Archiso
=======

Archiso is a small set of bash scripts capable of building fully
functional Arch Linux based live CD and USB images. It is the tool used
to generate the official CD/USB images, however it is a very generic
tool and it could potentially be used to generate anything from rescue
systems, install disks, to special interest live CD/DVD/USB systems, and
who knows what else. Simply put, if it involves Arch on a shiny coaster,
it can do it. The heart and soul of Archiso is mkarchiso. All of its
options are documented in its usage output, so its direct usage won't be
covered here. Instead, this wiki article will act as a guide for rolling
your own live media in no time!

Contents
--------

-   1 Setup
-   2 Configure our live medium
    -   2.1 Installing packages
        -   2.1.1 Custom local repository
        -   2.1.2 Avoid installation of packages belonging to base group
    -   2.2 Adding a user
    -   2.3 Adding files to image
    -   2.4 aitab
    -   2.5 Boot Loader
    -   2.6 Login manager
    -   2.7 Changing Automatic Login
-   3 Build the ISO
-   4 Using the ISO
    -   4.1 CD
    -   4.2 USB
    -   4.3 grub4dos
    -   4.4 Installation
-   5 See also

Setup
-----

Note:It is recommended to act as root in all the following steps. If
not, it is very likely to have problems with false permissions later.

Before we begin, we need to install archiso from the official
repositories. Alternatively, archiso-git can be found in the AUR.

Create a directory to work within, this is where all the modifications
to the live image will take place: ~/archlive should do fine.

    $ mkdir ~/archlive

The archiso scripts that were installed to the host system earlier now
need to be copied over into the newly created directory you will be
working within.

Archiso comes with two "profiles": releng and baseline.

If you wish to create a fully customised live version of Arch Linux,
pre-installed with all your favourite programs and configurations, use
releng.

If you just want to create the most basic live medium, with no
pre-installed packages and a minimalistic configuration, use baseline.

So, depending on your needs, execute the following, replacing 'PROFILE'
with either releng or baseline.

    # cp -r /usr/share/archiso/configs/PROFILE/ ~/archlive

If you are using the releng profile to make a fully customised image,
then you can proceed onto #Configure our live medium.

If you are using the baseline profile to create a bare image, then you
won't be needing to do any customisations and can proceed onto #Build
the ISO.

Configure our live medium
-------------------------

This section details configuring the image you will be creating,
allowing you to define the packages and configurations you want your
live image to contain.

Change into the directory we created earlier (~/archlive/releng/ if you
have been following this guide), you will see a number of files and
directories; we are only concerned with a few of these, mainly:
packages.* - this is where you list, line by line, the packages you want
to have installed, and the root-image directory - this directory acts as
an overlay and it is where you make all the customisations.

Generally, every admininistrative task that you would normally do after
a fresh install except for package installation can be scripted into
~/archlive/releng/root-image/root/customize-root-image.sh. It has to be
written from the perpective of the new environment, so / in the script
means the root of the live-iso which is created.

> Installing packages

You will want to create a list of packages you want installed on your
live CD system. A file full of package names, one-per-line, is the
format for this. This is great for special interest live CDs, just
specify packages you want in packages.both and bake the image. The
packages.i686 and packages.x86_64 files allow you to install software on
just 32bit or 64bit, respectively.

I recommend installing "rsync" if you wish to install the system later
on with no internet connection or skipping downloading it all over
again. (#Installation)

Custom local repository

  ------------------------ ------------------------ ------------------------
  [Tango-two-arrows.png]   This article or section  [Tango-two-arrows.png]
                           is a candidate for       
                           merging with Pacman      
                           Tips#Custom local        
                           repository.              
                           Notes: Move the general  
                           information (e.g. repo   
                           tree) into the main      
                           article. (Discuss)       
  ------------------------ ------------------------ ------------------------

You can also create a custom local repository for the purpose of
preparing custom packages or packages from AUR/ABS. When doing so with
packages for both architectures, you should follow a certain directory
order to not run into problems.

For instance:

-   ~/customrepo
    -   ~/customrepo/x86_64
        -   ~/customrepo/x86_64/foo-x86_64.pkg.tar.xz
        -   ~/customrepo/x86_64/customrepo.db.tar.gz
        -   ~/customrepo/x86_64/customrepo.db (symlink created by
            repo-add)
    -   ~/customrepo/i686
        -   ~/customrepo/i686/foo-i686.pkg.tar.xz
        -   ~/customrepo/i686/customrepo.db.tar.gz
        -   ~/customrepo/i686/customrepo.db (symlink created by
            repo-add)

You can then add your repository by putting the following into
~/archlive/releng/pacman.conf, above the other repository entries (for
top priority):

    # custom repository
    [customrepo]
    SigLevel = Optional TrustAll
    Server = file:///home/user/customrepo/$arch

So, the build scripts just look for the appropriate packages.

If this is not the case you will be running into error messages similar
to this:

    error: failed to prepare transaction (package architecture is not valid)
    :: package foo-i686 does not have a valid architecture

Avoid installation of packages belonging to base group

By, default /usr/bin/mkarchiso, a script which is used by
~/archlive/releng/build.sh, calls one of the arch-install-scripts named
pacstrap without the -i flag, which causes Pacman to not wait for user
input during the installation process.

When blacklisting base group packages by adding them to the IgnorePkg
line in ~/archlive/releng/pacman.conf, Pacman asks if they still should
be installed, which means they will when user input is bypassed. To get
rid of these packages there are several options:

-   Dirty: Add the -i flag to each line calling pacstrap in
    /usr/bin/mkarchiso.

-   Clean: Create a copy of /usr/bin/mkarchiso in which you add the flag
    and adapt ~/archlive/releng/build.sh so that it calls the modifed
    version of the mkarchiso script.

-   Advanced: Create a function for ~/archlive/releng/build.sh which
    explicitly removes the packages after the base installation. This
    would leave you the comfort of not having to type enter so much
    during the installation process.

> Adding a user

User management can be handled the same way as during the normal
installation process, except you put your commands scripted into
~/archlive/releng/root-image/root/customize-root-image.sh. For further
information have a look at User management.

> Adding files to image

Note:You must be root to do this, do not change the ownership of any of
the files you copy over, everything within the root-image directory must
be root owned. Proper ownerships will be sorted out shortly.

The root-image directory acts as an overlay, think of it as root
directory '/' on your current system, so any files you place within this
directory will be copied over on boot-up.

So if you have a set of iptables scripts on your current system you want
to be used on you live image, copy them over as such:

    # cp -r /etc/iptables ~/archlive/releng/root-image/etc

Placing files in the users home directory is a little different. Do not
place them within root-image/home, but instead create a skel directory
within root-image/ and place them there. We will then add the relevant
commands to the customize_root_image.sh which we are going to use to
copy them over on boot and sort out the permissions.

First, create the skel directory; making sure you are within
~/archlive/releng/root-image/etc directory (if this is where you are
working from):

    # cd ~/archlive/releng/root-image/etc && mkdir skel

Now copy the 'home' files to the skel directory, again doing everything
as root! e.g for .bashrc.

    # cp ~/.bashrc ~/archlive/releng/root-image/etc/skel/

When ~/archlive/releng/root-image/root/customize-root-image.sh is
executed and a new user is created, the files from the skel directory
will automatically be copied over to the new home folder, permissions
set right.

> aitab

The default file should work fine, so you should not need to touch it.

The aitab file holds information about the filesystems images that must
be created by mkarchiso and mounted at initramfs stage from the archiso
hook. It consists of some fields which define the behaviour of images.

    # <img>         <mnt>                 <arch>   <sfs_comp>  <fs_type>  <fs_size>

 <img>
    Image name without extension (.fs .fs.sfs .sfs).
 <mnt>
    Mount point.
 <arch>
    Architecture { i686 | x86_64 | any }.
 <sfs_comp>
    SquashFS compression type { gzip | lzo | xz }.
 <fs_type>
    Set the filesystem type of the image { ext4 | ext3 | ext2 | xfs |
    btrfs }. A special value of "none" denotes no usage of a filesystem.
    In that case all files are pushed directly to SquashFS filesystem.
 <fs_size>
    An absolute value of file system image size in MiB (example: 100,
    1000, 4096, etc) A relative value of file system free space [in
    percent] {1%..99%} (example 50%, 10%, 7%). This is an estimation,
    and calculated in a simple way. Space used + 10% (estimated for
    metadata overhead) + desired %

Note:Some combinations are invalid. Example both sfs_comp and fs_type
are set to none

> Boot Loader

The default file should work fine, so you should not need to touch it.

Due to the modular nature of isolinux, you are able to use lots of
addons since all *.c32 files are copied and available to you. Take a
look at the official syslinux site and the archiso git repo. Using said
addons, it is possible to make visually attractive and complex menus.
See here.

> Login manager

Starting X at boot time was done by modifying inittab on sysvinit
systems. On a systemd based system things are handled by enabling your
login manager's service. If you know which .service file needs a
softlink: Great. If not, you can easily find out in case you're using
the same program on the system you build your iso on. Just use

    # systemctl disable nameofyourloginmanager

to temporarily turn it off. Next type the same command again and replace
"disable" with "enable" to activate it again. Systemctl prints
information about softlink it creates. Now change to
~/archiso/releng/root-image/etc/systemd/system and create the same
softlink there.

An example (make sure you're either in
~/archiso/releng/root-image/etc/systemd/system or add it to the
command):

    # ln -s /usr/lib/systemd/system/lxdm.service display-manager.service

This will enable LXDM at system start on your live system.

> Changing Automatic Login

The configuration for getty's automatic login is located under
root-image/etc/systemd/system/getty@tty1.service.d/autologin.conf.

You can modify this file to change the auto login user:

    [Service]
    ExecStart=
    ExecStart=-/sbin/agetty --autologin isouser --noclear %I 38400 linux

Or remove it altogether to disable auto login.

Build the ISO
-------------

Now you are ready to turn your files into the .iso which you can then
burn to CD or USB: Inside the directory you are working with, either
~/archlive/releng, or ~/archlive/baseline, execute:

    # ./build.sh -v

The script will now download and install the packages you specified to
work/*/root-image, create the kernel and init images, apply your
customizations and finally build the iso into out/.

Using the ISO
-------------

> CD

Just burn the iso to a cd. You can follow CD Burning as you wish.

> USB

See USB Flash Installation Media.

> grub4dos

Grub4dos is a utility that can be used to create multiboot usbs, able to
boot multiple linux distros from the same usb stick.

To boot the generated system on a usb with grub4dos already installed,
loop mount the ISO and copy the entire /arch directory to the root of
the usb. Then edit the menu.lst file from the grub4dos (it must be on
the usb root) and add these lines:

    title Archlinux x86_64
    kernel /arch/boot/x86_64/vmlinuz archisolabel=<your usb label>
    initrd /arch/boot/x86_64/archiso.img

Change the x86_64 part as necessary and put your real usb label there.

> Installation

Boot the created CD/DVD/USB. If you wish to install the Archiso you
created -as it is-, there are several ways to do this, but either way
we're following the Beginners' guide mostly.

If you don't have an internet connection on that PC, or if you don't
want to download every packages you want again, follow the guide, and
when you get to Beginners' guide#Install_the_base_system, instead of
downloading, use this: Full System Backup with rsync. (more info here:
Talk:Archiso)

You can also try: Archboot, GUI installer.

See also
--------

-   Archiso project page
-   Archiso as pxe server
-   Creating a custom Arch Linux live USB tutorial
-   A live DJ distribution powered by ArchLinux and built with Archiso

Retrieved from
"https://wiki.archlinux.org/index.php?title=Archiso&oldid=305404"

Category:

-   Live Arch systems

-   This page was last modified on 18 March 2014, at 06:01.
-   Content is available under GNU Free Documentation License 1.3 or
    later unless otherwise noted.
-   Privacy policy
-   About ArchWiki
-   Disclaimers
