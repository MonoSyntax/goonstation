# DWAINE Full Proc Inventory

Machine-generated from `dwaine_proc_index.txt`.

- Total proc entries: 502
- Total files: 104

## `code/modules/networks/computer3/base_os.dm`
- `27:/datum/computer/file/terminal_program/os/main_os/disposing()`
- `37:/datum/computer/file/terminal_program/os/main_os/input_text(text)`
- `594:/datum/computer/file/terminal_program/os/main_os/initialize()`
- `629:/datum/computer/file/terminal_program/os/main_os/disk_ejected(var/obj/item/disk/data/thedisk)`
- `644:/datum/computer/file/terminal_program/os/main_os/receive_command(obj/source, command, datum/signal/signal)`

## `code/modules/networks/computer3/mainframe2/artifact_res.dm`
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

## `code/modules/networks/computer3/mainframe2/documents.dm`
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

## `code/modules/networks/computer3/mainframe2/emailserv.dm`
- `12:/datum/computer/file/mainframe_program/srv/email/initialize(var/initparams)`

## `code/modules/networks/computer3/mainframe2/filetypes.dm`
- `19:/datum/computer/file/user_data/disposing()`
- `40:/datum/mainframe2_user_data/disposing()`
- `64:/datum/computer/file/document/disposing()`

## `code/modules/networks/computer3/mainframe2/logreader.dm`
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

## `code/modules/networks/computer3/mainframe2/mainframe2.dm`
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

## `code/modules/networks/computer3/mainframe2/misc_terms.dm`
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

## `code/modules/networks/computer3/mainframe2/os_drivers.dm`
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

## `code/modules/networks/computer3/mainframe2/programs/_program_parent.dm`
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

## `code/modules/networks/computer3/mainframe2/programs/os/_os_parent.dm`
- `14:/datum/computer/file/mainframe_program/os/proc/new_connection(datum/terminal_connection/conn)`
- `18:/datum/computer/file/mainframe_program/os/proc/closed_connection(datum/terminal_connection/conn)`
- `22:/datum/computer/file/mainframe_program/os/proc/term_input(data, termid, datum/computer/file/file)`
- `29:/datum/computer/file/mainframe_program/os/proc/ping_reply(senderid, sendertype)`
- `33:/datum/computer/file/mainframe_program/os/proc/message_term(message, termid, render)`
- `43:/datum/computer/file/mainframe_program/os/proc/file_term(datum/computer/file/file, termid, exdata)`
- `63:/datum/computer/file/mainframe_program/os/proc/parse_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`
- `155:/datum/computer/file/mainframe_program/os/proc/parse_file_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`
- `236:/datum/computer/file/mainframe_program/os/proc/parse_datum_directory(string, datum/computer/folder/origin, create_if_missing, datum/mainframe2_user_data/user)`

## `code/modules/networks/computer3/mainframe2/programs/os/bootloader.dm`
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

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/kernel.dm`
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

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/_syscall.dm`
- `12:/datum/dwaine_syscall/New(datum/computer/file/mainframe_program/os/kernel/kernel)`
- `16:/datum/dwaine_syscall/disposing()`
- `21:/datum/dwaine_syscall/proc/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/confget.dm`
- `4:/datum/dwaine_syscall/confget/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dget.dm`
- `4:/datum/dwaine_syscall/dget/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dlist.dm`
- `4:/datum/dwaine_syscall/dlist/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dmsg.dm`
- `4:/datum/dwaine_syscall/dmsg/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/dscan.dm`
- `4:/datum/dwaine_syscall/dscan/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/exit.dm`
- `4:/datum/dwaine_syscall/exit/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fget.dm`
- `4:/datum/dwaine_syscall/fget/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fkill.dm`
- `4:/datum/dwaine_syscall/fkill/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fmode.dm`
- `4:/datum/dwaine_syscall/fmode/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fowner.dm`
- `4:/datum/dwaine_syscall/fowner/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/fwrite.dm`
- `4:/datum/dwaine_syscall/fwrite/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/mount.dm`
- `4:/datum/dwaine_syscall/mount/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/msg_term.dm`
- `4:/datum/dwaine_syscall/msg_term/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tfork.dm`
- `4:/datum/dwaine_syscall/tfork/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tkill.dm`
- `4:/datum/dwaine_syscall/tkill/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tlist.dm`
- `4:/datum/dwaine_syscall/tlist/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/tspawn.dm`
- `4:/datum/dwaine_syscall/tspawn/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ugroup.dm`
- `4:/datum/dwaine_syscall/ugroup/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/uinput.dm`
- `4:/datum/dwaine_syscall/uinput/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ulist.dm`
- `4:/datum/dwaine_syscall/ulist/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/ulogin.dm`
- `4:/datum/dwaine_syscall/ulogin/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/kernel/syscalls/umsg.dm`
- `4:/datum/dwaine_syscall/umsg/execute(sendid, list/data, datum/computer/file/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/login.dm`
- `13:/datum/computer/file/mainframe_program/login/initialize()`
- `26:/datum/computer/file/mainframe_program/login/receive_progsignal(sendid, list/data, datum/computer/file/record/file)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell.dm`
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

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/_shell_builtin.dm`
- `12:/datum/dwaine_shell_builtin/New(datum/computer/file/mainframe_program/shell/shell)`
- `16:/datum/dwaine_shell_builtin/disposing()`
- `21:/datum/dwaine_shell_builtin/proc/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/break.dm`
- `4:/datum/dwaine_shell_builtin/_break/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/cls.dm`
- `4:/datum/dwaine_shell_builtin/cls/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/echo.dm`
- `4:/datum/dwaine_shell_builtin/echo/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/else.dm`
- `4:/datum/dwaine_shell_builtin/_else/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/eval.dm`
- `4:/datum/dwaine_shell_builtin/eval/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/goonsay.dm`
- `4:/datum/dwaine_shell_builtin/goonsay/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/if.dm`
- `4:/datum/dwaine_shell_builtin/_if/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/logout.dm`
- `4:/datum/dwaine_shell_builtin/logout/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/man.dm`
- `4:/datum/dwaine_shell_builtin/man/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/mesg.dm`
- `4:/datum/dwaine_shell_builtin/mesg/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/sleep.dm`
- `4:/datum/dwaine_shell_builtin/_sleep/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/talk.dm`
- `4:/datum/dwaine_shell_builtin/talk/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/unset.dm`
- `4:/datum/dwaine_shell_builtin/unset/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/while.dm`
- `4:/datum/dwaine_shell_builtin/_while/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_builtins/who.dm`
- `4:/datum/dwaine_shell_builtin/who/execute(list/command_list, list/piped_list)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/_shell_script_operator.dm`
- `13:/datum/dwaine_shell_script_operator/New(datum/computer/file/mainframe_program/shell/shell)`
- `17:/datum/dwaine_shell_script_operator/disposing()`
- `22:/datum/dwaine_shell_script_operator/proc/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/add.dm`
- `10:/datum/dwaine_shell_script_operator/add/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/divide.dm`
- `10:/datum/dwaine_shell_script_operator/divide/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/modulo.dm`
- `9:/datum/dwaine_shell_script_operator/modulo/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/multiply.dm`
- `10:/datum/dwaine_shell_script_operator/multiply/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/rand.dm`
- `10:/datum/dwaine_shell_script_operator/rand/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/arithmetic/subtract.dm`
- `10:/datum/dwaine_shell_script_operator/subtract/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/exists.dm`
- `11:/datum/dwaine_shell_script_operator/exists/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/is_directory.dm`
- `11:/datum/dwaine_shell_script_operator/is_directory/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/is_executable.dm`
- `11:/datum/dwaine_shell_script_operator/is_executable/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/file/is_file.dm`
- `11:/datum/dwaine_shell_script_operator/is_file/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/and.dm`
- `11:/datum/dwaine_shell_script_operator/and/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/not.dm`
- `10:/datum/dwaine_shell_script_operator/not/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/or.dm`
- `11:/datum/dwaine_shell_script_operator/or/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/logic/xor.dm`
- `11:/datum/dwaine_shell_script_operator/xor/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/assignment.dm`
- `8:/datum/dwaine_shell_script_operator/assignment/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/escape_string.dm`
- `12:/datum/dwaine_shell_script_operator/escape_string/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_del_topmost.dm`
- `11:/datum/dwaine_shell_script_operator/stack_del_topmost/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_depth.dm`
- `11:/datum/dwaine_shell_script_operator/stack_depth/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_dup_topmost.dm`
- `13:/datum/dwaine_shell_script_operator/stack_dup_topmost/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_pop.dm`
- `11:/datum/dwaine_shell_script_operator/stack_pop/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/misc/stack_print.dm`
- `11:/datum/dwaine_shell_script_operator/stack_print/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/eq.dm`
- `10:/datum/dwaine_shell_script_operator/eq/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/ge.dm`
- `12:/datum/dwaine_shell_script_operator/ge/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/gt.dm`
- `12:/datum/dwaine_shell_script_operator/gt/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/le.dm`
- `12:/datum/dwaine_shell_script_operator/le/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/lt.dm`
- `12:/datum/dwaine_shell_script_operator/lt/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/os/shell/shell_script_operators/relational/ne.dm`
- `10:/datum/dwaine_shell_script_operator/ne/execute(list/token_stream)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/_utility.dm`
- `38:/proc/optparse(data)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/cat.dm`
- `4:/datum/computer/file/mainframe_program/utility/cat/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/cd.dm`
- `4:/datum/computer/file/mainframe_program/utility/cd/initialize(initparams)`
- `24:/datum/computer/file/mainframe_program/utility/cd/proc/trim_path(filepath)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/chmod.dm`
- `4:/datum/computer/file/mainframe_program/utility/chmod/initialize(initparams)`
- `37:/datum/computer/file/mainframe_program/utility/chmod/proc/process_permissions(permissions)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/chown.dm`
- `4:/datum/computer/file/mainframe_program/utility/chown/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/cp.dm`
- `4:/datum/computer/file/mainframe_program/utility/cp/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/date.dm`
- `5:/datum/computer/file/mainframe_program/utility/date/initialize(initparams)`
- `56:/datum/computer/file/mainframe_program/utility/date/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `76:/datum/computer/file/mainframe_program/utility/date/proc/usage()`
- `82:/datum/computer/file/mainframe_program/utility/date/proc/message_reply_and_user(message)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/getopt.dm`
- `5:/datum/computer/file/mainframe_program/utility/getopt/initialize(initparams)`
- `44:/datum/computer/file/mainframe_program/utility/getopt/proc/invoke(list/string_list)`
- `137:/datum/computer/file/mainframe_program/utility/getopt/proc/message_reply_and_user(message)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/grep.dm`
- `5:/datum/computer/file/mainframe_program/utility/grep/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/ln.dm`
- `4:/datum/computer/file/mainframe_program/utility/ln/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/ls.dm`
- `4:/datum/computer/file/mainframe_program/utility/ls/initialize(initparams)`
- `48:/datum/computer/file/mainframe_program/utility/ls/proc/print_file_description(datum/computer/C)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/mkdir.dm`
- `4:/datum/computer/file/mainframe_program/utility/mkdir/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/mount.dm`
- `4:/datum/computer/file/mainframe_program/utility/mount/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/mv.dm`
- `4:/datum/computer/file/mainframe_program/utility/mv/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/pwd.dm`
- `4:/datum/computer/file/mainframe_program/utility/pwd/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/rm.dm`
- `12:/datum/computer/file/mainframe_program/utility/rm/initialize(initparams)`
- `61:/datum/computer/file/mainframe_program/utility/rm/input_text(text)`
- `82:/datum/computer/file/mainframe_program/utility/rm/message_user(msg, render, file)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/scnt.dm`
- `4:/datum/computer/file/mainframe_program/utility/scnt/initialize(initparams)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/su.dm`
- `4:/datum/computer/file/mainframe_program/utility/su/initialize(initparams)`
- `11:/datum/computer/file/mainframe_program/utility/su/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `33:/datum/computer/file/mainframe_program/utility/su/input_text(text)`

## `code/modules/networks/computer3/mainframe2/programs/utilities/tar.dm`
- `23:/datum/computer/file/mainframe_program/utility/tar/initialize(initparams)`
- `173:/datum/computer/file/mainframe_program/utility/tar/receive_progsignal(sendid, list/data, datum/computer/file/file)`
- `193:/datum/computer/file/mainframe_program/utility/tar/message_user(msg, render, file)`
- `199:/datum/computer/file/mainframe_program/utility/tar/proc/usage()`
- `205:/datum/computer/file/mainframe_program/utility/tar/proc/message_reply_and_user(message)`
- `213:/datum/computer/file/mainframe_program/utility/tar/proc/temp_file_name()`
- `218:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_list(datum/computer/target, current_path = "", depth = 0)`
- `233:/datum/computer/file/mainframe_program/utility/tar/proc/recursive_extract(datum/computer/to_extract, target_path, current_path = "", depth = 0)`
- `276:/datum/computer/file/mainframe_program/utility/tar/proc/deep_copy(datum/computer/to_copy, current_path = "", depth = 0)`

## `code/modules/networks/computer3/mainframe2/tapes.dm`
- `16:/obj/item/disk/data/memcard/main2/New()`
- `143:/obj/item/disk/data/tape/master/New()`
- `179:/obj/item/disk/data/tape/boot2/New()`
- `224:/obj/item/disk/data/tape/test/New()`
- `236:/obj/item/disk/data/tape/guardbot_tools/New()`
- `255:/obj/item/disk/data/tape/artifact_research/New()`

## `code/modules/networks/computer3/mainframe2/telesci.dm`
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

## `code/modules/networks/computer3/terminal.dm`
- `19:/datum/computer/file/terminal_program/os/terminal_os/input_text(text)`
- `360:/datum/computer/file/terminal_program/os/terminal_os/initialize()`
- `409:/datum/computer/file/terminal_program/os/terminal_os/disk_ejected(var/obj/item/disk/data/thedisk)`
- `444:/datum/computer/file/terminal_program/os/terminal_os/restart()`
- `455:/datum/computer/file/terminal_program/os/terminal_os/process()`
- `469:/datum/computer/file/terminal_program/os/terminal_os/receive_command(obj/source, command, datum/signal/signal)`

