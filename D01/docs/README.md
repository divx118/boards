#Hisilicon Opensource Board D01 hacking manual

             .--,       .--,
            ( (  \.---./  ) )
             '.__/o   o\__.'
                {=  ^  =}
                 >  -  <
         ____.""`-------`"".___
        /                      \
        \                      /
        / Joy Xu               \
        \ xuwei5@hisilicon.com /
        /  V2.0                \
        \  2014-12-18          /
        /                      \
        \______________________/
              ___)( )(___
             (((__) (__)))

##Overview

* [D01 Board Hardware Featuers](#hardware)
* [D01 Software Solution and Status](#solution)
    * [UEFI](#uefi)
    * [BootWrapper](#bootwrapper)
    * [GRUB](#grub)
    * [EFI Stub](#stub)
    * [Kernel](#kernel)
    * [Distributions](#osv)
    * [Toolchain](#toolchain)
    * [Release](#release)
* [D01 Hacking](#hacking)
    * [UEFI hacking](#uefihacking)
        * [compile UEFI source code](#compileuefi)
        * [boot D01 to UEFI shell](#uefishell)
        * [upgrade UEFI](#upgradeuefi)
        * [how to restore the UEFI when the UEFI did not work](#restoreuefi)
    * [Bootwrapper hacking](#bootwrapperhacking)
        * [compile BootWrapper source code](#compilebootwrapper)
        * [upgrade BootWrapper](#upgradebootwrapper)
    * [Grub hacking](#grubhacking)
    * [Kernel hacking](#kernelhacking)
    * [boot via NAND](#nandboot)
        * [upgrade kernel and NAND rootfs](#upgradekernel)
        * [show & change kernel command line in UEFI](#changecommandline)
        * [boot into NAND Rootfs](#nandfsboot)
        * [boot into NFS](#nfsboot)
        * [boot into Ubuntu on SATA](#ubuntuboot)
    * [boot via PXE](#pxeboot)
        * [set up TFTP server](#installtftp)
        * [set up DHCP server](#installtftp)
        * [enter PXE in the UEFI shell](#enterpxe)
    * [boot via EFI-stub](#stubboot)
        * [compile kernel with EFI stub](#compilestubkernel)
        * [install EFI-stub kernel on SATA disk](#installefi)
        * [launch EFI-stub Kernel in the UEFI shell](#enterefi)
    * [boot via GRUB](#grubboot)
        * [install GRUB on SATA disk](#installgrub)
        * [launch GRUB in the UEFI shell](#entergrub)
    * [KVM on D01](#kvm)
    * [Xen on D01](#xen)
* [Reference](#refer)


<a name="hardware"/>
##D01 Board Hardware Featuers

Major hardware features of D01 are listed here.  
         
        +-----------+----------------------------------------------------------+
        |Features   | Description                                              |
        +-----------+----------------------------------------------------------|
        |Processors | Integrated 16 x ARM Cortex-A15 CPU Core @ max. 1.5GHz    |
        |           | Support for up to 84000 DMIPS                            |
        |           | Support for CPU configuration as AMP/SMP                 |
        |           | Configurable Big or Littile endian,default Little endian |
        |           | Support ARMv7-A instruction set                          |
        |           | Support float VFPv4 instruction set                      |
        +-----------+----------------------------------------------------------+
        |Memory     | Two 64bit DDR3 DRAM Dual Inline                          |
        |           | Memory Module(DIMM) sockets:(2)&(3)                      |
        |           | Maximum frequency of 1600 MHz                            |
        |           | Maximum capacity of 64GB                                 |
        |           | preassembled capacity:16GB                               |
        |           | Built-in two 1Gb NOR Flash;                              |
        |           | only one NOR flash could use by software:(29)            |
        |           | Built-in two 512MB NAND Flash:(30)                       |
        +-----------+----------------------------------------------------------+
        |GPU        | None                                                     |
        +-----------+----------------------------------------------------------+
        |Peripheral | TWO USB2.0 Host port:(11)                                |
        |Interfaces | Two UART interfaces:(12)&(13)                            |
        |           | Four I2C interfaces                                      |
        |           | One SPI interface, supporting four CSs                   |
        |           | One SD card interface:(10)                               |
        |           | GPIO: eight LED interfaces:(26);eight switchs:(25)       |
        |           | Three SATA interfaces(2.5in SATA 3.0 6Gbps):(4)&(5)&(6)  |
        |           | Tracer Connector x2:(19)&(20)                            |
        |           | One JTAG interfaces(5x2 pin, ARM Connector):(24)         |
        +-----------+----------------------------------------------------------+
        |BIOS       | BIOS resident in NOR Flash                               |
        |           | Support for update with FTP                              |
        +-----------+----------------------------------------------------------+
        |LAN        | Two 10/100/1000Mbit/s Gigabit Ethernet ports:(15)&(16)   |
        |           | One 10/100Mbit/s FE port:(14)                            |
        +-----------+----------------------------------------------------------+
        |Hardware   | Power consumption sense                                  |
        |Monitor    |                                                          |
        +-----------+----------------------------------------------------------+
        |Input      | None                                                     |
        |Devices    |                                                          |
        +-----------+----------------------------------------------------------+
        |Other      | One hardware button for power-off:(8)                    |
        |           | One hardware button for reset:(9)                        |
        +-----------+----------------------------------------------------------+

<a name="pic"/>
Refer to the following picture to know  
where each component below is physically located.    

<img src="./pic/d01-portrait.png" style="width:700px;height:500px;margin-left:30px"/> 

<a name="solution"/> 
##D01 Software Solution and Status 

The software architecture on D01 is consisted of [UEFI](#uefi), [BootWrapper](#bootwrapper)  
, [GRUB](#grub), [EFI Stub](#stub), [Kernel](#kernel), [Distributions](#osv) and   
[Toolchain](#toolchain). The big picture is like following:


          +--------------------------+ +-----+
          |  NANDRootfs Ubuntu       | |  T  |
          |  OpenSuse Debian Fedora  | |  o  |
          +--------------------------+ |  o  |
          |         Kernel           | |  l  |
          +-----------+              | |  C  |
          |    GRUB   |   EFI-STUB   | |  h  |
          +----+------+--------+-----+ |  a  |
          |    |  BootWrapper  |     | |  i  |
          |    +---------------+     | |  n  |
          |           UEFI           | |     |
          | +-------+-------+-----+  | |     |
          | | NANDC | SATAC | PXE |  | |     |
          | +-------+-------+-----+  | |     |
          +--------------------------+ +-----+

<a name="release"/>
And you could download the binary from the link:

        https://github.com/hisilicon/boards/tree/master/D01/release

        +------------------+------------------------------------+       
        |  filename        |    description                     |
        +------------------+------------------------------------+       
        |  D01.fd          |    UEFI binary                     |
        +------------------+------------------------------------+       
        |  .text           |	bootwrapper, HYP switch part    |   
        +------------------+------------------------------------+       
        |  .monitor	       |	bootwrapper, monitor part       |
        +------------------+------------------------------------+       
        |  grub2.efi       |    grub binary                     |
        +------------------+------------------------------------+       
        |  grub.cfg        |    grub configure                  |       
        +------------------+------------------------------------+       
        |  .kernel         |	zImage, with dtb concatenatea   |   
        +------------------+------------------------------------+       
        |  zImage          |    kernel image                    |
        +------------------+------------------------------------+       
        |  hip04-d01.dtb   |    D01 device tree bianry          |   
        +------------------+------------------------------------+       
        |  .filesystem	   |	initramfs stored in NAND        |   
        +------------------+------------------------------------+       

Linaro also provide a monthly release for it:

         http://www.linaro.org/downloads/ 

###The function of each component

<a name="uefi"/>
* [UEFI](http://uefi.org/about): responsible for loading and booting kernel  

    could get the source code from following git tree

        https://github.com/hisilicon/UEFI.git
        https://git.linaro.org/landing-teams/working/hisilicon/uefi.git
        https://git.linaro.org/uefi/linaro-edk2.git

    And could download monthly binary from:

        http://releases.linaro.org/latest/components/kernel/uefi-linaro

<a name="bootwrapper"/>
* BootWrapper: responsible for switching into HYP mode for slave cores  

    could get the source code from following git tree

        https://github.com/hisilicon/bootwrapper.git

    And it is based on following git tree:

        https://git.linaro.org/arm/models/boot-wrapper.git
        https://github.com/virtualopensystems/boot-wrapper.git

<a name="grub"/> 
* [GRUB](http://www.gnu.org/software/grub/): responsible for loading kernel 

    could get the source code from following git tree

        https://github.com/hisilicon/grub.git

    And it is based on Leif's grub: `http://bazaar.launchpad.net/~leif-lindholm/linaro-grub/arm-uefi`  
    I have confrimed that we could not use the mainline grub: `git://git.savannah.gnu.org/grub.git`

<a name="stub"/>
* EFI-STUB: responsible for adding PE HEAD into Kernel Image and making it like   
    a UEFI application.  

    In this case, UEFI could directly load and boot the kernel  
    from the **FAT32 partion**(because our UEFI SATA driver could only support this type)  
    in the harddisk without **GRUB**.

 <a name="kernel"/>
* [kernel](http://www.kernel.org): our operate system

    could get the source code from following git tree

        https://github.com/hisilicon/linux-hisi.git tag:D01-3.18-release

        https://git.linaro.org/landing-teams/working/hisilicon/kernel.git 
        branch: integration-hilt-linux-linaro or integration-hilt-working-v3.14

        https://git.linaro.org/kernel/linux-linaro-tracking.git
        https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

    Most of the featuers have already merged into kernel mainline.   
    But some of features is not accepted like: **Ethernet driver, VGIC, SMMU with SATA  
    and reboot**.  
    Following is the commits list need to maintain by ourselves
    
        Ethernet dirver:
        791a2af hip04: dts: update eth resource
        a162505 hip04: dts: add mdio resource
        90edc4d hip04: dts: add ether resource
        bbdab0b hip04: serdes: fix build error due to macro __DATE__
        2d326b0 misc: add sirdes driver

        VGIC:
        http://article.gmane.org/gmane.linux.ports.arm.kernel/344932/raw

        SMMU with SATA:
        commits: e5baf66 2921667 on branch: testing/0429-for-xuwei in
        https://git.linaro.org/landing-teams/working/hisilicon/kernel.git 
        
        Reboot:
        commit: 4469f84 at branch: integration-hilt-d01 in
        https://git.linaro.org/landing-teams/working/hisilicon/kernel.git 

    And you could run following commands to pick them up:

        git cherry-pick --strategy recursive -X theirs 2d326b0
        git cherry-pick bbdab0b
        git cherry-pick --strategy recursive -X theirs 90edc4d
        git cherry-pick --strategy recursive -X theirs a162505
        git cherry-pick --strategy recursive -X theirs 791a2af

<a name="osv"/> 
* Linux Distributions: each distribution is downloaded from itself website   
	or from Linaro.

        +-----------+------------------------------------------------+
        |  Ubuntu   |  http://releases.linaro.org/latest/ubuntu      |
        +-----------+------------------------------------------------+
        |  OpenSuse |                                                |
        +-----------+------------------------------------------------+
        |  Fedora   |                                                |
        +-----------+------------------------------------------------+
        |  Debian   |                                                |
        +-----------+------------------------------------------------+

    But the NAND Rootfs is created by ourselves and it could download from:
        
        https://github.com/hisilicon/boards/blob/master/D01/release/.filesystem
        http://releases.linaro.org/latest/ubuntu/boards/lt-d01/.filesystem

<a name="toolchain"/>        
* ToolChain: used to compile and debug and downdload from  

        http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux.tar.xz       

<a name="hacking"/> 
##D01 Hacking                                                                                        

* [UEFI hacking](#uefihacking)
    * [compile UEFI source code](#compileuefi)
    * [boot D01 to UEFI shell](#uefishell)
    * [upgrade UEFI](#upgradeuefi)
    * [how to restore the UEFI when the UEFI did not work](#restoreuefi)
* [Bootwrapper hacking](#bootwrapperhacking)
    * [compile BootWrapper source code](#compilebootwrapper)
    * [upgrade BootWrapper](#upgradebootwrapper)
* [Grub hacking](#grubhacking)
* [Kernel hacking](#kernelhacking)
* [boot via NAND](#nandboot)
    * [upgrade kernel and NAND rootfs](#upgradekernel)
    * [show & change kernel command line in UEFI](#changecommandline)
    * [boot into NAND Rootfs](#nandfsboot)
    * [boot into NFS](#nfsboot)
    * [boot into Ubuntu on SATA](#ubuntuboot)
* [boot via PXE](#pxeboot)
    * [set up TFTP server](#installtftp)
    * [set up DHCP server](#installtftp)
    * [enter PXE in the UEFI shell](#enterpxe)
* [boot via EFI-stub](#stubboot)
    * [compile kernel with EFI stub](#compilestubkernel)
    * [install EFI-stub kernel on SATA disk](#installefi)
    * [launch EFI-stub Kernel in the UEFI shell](#enterefi)
* [boot via GRUB](#grubboot)
    * [install GRUB on SATA disk](#installgrub)
    * [launch GRUB in the UEFI shell](#entergrub)
* [KVM on D01](#kvm)
* [Xen on D01](#xen)

<a name="uefihacking">
###UEFI hacking 

FTP protocol is used in D01 UEFI for file downloading.   
So, before this step, please make sure you have a working  
FTP server running in your local network.   
And D01 can use FTP to get files from network.

<a name="compileuefi"/>
####compile UEFI source code 

A general guideline is here: https://wiki.linaro.org/LEG/Engineering/Kernel/UEFI/build
For simplicity, to build UEFI for D01 board, follow these steps:

1. Prepare build environment on your Ubuntu PC:
		
		sudo apt-get install uuid-dev build-essential gcc-arm-linux-gnueabi
		sudo apt-get install gcc-arm-linux-gnueabihf

	Clone Linaro's uefi tools.

	 	cd ~
 		git clone git://git.linaro.org/arm/uefi/uefi-tools.git
 		export PATH=$PATH:~/uefi-tools.git
 		
	Enter uefi-tools, add a build item for D01.
	Note: This will be submitted into Linaro uefi-tools.git soon.  
	If you found it already there, don't panic.
	
		diff --git a/platforms.config b/platforms.config
		index 2c29a29..7710b80 100755
		--- a/platforms.config
		+++ b/platforms.config
		@@ -132,3 +132,10 @@ BUILDFLAGS=
		 DSC=BeagleBoardPkg/BeagleBoardPkg.dsc
		 ARCH=ARM
		 [/]
		+
		+[d01]
		+LONGNAME=HiSilicon D01 Cortex-A15 16-cores
		+BUILDFLAGS=-D EDK2_ARMVE_STANDALONE=1
		+DSC=HisiPkg/D01BoardPkg/D01BoardPkg.dsc
		+ARCH=ARM
		+[/]

2. download D01 UEFI source code from [above section](#uefi)

3. Enter your uefi source code folder, and run uefi-build.sh to build.
 
		cd your/path/to/uefi/source;
		uefi-build.sh -b DEBUG d01
		
	Note: for a release version.
		
		uefi-build.sh -b RELEASE d01
		
	When finished, you can find the build result here:
	
		ls -la Build/D01/DEBUG_GCC46/FV/D01.fd
		
	Note: for a release version:
	
		ls -la Build/D01/RELEASE_GCC46/FV/D01.fd
		
<a name="uefishell"/> 
####boot D01 to UEFI shell(EBL)                                                                         

1. Make sure Jumper J39 is in position 1 and 2.(This means to use custom UEFI.  
If position 2 and 3 are connected, it will boot with the default UEFI,   
which could never be flashed.)

2. Serial console (Connector UART0): 115200/8/N/1

3. Ethernet (FE, J44)

4. The board is shipped with a power adapter. Apply it to the board,   
and turn it on.You should be able to see output in serial console now.

5. Keep pressing 's' in minicom or other serial terminal to start UEFI Boot Menu.
Select [b] EBL. Type in 'help' to see all commands supported in EBL.

<a name="upgradeuefi"/>
####upgrade UEFI 

1. download [`D01.fd`](#release) the UEFI binary file or [compile](#compileuefi) it by yourself

2. copy the `D01.fd` to the FTP server

3. [Enter EBL](#uefishell) as described above. 

4. Type in these commands: 
       
		IP address config:
			> ifconfig -s eth0 [IP.address] [mask] [gateway]
			  eg. ifconfig -s eth0 192.168.10.155 255.255.255.0 192.168.10.1
		download from FTP server (Note, filename must be EFI-BOOT.fd)
			> provision [server.IP] -u [user.name] -p [passwd] -f EFI-BOOT.fd
			  eg. provision 192.168.10.6 -u dj -p dj -f EFI-BOOT.fd
		flash to NOR:
			> updateL1 EFI-BOOT.fd

5. reboot the D01 board

<a name="restoreuefi"/>
####how to restore the UEFI when the UEFI did not work 

In case of failure in UEFI, you can always switch to a factory default  
(known-good, non-erasable) UEFI by shorting Pin 2 and Pin 3 of J39.   
(Refer to Mark 18 of [d01-portrait.png](#pic))

1. Power off the board, disconnect power supply

2. Short Pin 2 and Pin 3 of J39 (Leave Pin 1 unconnected)

3. On your client (minicom or similar tools),   
change UART baudrate to 9600 or 115200

4. Apply power to the board, turn it on

5. Following above steps to [upgrade UEFI](#upgradeuefi)

6. Power off the board, disconnect power supply

7. Short Pin 1 and Pin 2 of J39 (Leave Pin 3 unconnected)

8. On your client, change UART baudrate to 115200. (That's usually the case.   
Unless you know that you are using a 9600 baudrate UEFI.)


<a name="bootwrapperhacking"/>
###BootWrapper Hacking

<a name="compilebootwrapper"/>
####compile BootWrapper source code

1. download BootWrapper source code from [above section](#bootwrapper)

2. use `make` command to compile and it will generate `.text` and `.monitor`

<a name="upgradebootwrapper"/>
####upgrade Bootwrapper

Boot D01 to UEFI shell. And in EBL, type in these commands:

1. IP address config:
        > ifconfig -s eth0 [IP.address] [mask] [gateway]
          eg. ifconfig -s eth0 192.168.10.155 255.255.255.0 192.168.10.1

2. download BootWrapper binary from FTP server (Note, filenames must not be changed):  

        > provision [server.IP] -u [user.name] -p [passwd] -f .text
        > provision [server.IP] -u [user.name] -p [passwd] -f .monitor
          eg. provision 192.168.10.6 -u dj -p dj -f .text
          eg. provision 192.168.10.6 -u dj -p dj -f .monitor
          
3. Reboot D01

<a name="grubhacking"/>
###Grub Hacking

1. Prepare build environment on your Ubuntu PC:

		sudo apt-get install autoconf autogen automake bison flex libfreetype6 libfreetype6-dev

2. Download Grub source code from [above section](#grub)

3. Run following commands to build:

		cd grub
		./autogen.sh
		CROSS_COMPILE=arm-linux-gnueabihf- ./configure --target=arm-linux-gnueabihf --with-platform=efi --prefix=$HOME/grub2-build/
		make && make install

<a name="kernelhacking"/>
###Kernel Hacking

1. download kernel source code from [above section](#kernel)

2. use following command to compile:

		make  ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- O=../linux-next.build hisi_defconfig
		make  ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- O=../linux-next.build -j8 zImage
		make  ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- O=../linux-next.build hip04-d01.dtb
		cat ../linux-next.build/arch/arm/boot/zImage ../linux-next.build/arch/arm/boot/dts/hip04-d01.dtb > /$(FTP-server-path)/.kernel

<a name="nandboot"/>
###boot via NAND

if booting via NAND, could use following method to [upgrade kernel and NAND rootfs](#upgradekernel).
And by [changing the command line](#changecommandline) to boot into different root filesystem.

<a name="upgradekernel"/>
####upgrade kernel and NAND rootfs
Boot D01 to UEFI shell. And in EBL, type in these commands:

1. IP address config:
        > ifconfig -s eth0 [IP.address] [mask] [gateway]
          eg. ifconfig -s eth0 192.168.10.155 255.255.255.0 192.168.10.1

2. download kernel binary from FTP server (Note, filenames must not be changed):  

        > provision [server.IP] -u [user.name] -p [passwd] -f .filesystem
        > provision [server.IP] -u [user.name] -p [passwd] -f .kernel
          eg. provision 192.168.10.6 -u dj -p dj -f .filesystem
          eg. provision 192.168.10.6 -u dj -p dj -f .kernel

3. Reboot D01

<a name="changecommandline"/>
####show & change kernel command line in UEFI

1. Boot D01 and enter EBL.

2. Use `getlinuxatag` to see current kernel parameter:

        > getlinuxatag

3. Use `changelinuxatag` to change kernel cmdline to this:

        > changelinuxatag
          ...
          console=ttyS0,115200 initrd=0x10d00000,0x1800000 rdinit=/linuxrc earlyprintk
          ...

4. Use setlinuxatag to save to FLASH
        > setlinuxatag

5. Reboot D01 

<a name="nandfsboot"/>
####boot into NAND rootfs

1. Change kernel command line as:

          console=ttyS0,115200 initrd=0x10d00000,0x1800000 rdinit=/linuxrc earlyprintk
          
2. Reboot D01 

<a name="nfsboot"/>	
####boot into NFS

Before doing that, please make sure you have UEFI, [.text, .monitor](#bootwrapperhacking),  
and [.kernel](#upgradekernel) installed on your D01.  
(.filesystem is not necessary because kernel will boot with NFS).

On your local env:

1. Choose a machine and use it as NFS server.   
Enable NFS service on it. Please follow the guide in your host Machine  
on how to do this.

2. Download Ubuntu NFS server [image release for D01](#osv)
Extract this file. Export this path in NFS's config.

On D01 board:

1. Boot up the board, and enter UEFI EBL shell.
2. Follow [this step](#changecommandline) to update kernel cmdline.   
	Change kernel cmdline to:
	
		console=ttyS0,115200 earlyprintk rootfstype=nfsroot root=/dev/nfs rw nfsroot=<NFS-server-ip>:<path-to-exported-NFS-files> ip=<client-ip>:<NFS-server-ip>:<gw-ip>:<netmask>::eth0:on:<dns0-ip>:<dns1-ip> user_debug=31 nfsrootdebug

	Here is an example:

		console=ttyS0,115200 earlyprintk rootfstype=nfsroot root=/dev/nfs rw nfsroot=192.168.0.108:/Users/docularxu/Downloads/mnt ip=192.168.0.150:192.168.0.108:192.168.0.1:255.255.255.0::eth0:on:192.168.0.1:8.8.8.8 user_debug=31 nfsrootdebug

	Note, this is an example. Please change the values according to your   
	local environment. Explanation of each parameter can be found in kernel   
	source Documentations. Like: 
			
			ip=<client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>:<dns0-ip>:<dns1-ip>
			
3. Reboot the board.

After kernel boots and the Console shell is shown up,   
this means the process is successful. It is recommended to config DNS server,  
so your board can access the Internet. sudo to edit this file `/etc/resolv.conf`,   
and add your DNS server into it. Eg.

		nameserver 192.168.0.1

<a name="ubuntuboot"/>		
####boot into Ubuntu on SATA

1. Download Ubuntu server [release for D01](#osv)

2. Boot the board with NFS 

3. Partition and Format SATA disk.   
Create one ext4 partition of at least 10G to host the root filesystem.   
In my case, it is `/dev/sda1`.

4. Mount `/dev/sda1` to `/mnt/sda1`

		mkdir /mnt/sda1 /mnt/sda3
		mount -t ext4 /dev/sda1 /mnt/sda1

5. Extract release from Step 1 to `/mnt/sda1`

6. Unmount `/dev/sda1`

7. Reboot, and enter UEFI EBL shell.

8. Change kernel cmdline. Refer to [above](#changecommandline) for details how-to.
		
		console=ttyS0,115200 root=/dev/sda1 rootfstype=ext4 rw earlyprintk ip=192.168.0.150:192.168.0.108:192.168.0.1:255.255.255.0::eth0:on:192.168.0.1:8.8.8.8
		
Note, you should change /dev/sda1 to according to your SATA disk situation.

9. Reboot the board.

After kernel boots and the Console shell is shown up,   
this means the process is successful. It is recommended to config DNS server,   
so your board can access the Internet. sudo to edit this file /etc/resolv.conf,  
and add your DNS server into it. Eg.

		nameserver 192.168.0.1

<a name="pxeboot"/>
###boot via PXE

PXE boot is built upon DHCP and TFTP. So, to verify PXE,   
the first thing you need to do is to create and set up a TFTP server and DHCP   
server on your local network. 

<a name="installtftp"/>
####set up TFTP server on Ubuntu

1. install TFTP server and TFTP client(optional, tftp-hpa is the client package)

		sudo apt-get install tftpd-hpa tftp-hpa

2. configure the TFTP server, update `/etc/default/tftpd-hpa` like following:

		TFTP_USERNAME=tftp
		TFTP_ADDRESS=0.0.0.0:69
		TFTP_DIRECTORY=/var/lib/tftpboot
		TFTP_OPTIONS=-l -c -s

3. set up TFTP server directory

		sudo mkdir /var/lib/tftpboot
		sudo chmod -R 777 /var/lib/tftpboot/
		
4. restart TFTP server

		service tftpd-hpa status
		service tftpd-hpa restart
		service tftpd-hpa force-reload

<a name="installtftp"/>
####set up DHCP server on Ubuntu

Refer to https://help.ubuntu.com/community/isc-dhcp-server
For a simplified direction, try these steps:

1. install DHCP server package

		sudo apt-get install isc-dhcp-server
		
2. Edit `/etc/dhcp/dhcpd.conf` to suit your needs and particular configuration.  
	Make sure filename is "grub2.efi". Here is an example:

		$ cat /etc/dhcp/dhcpd.conf
		# Sample /etc/dhcpd.conf
		# (add your comments here)
		default-lease-time 600;
		max-lease-time 7200;
		option subnet-mask 255.255.255.0;
		option broadcast-address 192.168.0.255;
		option routers 192.168.0.1;
		option domain-name-servers 192.168.0.1;
		option domain-name "mydomain.example";
		subnet 192.168.0.0 netmask 255.255.255.0 {
		        range 192.168.0.160 192.168.0.180;
		        option subnet-mask 255.255.255.0;
		        filename "grub2.efi";
		}
		#
		
3. Edit `/etc/default/isc-dhcp-server` to specify the interfaces dhcpd  
should listen to. By default it listens to eth0.  
    Assign a static ip to the interface that you will use for dhcp.
	
4. use these commands to start or check dhcp service
	
		sudo service isc-dhcp-server status
		sudo service isc-dhcp-server start
	
<a name="enterpxe"/>
####enter PXE in the UEFI shell

In the description below, we suppose you have config:

    * TFTP root path is: `/var/lib/tftpboot`
    * DHCP default download filename is: `grub2.efi`
	
1. Download PXE related files from [release](#release): `D01.fd, grub2.efi  
grub.cfg, hip04-d01.dtb and zImage` or compile them by yourself, could refer to  
[hacking section](#hacking) to compile and upgrade UEFI, BootWrapper,   
Grub and Kernel.

2. Create folder structure under your TFTP root path,  
and copy files into it like this:

		/var/lib/tftpboot$ tree .
        .
        ├── boot
        │   └── grub
        │       └── grub.cfg
        ├── grub2.efi
        ├── hip04-d01.dtb
        └── zImage

3. On D01 board need to upgrade to a later UEFI which can support PXE boot.

4. When booting D01, enter into UEFI Shell, then input these commands in turn:

		2      //Boot Manager
		1      //Add Boot Device Entry
		3      //PXE on MAC Address:...
		PXE      //Description for this new Entry
		5      //Return to main menu
		2      //PXE
		
These will enable the board to download 'grub2.efi' and launch it,   
then in 3 seconds, grub2 will download and boot linux.

A full log is attached here: [sample.pxe.boot.log.minicom.cap.txt](https://wiki.linaro.org/Boards/D01/PXE?action=AttachFile&do=view&target=sample.pxe.boot.log.minicom.cap.txt)

<a name="stubboot"/>
###boot via EFI-stub

<a name="compilestubkernel"/>
####compile kernel with EFI stub(#compilestubkernel)

<a name="installefi"/>
####install EFI-stub kernel on SATA disk(#installefi)

1. partition the disk to MBR. use `fdisk` and `mkfs`   
command to create a FAT partition. If using D01 

		fdisk, to create a "Win95" type of partition. Eg.
		Command (m for help): p
		
		Disk /dev/sdb: 500.1 GB, 500107862016 bytes
		255 heads, 63 sectors/track, 60801 cylinders, total 976773168 sectors
		Units = sectors of 1 * 512 = 512 bytes
		Sector size (logical/physical): 512 bytes / 4096 bytes
		I/O size (minimum/optimal): 4096 bytes / 4096 bytes
		Disk identifier: 0x90465146
		
		   Device Boot      Start         End      Blocks   Id  System
		/dev/sdb1            2048    41945087    20971520    b  W95 FAT32


2. copy zImage and device tree blob to the FAT partition. If using D01, you 
could do this by booting via [NAND](#nandboot) or [PXE](#pxeboot) firstly.

<a name="enterefi"/>
####launch EFI-stub Kernel in the UEFI shell(#enterefi)

1. boot the board.

2. Press 's' to Start Boot Menu.

3. Press '4' to select [4] Shell. You will see something like this

		UEFI Interactive Shell v2.0
		EDK II
		UEFI v2.40 (ARM D01 EFI May  9 2014 10:43:52, 0x00000000)
        Mapping table
        FS1: Alias(s):F0:;BLK5:
        VenMsg(06ED4DD0-FF78-11D3-BDC4-00A0C94053D1,0000000000000000)
        FS0: Alias(s):HD5b0a1:;BLK3:
        Pci(0x0,0x0)/Sata(0x1,0x0,0x0)/HD(1,MBR,0x05A8565C,0x3F,0x5FC6D)
        BLK0: Alias(s):
        Pci(0x0,0x0)/Sata(0x0,0x0,0x0)
        BLK1: Alias(s):
        Pci(0x0,0x0)/Sata(0x0,0x0,0x0)/HD(1,MBR,0x364C8C12,0x800,0x3A385830)
        BLK2: Alias(s):
        Pci(0x0,0x0)/Sata(0x1,0x0,0x0)
        BLK4: Alias(s):
        Pci(0x0,0x0)/Sata(0x1,0x0,0x0)/HD(2,MBR,0x05A8565C,0x5FCAC,0x302C4)
        Press ESC in 1 seconds to skip startup.nsh or any other key to continue.

4. Command sequence to launch zImage:

		Shell> FS0:
		FS0:\> ls
		Directory of: FS0:\
		05/09/2014  02:28           2,665,312  zImage_lt-d01
		05/09/2014  02:28               3,438  hip04-d01.dtb
		01/01/1980  00:00 <DIR>         1,536  boot
		          2 File(s)   2,668,750 bytes
		          1 Dir(s)
		FS0:\> zImage_lt-d01 dtb=hip04-d01.dtb console=ttyS0,115200 root=/dev/sda2 rw earlyprintk
		EFI stub: Booting Linux Kernel...
		
	Note: Attached here a reference booting log. minicom_sata.cap
	Note: only FAT partition is recognizable so far.

<a name="grubboot"/>
###boot via GRUB    

Create GRUB configure file like following, or you could download it from 
the [release](#release)

        #
        # Sample GRUB configuration file
        #

        # Boot automatically after 0 secs.
        set timeout=3

        # By default, boot the D01 kernel
        set default=NAND

        # For booting GNU/Linux
        menuentry "D01-NAND" --id NAND {
            devicetree (hd0,msdos1)/hip04-d01.dtb
            linux (hd0,msdos1)/zImage console=ttyS0,115200 earlyprintk initrd=0x10d00000,0x1800000 rdinit=/linuxrc ip=dhcp
        }

        menuentry "D01-Ubuntu" --id UBUNTU {
            devicetree (hd0,msdos1)/hip04-d01.dtb
            linux (hd0,msdos1)/zImage console=ttyS0,115200 earlyprintk root=/dev/sda2 rootfstype=ext4 rw ip=dhcp
        }

        menuentry "D01-Opensuse" --id SUSE {
            devicetree (hd0,msdos1)/hip04-d01.dtb
            linux (hd0,msdos1)/zImage console=ttyS0,115200 earlyprintk root=/dev/sdb1 rootfstype=ext4 rw ip=dhcp
        }

        menuentry "D01-NFS" --id NFS {
            devicetree (hd0,msdos1)/hip04-d01.dtb
            linux (hd0,msdos1)/zImage console=ttyS0,115200 earlyprintk rootfstype=nfsroot root=/dev/nfs rw nfsroot=192.168.201.25:/home/joyx/develop/d01/workspace/ubuntu-image/binary  ip=192.168.201.39:192.168.201.1::255.255.255.128::eth0:on:192.168.201.1:8.8.8.8 
        }


<a name="installgrub"/>
####install GRUB on SATA disk

installing GRUB on SATA disk is similiar with installing   
[EFI-stub kernel on SATA](#installefi).  
Copy following files into the FAT partition of the SATA disk and the directory tree  
is like this:

		├── boot
		│   └── grub
		│       └── grub.cfg
		├── grub2.efi
		├── hip04-d01.dtb
		└── zImage


<a name="entergrub"/>
####launch GRUB in the UEFI shell

When booting D01, enter into UEFI Shell, then input these commands in turn:

		2      //Boot Manager
		1      //Add Boot Device Entry
		2      //Pci(0x0,0x0)/Sata(0x1,0x0,0x0)...
        GRUB2  //File path of the EFI Application or the kernel
        y      //Is an EFI Application? [y/n]
        y      //Is your application is an OS loader? [y/n]
		5      //Return to main menu
		3      //GRUB


A quick view about GRUB booting is like following:

<img src="./pic/d01-grub.png" style="width:700px;height:300px;margin-left:30px"/> 

<a name="kvm"/>
###KVM on D01

Host Kernel on D01 is already support KVM.
steps to build kernel could refer to [Kernal hacking](#kernelhacking).

Steps to build and run guest:

1. Clone the Torvalds kernel

        git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

2. Checkout to v3.12 branch

        git checkout -b linux-3.12 v3.12

3. Cross compile the kernel

        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_defconfig
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j2

4. Grab Pawel's DTS tree: git://linux-arm.org/arm-dts.git

        git clone git://linux-arm.org/arm-dts.git
        git chechout -b v3_12 v3.12

5. From your kernel tree, run:

        ./script/dtc/dtc -O dtb -o rtsm_ve-cortex_a15x4.dtb  wherever_your_arm-dts_tree_is/fast_models/rtsm_ve-cortex_a15x4.dts

6. Launch QEMU on D01:

        qemu-system-arm -enable-kvm -kernel zImage -dtb rtsm_ve-cortex_a15x4.dtb -initrd initrd.cpio.gz -append "console=ttyAMA0 earlyprintk rdinit=/linuxrc" -nographic -machine vexpress-a15,kernel_irqchip=on -m 128 -smp 4 -cpu cortex-a15 -rtc base=localtime

<a name="xen"/>
###Xen on D01


<a name="refer"/>
##Reference

* https://wiki.linaro.org/Boards/D01
* https://wiki.linaro.org/LEG/Engineering/Kernel/ACPI/ACPIviaEFI
