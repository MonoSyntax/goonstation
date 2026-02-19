# DWAINE System VI Deep Report

## Scope and Method
This report documents how DWAINE works in this codebase, focusing on:
- `code/modules/networks/computer3/mainframe2/*`
- related global defines in `_std/defines/mainframe_defines/*`
- legacy context files `code/modules/networks/computer3/base_os.dm` and `code/modules/networks/computer3/terminal.dm`

It is based on direct source review and proc inventory extraction.

## 1) What DWAINE Is
DWAINE is a networked, multi-user OS stack for SS13 mainframes, implemented as DM datums plus networked machinery.

At runtime, it is split into four layers:
1. Physical network machine objects (`/obj/machinery/networked/*`) that send/receive wire signals.
2. Mainframe runtime (`/obj/machinery/networked/mainframe`) that owns terminals, program list, and OS reference.
3. OS and program layer (`kernel`, `shell`, `drivers`, `srv/*`, `utilities`).
4. Virtual filesystem layer (`/datum/computer/*`) with Unix-like paths, metadata, and permissions.

## 2) Load Order and Boot Chain
`goonstation.dme` includes DWAINE files in explicit order (mainframe2 core -> programs -> syscalls -> shell -> operators -> utilities).

### Boot flow
1. `mainframe2.dm` `mainframe/New()` creates net id, links to data terminal, creates memory card, creates bootloader datum.
2. `post_system()` runs POST:
- if `/sys` contains an OS program, it runs it.
- else it runs bootloader (`/datum/computer/file/mainframe_program/os/bootloader`).
3. Kernel init (`kernel.dm`) builds syscall table, initializes users, initializes drivers, optionally runs `/sys/init`.
4. New user terminals are logged in as TEMP and launched into `login`; successful login launches `shell`.

### Bootloader fallback
`bootloader.dm` handles disaster recovery:
- wipes core (`clear_core()`)
- disconnects devices (`disconnect_devices()`)
- pings databanks
- requests `bootreq`
- expects an archive containing an OS + programs/drivers
- writes recovered files into `/sys`, `/sys/drvr`, `/sys/srv`, `/bin`, then starts recovered OS.

## 3) Mainframe Runtime Model
Primary runtime file: `code/modules/networks/computer3/mainframe2/mainframe2.dm`.

### Core responsibilities
- maintain active terminal connections (`terminals`)
- maintain scheduled running programs (`processing`)
- own `os`, `bootloader`, runfolder `/proc`, and memory card
- process network events and forward them to OS
- run all program `process()` procs each machinery tick

### Important procs
- `run_program(...)`: clones program into runfolder, assigns `progid`, wires parent/user, calls `initialize()`.
- `unload_program(...)` and `unload_all()`: stop programs safely, including user cleanup.
- `relay_progsignal(...)`: inter-program signaling path used by all syscalls.
- `receive_signal(...)`: handles network protocol (`term_connect`, `term_message`, `term_ping`, `term_disconnect`, `ping_reply`).
- `reboot_mainframe()` and `post_system()`: reboot and POST behavior.

### Connection protocol
- Device connects with `term_connect` to mainframe net id.
- Mainframe records `/datum/terminal_connection` and notifies OS (`os.new_connection`).
- Terminals and devices exchange `term_ping` to keep alive.
- Timeout logic disconnects stale connections.

## 4) Program Base Class and OS Base Class

### Program parent (`programs/_program_parent.dm`)
This is the common base for every DWAINE executable/service/driver.

Key behavior:
- lifecycle: `ensure_program()`, `initialize()`, `process()`, `handle_quit()`
- user IO bridge: `message_user(...)`
- syscall bridge: `signal_program(...)`
- file lookup helpers: `get_folder_name`, `get_file_name`, `get_computer_datum`
- permission helpers: `check_read_permission`, `check_write_permission`, `check_mode_permission`
- parser helper: `parse_string(...)`

Also defines global `command2list(...)` used heavily by shell and command parsing.

### OS parent (`programs/os/_os_parent.dm`)
Provides OS-level helpers for:
- connection hooks (`new_connection`, `closed_connection`)
- terminal input routing (`term_input`)
- network output (`message_term`, `file_term`)
- filesystem path traversal (`parse_directory`, `parse_file_directory`, `parse_datum_directory`)

## 5) Kernel Design
File: `programs/os/kernel/kernel.dm`

Kernel owns:
- `users`: active `/datum/mainframe2_user_data` records
- `processing_drivers`: drivers open to kernel relays
- `syscalls`: cached `/datum/dwaine_syscall` objects indexed by command id
- scan timers and ping windows

Major responsibilities:
- temp/full login lifecycle
- per-terminal program routing
- driver instantiation from `/sys/drvr` prototypes
- mount/user/home folder initialization
- syscall dispatch
- global user messaging

### User lifecycle
- `login_temp_user`: create TEMP session for terminal
- `login_user`: create/attach full user record under `/usr`, home under `/home`
- `logout_user` / `logout_all_users`: terminate active program and cleanup

### Driver lifecycle
- `initialize_drivers`: copy driver prototypes into `/dev` and initialize
- `initialize_driver`: optionally register into mainframe `processing`
- dynamic connect/disconnect from `new_connection` / `closed_connection`

## 6) Syscall Layer
Files: `programs/os/kernel/syscalls/*.dm`

Each syscall file defines one `execute(...)` implementation. IDs come from `_std/defines/mainframe_defines/_syscalls.dm`.

### Syscall map
- `msg_term.dm` (`DWAINE_COMMAND_MSG_TERM`): send text/file to terminal
- `ulogin.dm`: login temp/full user
- `ugroup.dm`: update user group
- `ulist.dm`: list logged-in users
- `umsg.dm`: user-to-user message
- `uinput.dm`: inject input to user terminal/program
- `dmsg.dm`: send command to driver
- `dlist.dm`: list drivers by tag
- `dget.dm`: resolve driver index by tag/net id
- `dscan.dm`: force device rescan/ping window
- `exit.dm`: controlled task/program exit and parent/user reassignment
- `tspawn.dm`: spawn executable by path
- `tfork.dm`: fork current program type
- `tkill.dm`: kill child task
- `tlist.dm`: list caller-owned child tasks
- `fget.dm`: fetch file/folder by path
- `fkill.dm`: delete file/folder
- `fmode.dm`: chmod-like metadata permission update
- `fowner.dm`: chown/chgrp-like metadata update
- `fwrite.dm`: write file to path, with mkdir/append/replace behavior
- `confget.dm`: fetch config file from `/conf`
- `mount.dm`: create `/mnt` mountpoint and optional symlink

## 7) Filesystem and Permissions

### Directory conventions
From `_std/defines/mainframe_defines/_filepaths.dm` and tape setup:
- `/sys`: OS and core programs
- `/sys/drvr`: driver prototypes
- `/sys/srv`: services
- `/bin`: shell utilities
- `/usr`: user records
- `/home`: per-user homes
- `/dev`: active device driver instances
- `/mnt`: mounted driver filesystems
- `/conf`: configs
- `/etc/mail`: mail storage
- `/tmp`, `/var`
- `/proc`: runtime program copies

### Permissions
From `_permissions.dm`:
- owner/group/other read/write/delete flags (`COMP_*` bitflags)
- checks enforced by program parent helper procs and syscall handlers
- `group == 0` is sysop and bypasses most checks

## 8) Shell Architecture
File: `programs/os/shell/shell.dm`

The shell (`Msh`) is the main user-facing command interpreter.

### Command pipeline behavior
Execution order for each command token:
1. try `/bin/<cmd>`
2. try `<curpath>/<cmd>`
3. try builtin table
4. if piping output exists and command is not executable, treat command as output file path and write record

### Shell features
- command substitution `$(...)`
- piping via `pipetemp`
- variable expansion via `command2list` and `scriptvars`
- script execution by forking shell into child iterations
- shebang script detection (`#!`)
- stack-based RPN expression evaluator (`eval` + operators)

### Builtins (`shell_builtins/*.dm`)
- `break`: break script line processing
- `cls/clear`: clear terminal render
- `echo`: print/pipe text (`-n` supported)
- `if` / `else` / `while`: control flow flags for script engine
- `eval`: run RPN evaluator
- `logout/logoff`: terminate shell session
- `man/help`: read topics from `/conf/help`
- `mesg`: toggle message acceptance
- `talk`: send direct user message
- `sleep`: bounded sleep delay
- `unset`: clear script variables
- `who`: list current users
- `goonsay`: novelty ascii output builtin

### Script operators (`shell_script_operators/*.dm`)
- arithmetic: `+ - * / % rand`
- relational: `eq ne gt ge lt le`
- logical: `and or xor not`
- file tests: `e f d x`
- stack and misc: `dup del . .s # ' to`

## 9) Utility Programs (`/bin`)
File family: `programs/utilities/*.dm`

Core utilities and what they do:
- `cat`: print file contents
- `cd`: change current path
- `chmod`: permission bit conversion and update
- `chown`: owner/group metadata update
- `cp`: copy file
- `mv`: move file (copy + delete)
- `pwd`: print current path
- `ls`: list directory, optional `-l` metadata output
- `mkdir`: create dirs, optional `-p`
- `ln`: create folder symlink
- `rm`: delete, with `-i/-f/-r`
- `mount`: mount mountable driver into `/mnt`
- `scnt`: trigger device scan / reconnect by id
- `su`: elevate to sysop using auth card
- `grep`: regex search across record fields, supports recursive mode
- `date`: round tick timestamp formatting
- `getopt`: option parser utility used by other commands
- `tar`: archive create/list/extract utility over computer datums

## 10) Driver Layer (`os_drivers.dm`)
This file implements most OS-side hardware integration.

### Base patterns
- `/driver`: core driver abstraction
- `/driver/mountable`: drivers exposing filesystem contents under `/mnt`
- common proc pattern: `initialize`, `terminal_input`, `receive_progsignal`, `add_file`, `remove_file`, `change_metadata`

### Major driver modules
- `user_terminal`: mount path for user file delivery
- `databank`: tape/databank sync, file transfer, metadata updates
- `mountpoint`: folder wrapper that proxies add/remove into mountable driver
- `printer`: queue and status bridge for line printers
- `telepad` + `srv/telecontrol (teleman)`: telescience command translation
- `nuke` + `nuke_interface (nukeman)`: auth-gated arm/disarm/time control for nuke charge
- `guard_dock` + `guardbot_interface (prman)`: PR-6 task upload/wake/wipe/status/recall
- `radio`: radio terminal multiplexing + channel folders
- `secdetector`, `apc`, `hept_emitter`, `hept_interface`: security/power/emitter control
- `h7init`: autonomous security response init program
- `test_apparatus`: generalized GPTIO driver for artifact test devices
- `service_terminal`: pseudo-user for service endpoints
- `srv/print`: print service API for terminals
- `comm_dish`: communications report ingestion and bridge printing

## 11) Device Implementations

### `misc_terms.dm`
Contains physical networked devices and supporting classes.

Important families:
- base `/obj/machinery/networked` signal send helpers (`post_status`, `post_file`) and wiring
- `storage` databanks/tape devices with protocol for `sync`, `filestore`, `delfile`, `modfile`, `bootreq`
- bomb tester, nuke charge machine, radio bridge, printer, scanner, sec detector, H7 emitter, test apparatus suite

Proc pattern is consistent across devices:
- `New`/`disposing`: wiring lifecycle
- `attack_hand`/`Topic`/`ui_*`: user interface
- `process`: timeout and periodic work
- `receive_signal`: network protocol handling
- device-specific `update_icon`, `message_interface`, `return_html_interface`, etc.

### `telesci.dm`
Implements telescience hardware (`telepad`) and user console (`teleconsole`).

Key behavior:
- coordinate transforms and hidden random offsets
- `send`, `receive`, `relay`, `portal`, `scan`, and LRT variants
- teleport safety checks (`is_teleportation_allowed`)
- failure effects (`processbadeffect`) for invalid/interference cases
- teleconsole UI sends `teleman` commands through service terminal path

### `artifact_res.dm`
Implements GPTIO user program (`gptio`), artifact console driver, and physical artifact console UI.

Key behavior:
- `gptio` command-line control: list/info/peek/poke/activate/deactivate/pulse/sense/read
- `driver/artifact_console` translates UI button/select events to DWAINE driver messages
- physical `/obj/machinery/networked/artifact_console` renders custom browser panel and exchanges term protocol

### `logreader.dm`
Implements access log reader machine + accesslog service + mountable driver.

Key behavior:
- machine UI builds filtered queries
- service `/sys/srv/accesslog` supports add/list/filter log operations
- mountable logreader driver executes accesslog + tar pipeline to send archive responses

## 12) Data and Content Files
- `filetypes.dm`: user data datums (`mainframe2_user_data`) and document type
- `documents.dm`: help library, demo scripts, random mail records
- `emailserv.dm`: mail service (`index/get/send/delete`) and group dispatch logic
- `tapes.dm`: default mainframe filesystem image, backup tapes, artifact/guardbot content media

## 13) Legacy Context (Non-DWAINE OS)
- `base_os.dm`: older ThinkDOS-style terminal OS program model (`/terminal_program/os/main_os`)
- `terminal.dm`: older TermOS terminal program network protocol

These are parallel/legacy systems and not DWAINE System VI kernel/shell architecture, but still exist in the repo and include similar terminal command language.

## 14) File-by-File Role Map

### Mainframe2 core
- `code/modules/networks/computer3/mainframe2/mainframe2.dm`: defines the physical `/obj/machinery/networked/mainframe` runtime. It owns terminal connection state, processing task slots, the active OS instance, bootloader fallback, signal routing between programs, connection timeout logic, and POST/boot startup.
- `code/modules/networks/computer3/mainframe2/programs/_program_parent.dm`: defines the shared base type for all DWAINE programs (kernel, shell, drivers, utilities, services). It provides lifecycle hooks (`initialize`, `process`, `handle_quit`), inter-program signaling, user message/file forwarding, path helpers, and permission checks.
- `code/modules/networks/computer3/mainframe2/programs/os/_os_parent.dm`: defines common OS-level behavior used by kernel/bootloader-style programs, including terminal send/file send helpers, connection/ping hooks, and filesystem parsing APIs (`parse_directory`, `parse_file_directory`, `parse_datum_directory`).
- `code/modules/networks/computer3/mainframe2/programs/os/kernel/kernel.dm`: implements the DWAINE kernel. It caches syscall handlers, manages active users and active drivers, routes terminal input to user programs or drivers, performs login/logout/session handling, and runs periodic device scan ping cycles.
- `code/modules/networks/computer3/mainframe2/programs/os/bootloader.dm`: implements NETBOOT recovery behavior when no usable OS is present. It wipes storage, disconnects devices, discovers databanks, requests boot archives, restores system trees (`/sys`, `/sys/drvr`, `/sys/srv`, `/bin`), and starts a recovered OS.
- `code/modules/networks/computer3/mainframe2/programs/os/login.dm`: login gateway program for user terminals. It loads MOTD from `/conf`, prompts for credentials/card flow, validates login record structure, and asks kernel to transition TEMP session to full user.
- `code/modules/networks/computer3/mainframe2/programs/os/shell/shell.dm`: command interpreter (`Msh`) and script engine. It resolves and executes `/bin` programs and CWD executables, runs shell builtins, handles pipelines/command substitution, executes shebang scripts through controlled forks, and evaluates RPN expressions.

### Syscalls
- `.../syscalls/_syscall.dm`: abstract syscall base datum with shared `id` and `execute(...)` contract used by kernel dispatch.
- `.../syscalls/confget.dm`: reads named config record file from `/conf` and returns either the config datum or target/file errors.
- `.../syscalls/dget.dm`: resolves active driver slot index by driver tag or driver name/net-id-derived identifier and returns index marked with `ESIG_DATABIT`.
- `.../syscalls/dlist.dm`: enumerates active drivers filtered by terminal tag, optionally preserving sparse index slots, and returns status mapping.
- `.../syscalls/dmsg.dm`: routes a command payload to a target driver (by ID or name mode), remapping `dcommand/dtarget` into driver-facing fields.
- `.../syscalls/dscan.dm`: immediately opens a scan window and broadcasts ping, forcing early device rediscovery instead of waiting for periodic scan.
- `.../syscalls/exit.dm`: exits caller task with controlled user-session reassignment logic (parent return, base shell return, or fallback shell relaunch).
- `.../syscalls/fget.dm`: path resolver returning a permission-filtered file/folder datum for the caller.
- `.../syscalls/fkill.dm`: deletes a target datum by path with safeguards for root/runfolder and support for mountable driver-backed removal.
- `.../syscalls/fmode.dm`: updates metadata permission bitfield (`chmod` semantics) after mode-permission check.
- `.../syscalls/fowner.dm`: updates metadata owner/group (`chown/chgrp` semantics) with mode-permission gating.
- `.../syscalls/fwrite.dm`: writes a file datum to destination path, supporting path creation (`mkdir`), append semantics for records (`append`), and replacement (`replace`).
- `.../syscalls/mount.dm`: creates/refreshes a mountpoint under `/mnt` for a mountable driver and can create a symlink alias for easier access.
- `.../syscalls/msg_term.dm`: sends text render packets or file packets to a terminal endpoint.
- `.../syscalls/tfork.dm`: forks a new child task of caller program type, passing optional args, and returns child task ID with data bit.
- `.../syscalls/tkill.dm`: terminates a caller-owned child task and reattaches user focus to caller when needed.
- `.../syscalls/tlist.dm`: returns caller-owned child task listing from mainframe processing table.
- `.../syscalls/tspawn.dm`: executes program by filesystem path, optionally passing caller user context and argument string.
- `.../syscalls/ugroup.dm`: updates active user group field in user record (permission/elevation-sensitive workflows consume this).
- `.../syscalls/uinput.dm`: injects text/file into a target user session, or launches login/shell if no current program exists.
- `.../syscalls/ulist.dm`: builds and returns active-user status list including login time/group/name fields.
- `.../syscalls/ulogin.dm`: kernel login entrypoint handling TEMP user setup and TEMP-to-full-user transition.
- `.../syscalls/umsg.dm`: sends direct user-to-user messages by terminal ID or username and enforces recipient acceptance policy.

### Shell builtins
- `_shell_builtin.dm`: abstract builtin command class; binds one or more command names to an `execute` implementation with shell context access.
- `break.dm`: returns `BUILTIN_BREAK`, stopping current script command processing branch.
- `cls.dm`: clears terminal output buffer (`clear` render mode), aliasing `cls` and `clear`.
- `echo.dm`: emits supplied text and/or pipeline text, supports `-n`, and either prints or forwards to next pipeline stage.
- `else.dm`: conditional flow helper that skips execution when prior `if` branch already succeeded.
- `eval.dm`: invokes RPN script evaluator, reports stack/undefined errors, and can place final value in pipe stream.
- `goonsay.dm`: novelty output builtin that appends a fixed ASCII-art phrase, with pipeline support.
- `if.dm`: evaluates condition using script operators, sets `SCRIPT_IF_TRUE`, and rewrites active pipeline branch around `else`.
- `logout.dm`: prints farewell, terminates child script process if present, and exits active session program.
- `man.dm`: reads help topics from `/conf/help` record fields (`man`/`help` aliases).
- `mesg.dm`: reads/sets `accept_msg` preference (`y`/`n`) controlling whether direct messages are accepted.
- `sleep.dm`: bounded delay utility for scripts and command flow throttling.
- `talk.dm`: sends direct message through kernel `UMSG` path and handles invalid target/refusal/error cases.
- `unset.dm`: deletes selected script variables or resets full variable table.
- `while.dm`: loop control builtin setting/clearing loop state and controlling script line continuation.
- `who.dm`: lists active users from kernel `ULIST`, either printing or piping output.

### Shell script operators
- `_shell_script_operator.dm`: abstract RPN operator class used by `eval`; each operator reads/modifies the shell stack and returns script status codes.
- `arithmetic/add.dm` (`+`): adds two numeric operands, or concatenates two text operands.
- `arithmetic/subtract.dm` (`-`): subtracts numerics, or trims characters from end of a text operand using numeric count.
- `arithmetic/multiply.dm` (`*`): multiplies numerics, or repeats text N times.
- `arithmetic/divide.dm` (`/`): divides numerics (undefined on divide-by-zero), or splits text operand by delimiter text.
- `arithmetic/modulo.dm` (`%`): modulo for numeric operands (undefined on modulo-by-zero).
- `arithmetic/rand.dm` (`rand`): replaces top stack value X with a random value in `[1, X]`.
- `relational/eq.dm` (`eq`): pushes equality result of top two operands.
- `relational/ne.dm` (`ne`): pushes inequality result of top two operands.
- `relational/gt.dm` (`gt`): greater-than compare; mixed number/text operands compare against text length.
- `relational/ge.dm` (`ge`): greater-or-equal compare; mixed number/text operands compare against text length.
- `relational/lt.dm` (`lt`): less-than compare; mixed number/text operands compare against text length.
- `relational/le.dm` (`le`): less-or-equal compare; mixed number/text operands compare against text length.
- `logic/and.dm` (`and`): bitwise AND for numerics, logical AND for non-numeric truthy/falsy values.
- `logic/or.dm` (`or`): bitwise OR for numerics, logical OR for non-numeric values.
- `logic/xor.dm` (`xor`, `eor`): bitwise XOR for numerics, logical exclusive-or for non-numeric values.
- `logic/not.dm` (`not`, `!`): bitwise NOT for numerics, logical negation for non-numeric values.
- `file/exists.dm` (`e`): tests whether a filesystem path resolves to any computer datum.
- `file/is_file.dm` (`f`): tests whether a path resolves to a file datum.
- `file/is_directory.dm` (`d`): tests whether a path resolves to a folder datum.
- `file/is_executable.dm` (`x`): tests whether a path resolves to an executable mainframe program datum.
- `misc/assignment.dm` (`to`, `value`): assigns top stack item into a named script variable.
- `misc/escape_string.dm` (`'`): captures raw tokens until next apostrophe into one literal stack value.
- `misc/stack_del_topmost.dm` (`del`): drops top stack item.
- `misc/stack_depth.dm` (`#`): pushes current stack depth.
- `misc/stack_dup_topmost.dm` (`dup`): duplicates top stack item.
- `misc/stack_pop.dm` (`.`): prints and pops top stack item.
- `misc/stack_print.dm` (`.s`): prints stack depth and full stack contents for debugging.

### Utilities
- `_utility.dm`: utility program base and shared `optparse` helper that interprets normalized `getopt` output into option/argument lists.
- `cat.dm`: concatenates one or more files and prints contents to user terminal.
- `cd.dm`: resolves and validates path, then updates user `curpath` (supports relative/absolute navigation with path trimming).
- `chmod.dm`: parses numeric permission notation and applies metadata permissions through syscall.
- `chown.dm`: updates owner/group metadata, with privilege checks and input parsing (`owner[:group]`).
- `cp.dm`: copies source file datum to destination path/folder with collision and path validation.
- `date.dm`: prints round-time timestamp with optional format specifiers and option parsing via `getopt`.
- `getopt.dm`: parses option specification strings and command args into normalized option/value output for other utilities.
- `grep.dm`: regex search utility for record fields, supporting case flags, recursive traversal, output-mode controls, and quiet modes.
- `ln.dm`: creates folder symlinks by writing `/datum/computer/folder/link` entries.
- `ls.dm`: lists folder/file entries; `-l` produces metadata-rich, permission-aware descriptions.
- `mkdir.dm`: creates directories (multiple paths, optional `-p` full-path creation).
- `mount.dm`: privileged mount helper that validates driver ID/mountpoint name and invokes mount syscall.
- `mv.dm`: move utility implemented as copy-to-destination then delete-source.
- `pwd.dm`: prints current user working directory.
- `rm.dm`: remove utility with interactive (`-i`), force/silent (`-f`), and recursive (`-r`) behavior.
- `scnt.dm`: triggers network device scan globally or reconnects explicit net IDs.
- `su.dm`: privilege escalation flow requiring authorized card data and updates user group to sysop on success.
- `tar.dm`: archive utility supporting create/list/extract paths, quiet/verbose/skip modes, temporary archive generation, and recursive copy/extract helpers.

### Driver and device integration files
- `code/modules/networks/computer3/mainframe2/os_drivers.dm`: primary OS-side driver/service implementation file. It defines driver base abstractions, mountable driver behavior, and many device/service bridges used by kernel and shell utilities.
- `code/modules/networks/computer3/mainframe2/misc_terms.dm`: physical networked machine implementations and protocol handlers (databanks, printers, radios, scanners, detectors, and related support objects).
- `code/modules/networks/computer3/mainframe2/telesci.dm`: telescience hardware and control stack, including telepad behavior and teleconsole command translation.
- `code/modules/networks/computer3/mainframe2/artifact_res.dm`: artifact research integration including GPTIO utility, artifact console driver, and corresponding machine-side interface logic.
- `code/modules/networks/computer3/mainframe2/logreader.dm`: access log reader machine plus service/driver pathways used to query, package, and return access-log data.

### Data/content/setup files
- `code/modules/networks/computer3/mainframe2/filetypes.dm`: core DWAINE-specific file datum types (including user data wrappers and document cleanup behavior).
- `code/modules/networks/computer3/mainframe2/documents.dm`: static content records (help text, scripts, reference docs) and random email content helpers.
- `code/modules/networks/computer3/mainframe2/emailserv.dm`: email service program implementing mailbox index/read/send/delete flows and recipient/group dispatch logic.
- `code/modules/networks/computer3/mainframe2/tapes.dm`: default/backup media content and seeded filesystem payloads used for setup and recovery flows.

### Legacy adjacent files
- `code/modules/networks/computer3/base_os.dm`: legacy, non-System-VI terminal OS implementation kept for parallel compatibility.
- `code/modules/networks/computer3/terminal.dm`: legacy terminal protocol/program machinery adjacent to but separate from DWAINE VI architecture.

## 15) Global DWAINE Constants
From `_std/defines/mainframe_defines/*`:

### `_syscalls.dm` (command IDs and packet contract keys)
- `DWAINE_COMMAND_MSG_TERM` (`1`): send text or file to terminal endpoint; uses keys like `term`, `data`, `render`.
- `DWAINE_COMMAND_ULOGIN` (`2`): request user login flow; supports TEMP/full/service/sysop fields (`name`, `sysop`, `service`, `data`).
- `DWAINE_COMMAND_UGROUP` (`3`): request update of active user group metadata.
- `DWAINE_COMMAND_ULIST` (`4`): request active user list.
- `DWAINE_COMMAND_UMSG` (`5`): direct user-to-user message dispatch.
- `DWAINE_COMMAND_UINPUT` (`6`): alternate terminal input injection path.
- `DWAINE_COMMAND_DMSG` (`7`): relay command payload to driver.
- `DWAINE_COMMAND_DLIST` (`8`): list drivers by type/tag.
- `DWAINE_COMMAND_DGET` (`9`): resolve driver index by tag or net-id-derived name.
- `DWAINE_COMMAND_DSCAN` (`10`): force immediate device scan window.
- `DWAINE_COMMAND_EXIT` (`11`): instruct caller task to exit.
- `DWAINE_COMMAND_TSPAWN` (`12`): spawn program task by filesystem path.
- `DWAINE_COMMAND_TFORK` (`13`): fork a task of current program type.
- `DWAINE_COMMAND_TKILL` (`14`): terminate a caller-owned child task.
- `DWAINE_COMMAND_TLIST` (`15`): list caller-owned child tasks.
- `DWAINE_COMMAND_TEXIT` (`16`): parent notification that child task exited.
- `DWAINE_COMMAND_FGET` (`17`): resolve file/folder by path.
- `DWAINE_COMMAND_FKILL` (`18`): delete file/folder by path.
- `DWAINE_COMMAND_FMODE` (`19`): set metadata permission bits.
- `DWAINE_COMMAND_FOWNER` (`20`): set metadata owner/group.
- `DWAINE_COMMAND_FWRITE` (`21`): write file datum to path with optional mkdir/replace/append semantics.
- `DWAINE_COMMAND_CONFGET` (`22`): fetch named config file from `/conf`.
- `DWAINE_COMMAND_MOUNT` (`23`): create mountpoint for mountable device driver.
- `DWAINE_COMMAND_RECVFILE` (`24`): signal that a file payload has been delivered.
- `DWAINE_COMMAND_BREAK` (`25`): signal that script processing should break.
- `DWAINE_COMMAND_REPLY` (`30`): generic reply packet used by utility subprocess workflows.

### `_errors.dm` (status and error bitfields/codes)
- `ESIG_SUCCESS` (`0`): syscall completed successfully.
- `ESIG_GENERIC` (`1 << 0`): generic failure.
- `ESIG_NOTARGET` (`1 << 1`): required target missing/invalid.
- `ESIG_BADCOMMAND` (`1 << 2`): command ID invalid/unhandled.
- `ESIG_NOUSR` (`1 << 3`): required user context missing.
- `ESIG_IOERR` (`1 << 4`): I/O failure or endpoint refused operation.
- `ESIG_NOFILE` (`1 << 5`): required file/path missing.
- `ESIG_NOWRITE` (`1 << 6`): write permission denied.
- `ESIG_USR1` (`1 << 7`): user-defined application-specific status/error slot 1.
- `ESIG_USR2` (`1 << 8`): user-defined application-specific status/error slot 2.
- `ESIG_USR3` (`1 << 9`): user-defined application-specific status/error slot 3.
- `ESIG_USR4` (`1 << 10`): user-defined application-specific status/error slot 4.
- `ESIG_DATABIT` (`1 << 15`): marker bit indicating a numeric return includes payload data rather than error-only state.
- `BUILTIN_SUCCESS` (`0`): shell builtin completed normally.
- `BUILTIN_BREAK` (`1`): builtin requests shell/script break.
- `BUILTIN_CONTINUE` (`2`): builtin requests script continue behavior.
- `EXEC_FAILURE` (`0`): executable launch failed.
- `EXEC_SUCCESS` (`1`): executable launch succeeded.
- `EXEC_SCRIPT_ERROR` (`2`): shell script launch/evaluation failed.
- `EXEC_STACK_OVERFLOW` (`3`): script execution hit iteration/fork limit.
- `SCRIPT_SUCCESS` (`0`): script operator evaluation succeeded.
- `SCRIPT_STACK_OVERFLOW` (`-1`): stack exceeded configured depth/limits.
- `SCRIPT_STACK_UNDERFLOW` (`-2`): operator required more operands than available.
- `SCRIPT_UNDEFINED` (`-3`): undefined result (invalid type combo or undefined operation).

### `_permissions.dm` (filesystem ACL bitflags)
- `COMP_HIDDEN` (`0`): hidden/special value used by protected system folders.
- `COMP_ROWNER` (`1 << 0`): owner read permission.
- `COMP_WOWNER` (`1 << 1`): owner write permission.
- `COMP_DOWNER` (`1 << 2`): owner delete/mode-change permission.
- `COMP_RGROUP` (`1 << 3`): group read permission.
- `COMP_WGROUP` (`1 << 4`): group write permission.
- `COMP_DGROUP` (`1 << 5`): group delete/mode-change permission.
- `COMP_ROTHER` (`1 << 6`): other read permission.
- `COMP_WOTHER` (`1 << 7`): other write permission.
- `COMP_DOTHER` (`1 << 8`): other delete/mode-change permission.
- `COMP_ALLACC` (`~0`): full-access bitmask.

### `_shell.dm` (script state flags and interpreter limits)
- `SCRIPT_IF_TRUE` (`1 << 0`): shell script state bit indicating active `if` branch success.
- `SCRIPT_IN_LOOP` (`1 << 1`): shell script state bit indicating while-loop context.
- `MAX_PIPED_COMMANDS` (`16`): maximum number of pipeline segments parsed from one command line.
- `MAX_SCRIPT_COMPLEXITY` (`2048`): maximum per-run script command iterations before aborting.
- `MAX_SCRIPT_ITERATIONS` (`128`): maximum script fork depth/iteration chain.
- `MAX_STACK_DEPTH` (`128`): maximum RPN stack item count.
- `INT_MAX` (`2147483647`): script numeric upper bound.
- `INT_MIN` (`-2147483647`): script numeric lower bound.
- `SCRIPT_CLAMPVALUE(VALUE)`: macro that clamps and rounds script numeric values into safe internal range.

### `_filepaths.dm` (canonical directory macros)
- `setup_filepath_users` (`"/usr"`): user record directory.
- `setup_filepath_users_home` (`"/home"`): per-user home directory root.
- `setup_filepath_drivers` (`"/dev"`): active driver instance directory.
- `setup_filepath_drivers_proto` (`"/sys/drvr"`): driver prototype directory.
- `setup_filepath_volumes` (`"/mnt"`): mounted filesystem/mountpoint directory.
- `setup_filepath_system` (`"/sys"`): system program directory (kernel/shell/login/etc.).
- `setup_filepath_config` (`"/conf"`): config record directory.
- `setup_filepath_commands` (`"/bin"`): command utility directory.
- `setup_filepath_process` (`"/proc"`): runtime process copy directory.

### `mainframe.dm` (shared macros and global helpers)
- `ABSOLUTE_PATH(PATH, CURRENT_PATH)`: macro producing root-absolute path from absolute or current-directory-relative input.
- `generic_exit_list`: global packet list for standard program exit (`"command" = DWAINE_COMMAND_EXIT`).
- `mainframe_prog_exit`: macro shorthand to signal current program exit via parent/kernel path.
- `MIN_NUKE_TIME` (`120`): lower bound for networked nuke timer setting.
- `MAX_NUKE_TIME` (`600`): upper bound for networked nuke timer setting.

## 16) Critical Flows

### Boot -> Kernel -> Shell
1. `mainframe/post_system()` posts the machine and loads boot/runtime OS program state.
2. Bootloader (`initialize`/`process`) selects a valid core and hands control to kernel startup.
3. Kernel `initialize()` wires syscalls, users, and drivers; then terminals get sessions via `new_connection()` and `login_user()`.
4. Shell `initialize()` registers builtins/operators so user input can execute utilities and scripts.

### Command Execution Path
1. Terminal text enters kernel `term_input()` and is routed to the active program (usually shell).
2. Shell `input_text()` parses pipelines/tokens and dispatches commands through `execpath()` or builtin `execute()`.
3. Utilities and builtins issue syscall packets via `signal_program()`; kernel `receive_progsignal()` dispatches to syscall `execute()`.
4. Replies/files flow back via `receive_progsignal()` and are rendered with `message_user()` to terminal.

### Filesystem and Permission Path
1. OS parse helpers (`parse_directory`/`parse_file_directory`/`parse_datum_directory`) normalize and resolve target paths.
2. Program ACL helpers (`check_read_permission`/`check_write_permission`/`check_mode_permission`) gate access.
3. Syscalls (`fget`/`fwrite`/`fkill`/`fmode`/`fowner`) perform kernel-authoritative file operations.

### Script Evaluation Path
1. Shell `script_format()` tokenizes script lines and `script_process()` advances control flow.
2. `script_evaluate()` runs stack operators (arithmetic/logical/relational/file checks).
3. Builtins like `_if`, `_else`, `_while`, and `_break` mutate script state flags to control execution.

## 17) Testing Priorities

1. Boot resilience: power cycle and reboot (`power_change`, `reboot_mainframe`, bootloader stages) should never leave orphan tasks.
2. Session lifecycle: connect/disconnect, temp login, full login, and logout-all should maintain consistent user/task state.
3. Syscall contract correctness: each `DWAINE_COMMAND_*` path should return expected `ESIG_*` on success/failure edges.
4. Permission enforcement: verify read/write/mode checks across owner/group/other combinations and hidden files.
5. Shell safety limits: pipeline and script complexity limits must reject pathological input without hanging runtime.
6. Script semantics: operator stack underflow/undefined conditions should fail predictably with correct error codes.
7. Utility integration: `cd/chmod/ls/rm/su/tar/mount` should exercise syscall layer end-to-end with realistic paths.

## Appendix A: Core Proc Inventory
Filtered to runtime-critical paths only. The exhaustive machine-generated list is in `dwaine_proc_index_full.md`.

- Entries retained: 130
- Files retained: 61

### `code/modules/networks/computer3/mainframe2/mainframe2.dm`
- `244:/obj/machinery/networked/mainframe/process()`: Main machine tick: advances loaded programs, monitors timeout state, and forces safe shutdown/reboot paths when runtime health degrades.
- `302:/obj/machinery/networked/mainframe/receive_signal(datum/signal/signal)`: Wire packet ingress for the physical mainframe; routes terminal/device traffic into the active OS path and posts structured responses.
- `401:/obj/machinery/networked/mainframe/power_change()`: Synchronizes software state with power state transitions, including startup posting on restore and teardown on power loss.
- `456:/obj/machinery/networked/mainframe/proc/run_program(datum/computer/file/mainframe_program/program, datum/mainframe2_user_data/user, datum/computer/file/mainframe_program/caller_prog, runparams, allow_fork = FALSE)`: Primary task loader used by kernel/syscalls: validates executable files, creates runnable copies in process space, and wires parent-child ownership.
- `532:/obj/machinery/networked/mainframe/proc/unload_all()`: System-wide drain path that logs out sessions and unloads every active program before reboot/destruction.
- `543:/obj/machinery/networked/mainframe/proc/unload_program(datum/computer/file/mainframe_program/program)`: Stops one program instance, triggers its unload callback, and removes it from scheduler/process bookkeeping.
- `568:/obj/machinery/networked/mainframe/proc/relay_progsignal(datum/computer/file/mainframe_program/caller_prog, progid, list/data, datum/computer/file/file)`: Internal IPC relay that forwards command payloads and optional file attachments between program IDs.
- `579:/obj/machinery/networked/mainframe/proc/reconnect_all_devices()`: Bulk reconnection scan used by kernel and utilities to rebind driver endpoints after topology changes.
- `590:/obj/machinery/networked/mainframe/proc/reconnect_device(device_id)`: Targeted reconnect for one device identifier, replacing stale links and reinitializing driver communication.
- `606:/obj/machinery/networked/mainframe/proc/reboot_mainframe()`: Soft reboot orchestrator: tears down runtime programs then triggers a fresh system post sequence.
- `613:/obj/machinery/networked/mainframe/proc/post_system()`: Boot entrypoint that instantiates/initializes the configured OS stack after hardware startup.

### `code/modules/networks/computer3/mainframe2/programs/_program_parent.dm`
- `52:/datum/computer/file/mainframe_program/proc/ensure_program()`: Liveness guard that confirms holder/master bindings are still valid before any program action executes.
- `71:/datum/computer/file/mainframe_program/proc/input_text(text)`: Base text-input hook for program datums; subclasses override behavior while inheriting lifecycle safety checks.
- `78:/datum/computer/file/mainframe_program/proc/initialize(initparams)`: One-time program startup callback invoked with spawn parameters from kernel/mainframe.
- `86:/datum/computer/file/mainframe_program/proc/handle_quit()`: Standard exit routine for user-space programs that requests unload from the mainframe scheduler.
- `90:/datum/computer/file/mainframe_program/proc/process()`: Default per-tick callback for schedulable programs; used as the baseline runtime contract.
- `97:/datum/computer/file/mainframe_program/proc/parse_string(string, list/replace_list)`: Command preprocessing helper that applies variable substitutions before tokenization.
- `149:/datum/computer/file/mainframe_program/proc/is_name_invalid(string)`: Filename/path component validator used to block invalid or unsafe names before filesystem mutations.
- `163:/datum/computer/file/mainframe_program/proc/message_user(msg, render, file)`: Canonical user-output helper that emits terminal message packets through kernel/mainframe routing.
- `176:/datum/computer/file/mainframe_program/proc/read_user_field(field)`: Reads user profile/account fields from backing records with refresh semantics.
- `186:/datum/computer/file/mainframe_program/proc/write_user_field(field, data)`: Writes user profile/account fields and persists updates to the user data record.
- `202:/datum/computer/file/mainframe_program/proc/signal_program(progid, list/data, datum/computer/file/file)`: Program-to-program signaling wrapper that sends command IDs and payload lists over DWAINE IPC.
- `212:/datum/computer/file/mainframe_program/proc/receive_progsignal(sendid, list/data, datum/computer/file/file)`: Inbound IPC callback that programs override to react to command replies and asynchronous events.
- `220:/datum/computer/file/mainframe_program/proc/check_read_permission(datum/computer/target, datum/mainframe2_user_data/user)`: ACL evaluation for read access based on owner/group/other bits and current user context.
- `251:/datum/computer/file/mainframe_program/proc/check_write_permission(datum/computer/target, datum/mainframe2_user_data/user)`: ACL evaluation for write access, enforcing ownership/group constraints for file edits.
- `284:/datum/computer/file/mainframe_program/proc/check_mode_permission(datum/computer/target, datum/mainframe2_user_data/user)`: ACL evaluation for metadata-changing operations such as chmod/chown-style updates.
- `324:/proc/command2list(text, separator, list/replaceList, list/substitution_feedback_thing)`: Global tokenizer for shell-like command strings, handling separators, quoting, and substitution edge cases.

### `code/modules/networks/computer3/mainframe2/programs/os/_os_parent.dm`
- `22:/datum/computer/file/mainframe_program/os/proc/term_input(data, termid, datum/computer/file/file)`: OS-level terminal input entrypoint called by kernel/mainframe when user keystrokes arrive.
- `33:/datum/computer/file/mainframe_program/os/proc/message_term(message, termid, render)`: Sends rendered text output to a specific terminal session ID.
- `43:/datum/computer/file/mainframe_program/os/proc/file_term(datum/computer/file/file, termid, exdata)`: Transfers a file payload to a terminal endpoint, including auxiliary metadata when needed.
- `63:/datum/computer/file/mainframe_program/os/proc/parse_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`: Core absolute/relative path traversal routine for directory targets, with optional creation and permission enforcement.
- `155:/datum/computer/file/mainframe_program/os/proc/parse_file_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`: Path resolver variant that expects a file target and separates parent folder resolution from final file lookup.
- `236:/datum/computer/file/mainframe_program/os/proc/parse_datum_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`: Generic resolver that can return either file or folder datums for syscall and utility operations.

### `code/modules/networks/computer3/mainframe2/programs/os/bootloader.dm`
- `44:/datum/computer/file/mainframe_program/os/bootloader/initialize()`: Resets bootloader state machine, scans memory banks/core records, and prepares first interactive boot stage.
- `60:/datum/computer/file/mainframe_program/os/bootloader/process()`: Advances bootloader staging logic each tick until kernel handoff is complete or boot fails.
- `116:/datum/computer/file/mainframe_program/os/bootloader/term_input(data, termid, datum/computer/file/file)`: Handles boot-time operator commands (selection/confirmation paths) from connected terminals.
- `219:/datum/computer/file/mainframe_program/os/bootloader/proc/new_current()`: Rebuilds or rotates the active candidate bank list used by boot stage prompts.
- `233:/datum/computer/file/mainframe_program/os/bootloader/proc/clear_core()`: Clears existing core/runtime file payloads to guarantee clean load conditions.
- `246:/datum/computer/file/mainframe_program/os/bootloader/proc/disconnect_devices()`: Forcibly detaches connected devices before reboot/handoff so kernel can rebuild driver bindings cleanly.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/kernel.dm`
- `57:/datum/computer/file/mainframe_program/os/kernel/initialize()`: Kernel bootstrap: builds syscall registry, seeds system folders, and initializes user/driver runtime structures.
- `95:/datum/computer/file/mainframe_program/os/kernel/term_input(data, termid, datum/computer/file/file, is_break = FALSE)`: Primary per-terminal dispatch path, forwarding user input to session programs and handling break/file control packets.
- `127:/datum/computer/file/mainframe_program/os/kernel/new_connection(datum/terminal_connection/conn, datum/computer/file/connect_file)`: Session onboarding path that allocates temporary user context and starts login/session programs for a new terminal.
- `180:/datum/computer/file/mainframe_program/os/kernel/closed_connection(datum/terminal_connection/conn)`: Session teardown path that unwinds terminal bindings and logs users out when links drop.
- `215:/datum/computer/file/mainframe_program/os/kernel/receive_progsignal(sendid, list/data, datum/computer/file/file)`: Kernel syscall dispatcher: interprets command IDs from programs and executes matching syscall handlers.
- `231:/datum/computer/file/mainframe_program/os/kernel/process()`: Periodic kernel maintenance loop for driver pings, scan timers, and background runtime housekeeping.
- `246:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_users()`: Ensures account/home directory scaffolding exists and loads persistent user records into kernel state.
- `301:/datum/computer/file/mainframe_program/os/kernel/proc/login_temp_user(user_netid, datum/computer/file/record/login_record, datum/computer/file/mainframe_program/caller_prog_override)`: Creates ephemeral pre-auth sessions tied to terminal net IDs for login workflows.
- `330:/datum/computer/file/mainframe_program/os/kernel/proc/login_user(datum/mainframe2_user_data/account, user_name, sysop = FALSE, interactive = TRUE)`: Authenticates/attaches an account to a live session, assigns groups/privilege flags, and launches shell/login programs.
- `407:/datum/computer/file/mainframe_program/os/kernel/proc/logout_user(datum/mainframe2_user_data/user, disconnect = FALSE)`: Ends one session, kills associated program tree, and optionally disconnects the terminal endpoint.
- `421:/datum/computer/file/mainframe_program/os/kernel/proc/logout_all_users(disconnect = FALSE)`: Bulk session termination used during reboot/shutdown to leave no dangling user contexts.
- `430:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_drivers()`: Discovers and starts driver programs, then binds active devices into `/dev` and related kernel maps.
- `489:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_driver(datum/computer/file/mainframe_program/driver/driver, datum/computer/file/connect_file)`: Initializes one driver instance and performs validation/binding for its associated connect file.
- `511:/datum/computer/file/mainframe_program/os/kernel/proc/is_sysop(datum/mainframe2_user_data/udat)`: Privilege predicate used by privileged operations to verify sysop authority.
- `521:/datum/computer/file/mainframe_program/os/kernel/proc/change_metadata(datum/computer/file/file, field, newval)`: Controlled metadata mutator used by chmod/chown syscall paths to centralize file field updates.
- `533:/datum/computer/file/mainframe_program/os/kernel/proc/message_all_users(message, sender_name, ignore_user_file_setting)`: Broadcast helper that sends a notice to all currently connected user sessions.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/confget.dm`
- `4:/datum/dwaine_syscall/confget/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_CONFGET`: fetches named config records from `/conf` for callers.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dget.dm`
- `4:/datum/dwaine_syscall/dget/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_DGET`: resolves a driver/device ID from tags or driver naming data.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dlist.dm`
- `4:/datum/dwaine_syscall/dlist/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_DLIST`: enumerates available drivers/devices, optionally filtered by type/tag.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dmsg.dm`
- `4:/datum/dwaine_syscall/dmsg/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_DMSG`: sends a structured message packet to a target driver program.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dscan.dm`
- `4:/datum/dwaine_syscall/dscan/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_DSCAN`: triggers immediate device rescan/reconnect behavior.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/exit.dm`
- `4:/datum/dwaine_syscall/exit/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_EXIT`: requests clean termination of the calling task.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fget.dm`
- `4:/datum/dwaine_syscall/fget/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_FGET`: resolves and returns filesystem targets from absolute/relative paths.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fkill.dm`
- `4:/datum/dwaine_syscall/fkill/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_FKILL`: deletes a file/folder target after mode/ownership checks.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fmode.dm`
- `4:/datum/dwaine_syscall/fmode/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_FMODE`: updates permission bitmasks on a filesystem target.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fowner.dm`
- `4:/datum/dwaine_syscall/fowner/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_FOWNER`: changes owner/group metadata on a filesystem target.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fwrite.dm`
- `4:/datum/dwaine_syscall/fwrite/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_FWRITE`: writes/replaces/appends file content and can create missing paths.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/mount.dm`
- `4:/datum/dwaine_syscall/mount/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_MOUNT`: mounts a mountable driver/device into the filesystem tree.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/msg_term.dm`
- `4:/datum/dwaine_syscall/msg_term/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_MSG_TERM`: sends text or file output to a terminal endpoint.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tfork.dm`
- `4:/datum/dwaine_syscall/tfork/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_TFORK`: forks/spawns a child task from the caller context.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tkill.dm`
- `4:/datum/dwaine_syscall/tkill/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_TKILL`: terminates a caller-owned child task by ID.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tlist.dm`
- `4:/datum/dwaine_syscall/tlist/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_TLIST`: returns the caller-owned task list and status metadata.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tspawn.dm`
- `4:/datum/dwaine_syscall/tspawn/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_TSPAWN`: executes an explicit program path as a new task.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ugroup.dm`
- `4:/datum/dwaine_syscall/ugroup/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_UGROUP`: updates active user group/session group metadata.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/uinput.dm`
- `4:/datum/dwaine_syscall/uinput/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_UINPUT`: injects synthetic input into a user session terminal stream.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ulist.dm`
- `4:/datum/dwaine_syscall/ulist/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_ULIST`: returns the set of currently logged-in users/sessions.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ulogin.dm`
- `4:/datum/dwaine_syscall/ulogin/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_ULOGIN`: executes temp/full login logic and session attachment.

### `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/umsg.dm`
- `4:/datum/dwaine_syscall/umsg/execute(sendid, list/data, datum/computer/file/file)`: Implements `DWAINE_COMMAND_UMSG`: sends direct user-to-user messages through kernel routing.

### `code/modules/networks/computer3/mainframe2/programs/os/login.dm`
- `13:/datum/computer/file/mainframe_program/login/initialize()`: Starts login program state, requests MOTD/config data, and prints initial authentication prompts.
- `26:/datum/computer/file/mainframe_program/login/receive_progsignal(sendid, list/data, datum/computer/file/record/file)`: Handles login-related replies (config/file delivery/session responses) and advances login interaction state.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell.dm`
- `60:/datum/computer/file/mainframe_program/shell/initialize(list/supplied_config)`: Builds builtin/operator registries and applies shell startup configuration for the session.
- `114:/datum/computer/file/mainframe_program/shell/process()`: Shell tick loop that advances script execution and deferred command state.
- `121:/datum/computer/file/mainframe_program/shell/input_text(text)`: Parses command lines, handles pipelines, and enforces complexity/pipe limits before execution.
- `243:/datum/computer/file/mainframe_program/shell/receive_progsignal(sendid, list/data, datum/computer/file/file)`: Consumes syscall replies and file-return events for active commands/scripts.
- `275:/datum/computer/file/mainframe_program/shell/message_user(msg, render, file)`: Shell-level output wrapper that routes text/file replies back to the current terminal context.
- `293:/datum/computer/file/mainframe_program/shell/proc/execpath(file_path, list/command_list, scripting = 0)`: Resolves executable paths, spawns utilities/tasks, and wires piping/scripting context into launch params.
- `342:/datum/computer/file/mainframe_program/shell/proc/script_process()`: Drives queued script lines, handling control-flow state and fork/wait transitions.
- `380:/datum/computer/file/mainframe_program/shell/proc/script_format(list/script_list)`: Normalizes script tokens/line structure before evaluation and execution.
- `398:/datum/computer/file/mainframe_program/shell/proc/script_evaluate(list/token_stream, return_bool = FALSE)`: Stack-machine evaluator for script expressions with operator dispatch, bounds checks, and typed return behavior.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/break.dm`
- `4:/datum/dwaine_shell_builtin/_break/execute(list/command_list, list/piped_list)`: Script control builtin that requests loop break semantics for the current script execution frame.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/echo.dm`
- `4:/datum/dwaine_shell_builtin/echo/execute(list/command_list, list/piped_list)`: Writes literal/piped argument text to terminal output without spawning external utilities.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/else.dm`
- `4:/datum/dwaine_shell_builtin/_else/execute(list/command_list, list/piped_list)`: Companion conditional builtin that inverts/continues branch execution based on prior `if` state.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/eval.dm`
- `4:/datum/dwaine_shell_builtin/eval/execute(list/command_list, list/piped_list)`: Evaluates an inline shell expression and prints/returns the computed result.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/if.dm`
- `4:/datum/dwaine_shell_builtin/_if/execute(list/command_list, list/piped_list)`: Evaluates a condition expression and sets shell script branch flags for conditional execution.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/logout.dm`
- `4:/datum/dwaine_shell_builtin/logout/execute(list/command_list, list/piped_list)`: Ends the current session by signaling task/session termination through kernel commands.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/man.dm`
- `4:/datum/dwaine_shell_builtin/man/execute(list/command_list, list/piped_list)`: Requests and prints command help/manual text through configuration/document retrieval paths.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/mesg.dm`
- `4:/datum/dwaine_shell_builtin/mesg/execute(list/command_list, list/piped_list)`: Reads or updates per-user message preference fields used by inter-user chat notifications.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/talk.dm`
- `4:/datum/dwaine_shell_builtin/talk/execute(list/command_list, list/piped_list)`: Interactive messaging builtin that sends directed user messages via `UMSG` command routing.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/while.dm`
- `4:/datum/dwaine_shell_builtin/_while/execute(list/command_list, list/piped_list)`: Loop control builtin that evaluates loop predicates and maintains loop-context state bits.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/who.dm`
- `4:/datum/dwaine_shell_builtin/who/execute(list/command_list, list/piped_list)`: Queries and displays the active user/session list from kernel (`ULIST`).

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/add.dm`
- `10:/datum/dwaine_shell_script_operator/add/execute(list/token_stream)`: RPN arithmetic operator: pops two operands, adds them, clamps result, and pushes output back on stack.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/divide.dm`
- `10:/datum/dwaine_shell_script_operator/divide/execute(list/token_stream)`: RPN arithmetic operator: divides operands with underflow/undefined guards (e.g., divide-by-zero handling).

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/rand.dm`
- `10:/datum/dwaine_shell_script_operator/rand/execute(list/token_stream)`: RPN arithmetic operator: produces bounded random values from stack operands and pushes clamped output.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/exists.dm`
- `11:/datum/dwaine_shell_script_operator/exists/execute(list/token_stream)`: Filesystem predicate operator: tests whether a path resolves to an existing datum and pushes boolean result.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/is_file.dm`
- `11:/datum/dwaine_shell_script_operator/is_file/execute(list/token_stream)`: Filesystem predicate operator: tests whether resolved path target is a regular file object.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/and.dm`
- `11:/datum/dwaine_shell_script_operator/and/execute(list/token_stream)`: Logical operator: combines top stack values as boolean conjunction and pushes normalized truth value.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/not.dm`
- `10:/datum/dwaine_shell_script_operator/not/execute(list/token_stream)`: Logical unary operator: negates the top boolean-ish stack value and pushes result.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/assignment.dm`
- `8:/datum/dwaine_shell_script_operator/assignment/execute(list/token_stream)`: Script assignment operator: binds evaluated value into shell script variables/environment context.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/escape_string.dm`
- `12:/datum/dwaine_shell_script_operator/escape_string/execute(list/token_stream)`: String utility operator: unescapes encoded character sequences before further evaluation/use.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_pop.dm`
- `11:/datum/dwaine_shell_script_operator/stack_pop/execute(list/token_stream)`: Stack utility operator: removes top item and exposes/prints it for script debugging/output flows.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/eq.dm`
- `10:/datum/dwaine_shell_script_operator/eq/execute(list/token_stream)`: Relational operator: compares two operands for equality and pushes boolean outcome.

### `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/gt.dm`
- `12:/datum/dwaine_shell_script_operator/gt/execute(list/token_stream)`: Relational operator: compares whether left operand is greater than right operand.

### `code/modules/networks/computer3/mainframe2/programs/utilities/_utility.dm`
- `38:/proc/optparse(data)`: Global option parser used by multiple utilities to decode short/long flags and positional argument layouts.

### `code/modules/networks/computer3/mainframe2/programs/utilities/cd.dm`
- `4:/datum/computer/file/mainframe_program/utility/cd/initialize(initparams)`: Implements `cd` command startup: resolves requested path and updates session working directory state.
- `24:/datum/computer/file/mainframe_program/utility/cd/proc/trim_path(filepath)`: Normalizes path strings (`.`/`..` cleanup) before writing final cwd back to user context.

### `code/modules/networks/computer3/mainframe2/programs/utilities/chmod.dm`
- `4:/datum/computer/file/mainframe_program/utility/chmod/initialize(initparams)`: Implements `chmod` command flow: parses mode arguments and issues metadata-change syscall requests.
- `37:/datum/computer/file/mainframe_program/utility/chmod/proc/process_permissions(permissions)`: Parses symbolic/numeric permission specifications into DWAINE ACL bitmasks.

### `code/modules/networks/computer3/mainframe2/programs/utilities/ls.dm`
- `4:/datum/computer/file/mainframe_program/utility/ls/initialize(initparams)`: Implements `ls` command startup: resolves listing target and prepares directory/file output.
- `48:/datum/computer/file/mainframe_program/utility/ls/proc/print_file_description(datum/computer/C)`: Formats one file entry with type/permission/owner metadata for terminal listing output.

### `code/modules/networks/computer3/mainframe2/programs/utilities/mkdir.dm`
- `4:/datum/computer/file/mainframe_program/utility/mkdir/initialize(initparams)`: Implements `mkdir` command flow, including path resolution and permission-gated directory creation.

### `code/modules/networks/computer3/mainframe2/programs/utilities/mount.dm`
- `4:/datum/computer/file/mainframe_program/utility/mount/initialize(initparams)`: Implements `mount` utility command, forwarding mount requests to kernel mount syscall.

### `code/modules/networks/computer3/mainframe2/programs/utilities/rm.dm`
- `12:/datum/computer/file/mainframe_program/utility/rm/initialize(initparams)`: Implements `rm` startup: parses flags/targets and schedules file deletion workflow.
- `61:/datum/computer/file/mainframe_program/utility/rm/input_text(text)`: Interactive confirmation/input path for `rm` (used by prompt-driven deletes and follow-up actions).

### `code/modules/networks/computer3/mainframe2/programs/utilities/su.dm`
- `4:/datum/computer/file/mainframe_program/utility/su/initialize(initparams)`: Begins `su` authentication/target-user switch flow and prompts for required credentials.
- `11:/datum/computer/file/mainframe_program/utility/su/receive_progsignal(sendid, list/data, datum/computer/file/file)`: Handles async replies during `su` flow (auth result/session update) and updates user messaging.
- `33:/datum/computer/file/mainframe_program/utility/su/input_text(text)`: Consumes password/username interaction for `su` and drives transition between prompt states.

### `code/modules/networks/computer3/mainframe2/programs/utilities/tar.dm`
- `23:/datum/computer/file/mainframe_program/utility/tar/initialize(initparams)`: Entry for `tar` command; parses create/extract options and prepares recursive archive plan.
- `173:/datum/computer/file/mainframe_program/utility/tar/receive_progsignal(sendid, list/data, datum/computer/file/file)`: Processes async syscall/task replies during archive create/extract operations.
- `218:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_list(datum/computer/target, current_path = "", depth = 0)`: Recursive traversal helper that enumerates nested filesystem tree contents for archive creation.
- `233:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_extract(datum/computer/to_extract, target_path, current_path = "", depth = 0)`: Recursive extraction helper that recreates archived tree structure at a target path.
- `276:/datum/computer/file/mainframe_program/utility/tar/proc/deep_copy(datum/computer/to_copy, current_path = "", depth = 0)`: Recursive copy helper used by tar workflows to duplicate directory/file trees safely.

