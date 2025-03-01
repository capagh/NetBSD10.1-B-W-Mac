# Installing NetBSD 10.1 on a blue-and-white Power Macintosh G3

Direct questions or comments to capa150 at gmail.com

## Contents:

- Background story
- Performing a dual-boot MacOS/NetBSD install:
- How to build an install kernel and a regular kernel for NetBSD 10.1/macppc.
- Preventing caps-lock from freezing a blue-and-white G3
- X11 with the blue-and-white

There are several ways to install NetBSD 10.1 on a Mac. This text describes two of the ways -- one is a dual-boot Classic MacOS/NetBSD 10.1 system, and the second is a NetBSD 10.1-only system. There are other methods described in the NetBSD install document.

What I used for this install:
- A Blue-and-White Macintosh G3 with a CD/DVD drive.
- An IDE hard disk attached to a PCI IDE card such as a Sonnet Tempo or Acard/Ahard M6280, M6880, M6860 or M6260 (the ‘M’ means it is bootable by a Mac. Otherwise it’s for PCs.) I’ve found the motherboard IDE to be buggy.
- A copy of MacOS 9 on CD/DVD (to partition the drive using MacOS 9’s Drive Setup utility.)
- A copy of MacOS 8.6 (I like 8.6 more than 9. I used Drive Setup on 9 to partition the disk. Then I installed 8.6.)
- A copy of NetBSD 10.1 on CD/DVD. (https://cdn.netbsd.org/pub/NetBSD/images/10.1/ titled “NetBSD-10.1-macppc.iso.”) (Alternately, skip the ISO and just install via FTP.)
- The macOS X application “QuickFTP,” available free from the macOS app store.

Note: I’ve not tried a SCSI disk on my Blue-and White. I suspect SCSI would wholly avoid the problems I encountered. Consider using blueSCSI or ZuluSCSI as finding a good SCSI drive is difficult these days.

This install assumes the Mac has one hard disk. Be sure to back-up any important data before proceeding.

It’s important to note the Blue-and-White G3 Macs use Open Firmware version 3.x.
The install process for Macs that use Open Firmware 1.x or 2.x is different and not covered in this document. Refer to the official NetBSD install notes for more.

## The backstory:

### Problem No. 1:
My Blue-and-White G3 came with an IDE hard drive attached to the motherboard’s IDE slot. I installed NetBSD 10.1 and it seemed to work. But eventually I noticed a quirk: When issued a "shutdown -r now" command, the system will often state:

```  
About to run shutdown hooks ...  
Stopping cron.  
Stopping inetd.
```


And then it hangs. This is an incomplete shut-down, as there are supposed to be more lines after “Stopping inetd.” But at other times it would complete the shutdown process. I eventually found the process rndctl -- which is called during shutdown -- would cause the computer to sometimes hang. Rndctl is used for entropy-related work.

(See NetBSD Problem Report #59014 https://gnats.netbsd.org/cgi-bin/query-pr-single.pl?number=59014 for more on this problem.)

I determined the problem is caused by the CMD IDE controller chip the blue-and-white uses. This controller is known to be buggy, according to the Wikipedia entry on it. https://en.wikipedia.org/wiki/CMD640

I resolved the rndctl problem by using a Sonnet Tempo IDE PCI controller card instead of the motherboard controller. Which led to ...

### Problem No. 2:

Although I resolved the rndctrl problem, I gained a new problem.

When booting off the Tempo, the Mac would report hundreds of errors when running NetBSD, such as: "wd0: aborted command, interface CRC error" and “wd0c: error reading fsbn.”

I believe the problem is due to the netbsd kernel accessing my UDMA IDE hard disk in UDMA mode 6 (ATA/133), but the Blue and White G3 specifications indicate the Mac is designed for UDMA mode 2 (ATA/33).

The Mac can't handle the higher speed and thus complains profusely.

I resolved this problem by compiling two custom kernels -- one for installation, and one for regular use. Both are the same as the default kernels except they limit the IDE drive mode to UDMA mode 2. (I also changed the console text color to green text on a black background, with white text for kernel messages -- default is black text on white background. Nothing else has been changed.)

(See NetBSD Problem Report #59078 https://gnats.netbsd.org/cgi-bin/query-pr-single.pl?number=59078 for more.)

## Performing a dual-boot MacOS/NetBSD install:

1. Insert the OS 9 install disc into the Mac. Hold down the ‘C’ key to boot from the CD.

2. Open the “Utilities” folder on the disc. Launch “Drive Setup.” (The version of Drive Setup that comes with OS 9 is required. Older versions will not work, according to the NetBSD install document.)

3. For this example install, three partitions will be used. One for MacOS, one for NetBSD root and usr, and one for NetBSD swap. Select the drive to be initialized then click “Custom Setup.”  Set “Partitioning Scheme” to three partitions. The top partition will be for MacOS. It should be at least 400 megabytes in size but you can make it larger if you like. Set the “Type” to either “MacOS Standard” (HFS) or “MacOS Extended” (HFS+).

4. Select the second-from-top partition. This will be your NetBSD root and usr partition. Make this partition as large as possible, while keeping some space free for swap. Set the “Type” to “A/UX Root.”

5. Select the third-from-top partition. This will be your swap drive. It should be the same size as the amount of RAM available. “Type” should be set to “A/UX Swap.”

6. There may be a small amount of space at the bottom of the partition map labeled “Extra.” This is OK.

7. Click “OK” followed by “Initialize.” Quit Drive Setup. 

8. Rename your new Mac drive if you like. Run the MacOS installer (I have also installed MacOS 8.5/8.6 for a dual-boot system. However you must use the newer MacOS 9 Drive Setup application to initially partition the drive, not the version of Drive Setup that comes with 8.5/8.6.)

9. Reboot into the new MacOS. Quit the MacOS Setup Assistant. Insert the NetBSD 10.1 disc. Drag “ofwboot.xcf” to the Mac’s hard drive.

10. Two kernels on this repository will limit the drive to UDMA mode 2:

netbsd-INSTALL  
netbsd

Download netbsd-INSTALL and move it to the MacOS drive.

QuickFTP Server, running on my MacBook Air, was a helpful tool for this installation. I connected my blue-and-white G3 to the local area network and set the TCP/IP control panel to DHCP. I then used Internet Explorer on the G3 to access the files from QuickFTP, using the IP address displayed on QuickFTP. (example “ftp://10.0.0.55” into the Internet Explorer address bar.) I right-clicked (or control-click) each file and saved it to the base level of the Mac’s drive.

QuickFTP uses this default path to host files, but it can be changed:  
/Users/chris/Library/Containers/QuickFTP/Data/  
For this document, I changed this to /Users/chris instead, as it’s easier to access.
Set a username, password, and read and write access on QuickFTP.

Both ofwboot.xcf and the modified installer netbsd-INSTALL files must be in the base level of the Mac’s drive, not in any subfolder.

11. Reboot the Mac and hold down command-option-O-F to boot into Open Firmware. The NetBSD install  document describes how the Open Firmware bootloader works. It can be tricky to figure out, and the exact command varies depending on the hardware available. Experimentation is pretty much mandatory. Here’s the command my G3 uses:
```
0 > boot /pci@80000000/pci-bridge@d/Ultra-Tek100P@3/sd:,\ofwboot.xcf netbsd-INSTALL
```
The boot process should begin. You should see a line about the hard disk on the screen, so make note of the drive name. In my case the drive is “wd0.”

When asked, accept “vt100” as terminal type.

12. Select “(S)hell” when asked. 

11. Optional: type:
```
# disklabel /dev/wd0
```
to see some cool partition information. Make note of the letter assigned the partitions. On my install, “a” is assigned to 4.2BSD (root/usr) and “b” is assigned to swap. (Do not use disklabel for any other purpose as it will wreck your partitions.)

12. Create a NetBSD filesystem on partition “a,” mount it, and create an etc folder:
```
# newfs /dev/wd0a
# mount /dev/wd0a /targetroot
# mkdir /targetroot/etc
```
(The official NetBSD install notes say to mount this partition as /mnt but this will cause an error later during the install, so use /targetroot.)

13. Create an fstab file: Type the following (I used tabs to keep everything tidy, but you can also use spaces.):
```
# cat > /targetroot/etc/fstab
/dev/wd0a	/		ffs	rw	1	1
/dev/wd0b	none		swap	sw	0	0
ptyfs		/dev/pts	ptyfs	rw
```
Press the return key at the end of the ptyfs line, followed by ctrl-d to save the file.

Unmount the filesystem:
```
# umount /targetroot
```
14.(optional) Type “pdisk /dev/wd0c” (“c” here represents the entire drive) to learn more about partitions.
Type “p” to print the partition table. Do not run any other command. Make note of the leftmost numbers associated with the partitions as you’ll need this to boot from Open Firmware later on. In particular, note the number associated with the HFS/HFS+ partition and also with the AU/X Root partition. Type “q” to quit.

15. Actually install NetBSD! Type “sysinst” to run the installer. **Do NOT** select option “a” as this will demolish the partitions. Instead select option “c: Re-install sets or install additional set.” then “yes” then “b: wd0 ...” then “a: Full installation” (or as desired) then “a: CD-ROM / DVD” (or as desired.)

Note: If you use networking, be sure to specify “100baseTX” instead of the default “autoselect” so ethernet functions correctly.

16. You’ll  now be asked to set up entropy. The way I did it is with option “c: Load raw binary random data.”

In Terminal.app on my M1:
```
% dd if=/dev/random bs=32 count=1 of=/Users/chris/random.tmp
```
Select “b: Download via ftp”  
Select “a: bm0” (onboard ethernet)

When you see Sysinst show this message:  
“Network media type (empty to autoconfigure) [autoselect]:”  
Do not hit return to accept “autoselect”, as this does not work. Instead type in:
```
100baseTX
```
Then “Yes” to perform autoconfiguration.  
Type in a hostname as desired.  
Feel free to leave domain blank.

To configure FTP, enter fields for Host, User and Password (same as what QuickFTP uses) as necessary and start download.

Now you’ll be back at the sysinst main menu.

Do not reboot yet. Sysinst installed the regular default kernel and that needs to be replaced with the kernel modified for UDMA mode 2.

Select “Exit Install System”

Mount our new system and rename the default kernel:
```
# mount /dev/wd0a /mnt
# cd /mnt
# mv netbsd netbsd.original
```
Download the modified kernel:
```
# ftp 10.0.0.220 
ftp> get netbsd
ftp> quit
```
Modify permissions to match original:
```
# chmod 755 netbsd
# cd /
# umount /mnt
# reboot
```
When you hear the chime, hold down command-option-O-F again to get into Open Firmware. Boot off the modified regular kernel:
```
0 > boot /pci@80000000/pci-bridge@d/Ultra-Tek100P@3/sd:,\ofwboot.xcf netbsd
```
16. NetBSD will state /etc/rc.conf isn’t configured. Hit return to accept /bin/sh. Type:
```
# export TERM=vt100
# mount -uw /
```
Now edit the rc.conf file:
```
# vi /etc/rc.conf
```
A short lesson in how to use the vi text editor:  
Vi has two modes: command mode and edit mode.  
In command mode, use the arrow keys (or the HJKL keys) to move the cursor around.  
In command mode, use “x” key to delete text.  
Press “i” key to enter insert mode. Now you can edit text.  
Press “ESC” to exit insert mode and return to command mode.  
Press “ESC” followed by “:wq” to write the file to disk and quit.  

Edit rc.conf to make these changes and additions:
```
rc_configured=YES
dhcpcd=YES
wscons=YES
hostname=netbsd (or whatever name you prefer)
ifconfig_bm0=“media 100baseTX mediaopt full-duplex”
```
Esc :wq to write the file to disk and quit vi.

Set filesystem to read-only (so fsck doesn’t complain), and exit to resume booting:
```
# mount -ur / 
# exit
```
The boot process will continue.

Login as root (no password yet.)

Add a password for root:
```
# passwd root
```
19. Create a normal user (e.g. johndoe) and a home directory for that user, and add that user to the superuser group:
```# useradd -m -G wheel johndoe```

Give the user a password:
```# passwd johndoe```

All done.

To reboot:
```# shutdown -r now```
To shut-down and power-off:
```# shutdown -p now```

If you wish to boot into MacOS, boot without holding command-option-O-F.

------

## How to build an install kernel and a regular kernel for NetBSD 10.1/macppc.

This section is optional, as I’ve already made the two kernels available for download. However, if you’d like to build those kernels yourself, the steps are listed here:

For this example, two computers are used: a blue-and-white G3, and a Macbook Air M1.

Install UTM (Qemu) on the M1. Create an emulated amd64 machine and install NetBSD 10.1. (Running NetBSD for Arm would be more sensible for M1, but I couldn’t get it to install, hence the choice to emulate amd64 instead.)

In UTM preferences, set the number of cores to 8 (for M1).  
Create a regular user in group wheel.  
Enable dhcpcd with (as root):  
```
# /etc/rc.d/dhcpcd onestart
```
### OBTAINING SOURCES

Build sources as described here:  
https://www.netbsd.org/docs/guide/en/chap-fetch.html

(as root):
```
# mkdir /usr/src
# chown chris /usr/src
```
exit to return to regular user.

Grab the source files (no need for xsrc.tgz, as I’ll be using this emulated machine for compiling only.):
```
$ ftp -i ftp://ftp.NetBSD.org/pub/NetBSD/NetBSD-10.1/source/sets/
ftp> get gnusrc.tgz
ftp> get sharesrc.tgz
ftp> get src.tgz
ftp> get syssrc.tgz
ftp> quit
```
You should now have four *.tgz files in your home directory.

Extract all them all:
```
$ for file in *.tgz
> do
> tar -xzf $file -C /
> done
```
This will take a while. 

### CROSS-COMPILING

Since I’m using amd64 to create macppc/powerpc kernels, cross-compilation is required.

https://www.netbsd.org/docs/guide/en/chap-build.html#chap-boot-cross-build-kernel

Although the instructions call for placing sources in ~/cvs/src, I chose to keep them in /usr/src instead.
```
$ mkdir ~/obj
$ cd /usr/src
```
Build the tools:
```
$ ./build.sh -U -O ~/obj -j8 -m macppc -a powerpc tools
```
where  the 8 in -j8 is the number of cores on your system.

After a while, this process will print a summary of results.

Configure the kernel:
```
$ cd /usr/src/sys/arch/macppc/conf
```
The configuration file titled INSTALL is the one with the RAM disk for installation. Copy the original and work on the copy:
```
cp INSTALL INSTALL2
```
Use your preferred editor with INSTALL2.
Change the line:
```
wd*  at atabus? drive ? flags 0x0000
```
to
```wd* at atabus? drive? flags 0x0aac```
and save the change.

This will limit the drive to UDMA mode 2 (ATA/33) which is the drive specification for the blue-and-white G3. See the man page for wd for more information.

Build the install kernel as a “release.” This, apparently, also builds the Unix utilities required on the installer.
```
$ cd /usr/src
$ ./build.sh -U -j8 -O ~/obj -m macppc -a powerpc kernel=INSTALL2 release
```
(Running on an Apple M1 cpu, this task will take around 12 hours to complete.)

https://www.netbsd.org/docs/guide/en/chap-inst-media.html

There will be a newly-created install kernel with the changes for UDMA mode 2 in the directory ~/obj/distrib/macppc/floppies/md-kernel/.

However, this new file isn’t titled “INSTALL2,” but simply “netbsd-INSTALL.”

Transfer this file to the base level of the hard drive of the Classic MacOS machine.

In my case, how I did this was to sftp the file from my UTM machine to the M1 host OS. From the ~/obj/distrib/macppc/floppies/md-kernel/ directory:

Transfer files from the emulated amd64 machine to my M1 Mac’s OS:  
```
$ sftp chris@10.0.0.220  (change the address as necessary)
sftp> put netbsd-INSTALL
sftp> quit
```
netbsd-INSTALL is now in the M1’s home directory (which is, co-incidentally, also QuickFTP’s working directory.)

On the blue-and-white, running Classic MacOS, point Internet Explorer or Netscape to ftp://10.0.0.220 and right-click netbsd-INSTALL to save the file to the base level of the hard disk.

### COMPILE THE REGULAR KERNEL

The installation kernel is ready, but we need a regular kernel as well.

https://www.netbsd.org/docs/guide/en/chap-kernel.html

On the emulated amd64: 
```
$ cd /usr/src/sys/arch/macppc/conf
$ cp GENERIC GENERIC2
```
Use your preferred editor with GENERIC2.
Change the line:
```
wd*  at atabus? drive ? flags 0x0000
```
to
```wd* at atabus? drive? flags 0x0aac```
and save the change.
```
$ cd /usr/src
$ ./build.sh -U -j8 -O ~/obj -m macppc -a powerpc kernel=GENERIC2
```
A summery will tell you where the new kernel (called “netbsd”) is located.

Copy this from the UTM machine to the home directory on my M1:
```
$ cd /home/chris/obj/sys/arch/macppc/compile/GENERIC2/
$ sftp chris@10.0.0.220
sftp> put netbsd
sftp> quit
```
It’s now ready to be copied to the blue-and-white G3 (see elsewhere).

----

## Preventing caps-lock from freezing a blue-and-white G3

On my B&W G3 I noticed the caps-lock key would sometimes -- but not always -- cause the computer to freeze. Unplugging the USB cable would un-freeze the machine, but also cause strange behavior on the console. As such, I found it useful to simultaneously ssh into the B&W from my M1 Air while messing around with caps-lock so I could more easily reboot the machine.

The keyboard in question is a late-1990s Apple M2452.

My “solution” was to map the caps-lock key to the left shift key. Therefore I have two left shift keys, and no caps-lock. But at least I don’t accidentally freeze my Mac any more.

Edit /etc/wscons.conf. 
Underneath the line that begins: “#mapfile /usr/share/wscons/keymaps/pckbd.sv.svascii”
add a new, uncommented line:
```mapfile /usr/share/wscons/keymaps/pckbd.caps.shift```
and save the change.

Then:
```
# cd /usr/share/wscons/keymaps/
# vi pckbd.caps.shift
```
and add one line to this file:
```keysym Caps_Lock = Shift_L```

Save and reboot. Caps will now map to shift.

## X11 with the blue-and-white

Here’s a functional xorg.conf file for the Mac with a ATI Rage 128 PCI card.
It’s probably nowhere near optimal. 3D acceleration is not enabled. Let me know if you know how to enable acceleration.

Place this text as /etc/X11/xorg.conf
```
Section "ServerLayout"
	Identifier     "X.org Configured"
	Screen      0  "Screen0" 0 0
	InputDevice    "Mouse0" "CorePointer"
	InputDevice    "Keyboard0" "CoreKeyboard"
EndSection

Section "Files"
	ModulePath   "/usr/X11R7/lib/modules"
	FontPath     "/usr/X11R7/lib/X11/fonts/misc/"
	FontPath     "/usr/X11R7/lib/X11/fonts/TTF/"
	FontPath     "/usr/X11R7/lib/X11/fonts/Type1/"
	FontPath     "/usr/X11R7/lib/X11/fonts/75dpi/"
	FontPath     "/usr/X11R7/lib/X11/fonts/100dpi/"
EndSection

Section "Module"
	#Load  "dbe"  #action
	#Load  "extmod"  #action
	Load  "dri"
	Load  "dri2"
	Load  "glx"
	Load  "shadow"
EndSection

Section "InputDevice"
	Identifier  "Keyboard0"
	Driver      "kbd"
	Option	    "Protocol" "wskbd"
	Option	    "Device" "/dev/wskbd"
EndSection

Section "InputDevice"
	Identifier  "Mouse0"
	Driver      "mouse"
	Option	    "Protocol" "wsmouse"
	Option	    "Device" "/dev/wsmouse"
	Option	    "ZAxisMapping" "4 5 6 7"
EndSection

Section "Monitor"
	Identifier   "Monitor0"
	VendorName   "Monitor Vendor"
	ModelName    "Monitor Model"
	#HorizSync     58-62		#action
	#VertRefresh   75-117		#action
	#Option "VGA-0" "VGA monitor"  #action
	Modeline "1024x768" 78.75  1024 1040 1136 1312  768 769 772 800 -hsync -vsync 
	Modeline "1024x768" 78.75  1024 1040 1136 1312  768 769 772 800 +hsync +vsync 
EndSection

Section "Monitor"
    Identifier    "LVDS monitor"
    Option "LVDS" "LVDS note"
EndSection

Section "Device"
        ### Available Driver options are:-
        ### Values: <i>: integer, <f>: float, <bool>: "True"/"False",
        ### <string>: "String", <freq>: "<f> Hz/kHz/MHz",
        ### <percent>: "<f>%"
        ### [arg]: arg optional
        #Option     "NoAccel"            	# [<bool>]
        #Option     "Dac6Bit"            	# [<bool>]
        #Option     "VGAAccess"          	# [<bool>]
        #Option     "ShowCache"          	# [<bool>]
        #Option     "SWcursor"           	# [<bool>]
        #Option     "VideoKey"           	# <i>
        #Option     "PanelWidth"         	# <i>
        #Option     "PanelHeight"        	# <i>
        #Option     "ProgramFPRegs"      	# [<bool>]
        #Option     "DMAForXv"           	# [<bool>]
        #Option     "ForcePCIMode"       	# [<bool>]
        #Option     "CCEPIOMode"         	# [<bool>]
        #Option     "CCENoSecurity"      	# [<bool>]
        #Option     "CCEusecTimeout"     	# <i>
        #Option     "AGPMode"            	# <i>
        #Option     "AGPSize"            	# <i>
        #Option     "RingSize"           	# <i>
        #Option     "BufferSize"         	# <i>
        #Option     "EnablePageFlip"     	# [<bool>]
        #Option     "AccelMethod"        	# <str>
        #Option     "RenderAccel"        	# [<bool>]
	#Option		"NoAccel"	"off"  #action
	#Option		"UseFBDev"	"on"  #action
	#Screen		0	#action
	#Option		"ForcePCIMode"    "True"  #not used re manpage but in autogen
	#Option		"Display"	"CRT"  #not used re r128 manpage but x.org disagree
	Identifier  "Card0"
	Driver      "r128"
	BusID       "PCI:0:16:0"
EndSection


Section "Screen"
	Identifier "Screen0"
	Device     "Card0"
	Monitor    "Monitor0"
	Option	  "DPMS"

	DefaultDepth  24

	SubSection "Display"
		Viewport   0 0
		Depth     1
	EndSubSection
	SubSection "Display"
		Viewport   0 0
		Depth     4
	EndSubSection
	SubSection "Display"
		Viewport   0 0
		Depth     8
	EndSubSection
	SubSection "Display"
		Viewport   0 0
		Depth     15
	EndSubSection
	SubSection "Display"
		Viewport   0 0
		Depth     16
	EndSubSection
	SubSection "Display"
		Viewport   0 0
		Depth     24
	EndSubSection
EndSection

Section "DRI"
    Mode 0666
EndSection
```
