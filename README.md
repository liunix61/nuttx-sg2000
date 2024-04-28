![RISC-V Sophgo SG2000](https://lupyuen.github.io/images/sg2000-soc.jpg)

# Apache NuttX RTOS on 64-bit RISC-V Sophgo SG2000 (T-Head C906 / Milk-V Duo S)

64-bit RISC-V Sophgo SG2000 SoC ... Will it boot Apache NuttX RTOS? 🤔 (T-Head C906 / Milk-V Duo S)

- [SG2000 Overview](https://www.cnx-software.com/2024/02/07/sophgo-sg2000-sg2002-ai-soc-features-risc-v-arm-8051-cores-android-linux-freertos/)

- [SG2000 Reference Manual](https://github.com/sophgo/sophgo-doc/releases)

Let's find out!

_Is this a sponsored review?_

I was given a Milk-V Duo S, and I bought another. So it cancels out, I guess?

# Boot Milk-V Duo S without MicroSD

Connect our USB UART Dongle like so...

https://milkv.io/docs/duo/getting-started/duos

![Milk-V Duo S](https://lupyuen.github.io/images/sg2000-board.jpg)

USB UART Dongle must be CP2102, it doesn't like CH340G 😬

Flip the switch so it's set to "RV" (RISC-V) instead of "Arm"...

![Switch to "RV" (RISC-V) instead of "Arm"](https://lupyuen.github.io/images/sg2000-switch.jpg)

Power up via the USB-C Port. We should see in RISC-V Mode...

https://gist.github.com/lupyuen/a7c3af98be36dcd5cc5b45f5aadc5d16

```bash
C.SCS/0/0.WD.URPL.USBI.USBEF.BS/EMMC.EMI/25000000/12000000. E:bm_emmc_send_cmd_without_data CMD: 1 INT_STAT: 0x E:bm_emmc_send_cmd_without_data CMD: 0 INT_STAT: 0x E:eMMC init failed, 3
 E:eMMC initializing failed
PS. E:bm_emmc_send_cmd_without_data CMD: 6 INT_STAT: 0x E:load param1 (-5)
 E:Boot failed (8).
 E:RESET:plat/mars/platform.c:114
WD.C.SCS/0/0.WD.URPL.USBI.USBEF.BS/EMMC.EMI/25000000/12000000. E:bm_emmc_send_cmd_without_data CMD: 1 INT_STAT: 0x E:bm_emmc_send_cmd_without_data CMD: 0 INT_STAT: 0x E:eMMC init failed, 3
 E:eMMC initializing failed
```

If we see `B.SCS` instead of `C.SCS`...

https://gist.github.com/lupyuen/d55b77a51ee8b258d6d1c0799770742a

```bash
B.SCS/0/0.WD.URPL.USBI.USBEF.BS/EMMC.EMI/25000000/12000000.
```

Nope we're in Arm Mode! Flip the switch back to RISC-V!

If we use CH340G: UART Output will be garbled (happens on Linux and macOS, maybe Windows too)...

https://gist.github.com/lupyuen/1d5ba1b2a47c110ee7ff265102b1aae5

Milk-V Duo S doesn't ship with U-Boot Bootloader preinstalled. Too bad we can't boot NuttX over TFTP, our NuttX Porting will be a bit slower. Bummer :-(

TODO: Get the eMMC Version with U-Boot preinstalled. But flashing the eMMC only works on Windows. Sigh

# Boot Linux on Milk-V Duo S

Let's boot Linux on MicroSD...

https://github.com/Fishwaldo/sophgo-sg200x-debian/releases

We pick the Latest Release for Milk-V Duo S...

https://github.com/Fishwaldo/sophgo-sg200x-debian/releases/download/v1.1.0/duos_sd.img.lz4

```bash
$ brew install lz4
$ lz4 ~/Downloads/duos_sd.img.lz4
## TODO: Write duos_sd.img to MicroSD with Balena Etcher

→ ls -l /Volumes/boot
total 17488
-rwxrwxrwx  1 Luppy  staff  3494900 Apr 24 11:33 System.map-5.10.4-20240329-1+
-rwxrwxrwx  1 Luppy  staff   125534 Apr 24 11:33 config-5.10.4-20240329-1+
drwxrwxrwx  1 Luppy  staff     2048 Apr 24 11:33 extlinux
drwxrwxrwx  1 Luppy  staff     2048 Apr 24 11:33 fdt
-rwxrwxrwx  1 Luppy  staff   388608 Apr 24 11:33 fip.bin
-rwxrwxrwx  1 Luppy  staff  4937389 Apr 24 11:33 vmlinuz-5.10.4-20240329-1+

→ ls -l /Volumes/boot/extlinux
total 4
-rwxrwxrwx  1 Luppy  staff  749 Apr 24 11:33 extlinux.conf

→ ls -l /Volumes/boot/fdt
total 4
drwxrwxrwx  1 Luppy  staff  2048 Apr 24 11:33 linux-image-duos-5.10.4-20240329-1+

→ ls -l /Volumes/boot/fdt/linux-image-duos-5.10.4-20240329-1+
total 44
-rwxrwxrwx  1 Luppy  staff  21575 Apr 24 11:33 cv181x_milkv_duos_sd.dtb
```

Here's the Boot Config...

```bash
→ cat /Volumes/boot/extlinux/extlinux.conf
## /boot/extlinux/extlinux.conf
##
## IMPORTANT WARNING
##
## The configuration of this file is generated automatically.
## Do not edit this file manually, use: u-boot-update

default l0
menu title U-Boot menu
prompt 1
timeout 50


label l0
	menu label Debian GNU/Linux trixie/sid 5.10.4-20240329-1+
	linux /vmlinuz-5.10.4-20240329-1+

	fdtdir /fdt/linux-image-duos-5.10.4-20240329-1+/

	append root=/dev/root console=ttyS0,115200 earlycon=sbi root=/dev/mmcblk0p2 rootwait rw

label l0r
	menu label Debian GNU/Linux trixie/sid 5.10.4-20240329-1+ (rescue target)
	linux /vmlinuz-5.10.4-20240329-1+

	fdtdir /fdt/linux-image-duos-5.10.4-20240329-1+/
	append root=/dev/root console=ttyS0,115200 earlycon=sbi root=/dev/mmcblk0p2 rootwait rw single
```

Yep Linux boots OK on Milk-V Duo S! (Together with OpenSBI and U-Boot Bootloader)

https://gist.github.com/lupyuen/01d409b7bde9607a96cd4d460e53330a

```bash
OpenSBI v0.9
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name             : Milk-V DuoS
Platform Features         : mfdeleg
Platform HART Count       : 1
Platform IPI Device       : clint
Platform Timer Device     : clint
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform SysReset Device  : ---
Firmware Base             : 0x80000000
Firmware Size             : 132 KB
Runtime SBI Version       : 0.3

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000074000000-0x000000007400ffff (I)
Domain0 Region01          : 0x0000000080000000-0x000000008003ffff ()
Domain0 Region02          : 0x0000000000000000-0xffffffffffffffff (R,W,X)
Domain0 Next Address      : 0x0000000080200020
Domain0 Next Arg1         : 0x0000000080080000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART ISA             : rv64imafdcvsux
Boot HART Features        : scounteren,mcounteren,time
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4096
Boot HART PMP Address Bits: 38
Boot HART MHPM Count      : 8
Boot HART MHPM Count      : 8
Boot HART MIDELEG         : 0x0000000000000222
Boot HART MEDELEG         : 0x000000000000b109
...
Debian GNU/Linux trixie/sid duos ttyS0
duos login: 
```

Let's dump the U-Boot Config...

https://gist.github.com/lupyuen/000b55a46336cddf217a589f469d60e2

```bash
U-Boot 2021.10-ga57aa1f2-dirty (Apr 24 2024 - 11:24:46 +0000) cvitek_cv181x

DRAM:  510 MiB
gd->relocaddr=0x9fbc7000. offset=0x1f9c7000
set_rtc_register_for_power
MMC:   cv-sd@4310000: 0, wifi-sd@4320000: 1
Loading Environment from nowhere... OK
In:    serial
Out:   serial
Err:   serial
Net:   
Warning: ethernet@4070000 (eth0) using random MAC address - 82:c2:1a:b2:ee:b5
eth0: ethernet@4070000
Hit any key to stop autoboot:  0

cv181x_c906# printenv

arch=riscv
baudrate=115200
board=mars
board_name=mars
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
boot_efi_binary=load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootriscv64.efi; if fdt addr ${fdt_addr_r}; then bootefi ${kernel_addr_r} ${fdt_addr_r};else bootefi ${kernel_addr_r} ${fdtcontroladdr};fi
boot_efi_bootmgr=if fdt addr ${fdt_addr_r}; then bootefi bootmgr ${fdt_addr_r};else bootefi bootmgr;fi
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}${boot_syslinux_conf}
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_syslinux_conf=extlinux/extlinux.conf
boot_targets=mmc0 dhcp pxe 
bootcmd=run distro_bootcmd || run sdboot || run sdbootauto
bootcmd_dhcp=devtype=dhcp; if dhcp ${scriptaddr} ${boot_script_dhcp}; then source ${scriptaddr}; fi;setenv efi_fdtfile ${fdtfile}; setenv efi_old_vci ${bootp_vci};setenv efi_old_arch ${bootp_arch};setenv bootp_vci PXEClient:Arch:00027:UNDI:003000;setenv bootp_arch 0x1b;if dhcp ${kernel_addr_r}; then tftpboot ${fdt_addr_r} dtb/${efi_fdtfile};if fdt addr ${fdt_addr_r}; then bootefi ${kernel_addr_r} ${fdt_addr_r}; else bootefi ${kernel_addr_r} ${fdtcontroladdr};fi;fi;setenv bootp_vci ${efi_old_vci};setenv bootp_arch ${efi_old_arch};setenv efi_fdtfile;setenv efi_old_arch;setenv efi_old_vci;
bootcmd_mmc0=devnum=0; run mmc_boot
bootcmd_pxe=dhcp; if pxe get; then pxe boot; fi
bootdelay=1
consoledev=ttyS0
cpu=generic
distro_bootcmd=for target in ${boot_targets}; do run bootcmd_${target}; done
efi_dtb_prefixes=/ /dtb/ /dtb/current/
fdt_addr_r=0x81200000
fdtcontroladdr=9f280810
fdtfile=cv181x_milkv_duos_sd.dtb
fdtoverlay_addr_r=0x81300000
gatewayip=192.168.0.11
ipaddr=192.168.0.3
kernel_addr_r=0x80200000
kernel_comp_addr_r=0x81800000
kernel_comp_size=0x1000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
mmc_boot=if mmc dev ${devnum}; then devtype=mmc; run scan_dev_for_boot_part; fi
netdev=eth0
netmask=255.255.255.0
othbootargs=earlycon=sbi riscv.fwsz=0x80000   loglevel=9
pxefile_addr_r=0x81400000
ramdisk_addr_r=0x81600000
root=root=/dev/mmcblk0p2 rootwait rw
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_prefixes}; do run scan_dev_for_extlinux; run scan_dev_for_scripts; done;run scan_dev_for_efi;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || setenv devplist 1; for distro_bootpart in ${devplist}; do if fstype ${devtype} ${devnum}:${distro_bootpart} bootfstype; then run scan_dev_for_boot; fi; done; setenv devplist
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; for prefix in ${efi_dtb_prefixes}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${efi_fdtfile}; then run load_efi_dtb; fi;done;run boot_efi_bootmgr;if test -e ${devtype} ${devnum}:${distro_bootpart} efi/boot/bootriscv64.efi; then echo Found EFI removable media binary efi/boot/bootriscv64.efi; run boot_efi_binary; echo EFI LOAD FAILED: continuing...; fi; setenv efi_fdtfile
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${boot_syslinux_conf}; then echo Found ${prefix}${boot_syslinux_conf}; run boot_extlinux; echo SCRIPT FAILED: continuing...; fi
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${script}; then echo Found U-Boot script ${prefix}${script}; run boot_a_script; echo SCRIPT FAILED: continuing...; fi; done
scriptaddr=0x81500000
sdboot=setenv bootargs ${reserved_mem} ${root} ${mtdparts} console=$consoledev,$baudrate $othbootargs;echo Boot from SD dev ${sddev} ...;mmc dev ${sddev} && fatload mmc ${sddev} ${uImage_addr} boot.sd;if test $? -eq 0; then bootm ${uImage_addr}#config-cv181x_milkv_duos_sd;fi;
sdbootauto=cvi_sd_boot;setenv bootargs ${reserved_mem} ${root} ${mtdparts} console=$consoledev,$baudrate $othbootargs;echo Boot from SD dev ${sddev} auto ...;mmc dev ${sddev} && fatload mmc ${sddev} ${uImage_addr} boot.sd;if test $? -eq 0; then bootm ${uImage_addr}#config-cv181x_milkv_duos_sd;fi;
sddev=0
serverip=192.168.56.101
stderr=serial
stdin=serial
stdout=serial
uImage_addr=0x81800000
update_addr=0x9fe00000
vendor=cvitek

Environment size: 4333/131068 bytes
```

Aha Ethernet Driver is available in U-Boot. Which means we can boot NuttX over TFTP yay!

```bash
cv181x_c906# net list

eth0 : ethernet@4070000 00:00:00:00:00:00 active
```

(See below for the U-Boot Commands)

Here's another Linux Image: https://github.com/logicethos/Milk-V_Duo_Linux2SD

# Boot Milk-V Duo S over TFTP 

We'll port NuttX by booting over TFTP, but let's test TFTP first. Here are the steps to boot Milk-V Duo S over TFTP...

https://lupyuen.github.io/articles/tftp#configure-u-boot-for-tftp

```bash
## Set U-Boot TFTP Server
setenv tftp_server 192.168.31.10
printenv tftp_server

## Set RAM Disk Size (assume the max)
setenv ramdisk_size 0x1000000
printenv ramdisk_size

## TODO: saveenv doesn't exist, can't save Environment Variables!
## TODO: Update the Boot Config in MicroSD instead
## saveenv

## Fetch the IP Address over DHCP
## Load the NuttX Image from TFTP Server
## kernel_addr_r=0x80200000
## tftp_server=192.168.x.x
## TODO: Fix the NuttX Image
dhcp ${kernel_addr_r} ${tftp_server}:Image-sg2000

## Load the Device Tree from TFTP Server
## fdt_addr_r=0x81200000
## TODO: Fix the Device Tree
tftpboot ${fdt_addr_r} ${tftp_server}:jh7110-star64-pine64.dtb

## Set the RAM Address of Device Tree
## fdt_addr_r=0x81200000
fdt addr ${fdt_addr_r}

## Load the Intial RAM Disk from TFTP Server
## ramdisk_addr_r=0x81600000
## TODO: Fix the Initial RAM Disk
tftpboot ${ramdisk_addr_r} ${tftp_server}:initrd

## Boot the NuttX Image with the Initial RAM Disk and Device Tree
## kernel_addr_r=0x80200000
## ramdisk_addr_r=0x81600000
## ramdisk_size=0x1000000
## fdt_addr_r=0x81200000
booti ${kernel_addr_r} ${ramdisk_addr_r}:${ramdisk_size} ${fdt_addr_r}
```

_What happens when we run this?_

```bash
cv181x_c906# setenv tftp_server 192.168.31.10
cv181x_c906# printenv tftp_server
tftp_server=192.168.31.10

cv181x_c906# setenv ramdisk_size 0x1000000
cv181x_c906# printenv ramdisk_size
ramdisk_size=0x1000000

cv181x_c906# dhcp ${kernel_addr_r} ${tftp_server}:Image
Speed: 100, full duplex
BOOTP broadcast 1
BOOTP broadcast 2
*** Unhandled DHCP Option in OFFER/ACK: 43
*** Unhandled DHCP Option in OFFER/ACK: 43
DHCP client bound to address 192.168.31.47 (550 ms)
Using ethernet@4070000 device
TFTP from server 192.168.31.10; our IP address is 192.168.31.47
Filename 'Image'.
Load address: 0x80200000
Loading: #################################################################
. #################################################################
. #############
. 1.2 MiB/s
done
Bytes transferred = 2097800 (200288 hex)

cv181x_c906# tftpboot ${fdt_addr_r} ${tftp_server}:jh7110-star64-pine64.dtb
Speed: 100, full duplex
Using ethernet@4070000 device
TFTP from server 192.168.31.10; our IP address is 192.168.31.47
Filename 'jh7110-star64-pine64.dtb'.
Load address: 0x81200000
Loading: ####
. 1.1 MiB/s
done
Bytes transferred = 50235 (c43b hex)

cv181x_c906# fdt addr ${fdt_addr_r}

cv181x_c906# tftpboot ${ramdisk_addr_r} ${tftp_server}:initrd
Speed: 100, full duplex
Using ethernet@4070000 device
TFTP from server 192.168.31.10; our IP address is 192.168.31.47
Filename 'initrd'.
Load address: 0x81600000
Loading: #################################################################
. #################################################################
. #################################################################
. #################################################################
. #################################################################
. #################################################################
. #################################################################
. #################################################################
. ##################################################
. 1.2 MiB/s
done
Bytes transferred = 8354816 (7f7c00 hex)

cv181x_c906# booti ${kernel_addr_r} ${ramdisk_addr_r}:${ramdisk_size} ${fdt_addr_r}
## Flattened Device Tree blob at 81200000
   Booting using the fdt blob at 0x81200000
   Loading Ramdisk to 9e27f000, end 9f27f000 ... OK
   Loading Device Tree to 000000009e26f000, end 000000009e27e43a ... OK

Starting kernel ...
```

NuttX Kernel hangs, but that's OK! We haven't modified NuttX Kernel for SG2000 yet. Let's fix this...

# Boot Apache NuttX RTOS over TFTP

TODO

# UART Controller for SG2000

According to the [SG2000 Reference Manual](https://github.com/sophgo/sophgo-doc/releases), the UART Controller Base Addresses are...

| GPIO Module | Base Address |
|-------------|--------------|
| UART0 | 0x04140000 |
| UART1 | 0x04150000 |
| UART2 | 0x04160000 |
| UART3 | 0x04170000 |
| UART4 | 0x041C0000 |
| RTCSYS_UART | 0x05022000 |

We'll print to UART0 in NuttX.

TODO: What UART Controller is inside Milk-V Duo S? Modify the NuttX Boot Code to write UART Output Register in RISC-V Assembly

# NuttX prints to SG2000 Serial Console

TODO

https://lupyuen.github.io/articles/nuttx2#print-to-qemu-console

# U-Boot Commands for Milk-V Duo S

Here are the U-Boot Commands...

```bash
cv181x_c906# help

?         - alias for 'help'
base      - print or set address offset
bdinfo    - print Board Info structure
blkcache  - block cache diagnostics and control
boot      - boot default, i.e., run 'bootcmd'
bootd     - boot default, i.e., run 'bootcmd'
bootefi   - Boots an EFI payload from memory
bootelf   - Boot from an ELF image in memory
booti     - boot Linux kernel 'Image' format from memory
bootm     - boot application image from memory
bootp     - boot image via network using BOOTP/TFTP protocol
bootvx    - Boot vxWorks from an ELF image
cmp       - memory compare
cp        - memory copy
cpu       - display information about CPUs
cvi_sd_boot- boot from SD card
cvi_update- cvi_update [eth, sd, usb]- check boot status and update if necessary

cvi_utask - bootloader control block command
dcache    - enable or disable data cache
dhcp      - boot image via network using DHCP/TFTP protocol
echo      - echo args to console
efuser    - Read efuse
efuser_dump- Read/Dump efuse
efusew    - Write efuse
efusew_word- Write word to efuse
env       - environment handling commands
erase     - erase FLASH memory
exit      - exit script
ext2load  - load binary file from a Ext2 filesystem
ext2ls    - list files in a directory (default /)
ext4load  - load binary file from a Ext4 filesystem
ext4ls    - list files in a directory (default /)
ext4size  - determine a file's size
false     - do nothing, unsuccessfully
fatinfo   - print information about filesystem
fatload   - load binary file from a dos filesystem
fatls     - list files in a directory (default /)
fatmkdir  - create a directory
fatrm     - delete a file
fatsize   - determine a file's size
fatwrite  - write file into a dos filesystem
fdt       - flattened device tree utility commands
flinfo    - print FLASH memory information
fstype    - Look up a filesystem type
fstypes   - List supported filesystem types
go        - start application at address 'addr'
help      - print command description/usage
icache    - enable or disable instruction cache
iminfo    - print header information for application image
ln        - Create a symbolic link
load      - load binary file from a filesystem
loadb     - load binary file over serial line (kermit mode)
loadx     - load binary file over serial line (xmodem mode)
loady     - load binary file over serial line (ymodem mode)
loop      - infinite loop on address range
ls        - list files in a directory (default /)
md        - memory display
mdio      - MDIO utility commands
mii       - MII utility commands
mm        - memory modify (auto-incrementing address)
mmc       - MMC sub system
mmcinfo   - display MMC info
mw        - memory write (fill)
net       - NET sub-system
nfs       - boot image via network using NFS protocol
nm        - memory modify (constant address)
panic     - Panic with optional message
part      - disk partition related commands
ping      - send ICMP ECHO_REQUEST to network host
printenv  - print environment variables
protect   - enable or disable FLASH write protection
pxe       - commands to get and boot from pxe
reset     - Perform RESET of the CPU
run       - run commands in an environment variable
save      - save file to a filesystem
setenv    - set environment variables
setexpr   - set environment variable as the result of eval expression
showvar   - print local hushshell variables
size      - determine a file's size
sleep     - delay execution for some time
source    - run script from memory
sysboot   - command to get and boot from syslinux files
test      - minimal test like /bin/sh
tftpboot  - boot image via network using TFTP protocol
true      - do nothing, successfully
version   - print monitor, compiler and linker version
```
