# HDDをフォーマット＆マウントしてみる

NAS 構築の作業ログ。あとからこれをmdadmに移行するという前提。

## 使ったコマンド

`lsblk`, `mkfs` と `vim` しか使っていません

### ブロックを一覧

ブロック（何のことなのかまだ理解していない）を一覧。フォーマットしているとここでext4とかUUIDとかが表示される。

```
neet@mitaka-nas:~$ lsblk -f
NAME                      FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                       ext4        1.0            c724e10a-4363-4aba-b7d4-959282e36848
nvme0n1
├─nvme0n1p1               vfat        FAT32          D0CE-538B                                   1G     1% /boot/efi
├─nvme0n1p2               ext4        1.0            9f1fe51d-2cc3-406a-8656-0e929daffe20      1.7G     5% /boot
└─nvme0n1p3               LVM2_member LVM2 001       lf9HFN-Bvtk-29jQ-r3L8-WbXY-s2Yk-4pUVnP
  └─ubuntu--vg-ubuntu--lv ext4        1.0            7030fe26-eae4-495f-a66c-834e479a5ad2     86.3G     7% /
```

### フォーマット

`mkfs.ext4` というコマンドで `/dev/sda` （SATAで繋がっているHDD）をフォーマットした。

```
neet@mitaka-nas:~$ sudo mkfs.ext4 /dev/sda
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 1953506646 4k blocks and 244191232 inodes
Filesystem UUID: c724e10a-4363-4aba-b7d4-959282e36848
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848, 512000000, 550731776, 644972544, 1934917632

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```
これでUUIDが割り当てられるようになる。

### 永続的にマウント

とりあえずこのように書いてみた。書き込んだら再起動すればいいっぽい。

```
UUID=c724e10a-4363-4aba-b7d4-959282e36848       /nas    ext4    defaults        0       2
```

オプションは[/etc/fstabに記述されている数字の意味](https://atmarkit.itmedia.co.jp/flinux/rensai/linuxtips/756fstabnum.html)が役立つ。

これで `/dev/sda` が使えるようになった。

```
neet@mitaka-nas:~$ ls /
bin                boot   dev  home  lib64              lost+found  mnt  opt   root  sbin                snap  swap.img  tmp  var
bin.usr-is-merged  cdrom  etc  lib   lib.usr-is-merged  media       nas  proc  run   sbin.usr-is-merged  srv   sys       usr
```

### フォーマットを消す

まだ試していない。飽きたらこれで消せるらしいです。データが消えるわけじゃなくて、ヘッダー領域？みたいなのが消えるだけらしい。

```
sudo wipefs /dev/sda
```

## 参考にしたウェブサイト

- [Ubuntuで外付けHDDをフォーマットする](https://blog.hn-pgtech.com/2024-03-20/)
- [/etc/fstabに記述されている数字の意味](https://atmarkit.itmedia.co.jp/flinux/rensai/linuxtips/756fstabnum.html)
