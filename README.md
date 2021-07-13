# arch-linux-virtualbox-installation
This is a step by step guide to install Arch Linux in a VirtualBox environment, using this profile's dotfiles and post-installation script. I use an EFI installation, and this is based in [Arch Linux Installation Guide](https://wiki.archlinux.org/title/installation_guide). 

# Configuring VirtualBox
1. Create a new virtual machine, and define its parameters. For Arch Linux, recommended specs are:
    - 1024 MB RAM.
    - 8 GB of hard disk space.
    - For a more detailed tutorial, access this [link](https://gist.github.com/dannycastonguay/46e8cca629d67105a2db#mount-arch-linux-on-virtualbox)
2. Don't forget to set "enable EFI" on "System" tab.
3. Run machine.

# Run installation
4. When system is ready, change layout to your preference with `loadkeys {layout}`. I use `loadkeys la-latin1`.
5. Enable Ethernet with `ip link`. Check if it works with `ping google.com`.
6. Set system clock with `timedatectl set-ntp true`.
7. After this, you have to set disk partitions. Personally, i prefer to use `cfdisk` utility.
  - Run `cfdisk`.
  - Select `gpt`.
  - Create a partition with `New` option. Assign it 1G.
  - Set its type (`Type` option) to `EFI System`. 
  - Create another partition, assign it the space you want for root partition.
  > I prefer to create a swap partition. Many users not do this, but [here](https://serverfault.com/questions/351093/should-linux-vms-in-vmware-esx-have-a-swap-partition/351176#351176) you can read why I'll do it.
  - Create a last partition, to use a swap. It's recommended to use at least double of RAM assigned for this one.
  - Finally, select `Write` option.
8. After that, is needed to format them. Considering like /dev/sda1 the EFI partition, /dev/sda2 the root partition, and /dev/sda3 the swap partition, run:
  - `mkfs.ext4 /dev/sda2`.
  - `mkfs.fat -F32 /dev/sda1`.
  - `mkswap /dev/sda3`.
  - `swapon /dev/sda3`.
9. We'll mount them:
  - `mount /dev/sda2 /mnt`.
  - `mkdir /mnt/efi`.
  - `mount /dev/sda1 /mnt/efi`.
10. Finally, we must have root mounted in mnt folder, and the EFI partition reachable through /mnt/efi. Continue installing kernel. Run `pacstrap /mnt base linux linux-firmware base-devel nano`. I have also added some nano text editor.
11. Generate filesystem file `genfstab -U /mnt >> /mnt/etc/fstab`.
12. Change root into the new system `arch-chroot /mnt`.
13. Set the time zone `ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime`. "America/Argentina/Buenos_Aires" is which I use. You can list all file in `/usr/share/zoneinfo/` directory to see zones available.
14. Run hwclock(8) to generate /etc/adjtime `hwclock --systohc`.
15. Edit `/etc/locale.gen` and uncomment es_ES.UTF-8 UTF-8 or which corresponds to you.
16. Run `locale-gen`.
17. Edit `/etc/locale.conf` and add `LANG=es_ES.UTF-8` or which you uncomment before.
18. Edit `/etc/vconsole.conf` and add `KEYMAP=la-latin1` or which layout you prefer.
19. Edit `/etc/hostname` and write the PC name you want it to have.
20. Edit `/etc/hosts` and write this replacing "pcname" with the name your write on step 19:
```
127.0.0.1	  localhost
::1		      localhost
127.0.1.1	  pcname.localdomain	pcname
```
21. Run `passwd` and create a root password.
22. `useradd -m "yourname"`.
23. `passwd "yourname`.
24. `usermod -aG wheel,audio,video,optical,storage yourname`.
25. We'll install and configure sudo:
  - `pacman -S sudo`.
  - Run `EDITOR=nano visudo` and uncomment line which says `%wheel ALL=(ALL) ALL`.
26. After that, its needed to configure grub:
  - Run `pacman -S grub efibootmgr`.
  - `grub-install --target=x86_64-efi --efi-directory=/efi/ --bootloader-id=Arch`.
  - `grub-mkconfig -o /boot/grub/grub.cfg`.
27. Finally, before rebooting, remainds to enable network:
  - `pacman -S networkmanager`.
  - `systemctl enable NetworkManager`.
28. Exit (`exit`) and shutdown (`shutdown now`).
