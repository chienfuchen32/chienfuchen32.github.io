---
title: "Disk encryption: LUKS ( Linux Unified Key Setup) with Tang"
date: 2022-03-17T10:16:41+08:00
draft: true
tags: ["Linux Unified Key Setup", "encryption"]
---
## Key component
* `tang server` is responsible for helping dracut to decrypt the target disk. It won't store any client key.
* `encrypted server` is required to use `clevis`, `dracut`. It provide a easier way that integrate with tang server to decrypt LUKS disk.
## Network topology
```txt
  |-------------------------|                      |------------|
  |LUKS encrypted server    |-- disk decryption -->|tang server |
  |(clevis, dracut) [env]   |<----- response ------|[tang]      |
  |-------------------------|                      |____________|
```

## Tang server
* software installation via apt on x86x64 Ubuntu 20.04
```bash
adm@tang:~$ sudo apt-get install tang -y
## check version
adm@tang:~$ apt list --installed | grep tang
tang/focal,now 7-1build1 amd64 [installed]
## Enable the tangd service
adm@tang:~$ sudo systemctl enable tangd.socket
```
* Create an override file with 7500 to prevent port conflict
```bash
adm@tang:~$ sudo systemctl edit tangd.socket
```
```yaml
# tangd.socket
[Socket]
ListenStream=
ListenStream=7500 
```
```bash
adm@tang:~$ sudo systemctl daemon-reload
## Check that your configuration is working:
adm@tang:~$ sudo systemctl show tangd.socket -p Listen
Listen=[::]:7500 (Stream)
## Start the tangd service
adm@tang:~$ sudo systemctl restart tangd.socket
adm@tang:~$ sudo systemctl status tangd.socket
● tangd.socket - Tang Server socket
     Loaded: loaded (/lib/systemd/system/tangd.socket; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/tangd.socket.d
             └─override.conf
     Active: active (listening) since Mon 2022-03-14 00:54:03 UTC; 1h 25min ago
   Triggers: ● tangd@0.service
     Listen: [::]:7500 (Stream)
   Accepted: 0; Connected: 0;
      Tasks: 0 (limit: 984)
     Memory: 44.0K
     CGroup: /system.slice/tangd.socket

Mar 14 00:54:03 d systemd[1]: Listening on Tang Server socket.
```

## encrypted server: try clevis, luks to bind with tang
Assume that tang server is now running on `192.168.100.10:7500`, we need to run `clevis` to bind local encrypted disk (`/dev/md0` in this case) with tang.
* software installation via apt on x86x64 Ubuntu 20.04
```bash
adm@enc:~$ sudo apt-get install clevis clevis-luks clevis-dracut -y
## check version
adm@enc:~$ apt list --installed | grep clevis
clevis-dracut/focal-updates,now 12-1ubuntu2.3 all [installed]
clevis-luks/focal-updates,now 12-1ubuntu2.3 all [installed]
clevis-systemd/focal-updates,now 12-1ubuntu2.3 amd64 [installed,automatic]
clevis/focal-updates,now 12-1ubuntu2.3 amd64 [installed]
adm@enc:~$ apt list --installed | grep dracut
dracut-core/focal,now 048+80-2 amd64 [installed,automatic]
dracut-network/focal,now 048+80-2 all [installed]
```
* check network configuration and check if tang server available
```bash
adm@enc:~$ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether c4:00:ad:94:d8:76 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.61/24 brd 192.168.100.255 scope global enp1s0
       valid_lft forever preferred_lft forever

adm@enc:~$ curl -sfg http://192.168.100.10:7500/adv -o adv.jws
```
* check encrypted disk, partition
```bash
adm@enc:~$ sudo lsblk
NAME              MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
loop0               7:0    0    55M  1 loop  /snap/core18/1880
loop1               7:1    0  55.5M  1 loop  /snap/core18/2344
loop2               7:2    0  67.9M  1 loop  /snap/lxd/22526
loop3               7:3    0  67.8M  1 loop  /snap/lxd/22753
loop4               7:4    0  61.9M  1 loop  /snap/core20/1376
loop5               7:5    0  43.6M  1 loop  /snap/snapd/15177
sda                 8:0    0 953.9G  0 disk
├─sda1              8:1    0   512M  0 part  /boot/efi
├─sda2              8:2    0     1G  0 part  /boot
├─sda3              8:3    0 945.9G  0 part
│ └─md0             9:0    0 945.8G  0 raid1
│   └─dm_crypt-1  253:0    0 945.7G  0 crypt
│     └─vg0-lv--0 253:1    0 945.7G  0 lvm   /
└─sda4              8:4    0   6.5G  0 part
sdb                 8:16   0 953.9G  0 disk
├─sdb1              8:17   0     8G  0 part  [SWAP]
└─sdb2              8:18   0 945.9G  0 part
  └─md0             9:0    0 945.8G  0 raid1
    └─dm_crypt-1  253:0    0 945.7G  0 crypt
      └─vg0-lv--0 253:1    0 945.7G  0 lvm   /

# check LUKS on encrpyt partition, total 8 keyslots
$ sudo cryptsetup luksDump /dev/md0
LUKS header information
Version:        2
Epoch:          5
Metadata area:  16384 [bytes]
Keyslots area:  16744448 [bytes]
UUID:           b3abd9e0-b3be-4dab-9928-e3f29c749b93
Label:          (no label)
Subsystem:      (no subsystem)
Flags:          (no flags)

Data segments:
  0: crypt
        offset: 16777216 [bytes]
        length: (whole device)
        cipher: aes-xts-plain64
        sector: 512 [bytes]

Keyslots:
  0: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        Cipher key: 512 bits
        PBKDF:      argon2i
        Time cost:  5
        Memory:     1048576
        Threads:    4
        Salt:       44 01 7e f7 be 90 fa d9 fe 24 8f b7 95 6e 28 91
                    53 7c 47 5d 6f 0a cf 34 18 c4 ee 13 d6 91 28 e5
        AF stripes: 4000
        AF hash:    sha256
        Area offset:32768 [bytes]
        Area length:258048 [bytes]
        Digest ID:  
Tokens:
  0: clevis
        Keyslot:  1
Digests:
  0: pbkdf2
        Hash:       sha256
        Iterations: 100054
        Salt:       8b 7d 45 cf 2b 25 07 bd 7d 48 47 04 1e 70 d0 41
                    46 7c fa 1f ec ed 12 2a c5 36 fc 24 56 b8 6c 7e
        Digest:     06 89 5e 90 07 1c 83 eb 54 0a 14 d5 ad f2 f6 50
                    5f 13 00 58 62 ee 11 62 db 7c e0 1a 45 c5 fa 09

adm@enc:~$ sudo clevis luks bind -d /dev/md0 tang '{"url":"http://192.168.100.10:7500"}'
The advertisement contains the following signing keys:

ADIMDQW1mFAAoGFlQJIG9SioKDW

Do you wish to trust these keys? [ynYN] y
Enter existing LUKS password:
...
# check current state
adm@enc:~$ sudo clevis luks list -d /dev/md0
1: tang '{"url":"http://192.168.100.100:7500"}'

# try dracut
adm@enc:~$ dracut -fv --regenerate-all --kernel-cmdline " ip=192.168.100.61:192.168.100.0/24::255.255.255.0:enc:eth0:off " --kernel-cmdline " rd.net.timeout.carrier=3153600 "
```

## generate a initramfs for bootloader
Make grub load our image that contains clevis luks disk encryption script. The image is built by `dracut`. Our purpose is no matter when the `grub.cfg` will be updated, the main menuentry should be our dracut environment (`linux`, `initrd`)
* check grub version
```bash
adm@enc:~$ sudo apt list --installed | grep grub

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

grub-common/now 2.04-1ubuntu26.2 amd64 [installed,upgradable to: 2.04-1ubuntu26.13]
grub-efi-amd64-bin/focal-updates,focal-security,now 2.04-1ubuntu44.2 amd64 [installed,automatic]
grub-efi-amd64-signed/focal-updates,focal-security,now 1.167.2+2.04-1ubuntu44.2 amd64 [installed]
grub-efi-amd64/focal-updates,focal-security,now 2.04-1ubuntu44.2 amd64 [installed]
grub2-common/now 2.04-1ubuntu26.2 amd64 [installed,upgradable to: 2.04-1ubuntu26.13]
```

1. You can try to ping gateway IP and tang server's IP, if gateway IP is not available, don't fill in that field.
```
adm@enc:~$ ping 192.168.100.10
PING 192.168.100.10 (192.168.100.100) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=64 time=10.9 ms
64 bytes from 192.168.100.10: icmp_seq=2 ttl=64 time=1.00 ms
...
```
To enumerate all possible environment to generate new grub.cfg, please check our modified `10_linux file`
Add new dracut script with customized argument for your environment to grub folder, please check detail in Misc `dracut cmdline argument` part and edit files `/etc/dracut.conf.d/` if needed.
```bash
adm@enc:~$ sudo vi dracut-config/etc-dracut.conf.d/12-kernel-cmdline.conf
```
```conf
kernel_cmdline+=" ip=192.168.100.61:192.168.100.0/24::255.255.255.0:enc:eth0:off "
kernel_cmdline+="rd.net.timeout.carrier=3153600 "
```

```bash
# copy to grub config path
adm@enc:~$ sudo cp dracut-config/etc-dracut.conf.d/12-kernel-cmdline.conf /etc/dracut.conf.d/12-kernel-cmdline.conf
# backup first
adm@enc:~$ sudo cp /etc/grub.d/10_linux /etc/grub.d/.10_linux.bk
# add our include running dracut script and set initramfs as default menuentry
adm@enc:~$ sudo cp etc-grub.d/10_linux /etc/grub.d/10_linux
# update grub.cfg manually
adm@enc:~$ sudo update-grub
# you might see lot of log including dracut, correct them if there's anything wrong.

Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
dracut: Executing: /usr/bin/dracut --kver=5.4.0-104-generic -fv --kernel-cmdline " ip=192.168.100.61:192.168.100.0/24::255.255.255.0:enc:eth0:off "
dracut: dracut module 'bootchart' will not be installed, because command '/sbin/bootchartd' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'clevis-decrypt-tpm2' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_createprimary' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_unseal' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_load' could not be found!
dracut: dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut: dracut module 'biosdevname' will not be installed, because command 'biosdevname' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'clevis-decrypt-tpm2' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_createprimary' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_unseal' could not be found!
dracut: dracut module 'clevis-pin-tpm2' will not be installed, because command 'tpm2_load' could not be found!
dracut: dracut module 'cifs' will not be installed, because command 'mount.cifs' could not be found!
dracut: *** Including module: bash ***
dracut: *** Including module: dash ***
...
dracut: *** Creating initramfs image file '/boot/initramfs-5.4.0-42-generic.img" done ***
Found linux image: /boot/vmlinuz-5.4.0-104-generic
Found initrd image: /boot/initramfs-5.4.0-104-generic.img
Found initrd image: /boot/initramfs-5.4.0-104-generic.img
Found initrd image: /boot/initrd.img-5.4.0-104-generic
Found initrd image: /boot/initrd.img-5.4.0-104-generic
Found linux image: /boot/vmlinuz-5.4.0-42-generic
Found initrd image: /boot/initramfs-5.4.0-42-generic.img
Found initrd image: /boot/initramfs-5.4.0-42-generic.img
Found initrd image: /boot/initrd.img-5.4.0-42-generic
Found initrd image: /boot/initrd.img-5.4.0-42-generic
Adding boot menu entry for UEFI Firmware Settings
done
```
* check if initramfs created by dracut
```bash
adm@enc:~$ ls -lh /boot
total 250M
-rw-r--r-- 1 root root 233K Feb  3 18:16 config-5.4.0-100-generic
-rw-r--r-- 1 root root 233K Mar  2 17:10 config-5.4.0-104-generic
drwxr-xr-x 3 root root 4.0K Jan  1  1970 efi
drwxr-xr-x 4 root root 4.0K Mar 12 06:33 grub
-rw------- 1 root root  24M Mar 11 06:29 initramfs-5.4.0-100-generic.img
-rw------- 1 root root  24M Mar 11 06:29 initramfs-5.4.0-42-generic.img
lrwxrwxrwx 1 root root   28 Mar 11 07:55 initrd.img -> initrd.img-5.4.0-104-generic
-rw-r--r-- 1 root root  84M Feb 26 06:34 initrd.img-5.4.0-100-generic
-rw-r--r-- 1 root root  84M Mar 11 07:56 initrd.img-5.4.0-104-generic
lrwxrwxrwx 1 root root   28 Mar 11 07:55 initrd.img.old -> initrd.img-5.4.0-100-generic
drwx------ 2 root root  16K Feb 25 06:13 lost+found
-rw------- 1 root root 4.6M Feb  3 18:16 System.map-5.4.0-100-generic
-rw------- 1 root root 4.6M Mar  2 17:10 System.map-5.4.0-104-generic
lrwxrwxrwx 1 root root   25 Mar 11 07:55 vmlinuz -> vmlinuz-5.4.0-104-generic
-rw------- 1 root root  14M Feb  4 17:04 vmlinuz-5.4.0-100-generic
-rw------- 1 root root  14M Mar  2 18:37 vmlinuz-5.4.0-104-generic
lrwxrwxrwx 1 root root   25 Mar 11 07:55 vmlinuz.old -> vmlinuz-5.4.0-100-generic
```
* check if grub config (`/boot/grub/grub.cfg`) changed and the first one should be direct to our image
```yaml
menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-eec358a5-d009-4f70-9210-424330e9adb4' {
        recordfail
        load_video
        gfxmode $linux_gfx_mode
        insmod gzio
        if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
        insmod part_gpt
        insmod ext2
        set root='hd0,gpt2'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  a5963b85-5235-4b31-9776-5f6251c206f3
        else
          search --no-floppy --fs-uuid --set=root a5963b85-5235-4b31-9776-5f6251c206f3
        fi
        linux   /vmlinuz-5.4.0-104-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro
        initrd  /initramfs-5.4.0-104-generic.img
}
...
```
* you may try clevis-luks boot with disk decryption flow!
```bash
adm@enc:~$ sudo reboot
```

### Enhancement in the future

* Shamir's Secret Sharing (SSS) algorithm with clevis multiple constriant: tpm, tangd, ... etc.
* add tpm to protect tangd key on site controller
* make dracut image name to be fixed

### Misc

* `Tangd` tech, McCallum-Relyea Exchange: client(clevis) need server(tang) to decrypt data, there's no client data stored in server.

* dracut cmdline argument
```
   ip=<client-IP>:[<peer>]:<gateway-IP>:<netmask>:<client_hostname>:<interface>:{none|off|dhcp|on|any|dhcp6|auto6|ibft}[:[<mtu>][:<macaddr>]]
       explicit network configuration. If you want do define a IPv6
       address, put it in brackets (e.g. [2001:DB8::1]). This
       parameter can be specified multiple times.  <peer> is
       optional and is the address of the remote endpoint for
       pointopoint interfaces and it may be followed by a slash and
       a decimal number, encoding the network prefix length.

       <macaddr>
           optionally set <macaddr> on the <interface>. This cannot
           be used in conjunction with the ifname argument for the
           same <interface>.
Ref: https://man7.org/linux/man-pages/man7/dracut.cmdline.7.html
```

### bootloader alternative
* you can still use grub shell, if the `/boot/grub/grub.cfg` is broken.

* bootable Ubuntu USB stick

#### Ref
* luks, clevis, tang
```
https://mdtsai.medium.com/%E8%A7%A3%E9%99%A4-linux-%E5%8A%A0%E5%AF%86%E7%A3%81%E5%8D%80-5b8e597e4c37
https://www.osslab.com.tw/research-luks/
https://www.itread01.com/content/1542818777.html
https://kknews.cc/zh-tw/code/n22kq38.html
https://www.tecmint.com/file-and-disk-encryption-tools-for-linux/
https://www.tecmint.com/disk-encryption-in-linux/
https://help.ubuntu.com/community/Full_Disk_Encryption_Howto_2019
https://wiki.archlinux.org/title/Data-at-rest_encryption
https://docs.microsoft.com/zh-tw/azure/virtual-machines/linux/how-to-configure-lvm-raid-on-crypt
https://linuxsecurity.com/features/top-8-file-and-disk-encryption-tools-for-linux
https://nakedsecurity.sophos.com/2022/01/14/serious-security-linux-full-disk-encryption-bug-fixed-patch-now/
https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/
https://docs.nvidia.com/drive/drive_os_5.1.6.1L/nvvib_docs/index.html#page/DRIVE_OS_Linux_SDK_Development_Guide/Windows%20Systems/security_disk_encryption_lnx.html
https://www.wibu.com/
https://wiki.centos.org/zh-tw/HowTos/EncryptedFilesystem
https://gitlab.com/cryptsetup/cryptsetup/-/wikis/DMCrypt
https://www.digitalocean.com/community/tutorials/how-to-use-dm-crypt-to-create-an-encrypted-volume-on-an-ubuntu-vps
https://www.howtoforge.com/ubuntu_dm_crypt_luks
https://ubuntuqa.com/zh-tw/article/8257.html
https://www.gushiciku.cn/pl/g4f9/zh-tw
https://askubuntu.com/questions/996155/how-do-i-automatically-decrypt-an-encrypted-filesystem-on-the-next-reboot
https://dywang.csie.cyut.edu.tw/dywang/linuxsecurity/node59.html
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/configuring-automated-unlocking-of-encrypted-volumes-using-policy-based-decryption_security-hardening
https://www.youtube.com/watch?v=Dk6ZuydQt9I
https://www.youtube.com/watch?v=h5H6_oxpFA0
https://www.admin-magazine.com/Archive/2018/43/Automatic-data-encryption-and-decryption-with-Clevis-and-Tang
https://www.snia.org/sites/default/files/SDC15_presentations/security/NathanielMcCallum_Network_Bound_Encryption.pdf
https://www.youtube.com/watch?v=p_M0YEE-esA
https://www.cs.purdue.edu/homes/hmaji/teaching/Fall%202018/lectures/19.pdf
https://docs.okd.io/latest/security/network_bound_disk_encryption/nbde-managing-encryption-keys.html
# systemd, init
https://hugh712.gitbooks.io/buildroot/content/system-initialization.html
http://felix-lin.com/linux/init%e6%bc%94%e5%8c%96%e6%ad%b7%e7%a8%8b-%e8%bd%89%e8%b2%bc-%e6%b7%ba%e6%9e%90-linux-%e5%88%9d%e5%a7%8b%e5%8c%96-init-%e7%b3%bb%e7%b5%b1%ef%bc%8c%e7%ac%ac-1-%e9%83%a8%e5%88%86-sysvinit/
https://linux.die.net/man/5/init
http://felix-lin.com/linux/init%E6%BC%94%E5%8C%96%E6%AD%B7%E7%A8%8B-%E8%BD%89%E8%B2%BC-%E6%B7%BA%E6%9E%90-linux-%E5%88%9D%E5%A7%8B%E5%8C%96-init-%E7%B3%BB%E7%B5%B1%EF%BC%8C%E7%AC%AC-2-%E9%83%A8%E5%88%86-upstart/
# grub
https://hugh712.gitbooks.io/grub/content/configuration-files.html
https://wiki.ubuntu-tw.org/index.php?title=Grub2
```
