---
layout: post
title: Pork Explosion Unleashed
---

While I did "mock hype" this vulnerability, I was mostly making fun of those companies using bland and boring vulnerability disclosures as a PR stunt, Pork Explosion is certainly real and today we feast.

Going into this I want to thank Mike Chan and the others at Nextbit for their prompt action to mitigate this in the Nextbit Robin. I also appreciate the QPSI & Android Security teams for their willingness to assist in contacting Foxconn.

and to the meat (or Tofu for you BBQ haters)...

Pork Explosion is a backdoor found in the apps bootloader provided by Foxconn. For those that are not aware, Foxconn assembles phones for many many vendors, some (but not all) also choose to allow Foxconn to build many low level pieces of firmware. To date we have identified at least two vendors (likely many more) with vulnerable devices, InFocus (M810) and Nextbit (Robin). Pork Explosion allows an attack with physical access to a device to gain a root shell, with selinux disabled through usb. The attack can be made through fastboot and the apps bootloader, or through adb if access is available. Due to the ability to get a root shell on a password protected or encrypted device, Pork Explosion would be of value for forensic data extraction, brute forcing encryption keys, or unlocking the boot loader of a device without resetting user data. Phone vendors were unaware this backdoor has been placed into their products.

While taking a peek at the Nexbit Robin’s apps bootloader, (based on Qualcomm’s lk bootloader, with customizations made by Foxconn International Holdings), a fastboot command was noticed that seemed out of place. The Nextbit Robin’s apps boot loader is based on the lk bootloader with customizations made by Foxconn International Holdings.

~~~ asm
LOAD:0F92F8DC fastboot_table                          ; CODE XREF: sub_F939888+174p
LOAD:0F92F8DC
LOAD:0F92F8DC var_C           = -0xC
LOAD:0F92F8DC
LOAD:0F92F8DC                 STMFD           SP!, {R4,LR}
LOAD:0F92F8E0                 MOV             R4, #0xF9B9C14
LOAD:0F92F8E8                 SUB             SP, SP, #8
LOAD:0F92F8EC                 MOV             R0, #0x2AE4
LOAD:0F92F8F0                 MOV             R1, #0xF140
LOAD:0F92F8F4                 LDR             R3, [R4]
LOAD:0F92F8F8                 MOVT            R0, #0xF97
LOAD:0F92F8FC                 MOVT            R1, #0xF92
LOAD:0F92F900                 STR             R3, [SP,#0x10+var_C]
LOAD:0F92F904                 BL              fastboot_register
LOAD:0F92F908                 MOV             R0, #0x2AEC
LOAD:0F92F90C                 MOV             R1, #0xDCB4
LOAD:0F92F910                 MOVT            R0, #0xF97
LOAD:0F92F914                 MOVT            R1, #0xF92
LOAD:0F92F918                 BL              fastboot_register
LOAD:0F92F91C                 MOV             R0, #0x2B00
LOAD:0F92F920                 MOV             R1, #0xEB58
LOAD:0F92F924                 MOVT            R0, #0xF97
LOAD:0F92F928                 MOVT            R1, #0xF92
LOAD:0F92F92C                 BL              fastboot_register
LOAD:0F92F930                 MOV             R0, #0x2B10
LOAD:0F92F934                 MOV             R1, #0xE3A0
LOAD:0F92F938                 MOVT            R0, #0xF97
LOAD:0F92F93C                 MOVT            R1, #0xF92
LOAD:0F92F940                 BL              fastboot_register
LOAD:0F92F944                 MOV             R0, #0x2B24
LOAD:0F92F948                 MOV             R1, #0xE598
LOAD:0F92F94C                 MOVT            R0, #0xF97
LOAD:0F92F950                 MOVT            R1, #0xF92
LOAD:0F92F954                 BL              fastboot_register
LOAD:0F92F958                 MOV             R0, #0x2B34
LOAD:0F92F95C                 MOV             R1, #0xE6C4
LOAD:0F92F960                 MOVT            R0, #0xF97
LOAD:0F92F964                 MOVT            R1, #0xF92
LOAD:0F92F968                 BL              fastboot_register
LOAD:0F92F96C                 MOV             R0, #0x2B44
LOAD:0F92F970                 MOV             R1, #0xE7C8
LOAD:0F92F974                 MOVT            R0, #0xF97
LOAD:0F92F978                 MOVT            R1, #0xF92
LOAD:0F92F97C                 BL              fastboot_register
LOAD:0F92F980                 MOV             R0, #0x2B50
LOAD:0F92F984                 MOV             R1, #0xE954
LOAD:0F92F988                 MOVT            R0, #0xF97
LOAD:0F92F98C                 MOVT            R1, #0xF92
LOAD:0F92F990                 BL              fastboot_register
LOAD:0F92F994                 MOV             R0, #0x2B60 ; reboot-recovery
LOAD:0F92F998                 MOV             R1, #0xE500
LOAD:0F92F99C                 MOVT            R0, #0xF97
LOAD:0F92F9A0                 MOVT            R1, #0xF92
LOAD:0F92F9A4                 BL              fastboot_register
LOAD:0F92F9A8                 MOV             R0, #0x2B70 ; reboot-ftm
LOAD:0F92F9AC                 MOV             R1, #0xE54C
LOAD:0F92F9B0                 MOVT            R0, #0xF97
LOAD:0F92F9B4                 MOVT            R1, #0xF92
LOAD:0F92F9B8                 BL              fastboot_register
LOAD:0F92F9BC                 MOV             R0, #0x2B7C
LOAD:0F92F9C0                 MOV             R1, #0xEE1C
LOAD:0F92F9C4                 MOVT            R0, #0xF97
LOAD:0F92F9C8                 MOVT            R1, #0xF92
LOAD:0F92F9CC                 BL              fastboot_register
LOAD:0F92F9D0                 MOV             R0, #0x2B88
LOAD:0F92F9D4                 MOV             R1, #0xF804
LOAD:0F92F9D8                 MOVT            R0, #0xF97
LOAD:0F92F9DC                 MOVT            R1, #0xF92
LOAD:0F92F9E0                 BL              fastboot_register
LOAD:0F92F9E4                 MOV             R0, #0x2B98
LOAD:0F92F9E8                 MOV             R1, #0xF518
LOAD:0F92F9EC                 MOVT            R0, #0xF97
LOAD:0F92F9F0                 MOVT            R1, #0xF92
LOAD:0F92F9F4                 BL              fastboot_register
LOAD:0F92F9F8                 MOV             R0, #0x2BA0
LOAD:0F92F9FC                 MOV             R1, #0xEFE0
LOAD:0F92FA00                 MOVT            R0, #0xF97
LOAD:0F92FA04                 MOVT            R1, #0xF92
LOAD:0F92FA08                 BL              fastboot_register
LOAD:0F92FA0C                 LDR             R2, [SP,#0x10+var_C]
LOAD:0F92FA10                 LDR             R3, [R4]
LOAD:0F92FA14                 CMP             R2, R3
LOAD:0F92FA18                 BNE             loc_F92FA38
LOAD:0F92FA1C                 MOV             R0, #0x2BA8
LOAD:0F92FA20                 MOV             R1, #0xF0D0
LOAD:0F92FA24                 MOVT            R0, #0xF97
LOAD:0F92FA28                 MOVT            R1, #0xF92
LOAD:0F92FA2C                 ADD             SP, SP, #8
LOAD:0F92FA30                 LDMFD           SP!, {R4,LR}
LOAD:0F92FA34                 B               fastboot_register
~~~

Two types of fastboot commands exist, normal ones and OEM ones. Normal commands result in the command being sent over usb in a “command:parameters” format. Such as “getvar all” being sent as “getvar:all” or “reboot” simply sent as “reboot”.  These normal commands are hard coded into the fastboot client. OEM commands are where non standard commands belonging to the device manufacturer are implemented. The OEM commands are sent as “oem command parameters”, such as “oem writeimei 1234567899876543”.

The command in question is “reboot-ftm”, but it was not an “OEM” command nor normal command. It could not be accessed by prefacing it with “oem” and it couldn’t be sent with the fastboot client, as normal commands are hard coded. This command was supported by the mobile device, but a means of accessing it was not implemented in fastboot. In order to interface with this command, a custom client had to be created. The custom client connects to the device, and sends the bytes 7265626F6F742D66746D (reboot-ftm) to the bootloader.

~~~
Jons-Mac-Pro:nextbit jcase$ adb shell
root@ether:/ # id
uid=0(root) gid=0(root) groups=0(root),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats)
root@ether:/ # getenforce                                                      
Disabled
root@ether:/ # grep finger /system/build.prop                                  
# Do not try to parse description, fingerprint, or thumbprint
ro.build.fingerprint=Nextbit/ether/ether:6.0.1/MMB29M/00WW_1_350:user/release-keys
~~~

After issuing this command the phone would boot into a factory test mode, this mode would show various details about the software/hardware, including the emmc brand, mac address, and serial number. If you had access to adb, this mode could also be reached with “adb reboot ftm”. Booting to FTM boots the device with a kernel/ramdisk from the ftmboot partition, instead of boot or recovery partitions.

While in factory test mode adbd is running as root, and it requires no authentication to access a shell through adb. SELinux is in a disabled state, and it is not permissive but fully disabled.

In short, this is a full compromise over usb, which requires no logon access to the device. This vulnerability completely bypasses authentication and authorization controls on the device. It is a prime target for forensic data extraction. While it is obviously a debugging feature, it is a backdoor, it isn't something we should see in modern devices, and it is a sign of great neglect on Foxconn's part.

For those looking to detect vulnerable devices, you can check for the partitions "ftmboot" and "ftmdata". The "ftmboot" partition contacts a traditional Android kernel/ramdisk image. This one has SELinux disabled, and adb running as root. The "ftmdata" partition is mounted on /data during ftm bootmode. These partitions are only a sign that the device is vulnerable. Nextbit has mitigated this vulnerability by zeroing out these two partitions.

Author: Jon "jcase" Sawyer

Special thanks:

- beaups
- ben_ra
- QPSI
- Android Security
- Nextbit
- All those that know BBQ is pork, and "Texas BBQ" is fake BBQ.

Timeline

- 08-31-2016   Initial Discovery
- 08-31-2016   Notified Nextbit through Mike Chan (CTO of Nextbit)
- 08-31-2016   Attempt to notify Foxconn through Android Security Team
- 08-31-2016   Attempt to notify Foxconn through Qualcomm Product Security Initiative
- 08-31-2016   Nextbit through Mike Chan (CTO of Nextbit)
- 09-07-2016   Nextbit states they plan to zero out the partitions
- 10-11-2016   Nextbit releases fix
- 10-12-2016   Disclosure 


Bonus: Check out Foxconn's dump command for more fun!
