# SSD の容量が 98GB としてしか認識されてくれない

## 問題点

lsblk すると 1TB がちゃんと出ている

```shellsession
$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1                   259:0    0 931.5G  0 disk
├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
├─nvme0n1p2               259:2    0     2G  0 part /boot
└─nvme0n1p3               259:3    0 928.5G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   100G  0 lvm  /
```

df するとなぜか 98GB しか割り当てられていないことになっている

```shellsession
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              6.0G  2.4M  6.0G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   98G   75G   18G  81% /
tmpfs                               15G  276K   15G   1% /dev/shm
efivarfs                           128K   76K   48K  62% /sys/firmware/efi/efivars
none                               1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
none                               1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs                               15G  640K   15G   1% /tmp
/dev/nvme0n1p2                     2.0G  178M  1.7G  10% /boot
/dev/nvme0n1p1                     1.1G  6.4M  1.1G   1% /boot/efi
none                               1.0M     0  1.0M   0% /run/credentials/systemd-networkd.service
none                               1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                              3.0G  8.0K  3.0G   1% /run/user/1000
```

## やったこと

なんか公式のドキュメントでの言及を見つけられなかったが、 Ubuntu インストーラーのデフォルトの挙動で LVM (Logical Volume Manager) が 98GB しか割り当てないようにしているらしい。ググったらだいたい以下のやり方がでてくるのでそれに従う。

```shellsession
# lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 100.00 GiB (25600 extents) to <928.46 GiB (237685 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
# resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs 1.47.2 (1-Jan-2025)
Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required
old_desc_blocks = 13, new_desc_blocks = 117
The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 243389440 (4k) blocks long.
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              6.0G  2.4M  6.0G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  914G   75G  801G   9% /
tmpfs                               15G  368K   15G   1% /dev/shm
efivarfs                           128K   76K   48K  62% /sys/firmware/efi/efivars
none                               1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
none                               1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs                               15G  640K   15G   1% /tmp
/dev/nvme0n1p2                     2.0G  178M  1.7G  10% /boot
/dev/nvme0n1p1                     1.1G  6.4M  1.1G   1% /boot/efi
none                               1.0M     0  1.0M   0% /run/credentials/systemd-networkd.service
none                               1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                              3.0G  8.0K  3.0G   1% /run/user/1000
```

## 参考

- [5.4.14. 論理ボリュームの拡張 | 論理ボリュームマネージャの管理 | Red Hat Enterprise Linux | 6 | Red Hat Documentation](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/lv_extend)
- [Ubuntu Serverで500GBのSSDが98GBしか使えなかった話 - LVMの未割り当て領域を拡張する #Docker - Qiita](https://qiita.com/atsushi11o7/items/a8857aec45d0de007c1f#%E3%81%AA%E3%81%9C%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%A7%E3%83%87%E3%82%A3%E3%82%B9%E3%82%AF%E3%82%92%E4%BD%BF%E3%81%84%E5%88%87%E3%82%89%E3%81%AA%E3%81%84%E3%81%AE%E3%81%8B)
