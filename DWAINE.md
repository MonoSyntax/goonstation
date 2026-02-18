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
- `code/modules/networks/computer3/mainframe2/mainframe2.dm`: physical mainframe runtime, scheduler, connection table, signal relay, POST/boot.
- `code/modules/networks/computer3/mainframe2/programs/_program_parent.dm`: common program lifecycle, syscall bridge, permission checks.
- `code/modules/networks/computer3/mainframe2/programs/os/_os_parent.dm`: OS-level connection + filesystem parsing helpers.
- `code/modules/networks/computer3/mainframe2/programs/os/kernel/kernel.dm`: kernel, users, drivers, syscall dispatch, scans.
- `code/modules/networks/computer3/mainframe2/programs/os/bootloader.dm`: core wipe and databank recovery boot.
- `code/modules/networks/computer3/mainframe2/programs/os/login.dm`: card login gateway and MOTD.
- `code/modules/networks/computer3/mainframe2/programs/os/shell/shell.dm`: shell execution engine.

### Syscalls
- `.../syscalls/_syscall.dm`: syscall base datum.
- `.../syscalls/confget.dm`: fetch config record.
- `.../syscalls/dget.dm`: resolve driver index.
- `.../syscalls/dlist.dm`: list drivers by tag.
- `.../syscalls/dmsg.dm`: relay command to driver.
- `.../syscalls/dscan.dm`: force rescan ping window.
- `.../syscalls/exit.dm`: controlled task exit.
- `.../syscalls/fget.dm`: file/folder lookup by path.
- `.../syscalls/fkill.dm`: delete target datum.
- `.../syscalls/fmode.dm`: set permission metadata.
- `.../syscalls/fowner.dm`: set owner/group metadata.
- `.../syscalls/fwrite.dm`: write/copy file to destination.
- `.../syscalls/mount.dm`: mount mountable driver filesystem.
- `.../syscalls/msg_term.dm`: terminal message/file send.
- `.../syscalls/tfork.dm`: fork current program type.
- `.../syscalls/tkill.dm`: kill child task.
- `.../syscalls/tlist.dm`: list caller child tasks.
- `.../syscalls/tspawn.dm`: spawn executable by path.
- `.../syscalls/ugroup.dm`: change user group.
- `.../syscalls/uinput.dm`: push input to user program.
- `.../syscalls/ulist.dm`: list active users.
- `.../syscalls/ulogin.dm`: temp/full login entrypoint.
- `.../syscalls/umsg.dm`: send user message.

### Shell builtins
- `_shell_builtin.dm`: builtin base class.
- `break.dm`: terminate script processing.
- `cls.dm`: clear screen.
- `echo.dm`: emit text/pipe.
- `else.dm`: conditional branch helper.
- `eval.dm`: RPN evaluator wrapper.
- `goonsay.dm`: novelty text output.
- `if.dm`: conditional execution.
- `logout.dm`: log off current user.
- `man.dm`: help topic reader.
- `mesg.dm`: message permission toggle.
- `sleep.dm`: sleep utility.
- `talk.dm`: direct user message.
- `unset.dm`: clear shell variables.
- `while.dm`: loop execution helper.
- `who.dm`: list active users.

### Shell script operators
- `_shell_script_operator.dm`: operator base class.
- arithmetic: `add`, `subtract`, `multiply`, `divide`, `modulo`, `rand`.
- relational: `eq`, `ne`, `gt`, `ge`, `lt`, `le`.
- logic: `and`, `or`, `xor`, `not`.
- file tests: `exists`, `is_file`, `is_directory`, `is_executable`.
- stack/misc: `assignment`, `escape_string`, `stack_del_topmost`, `stack_depth`, `stack_dup_topmost`, `stack_pop`, `stack_print`.

### Utilities
- `_utility.dm`: utility base + `optparse` helper.
- `cat`, `cd`, `chmod`, `chown`, `cp`, `date`, `getopt`, `grep`, `ln`, `ls`, `mkdir`, `mount`, `mv`, `pwd`, `rm`, `scnt`, `su`, `tar`.

### Driver and device integration files
- `code/modules/networks/computer3/mainframe2/os_drivers.dm`: all major OS-side drivers/services.
- `code/modules/networks/computer3/mainframe2/misc_terms.dm`: major physical network devices.
- `code/modules/networks/computer3/mainframe2/telesci.dm`: telepad + teleconsole implementation.
- `code/modules/networks/computer3/mainframe2/artifact_res.dm`: GPTIO app + artifact console stack.
- `code/modules/networks/computer3/mainframe2/logreader.dm`: access log hardware + service.

### Data/content/setup files
- `code/modules/networks/computer3/mainframe2/filetypes.dm`
- `code/modules/networks/computer3/mainframe2/documents.dm`
- `code/modules/networks/computer3/mainframe2/emailserv.dm`
- `code/modules/networks/computer3/mainframe2/tapes.dm`

### Legacy adjacent files
- `code/modules/networks/computer3/base_os.dm`
- `code/modules/networks/computer3/terminal.dm`

## 15) Global DWAINE Constants
From `_std/defines/mainframe_defines/*`:
- `_syscalls.dm`: command ids and argument contracts
- `_errors.dm`: `ESIG_*`, builtin/exec/script status codes
- `_permissions.dm`: `COMP_*` ACL bits
- `_shell.dm`: script limits (`MAX_PIPED_COMMANDS`, stack/iteration limits)
- `_filepaths.dm`: canonical directories
- `mainframe.dm`: shared macros (`ABSOLUTE_PATH`, `mainframe_prog_exit`)

## Appendix A: Full Proc Inventory
Auto-generated from `dwaine_proc_index.txt` so every discovered proc in DWAINE-related files is listed.

### `code\modules\networks\computer3\base_os.dm`
- `27:/datum/computer/file/terminal_program/os/main_os/disposing()`
- `37:/datum/computer/file/terminal_program/os/main_os/input_text(text)`
- `594:/datum/computer/file/terminal_program/os/main_os/initialize()`
- `629:/datum/computer/file/terminal_program/os/main_os/disk_ejected(var/obj/item/disk/data/thedisk)`
- `644:/datum/computer/file/terminal_program/os/main_os/receive_command(obj/source, command, datum/signal/signal)`

### `code\modules\networks\computer3\mainframe2\artifact_res.dm`
- `12:/datum/computer/file/mainframe_program/test_interface/process()`
- `24:/datum/computer/file/mainframe_program/test_interface/message_user(var/msg, var/render=null)`
- `30:/datum/computer/file/mainframe_program/test_interface/initialize(var/initparams)`
- `367:/datum/computer/file/mainframe_program/test_interface/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `457:/datum/computer/file/mainframe_program/driver/artifact_console/disposing()`
- `461:/datum/computer/file/mainframe_program/driver/artifact_console/New()`
- `468:/datum/computer/file/mainframe_program/driver/artifact_console/initialize()`
- `494:/datum/computer/file/mainframe_program/driver/artifact_console/terminal_input(var/data, var/datum/computer/file/file)`
- `695:/datum/computer/file/mainframe_program/driver/artifact_console/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `940:/obj/machinery/networked/artifact_console/New()`
- `956:/obj/machinery/networked/artifact_console/power_change()`
- `964:/obj/machinery/networked/artifact_console/receive_signal(datum/signal/signal)`
- `1119:/obj/machinery/networked/artifact_console/attack_hand(var/mob/user)`
- `1341:/obj/machinery/networked/artifact_console/attackby(obj/item/W, mob/user)`
- `1355:/obj/machinery/networked/artifact_console/Topic(href, href_list)`
- `1406:/obj/machinery/networked/artifact_console/updateUsrDialog(var/forceUpdate)`

### `code\modules\networks\computer3\mainframe2\documents.dm`
- `16:/datum/computer/file/record/dwaine_help/New()`
- `48:/datum/computer/file/record/pr6_readme/New()`
- `65:/datum/computer/file/record/patrol_script/New()`
- `84:/datum/computer/file/record/bodyguard_script/New()`
- `100:/datum/computer/file/record/roomguard_script/New()`
- `113:/datum/computer/file/record/bodyguard_conf/New()`
- `133:/datum/computer/file/record/artlab_activate/New()`
- `142:/datum/computer/file/record/artlab_deactivate/New()`
- `151:/datum/computer/file/record/artlab_read/New()`
- `160:/datum/computer/file/record/artlab_info/New()`
- `169:/datum/computer/file/record/artlab_xray/New()`
- `178:/datum/computer/file/record/artlab_heater/New()`
- `187:/datum/computer/file/record/artlab_elecbox/New()`
- `198:/datum/computer/file/record/artlab_pitcher/New()`
- `207:/datum/computer/file/record/artlab_impactpad/New()`
- `217:/proc/get_random_email_list()`
- `222:/datum/computer/file/record/random_email/New(mailName as text)`

### `code\modules\networks\computer3\mainframe2\emailserv.dm`
- `12:/datum/computer/file/mainframe_program/srv/email/initialize(var/initparams)`

### `code\modules\networks\computer3\mainframe2\filetypes.dm`
- `19:/datum/computer/file/user_data/disposing()`
- `40:/datum/mainframe2_user_data/disposing()`
- `64:/datum/computer/file/document/disposing()`

### `code\modules\networks\computer3\mainframe2\logreader.dm`
- `7:/datum/computer/file/record/accesslog_default_config/New()`
- `38:/obj/machinery/networked/logreader/New()`
- `50:/obj/machinery/networked/logreader/attack_hand(var/mob/user)`
- `148:/obj/machinery/networked/logreader/Topic(href, href_list)`
- `272:/obj/machinery/networked/logreader/process()`
- `294:/obj/machinery/networked/logreader/receive_signal(datum/signal/signal)`
- `434:/datum/computer/file/mainframe_program/srv/accesslog/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `481:/datum/computer/file/mainframe_program/srv/accesslog/initialize(var/initparams)`
- `679:/datum/computer/file/mainframe_program/driver/mountable/logreader/initialize(var/initparams)`
- `686:/datum/computer/file/mainframe_program/driver/mountable/logreader/process()`
- `689:/datum/computer/file/mainframe_program/driver/mountable/logreader/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `707:/datum/computer/file/mainframe_program/driver/mountable/logreader/terminal_input(var/data, var/datum/computer/file/theFile)`
- `739:/datum/computer/file/mainframe_program/driver/mountable/logreader/add_file(var/datum/computer/file/theFile)`
- `751:/datum/computer/file/mainframe_program/driver/mountable/logreader/change_metadata(var/datum/computer/file/file, var/field, var/newval)`

### `code\modules\networks\computer3\mainframe2\mainframe2.dm`
- `12:/datum/terminal_connection/New(obj/machinery/networked/master, net_id, term_type)`
- `21:/datum/terminal_connection/disposing()`
- `74:/obj/machinery/networked/mainframe/New()`
- `106:/obj/machinery/networked/mainframe/disposing()`
- `139:/obj/machinery/networked/mainframe/attack_ai(mob/user)`
- `142:/obj/machinery/networked/mainframe/attack_hand(mob/user)`
- `168:/obj/machinery/networked/mainframe/Topic(href, href_list)`
- `224:/obj/machinery/networked/mainframe/attackby(obj/item/W, mob/user)`
- `241:/obj/machinery/networked/mainframe/zeta/boutput(user, "You pry out the memory core.")`
- `244:/obj/machinery/networked/mainframe/process()`
- `302:/obj/machinery/networked/mainframe/receive_signal(datum/signal/signal)`
- `401:/obj/machinery/networked/mainframe/power_change()`
- `418:/obj/machinery/networked/mainframe/clone()`
- `428:/obj/machinery/networked/mainframe/meteorhit(obj/O)`
- `437:/obj/machinery/networked/mainframe/ex_act(severity)`
- `448:/obj/machinery/networked/mainframe/blob_act(power)`
- `456:/obj/machinery/networked/mainframe/proc/run_program(datum/computer/file/mainframe_program/program, datum/mainframe2_user_data/user, datum/computer/file/mainframe_program/caller_prog, runparams, allow_fork = FALSE)`
- `532:/obj/machinery/networked/mainframe/proc/unload_all()`
- `543:/obj/machinery/networked/mainframe/proc/unload_program(datum/computer/file/mainframe_program/program)`
- `568:/obj/machinery/networked/mainframe/proc/relay_progsignal(datum/computer/file/mainframe_program/caller_prog, progid, list/data, datum/computer/file/file)`
- `579:/obj/machinery/networked/mainframe/proc/reconnect_all_devices()`
- `590:/obj/machinery/networked/mainframe/proc/reconnect_device(device_id)`
- `606:/obj/machinery/networked/mainframe/proc/reboot_mainframe()`
- `613:/obj/machinery/networked/mainframe/proc/post_system()`

### `code\modules\networks\computer3\mainframe2\misc_terms.dm`
- `76:/obj/machinery/networked/Topic(href, href_list)`
- `96:/obj/machinery/networked/disposing()`
- `103:/obj/machinery/networked/set_loc(atom/target)`
- `153:/obj/machinery/networked/storage/clone()`
- `172:/obj/machinery/networked/storage/New()`
- `201:/obj/machinery/networked/storage/disposing()`
- `208:/obj/machinery/networked/storage/process()`
- `232:/obj/machinery/networked/storage/attack_hand(mob/user)`
- `274:/obj/machinery/networked/storage/Topic(href, href_list)`
- `348:/obj/machinery/networked/storage/attackby(obj/item/W, mob/user)`
- `378:/obj/machinery/networked/storage/receive_signal(datum/signal/signal)`
- `617:/obj/machinery/networked/storage/power_change()`
- `705:/obj/machinery/networked/storage/bomb_tester/power_change()`
- `717:/obj/machinery/networked/storage/bomb_tester/process()`
- `797:/obj/machinery/networked/storage/bomb_tester/ui_act(action, params)`
- `851:/obj/machinery/networked/storage/bomb_tester/ui_interact(mob/user, datum/tgui/ui)`
- `861:/obj/machinery/networked/storage/bomb_tester/ui_data()`
- `875:/obj/machinery/networked/storage/bomb_tester/update_icon()`
- `898:/obj/machinery/networked/storage/bomb_tester/attackby(obj/item/I, mob/user)`
- `903:/obj/machinery/networked/storage/bomb_tester/attack_hand(mob/user)`
- `1056:/obj/machinery/networked/nuclear_charge/New()`
- `1068:/obj/machinery/networked/nuclear_charge/attack_hand(mob/user)`
- `1097:/obj/machinery/networked/nuclear_charge/Topic(href, href_list)`
- `1124:/obj/machinery/networked/nuclear_charge/was_deconstructed_to_frame(mob/user)`
- `1129:/obj/machinery/networked/nuclear_charge/process()`
- `1171:/obj/machinery/networked/nuclear_charge/power_change()`
- `1188:/obj/machinery/networked/nuclear_charge/attackby(obj/item/W, mob/user)`
- `1198:/obj/machinery/networked/nuclear_charge/receive_signal(datum/signal/signal)`
- `1398:/obj/machinery/networked/radio/New()`
- `1419:/obj/machinery/networked/radio/attack_hand(mob/user)`
- `1461:/obj/machinery/networked/radio/attackby(obj/item/W, mob/user)`
- `1471:/obj/machinery/networked/radio/Topic(href, href_list)`
- `1499:/obj/machinery/networked/radio/power_change()`
- `1508:/obj/machinery/networked/radio/process()`
- `1530:/obj/machinery/networked/radio/receive_signal(datum/signal/signal, transmission_type, range, connection_id)`
- `1727:/obj/machinery/networked/printer/New()`
- `1747:/obj/machinery/networked/printer/disposing()`
- `1752:/obj/machinery/networked/printer/attackby(obj/item/W, mob/user)`
- `1807:/obj/machinery/networked/printer/onDestroy()`
- `1813:/obj/machinery/networked/printer/attack_hand(mob/user)`
- `1849:/obj/machinery/networked/printer/Topic(href, href_list)`
- `1891:/obj/machinery/networked/printer/process()`
- `1920:/obj/machinery/networked/printer/receive_signal(datum/signal/signal)`
- `2158:/obj/machinery/networked/printer/update_icon()`
- `2202:/obj/machinery/networked/storage/scanner/New()`
- `2207:/obj/machinery/networked/storage/scanner/attack_hand(mob/user)`
- `2244:/obj/machinery/networked/storage/scanner/attackby(obj/item/W, mob/user)`
- `2262:/obj/machinery/networked/storage/scanner/MouseDrop_T(obj/item/W, mob/user)`
- `2282:/obj/machinery/networked/storage/scanner/Topic(href, href_list)`
- `2323:/obj/machinery/networked/storage/scanner/power_change()`
- `2333:/obj/machinery/networked/storage/scanner/process()`
- `2462:/obj/machinery/networked/secdetector/New()`
- `2484:/obj/machinery/networked/secdetector/disposing()`
- `2490:/obj/machinery/networked/secdetector/disposing()`
- `2500:/obj/machinery/networked/secdetector/attack_hand(mob/user)`
- `2542:/obj/machinery/networked/secdetector/Topic(href, href_list)`
- `2569:/obj/machinery/networked/secdetector/process()`
- `2618:/obj/machinery/networked/secdetector/receive_signal(datum/signal/signal)`
- `2704:/obj/machinery/networked/secdetector/ex_act(severity)`
- `2718:/obj/machinery/networked/secdetector/power_change()`
- `2727:/obj/machinery/networked/secdetector/update_icon(var/newState = 1)`
- `2798:/obj/beam/disposing()`
- `2804:/obj/beam/Bumped()`
- `2808:/obj/beam/Crossed(atom/movable/AM as mob|obj)`
- `2853:/obj/beam/ir_beam/New(location, newLimit)`
- `2861:/obj/beam/ir_beam/disposing()`
- `2867:/obj/beam/ir_beam/disposing()`
- `2874:/obj/beam/ir_beam/Crossed(atom/movable/AM as mob|obj)`
- `2884:/obj/beam/ir_beam/hit()`
- `2891:/obj/beam/ir_beam/generate_next()`
- `2918:/obj/machinery/networked/h7_emitter/New()`
- `2934:/obj/machinery/networked/h7_emitter/disposing()`
- `2940:/obj/machinery/networked/h7_emitter/disposing()`
- `2955:/obj/machinery/networked/h7_emitter/attackby(obj/item/W, mob/user)`
- `2968:/obj/machinery/networked/h7_emitter/attack_hand(mob/user)`
- `3018:/obj/machinery/networked/h7_emitter/Topic(href, href_list)`
- `3098:/obj/machinery/networked/h7_emitter/process()`
- `3107:/obj/machinery/networked/h7_emitter/receive_signal(datum/signal/signal)`
- `3195:/obj/machinery/networked/h7_emitter/power_change()`
- `3204:/obj/machinery/networked/h7_emitter/ex_act(severity)`
- `3219:/obj/machinery/networked/h7_emitter/update_icon()`
- `3273:/obj/machinery/networked/test_apparatus/New()`
- `3289:/obj/machinery/networked/test_apparatus/attackby(obj/item/W, mob/user)`
- `3303:/obj/machinery/networked/test_apparatus/attack_hand(mob/user)`
- `3334:/obj/machinery/networked/test_apparatus/MouseDrop_T(atom/movable/O as mob|obj, mob/user as mob)`
- `3347:/obj/machinery/networked/test_apparatus/mouse_drop(obj/over_object as obj, src_location, over_location)`
- `3369:/obj/machinery/networked/test_apparatus/Topic(href, href_list)`
- `3399:/obj/machinery/networked/test_apparatus/process()`
- `3411:/obj/machinery/networked/test_apparatus/receive_signal(datum/signal/signal)`
- `3493:/obj/machinery/networked/test_apparatus/power_change()`
- `3502:/obj/machinery/networked/test_apparatus/ex_act(severity)`
- `3517:/obj/machinery/networked/test_apparatus/update_icon()`
- `3573:/obj/machinery/networked/test_apparatus/pitching_machine/return_html_interface()`
- `3576:/obj/machinery/networked/test_apparatus/pitching_machine/message_interface(var/list/packetData)`
- `3632:/obj/machinery/networked/test_apparatus/pitching_machine/process()`
- `3659:/obj/machinery/networked/test_apparatus/pitching_machine/attackby(var/obj/item/I, mob/user)`
- `3694:/obj/machinery/networked/test_apparatus/pitching_machine/MouseDrop_T(atom/movable/O as mob|obj, mob/user as mob)`
- `3713:/obj/machinery/networked/test_apparatus/impact_pad/return_html_interface()`
- `3716:/obj/machinery/networked/test_apparatus/impact_pad/message_interface(var/list/packetData)`
- `3774:/obj/machinery/networked/test_apparatus/impact_pad/attackby(var/obj/item/I, mob/user)`
- `3808:/obj/machinery/networked/test_apparatus/impact_pad/MouseDrop_T(atom/movable/O as mob|obj, mob/user as mob)`
- `3814:/obj/machinery/networked/test_apparatus/impact_pad/hitby(atom/movable/M, datum/thrown_thing/thr)`
- `3824:/obj/machinery/networked/test_apparatus/impact_pad/bullet_act(var/obj/projectile/P)`
- `3890:/obj/machinery/networked/test_apparatus/electrobox/return_html_interface()`
- `3893:/obj/machinery/networked/test_apparatus/electrobox/update_icon()`
- `3900:/obj/machinery/networked/test_apparatus/electrobox/process()`
- `3937:/obj/machinery/networked/test_apparatus/electrobox/attackby(var/obj/item/I, mob/user)`
- `3965:/obj/machinery/networked/test_apparatus/electrobox/message_interface(var/list/packetData)`
- `4100:/obj/machinery/networked/test_apparatus/xraymachine/return_html_interface()`
- `4103:/obj/machinery/networked/test_apparatus/xraymachine/update_icon()`
- `4110:/obj/machinery/networked/test_apparatus/xraymachine/attackby(var/obj/item/I, mob/user)`
- `4138:/obj/machinery/networked/test_apparatus/xraymachine/message_interface(var/list/packetData)`
- `4297:/obj/machinery/networked/test_apparatus/heater/New()`
- `4301:/obj/machinery/networked/test_apparatus/heater/return_html_interface()`
- `4304:/obj/machinery/networked/test_apparatus/heater/update_icon()`
- `4323:/obj/machinery/networked/test_apparatus/heater/attackby(var/obj/item/I, mob/user)`
- `4347:/obj/machinery/networked/test_apparatus/heater/process()`
- `4379:/obj/machinery/networked/test_apparatus/heater/message_interface(var/list/packetData)`
- `4502:/obj/machinery/networked/test_apparatus/laserE/return_html_interface()`
- `4505:/obj/machinery/networked/test_apparatus/laserE/process()`
- `4523:/obj/machinery/networked/test_apparatus/laserE/message_interface(var/list/packetData)`
- `4611:/obj/machinery/networked/test_apparatus/gas_sensor/New()`
- `4616:/obj/machinery/networked/test_apparatus/gas_sensor/return_html_interface()`
- `4619:/obj/machinery/networked/test_apparatus/gas_sensor/message_interface(var/list/packetData)`
- `4684:/obj/machinery/networked/test_apparatus/mechanics/New()`
- `4697:/obj/machinery/networked/test_apparatus/mechanics/return_html_interface()`
- `4707:/obj/machinery/networked/test_apparatus/mechanics/message_interface(var/list/packetData)`
- `4792:/obj/machinery/networked/test_apparatus/mechanics/process()`

### `code\modules\networks\computer3\mainframe2\os_drivers.dm`
- `31:/datum/computer/file/mainframe_program/driver/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `37:/datum/computer/file/mainframe_program/driver/asText()`
- `60:/datum/computer/file/mainframe_program/driver/mountable/disposing()`
- `83:/datum/computer/file/mainframe_program/driver/mountable/disposing()`
- `137:/datum/computer/file/mainframe_program/driver/mountable/user_terminal/initialize(var/initparams)`
- `144:/datum/computer/file/mainframe_program/driver/mountable/user_terminal/add_file(var/datum/computer/file/a_file, var/datum/mainframe2_user_data/user)`
- `170:/datum/computer/file/mainframe_program/driver/mountable/databank/initialize(var/initparams)`
- `178:/datum/computer/file/mainframe_program/driver/mountable/databank/terminal_input(var/data, var/datum/computer/file/file)`
- `304:/datum/computer/file/mainframe_program/driver/mountable/databank/remove_file(var/datum/computer/file/file)`
- `321:/datum/computer/file/mainframe_program/driver/mountable/databank/change_metadata(var/datum/computer/file/file, var/field, var/newval)`
- `378:/datum/computer/folder/mountpoint/New(var/datum/computer/file/mainframe_program/driver/mountable/newdriver)`
- `389:/datum/computer/folder/mountpoint/disposing()`
- `395:/datum/computer/folder/mountpoint/disposing()`
- `404:/datum/computer/folder/mountpoint/add_file(datum/computer/R, misc)`
- `411:/datum/computer/folder/mountpoint/remove_file(datum/computer/R, misc)`
- `428:/datum/computer/file/mainframe_program/driver/mountable/printer/disposing()`
- `434:/datum/computer/file/mainframe_program/driver/mountable/printer/initialize(var/initparams)`
- `455:/datum/computer/file/mainframe_program/driver/mountable/printer/add_file(var/datum/computer/file/theFile)`
- `466:/datum/computer/file/mainframe_program/driver/mountable/printer/remove_file(var/datum/computer/file/file)`
- `477:/datum/computer/file/mainframe_program/driver/mountable/printer/change_metadata(var/datum/computer/file/file, var/field, var/newval)`
- `480:/datum/computer/file/mainframe_program/driver/mountable/printer/process()`
- `518:/datum/computer/file/mainframe_program/driver/mountable/printer/terminal_input(var/data, var/datum/computer/file/file)`
- `633:/datum/computer/file/mainframe_program/driver/telepad/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `738:/datum/computer/file/mainframe_program/driver/telepad/terminal_input(var/data, var/datum/computer/file/file)`
- `775:/datum/computer/file/mainframe_program/srv/telecontrol/initialize(var/initparams)`
- `1035:/datum/computer/file/mainframe_program/driver/nuke/initialize(var/initparams)`
- `1042:/datum/computer/file/mainframe_program/driver/nuke/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `1132:/datum/computer/file/mainframe_program/driver/nuke/process()`
- `1148:/datum/computer/file/mainframe_program/driver/nuke/terminal_input(var/data, var/datum/computer/file/file)`
- `1204:/datum/computer/file/mainframe_program/nuke_interface/initialize(var/initparams)`
- `1328:/datum/computer/file/mainframe_program/nuke_interface/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `1391:/datum/computer/file/mainframe_program/driver/mountable/guard_dock/initialize(var/initparams)`
- `1410:/datum/computer/file/mainframe_program/driver/mountable/guard_dock/terminal_input(var/data, var/datum/computer/file/file)`
- `1577:/datum/computer/file/mainframe_program/driver/mountable/guard_dock/add_file(var/datum/computer/file/theFile)`
- `1598:/datum/computer/file/mainframe_program/driver/mountable/guard_dock/remove_file(var/datum/computer/file/theFile)`
- `1609:/datum/computer/file/mainframe_program/driver/mountable/guard_dock/change_metadata(var/datum/computer/file/file, var/field, var/newval)`
- `1619:/datum/computer/file/mainframe_program/guardbot_interface/initialize(var/initparams)`
- `1836:/datum/computer/file/mainframe_program/driver/mountable/radio/initialize(var/initparams)`
- `1843:/datum/computer/file/mainframe_program/driver/mountable/radio/process()`
- `1846:/datum/computer/file/mainframe_program/driver/mountable/radio/disposing()`
- `1858:/datum/computer/file/mainframe_program/driver/mountable/radio/terminal_input(var/data, var/datum/computer/file/theFile)`
- `1945:/datum/computer/file/mainframe_program/driver/mountable/radio/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `1973:/datum/computer/file/mainframe_program/driver/mountable/radio/add_file(var/datum/computer/file/theFile, var/freq)`
- `2011:/datum/computer/file/mainframe_program/driver/mountable/radio/remove_file(var/datum/computer/file/file)`
- `2024:/datum/computer/file/mainframe_program/driver/mountable/radio/change_metadata(var/datum/computer/file/file, var/field, var/newval)`
- `2030:/datum/computer/folder/radio_channel/New(var/datum/computer/file/mainframe_program/driver/mountable/radio/newDriver)`
- `2036:/datum/computer/folder/radio_channel/can_add_file(datum/computer/file/R)`
- `2039:/datum/computer/folder/radio_channel/add_file(var/datum/computer/file/theFile)`
- `2045:/datum/computer/folder/radio_channel/remove_file(var/datum/computer/file/theFile)`
- `2067:/datum/computer/file/mainframe_program/driver/secdetector/process()`
- `2070:/datum/computer/file/mainframe_program/driver/secdetector/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `2101:/datum/computer/file/mainframe_program/driver/secdetector/terminal_input(var/data, var/datum/computer/file/file)`
- `2150:/datum/computer/file/mainframe_program/driver/apc/process()`
- `2153:/datum/computer/file/mainframe_program/driver/apc/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `2186:/datum/computer/file/mainframe_program/driver/apc/terminal_input(var/data, var/datum/computer/file/file)`
- `2230:/datum/computer/file/mainframe_program/driver/hept_emitter/process()`
- `2233:/datum/computer/file/mainframe_program/driver/hept_emitter/terminal_input(var/data, var/datum/computer/file/file)`
- `2247:/datum/computer/file/mainframe_program/driver/hept_emitter/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `2279:/datum/computer/file/mainframe_program/hept_interface/initialize(var/initparams)`
- `2342:/datum/computer/file/mainframe_program/h7init/initialize()`
- `2350:/datum/computer/file/mainframe_program/h7init/process()`
- `2431:/datum/computer/file/mainframe_program/driver/test_apparatus/process()`
- `2434:/datum/computer/file/mainframe_program/driver/test_apparatus/receive_progsignal(var/sendid, var/list/data, var/datum/computer/file/file)`
- `2525:/datum/computer/file/mainframe_program/driver/test_apparatus/terminal_input(var/data, var/datum/computer/file/file)`
- `2738:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/process()`
- `2742:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/disposing()`
- `2748:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/disposing()`
- `2755:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/initialize(initparams)`
- `2794:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/terminal_input(var/data, var/datum/computer/file/theFile)`
- `2833:/datum/computer/file/mainframe_program/driver/mountable/service_terminal/receive_progsignal(var/sendid, var/list/data = null, var/datum/computer/file/file)`
- `2867:/datum/computer/file/mainframe_program/srv/print/initialize(var/initparams)`
- `2957:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/disposing()`
- `2962:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/initialize(var/initparams)`
- `2982:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/add_file(var/datum/computer/file/theFile)`
- `2992:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/remove_file(var/datum/computer/file/file)`
- `3002:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/change_metadata(var/datum/computer/file/file, var/field, var/newval)`
- `3005:/datum/computer/file/mainframe_program/driver/mountable/comm_dish/terminal_input(var/data, var/datum/computer/file/record/recfile)`

### `code\modules\networks\computer3\mainframe2\programs\_program_parent.dm`
- `29:/datum/computer/file/mainframe_program/New(obj/holding)`
- `38:/datum/computer/file/mainframe_program/disposing()`
- `45:/datum/computer/file/mainframe_program/asText()`
- `52:/datum/computer/file/mainframe_program/proc/ensure_program()`
- `71:/datum/computer/file/mainframe_program/proc/input_text(text)`
- `78:/datum/computer/file/mainframe_program/proc/initialize(initparams)`
- `86:/datum/computer/file/mainframe_program/proc/handle_quit()`
- `90:/datum/computer/file/mainframe_program/proc/process()`
- `97:/datum/computer/file/mainframe_program/proc/parse_string(string, list/replace_list)`
- `104:/datum/computer/file/mainframe_program/proc/get_folder_name(string, datum/computer/folder/check_folder, datum/mainframe2_user_data/user)`
- `119:/datum/computer/file/mainframe_program/proc/get_file_name(string, datum/computer/folder/check_folder, datum/mainframe2_user_data/user)`
- `134:/datum/computer/file/mainframe_program/proc/get_computer_datum(string, datum/computer/folder/check_folder, datum/mainframe2_user_data/user)`
- `149:/datum/computer/file/mainframe_program/proc/is_name_invalid(string)`
- `163:/datum/computer/file/mainframe_program/proc/message_user(msg, render, file)`
- `176:/datum/computer/file/mainframe_program/proc/read_user_field(field)`
- `186:/datum/computer/file/mainframe_program/proc/write_user_field(field, data)`
- `202:/datum/computer/file/mainframe_program/proc/signal_program(progid, list/data, datum/computer/file/file)`
- `212:/datum/computer/file/mainframe_program/proc/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `216:/datum/computer/file/mainframe_program/proc/unloaded()`
- `220:/datum/computer/file/mainframe_program/proc/check_read_permission(datum/computer/target, datum/mainframe2_user_data/user)`
- `251:/datum/computer/file/mainframe_program/proc/check_write_permission(datum/computer/target, datum/mainframe2_user_data/user)`
- `284:/datum/computer/file/mainframe_program/proc/check_mode_permission(datum/computer/target, datum/mainframe2_user_data/user)`
- `324:/proc/command2list(text, separator, list/replaceList, list/substitution_feedback_thing)`

### `code\modules\networks\computer3\mainframe2\programs\os\_os_parent.dm`
- `14:/datum/computer/file/mainframe_program/os/proc/new_connection(datum/terminal_connection/conn)`
- `18:/datum/computer/file/mainframe_program/os/proc/closed_connection(datum/terminal_connection/conn)`
- `22:/datum/computer/file/mainframe_program/os/proc/term_input(data, termid, datum/computer/file/file)`
- `29:/datum/computer/file/mainframe_program/os/proc/ping_reply(senderid, sendertype)`
- `33:/datum/computer/file/mainframe_program/os/proc/message_term(message, termid, render)`
- `43:/datum/computer/file/mainframe_program/os/proc/file_term(datum/computer/file/file, termid, exdata)`
- `63:/datum/computer/file/mainframe_program/os/proc/parse_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`
- `155:/datum/computer/file/mainframe_program/os/proc/parse_file_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`
- `236:/datum/computer/file/mainframe_program/os/proc/parse_datum_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`

### `code\modules\networks\computer3\mainframe2\programs\os\bootloader.dm`
- `36:/datum/computer/file/mainframe_program/os/bootloader/New()`
- `40:/datum/computer/file/mainframe_program/os/bootloader/disposing()`
- `44:/datum/computer/file/mainframe_program/os/bootloader/initialize()`
- `60:/datum/computer/file/mainframe_program/os/bootloader/process()`
- `91:/datum/computer/file/mainframe_program/os/bootloader/new_connection(datum/terminal_connection/conn)`
- `97:/datum/computer/file/mainframe_program/os/bootloader/closed_connection(datum/terminal_connection/conn)`
- `103:/datum/computer/file/mainframe_program/os/bootloader/ping_reply(senderid, sendertype)`
- `116:/datum/computer/file/mainframe_program/os/bootloader/term_input(data, termid, datum/computer/file/file)`
- `219:/datum/computer/file/mainframe_program/os/bootloader/proc/new_current()`
- `233:/datum/computer/file/mainframe_program/os/bootloader/proc/clear_core()`
- `246:/datum/computer/file/mainframe_program/os/bootloader/proc/disconnect_devices()`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\kernel.dm`
- `36:/datum/computer/file/mainframe_program/os/kernel/New()`
- `42:/datum/computer/file/mainframe_program/os/kernel/disposing()`
- `57:/datum/computer/file/mainframe_program/os/kernel/initialize()`
- `95:/datum/computer/file/mainframe_program/os/kernel/term_input(data, termid, datum/computer/file/file, is_break = FALSE)`
- `127:/datum/computer/file/mainframe_program/os/kernel/new_connection(datum/terminal_connection/conn, datum/computer/file/connect_file)`
- `180:/datum/computer/file/mainframe_program/os/kernel/closed_connection(datum/terminal_connection/conn)`
- `192:/datum/computer/file/mainframe_program/os/kernel/ping_reply(senderid, sendertype)`
- `215:/datum/computer/file/mainframe_program/os/kernel/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `231:/datum/computer/file/mainframe_program/os/kernel/process()`
- `246:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_users()`
- `301:/datum/computer/file/mainframe_program/os/kernel/proc/login_temp_user(user_netid, datum/computer/file/record/login_record, datum/computer/file/mainframe_program/caller_prog_override)`
- `330:/datum/computer/file/mainframe_program/os/kernel/proc/login_user(datum/mainframe2_user_data/account, user_name, sysop = FALSE, interactive = TRUE)`
- `407:/datum/computer/file/mainframe_program/os/kernel/proc/logout_user(datum/mainframe2_user_data/user, disconnect = FALSE)`
- `421:/datum/computer/file/mainframe_program/os/kernel/proc/logout_all_users(disconnect = FALSE)`
- `430:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_drivers()`
- `489:/datum/computer/file/mainframe_program/os/kernel/proc/initialize_driver(datum/computer/file/mainframe_program/driver/driver, datum/computer/file/connect_file)`
- `511:/datum/computer/file/mainframe_program/os/kernel/proc/is_sysop(datum/mainframe2_user_data/udat)`
- `521:/datum/computer/file/mainframe_program/os/kernel/proc/change_metadata(datum/computer/file/file, field, newval)`
- `533:/datum/computer/file/mainframe_program/os/kernel/proc/message_all_users(message, sender_name, ignore_user_file_setting)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\_syscall.dm`
- `12:/datum/dwaine_syscall/New(datum/computer/file/mainframe_program/os/kernel/kernel)`
- `16:/datum/dwaine_syscall/disposing()`
- `21:/datum/dwaine_syscall/proc/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\confget.dm`
- `4:/datum/dwaine_syscall/confget/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\dget.dm`
- `4:/datum/dwaine_syscall/dget/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\dlist.dm`
- `4:/datum/dwaine_syscall/dlist/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\dmsg.dm`
- `4:/datum/dwaine_syscall/dmsg/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\dscan.dm`
- `4:/datum/dwaine_syscall/dscan/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\exit.dm`
- `4:/datum/dwaine_syscall/exit/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\fget.dm`
- `4:/datum/dwaine_syscall/fget/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\fkill.dm`
- `4:/datum/dwaine_syscall/fkill/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\fmode.dm`
- `4:/datum/dwaine_syscall/fmode/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\fowner.dm`
- `4:/datum/dwaine_syscall/fowner/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\fwrite.dm`
- `4:/datum/dwaine_syscall/fwrite/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\mount.dm`
- `4:/datum/dwaine_syscall/mount/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\msg_term.dm`
- `4:/datum/dwaine_syscall/msg_term/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\tfork.dm`
- `4:/datum/dwaine_syscall/tfork/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\tkill.dm`
- `4:/datum/dwaine_syscall/tkill/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\tlist.dm`
- `4:/datum/dwaine_syscall/tlist/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\tspawn.dm`
- `4:/datum/dwaine_syscall/tspawn/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\ugroup.dm`
- `4:/datum/dwaine_syscall/ugroup/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\uinput.dm`
- `4:/datum/dwaine_syscall/uinput/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\ulist.dm`
- `4:/datum/dwaine_syscall/ulist/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\ulogin.dm`
- `4:/datum/dwaine_syscall/ulogin/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\kernel\syscalls\umsg.dm`
- `4:/datum/dwaine_syscall/umsg/execute(sendid, list/data, datum/computer/file/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\login.dm`
- `13:/datum/computer/file/mainframe_program/login/initialize()`
- `26:/datum/computer/file/mainframe_program/login/receive_progsignal(sendid, list/data, datum/computer/file/record/file)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\_shell_builtin.dm`
- `12:/datum/dwaine_shell_builtin/New(datum/computer/file/mainframe_program/shell/shell)`
- `16:/datum/dwaine_shell_builtin/disposing()`
- `21:/datum/dwaine_shell_builtin/proc/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\break.dm`
- `4:/datum/dwaine_shell_builtin/_break/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\cls.dm`
- `4:/datum/dwaine_shell_builtin/cls/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\echo.dm`
- `4:/datum/dwaine_shell_builtin/echo/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\else.dm`
- `4:/datum/dwaine_shell_builtin/_else/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\eval.dm`
- `4:/datum/dwaine_shell_builtin/eval/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\goonsay.dm`
- `4:/datum/dwaine_shell_builtin/goonsay/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\if.dm`
- `4:/datum/dwaine_shell_builtin/_if/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\logout.dm`
- `4:/datum/dwaine_shell_builtin/logout/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\man.dm`
- `4:/datum/dwaine_shell_builtin/man/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\mesg.dm`
- `4:/datum/dwaine_shell_builtin/mesg/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\sleep.dm`
- `4:/datum/dwaine_shell_builtin/_sleep/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\talk.dm`
- `4:/datum/dwaine_shell_builtin/talk/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\unset.dm`
- `4:/datum/dwaine_shell_builtin/unset/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\while.dm`
- `4:/datum/dwaine_shell_builtin/_while/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_builtins\who.dm`
- `4:/datum/dwaine_shell_builtin/who/execute(list/command_list, list/piped_list)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\_shell_script_operator.dm`
- `13:/datum/dwaine_shell_script_operator/New(datum/computer/file/mainframe_program/shell/shell)`
- `17:/datum/dwaine_shell_script_operator/disposing()`
- `22:/datum/dwaine_shell_script_operator/proc/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\add.dm`
- `10:/datum/dwaine_shell_script_operator/add/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\divide.dm`
- `10:/datum/dwaine_shell_script_operator/divide/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\modulo.dm`
- `9:/datum/dwaine_shell_script_operator/modulo/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\multiply.dm`
- `10:/datum/dwaine_shell_script_operator/multiply/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\rand.dm`
- `10:/datum/dwaine_shell_script_operator/rand/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\arithmetic\subtract.dm`
- `10:/datum/dwaine_shell_script_operator/subtract/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\file\exists.dm`
- `11:/datum/dwaine_shell_script_operator/exists/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\file\is_directory.dm`
- `11:/datum/dwaine_shell_script_operator/is_directory/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\file\is_executable.dm`
- `11:/datum/dwaine_shell_script_operator/is_executable/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\file\is_file.dm`
- `11:/datum/dwaine_shell_script_operator/is_file/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\logic\and.dm`
- `11:/datum/dwaine_shell_script_operator/and/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\logic\not.dm`
- `10:/datum/dwaine_shell_script_operator/not/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\logic\or.dm`
- `11:/datum/dwaine_shell_script_operator/or/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\logic\xor.dm`
- `11:/datum/dwaine_shell_script_operator/xor/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\assignment.dm`
- `8:/datum/dwaine_shell_script_operator/assignment/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\escape_string.dm`
- `12:/datum/dwaine_shell_script_operator/escape_string/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\stack_del_topmost.dm`
- `11:/datum/dwaine_shell_script_operator/stack_del_topmost/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\stack_depth.dm`
- `11:/datum/dwaine_shell_script_operator/stack_depth/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\stack_dup_topmost.dm`
- `13:/datum/dwaine_shell_script_operator/stack_dup_topmost/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\stack_pop.dm`
- `11:/datum/dwaine_shell_script_operator/stack_pop/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\misc\stack_print.dm`
- `11:/datum/dwaine_shell_script_operator/stack_print/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\eq.dm`
- `10:/datum/dwaine_shell_script_operator/eq/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\ge.dm`
- `12:/datum/dwaine_shell_script_operator/ge/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\gt.dm`
- `12:/datum/dwaine_shell_script_operator/gt/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\le.dm`
- `12:/datum/dwaine_shell_script_operator/le/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\lt.dm`
- `12:/datum/dwaine_shell_script_operator/lt/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell_script_operators\relational\ne.dm`
- `10:/datum/dwaine_shell_script_operator/ne/execute(list/token_stream)`

### `code\modules\networks\computer3\mainframe2\programs\os\shell\shell.dm`
- `39:/datum/computer/file/mainframe_program/shell/disposing()`
- `60:/datum/computer/file/mainframe_program/shell/initialize(list/supplied_config)`
- `114:/datum/computer/file/mainframe_program/shell/process()`
- `121:/datum/computer/file/mainframe_program/shell/input_text(text)`
- `243:/datum/computer/file/mainframe_program/shell/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `275:/datum/computer/file/mainframe_program/shell/message_user(msg, render, file)`
- `293:/datum/computer/file/mainframe_program/shell/proc/execpath(file_path, list/command_list, scripting = 0)`
- `342:/datum/computer/file/mainframe_program/shell/proc/script_process()`
- `380:/datum/computer/file/mainframe_program/shell/proc/script_format(list/script_list)`
- `381:/datum/computer/file/mainframe_program/shell/RETURN_TYPE(/list)`
- `398:/datum/computer/file/mainframe_program/shell/proc/script_evaluate(list/token_stream, return_bool = FALSE)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\_utility.dm`
- `38:/proc/optparse(data)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\cat.dm`
- `4:/datum/computer/file/mainframe_program/utility/cat/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\cd.dm`
- `4:/datum/computer/file/mainframe_program/utility/cd/initialize(initparams)`
- `24:/datum/computer/file/mainframe_program/utility/cd/proc/trim_path(filepath)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\chmod.dm`
- `4:/datum/computer/file/mainframe_program/utility/chmod/initialize(initparams)`
- `37:/datum/computer/file/mainframe_program/utility/chmod/proc/process_permissions(permissions)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\chown.dm`
- `4:/datum/computer/file/mainframe_program/utility/chown/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\cp.dm`
- `4:/datum/computer/file/mainframe_program/utility/cp/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\date.dm`
- `5:/datum/computer/file/mainframe_program/utility/date/initialize(initparams)`
- `56:/datum/computer/file/mainframe_program/utility/date/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `76:/datum/computer/file/mainframe_program/utility/date/proc/usage()`
- `82:/datum/computer/file/mainframe_program/utility/date/proc/message_reply_and_user(message)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\getopt.dm`
- `5:/datum/computer/file/mainframe_program/utility/getopt/initialize(initparams)`
- `44:/datum/computer/file/mainframe_program/utility/getopt/proc/invoke(list/string_list)`
- `137:/datum/computer/file/mainframe_program/utility/getopt/proc/message_reply_and_user(message)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\grep.dm`
- `5:/datum/computer/file/mainframe_program/utility/grep/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\ln.dm`
- `4:/datum/computer/file/mainframe_program/utility/ln/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\ls.dm`
- `4:/datum/computer/file/mainframe_program/utility/ls/initialize(initparams)`
- `48:/datum/computer/file/mainframe_program/utility/ls/proc/print_file_description(datum/computer/C)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\mkdir.dm`
- `4:/datum/computer/file/mainframe_program/utility/mkdir/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\mount.dm`
- `4:/datum/computer/file/mainframe_program/utility/mount/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\mv.dm`
- `4:/datum/computer/file/mainframe_program/utility/mv/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\pwd.dm`
- `4:/datum/computer/file/mainframe_program/utility/pwd/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\rm.dm`
- `12:/datum/computer/file/mainframe_program/utility/rm/initialize(initparams)`
- `61:/datum/computer/file/mainframe_program/utility/rm/input_text(text)`
- `82:/datum/computer/file/mainframe_program/utility/rm/message_user(msg, render, file)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\scnt.dm`
- `4:/datum/computer/file/mainframe_program/utility/scnt/initialize(initparams)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\su.dm`
- `4:/datum/computer/file/mainframe_program/utility/su/initialize(initparams)`
- `11:/datum/computer/file/mainframe_program/utility/su/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `33:/datum/computer/file/mainframe_program/utility/su/input_text(text)`

### `code\modules\networks\computer3\mainframe2\programs\utilities\tar.dm`
- `23:/datum/computer/file/mainframe_program/utility/tar/initialize(initparams)`
- `173:/datum/computer/file/mainframe_program/utility/tar/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `193:/datum/computer/file/mainframe_program/utility/tar/message_user(msg, render, file)`
- `199:/datum/computer/file/mainframe_program/utility/tar/proc/usage()`
- `205:/datum/computer/file/mainframe_program/utility/tar/proc/message_reply_and_user(message)`
- `213:/datum/computer/file/mainframe_program/utility/tar/proc/temp_file_name()`
- `218:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_list(datum/computer/target, current_path = "", depth = 0)`
- `233:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_extract(datum/computer/to_extract, target_path, current_path = "", depth = 0)`
- `276:/datum/computer/file/mainframe_program/utility/tar/proc/deep_copy(datum/computer/to_copy, current_path = "", depth = 0)`

### `code\modules\networks\computer3\mainframe2\tapes.dm`
- `16:/obj/item/disk/data/memcard/main2/New()`
- `143:/obj/item/disk/data/tape/master/New()`
- `179:/obj/item/disk/data/tape/boot2/New()`
- `224:/obj/item/disk/data/tape/test/New()`
- `236:/obj/item/disk/data/tape/guardbot_tools/New()`
- `255:/obj/item/disk/data/tape/artifact_research/New()`

### `code\modules\networks\computer3\mainframe2\telesci.dm`
- `62:/obj/machinery/networked/telepad/New()`
- `84:/obj/machinery/networked/telepad/ex_act()`
- `87:/obj/machinery/networked/telepad/examine()`
- `92:/obj/machinery/networked/telepad/ui_interact(mob/user, datum/tgui/ui)`
- `98:/obj/machinery/networked/telepad/ui_data(mob/user)`
- `103:/obj/machinery/networked/telepad/ui_act(action, params)`
- `126:/obj/machinery/networked/telepad/attack_hand(mob/user)`
- `131:/obj/machinery/networked/telepad/updateUsrDialog()`
- `138:/obj/machinery/networked/telepad/receive_signal(datum/signal/signal)`
- `432:/obj/machinery/networked/telepad/process()`
- `963:/obj/machinery/networked/teleconsole/New()`
- `976:/obj/machinery/networked/teleconsole/disposing()`
- `980:/obj/machinery/networked/teleconsole/receive_signal(datum/signal/signal)`
- `1072:/obj/machinery/networked/teleconsole/attackby(obj/item/W, mob/user)`
- `1089:/obj/machinery/networked/teleconsole/process()`
- `1110:/obj/machinery/networked/teleconsole/ui_interact(mob/user, datum/tgui/ui)`
- `1116:/obj/machinery/networked/teleconsole/ui_data(mob/user)`
- `1150:/obj/machinery/networked/teleconsole/ui_act(action, params)`

### `code\modules\networks\computer3\terminal.dm`
- `19:/datum/computer/file/terminal_program/os/terminal_os/input_text(text)`
- `360:/datum/computer/file/terminal_program/os/terminal_os/initialize()`
- `409:/datum/computer/file/terminal_program/os/terminal_os/disk_ejected(var/obj/item/disk/data/thedisk)`
- `444:/datum/computer/file/terminal_program/os/terminal_os/restart()`
- `455:/datum/computer/file/terminal_program/os/terminal_os/process()`
- `469:/datum/computer/file/terminal_program/os/terminal_os/receive_command(obj/source, command, datum/signal/signal)`

