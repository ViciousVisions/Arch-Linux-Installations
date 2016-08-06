Arch Linux (EFI boot) Install - Virtualbox Guest
================================================

## Part I:  Prepare storage space to hold operating system components ##

1. View storage devices
  * ### [input] lsblk
    + Identify the name of the disk to be partitioned (sda, sdb, ...)
2. Partition storage device
  * ## [input] cgdisk /dev/sda
    + #### Create efi boot partition ####
      - **First Sector:** default
      - Size: 512MiB
      - Hex Code: ef00
        * EFI System◾
      - Partition Name: boot
    + #### Create swap partition ####
      - First Sector: default
      - Size: 3GiB
        * 1.5 x Memory Size
      - Hex Code: 8200
        * Linux swap
      - Partition Name: swap
    + #### Create root partition ####
      - First Sector: default
      - Size: 15GiB
      - Hex Code: 8304
        * Linux x86-64 root (/)
      - Partition Name: root
    + #### Create home partition ####
      - First Sector: default
      - Size: default
      - Hex Code: 8302
      - Linux /home
      - Partition Name: home
3. Review storage devices
  * lsblk
    + Identify the partitions you just created
4. Format the partitions
  * ### mkfs.fat -F32 /dev/sda1 ####
  * ### mkswap /dev/sda2
  * ### mkfs.ext4 /dev/sda3
  * ### mkfs.ext4 /dev/sda4
5. Connect a directory within /mnt to the partitions
  * ### [input] mount /dev/sda3 /mnt ###
    + root directory
  * ### [input] mkdir /mnt/boot ###
  * ### [input] mkdir /mnt/home ###
  * ### [input] mount /dev/sda1 /mnt/boot ###
    + boot directory
  * ### [input] mount /dev/sda4 /mnt/home ###
    + users home directories (~/ = /home/username/)

## Part II: Install operating system components ##

1. Update mirrorlist to the fastest repositories near you
  * ### [input] cat /etc/pacman.d/mirrorlist
  * ### [input] cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
  * ### [input] cat /etc/pacman.d/mirrorlist.backup
  * ### [input] sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
    + -i (Save changes in the same file)
    + s/"Text2Find"/"Text2ReplaceWith"
    + /etc/pacman.d/mirrorlist.backup (file to be searched)
  * ### [input] rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
    + -n # (number of mirrors to select)
    + /etc/pacman.d/mirrorlist (file to be searched)
    + Place only the 6 (in this case) fastest repositories in the mirrorlist

2. Download core and developer components of Arch Linux
  * ### [input] pacstrap -i /mnt base base-devel
    + -i (Avoid auto-confirmation of package selection)
    + /mnt (location of root directory to hold arch installation)
    + base (the package that holds the core of Arch Linux)
    + base-devel (developer package that holds stuff like gcc, automake, and grep)

3. Automate the mounting of partitions with the fstab (file system table) file
  * ### [input] genfstab -U /mnt >> /mnt/etc/fstab
    + -U (use UUID's as source identifiers)
    + /mnt (root directory)
    + '>>' (take the output of genfstab and appends it to the fstab file, creates fstab file if it does not exist)
    + /mnt/etc/fstab (path/name of the file that will be appended)

4. Stop using root from virtual CDRom iso and start using /mnt for the system root
  * ### [input] arch-chroot /mnt
    + Change the root to /mnt

5. Choose text langue formats
  * ### [input] nano /etc/locale.gen
    + uncomment #en_US.UTF-8 UTF-8
      - Should look like this:
        * en_US.UTF-8 UTF-8
      - Sets the langue (region of the world) for text formats

6. Generate the locale information based on the locale.gen file
  * ### [input] locale-gen
  * ### [input] echo LANG=en_US.UTF-8 > /etc/locale.conf
    + echo
      - writes out whatever is typed before the ">"
      -  (redirects the output of echo to the file locale.conf)
  * ### [input] export LANG=en_US.UTF-8
    + export (bash shell command that loads variables so that they can be inherited by any newly forked child processes)
    + LANG (environmental variable to be loaded)
    + en_US.UTF-8 (info to be stored in the environmental variable)

7. Setup the time information
  * ### [input] ls /usr/share/zoneinfo
    + identify your timezone
  * ### [input] ls /usr/share/zoneinfo/America
    + If America is not your country, then replace it with your country
    + Identify the city /region that is in your timezone
  * ### [input] ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
    + ln -s (creates a symbolic link [shortcut] to the New_York timezone and stores it in localtime)
  * ### [input] hwclock --systohc --utc
    + hwclock (hardware clock)
    + --systohc (synchronizes the hardware clock to the system clock)
    + --utc (coordinated universal time)

8. Setup host
  * ### [input] echo computername > /etc/hostname

9. Modify the package manager (pacman) configuration file
  * ### [input] nano /etc/pacman.conf
    + **Step 1:** Enable the multilib repository
      - The multilib repository is an official repository which allows the user to run and build 32-bit applications on 64-bit installations of Arch Linux.
      - uncomment: #[multilib]
        * #### **[delete this]** '#'
        * should now look like this:
          + [multilib]
      - uncomment #Include = /etc/pacman.d/mirrorlist
        * #### **[delete this]** '#'
        * should now look like this:
          + Include = /etc/pacman.d/mirrorlist
        * pacman (when updating) will now load and update the multilib [multi-library] which allows for 32 bit programs
      - **Step 2:** Enabke the Arch User Repository (AUR)
        * at the bottom of the file add:
        * #### **[type this]** [archlinuxfr]
        * #### **[type this]** SigLevel = Never
        * #### **[type this]** Server = http://repo.archlinux.fr/$arch
        * Not officially supported by arch Linux but contains useful programs created/compiled by the users. Considered one of the benefits of using Arch Linux
        * More info in the Arch Linux Wiki
        * This package is managed by yaourt instead of pacman once it is added
        * Save (left control button plus o) and exit (left control button plus x)
  * ### [type this] pacman -Syu
    + -S (sync)
    + y and u are Sync options
      - y (downloads a fresh copy of the master package database from the server)
      - u (upgrades all packages that are out of date)
  * ### [type this] pacman -S yaourt

10. Set root password
    * ### [type this] passwd
      + Enter new password hit enter and repeat
      + This password will be used to lock away the operating system to help prevent accidental/malicious modifications/deletions to your operating system

11. Create a user account
    * Add a user
      + ### [type this] useradd -m -g users -G wheel,root,adm -s /bin/bash bruce.lee
        - useradd - used for adding user accounts
        - -m (creates a home directory for the user)
        - -g (sets the initial group that the user belongs to. in this case "users")
        - -G (sets the supplementary groups that the user belongs to)
          * wheel (short for Big Wheel, is the administration group that gets special privileges like super user access)
        - -s /bin/bash (set the users shell to bash)
        - bruce.lee (username, replace this with the name you want the user to have)
    * Set user password
      + ### [type this] passwd bruce.lee
        - bruce.lee (replace with the appropriate username)
      + Enter new password hit enter and repeat
    * Setup sudo access - setup which user groups are allowed to use sudo and how they have to access it
      + ### [type this] EDITOR=nano visudo
        - Special command to gain access to the file that controls how sudo operates
        - Uncomment: #%wheel ALL=(ALL) ALL
        - #### [delete this] '#'
        - Then add the following line after the %wheel line
        - #### [type this] Defaults rootpw
        - Save (left control button plus o) and exit (left control button plus x)

12. Add tab completion to bash shell
    * ### [type this] pacman -S bash-completion

13. Install bootloader
    * ### [type this]  mount -t efivarfs efivarfs /sys/firmware/efi/efivars
      + Makes sure efivars are available
      + If it says efivarfs is already mounted, then that is fine
    * ### [type this] bootctl install
      + Installs everything needed for the bootloader
    * ### [type this] blkid -s PARTUUID -o value /dev/sda3 > /boot/loader/entries/arch.conf
      + Edit the arch.conf file to give root partition when arch boots up (add root partition UUID to arch.conf)
      + Find UUID of root drive because the system needs to know this upon booting
    * ### [type this] nano /boot/loader/entries/arch.conf
      + Above the UUID add the following lines
        - #### [type this] title Arch Linux
        - #### [type this] linux /vmlinuz-linux
        - #### [type this] initrd /initramfs-linux.img
      + Edit the line containing the UUID as follows
        - #### [type this] options root=PARTUUID=48eb0f0-1549-455e-9818-232f73ffde37 rw
          * 48eb0f0-1549-455e-9818-232f73ffde37 should be the UUID of the root partition already found in the file
        - Save (left control button plus o) and exit (left control button plus x)
    * If you have an intel processor then do the following (add intel micro-code)
      + ### [type this] pacman -S intel-ucode
      + ### [type this] nano /boot/loader/entries/arch.conf
        - add the following line between the  linux /vmlinuz-linux line and the initrd /initramfs-linux.img line
        - #### [type this] initrd /intel-ucode.img
        - Save (left control button plus o) and exit (left control button plus x)

14. Disconnecting, logging out and logging back in
    * Disconnect from the setup state (remove training wheels)
      + ### [type this] exit
      + ### [type this] umount -R /mnt
    * Go to virtualbox menu and choose file/close/power off machine
      + Click ok
    * Go to setting/storage in virtualbox manager and remove the iso from the optical drive (CD)
    * Boot back into Arch Linux from virutalbox manager
    * If everything went well you should get a login screen (login and congratulations)
      + At this point Arch Linux is only accessible via the command line (terminal)

** The core operating system is now installed (no graphical user interfaces have been added yet) **

15. Autologin to console
    * ### [type this] sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/
      + creates all the subdirectories if they are missing
    * ### [type this] sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf
      + #### [type this] [Service]
      + #### [type this] ExecStart=
      + #### [type this] ExecStart=-/usr/bin/agetty --autologin **username** --noclear %I $TERM
        - replace **username** with the username that you want to autologin with
          * **Example:** ExecStart=-/usr/bin/agetty --autologin Jeff --noclear %I $TERM
      + save (left control button plus o) and exit (left control button plus x)

    * ### [type this] systemctl edit getty@tty1
      + You could have used this command instead of nano
      + Save (left control button plus o) and exit (left control button plus x)
      + This command will reload the getty@tty1 unit so that your changes are now active

## Part III: Install graphical interface for Arch Linux (i.e. window manager, desktop environment)
1. Install internet connection
  * ### [type this] ping -c 3 www.google.com
    + Output should be Name or server not known
    + The core Arch Linux is installed but not connected to the internet
  * ### [type this] ip link
    + Find the name of the device on the line that contains <BROADCAST,MULTICAST>
    + Example: enp0s3
  * ### [type this] sudo systemctl enable dhcpcd@enp0s3.service
  * ### [type this] sudo reboot
  * Log back in
  * ### [type this] ping -c 3 www.google.com
    + Should now have internet access
2. Install xserver
  * provides a basic graphical interface to Arch Linux
  * Food analogy: think of xserver as providing all the ingredients for a cake and can even bake a simple cake.  If you want a more fancy/elaborate cake then you have to download a desktop environment to convert the xserver ingredients into the type of cake you want
  * ### [type this] sudo pacman -S xorg-server xorg-server-utils xorg-xinit mesa
    + Choose your libgl provider
      - #### Choose the default 1) mesa-libgl
      - ### WARNING: Do **not** choose nvidia drivers when installing as a guest in virtualbox
        * **(On a non-virtual machine install I would have chosen 4 the nvidia-libgl)**
    + #### Choose your xf86-input-driver (I choose 2 the xf86-input-libinput)
  * ### [type this] sudo pacman -S xorg-twm xorg-xclock xterm
    + Adding some graphical components that can be rendered
    + ### [type this] pacman -S linux-headers
      - Helps with compatibility issues (especially since we are running inside of virtualbox
3. Add virtualbox guest additions
  * #### Go to virtualbox menu and select Device/"Insert Guest Additions CD image..."
  * ### [type this] sudo mkdir /mnt/cdrom
  * ### [type this] sudo mount /dev/cdrom /mnt/cdrom
  * ### [type this] sudo /mnt/cdrom/VBoxLinuxAdditions.run
  * ### [type this] pacman -Syu
  * ### [type this] startx
