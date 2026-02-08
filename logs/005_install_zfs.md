# ZFSを試してみよう

なんか調べてみたらmdadmよりもzfsのほうがイケてるらしいので、そっちにすることにした。

## やったこと

### マウントを削除

Sambaを削除

```diff
# /etc/samba/smb.conf
- [sambashare]
-     comment = Samba on Ubuntu
-     path = /nas/sambashare
-     read only = no
-     browsable = yes
-     vfs objects = catia fruit streams_xattr
```

マウントを削除

```diff
# /etc/fstab
- UUID=c724e10a-4363-4aba-b7d4-959282e36848       /nas    ext4    defaults        0       2
```

マウントを取り消すと、空のディレクトリが残るっぽい

![](https://i.imgur.com/obiX8O7.png)

再起動

```
sudo reboot
```

### ディスクを初期化

wipefsでsdaを消す

```
neet@mitaka-nas:~$ sudo wipefs -a /dev/sda
/dev/sda: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
```

lsblkで削除されたことを確認

```
neet@mitaka-nas:~$ lsblk -f
NAME                      FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
sdb
sdc
nvme0n1
├─nvme0n1p1               vfat        FAT32          D0CE-538B                                   1G     1% /boot/efi
├─nvme0n1p2               ext4        1.0            9f1fe51d-2cc3-406a-8656-0e929daffe20      1.7G     5% /boot
└─nvme0n1p3               LVM2_member LVM2 001       lf9HFN-Bvtk-29jQ-r3L8-WbXY-s2Yk-4pUVnP
  └─ubuntu--vg-ubuntu--lv ext4        1.0            7030fe26-eae4-495f-a66c-834e479a5ad2     86.1G     7% /
```

### ZFSでプールを作る

いつもの

```
sudo apt install zfsutils-linux
```

なんかID指定しなきゃぶっ壊れるという情報があるので一応ID指定にしておく？

```
neet@mitaka-nas:~$ ls -l /dev/disk/by-id
total 0
lrwxrwxrwx 1 root root  9 Feb  8 00:22 ata-ST8000VN002-2ZM188_WPV2XR8L -> ../../sda
lrwxrwxrwx 1 root root  9 Feb  8 00:22 ata-ST8000VN002-2ZM188_WPV2Z7XF -> ../../sdc
lrwxrwxrwx 1 root root  9 Feb  8 00:22 ata-ST8000VN002-2ZM188_WPV2Z8EL -> ../../sdb
```

これでプールを作る。`tank`の部分はプール名で、慣例的に `tank` にすることが多いらしい。

```
sudo zpool create tank raidz1 \
  /dev/disk/by-id/ata-ST8000VN002-2ZM188_WPV2XR8L \
  /dev/disk/by-id/ata-ST8000VN002-2ZM188_WPV2Z7XF \
  /dev/disk/by-id/ata-ST8000VN002-2ZM188_WPV2Z8EL
```

ステータスを確認

```
neet@mitaka-nas:~$ sudo zpool status
  pool: tank
 state: ONLINE
config:

        NAME                                 STATE     READ WRITE CKSUM
        tank                                 ONLINE       0     0     0
          raidz1-0                           ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2XR8L  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z7XF  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z8EL  ONLINE       0     0     0
```

### Samba共有用のデータセットを作る

`tank/share` を作る。自動でマウントされる。

```
sudo zfs create tank/share
```

Linux側のユーザーを変更

```
sudo chown neet tank/share
```

Sambaの設定を元に戻す

```
# /etc/samba/smb.conf
[sambashare]
    comment = Samba on Ubuntu
    path = /tank/share
    read only = no
    browsable = yes
    vfs objects = catia fruit streams_xattr
```

### 空き容量の確認

`zfs list`を使う。`zpool list`で出てくる数字はパリティも足されてあるので注意が必要。

```
neet@mitaka-nas:/tank$ zfs list
NAME         USED  AVAIL  REFER  MOUNTPOINT
tank        29.6G  14.4T   139K  /tank
tank/share  29.6G  14.4T  29.6G  /tank/share
```

### （やり直し用）プールを削除

失敗したとき用

```
sudo zpool destroy 
```

## 不明点

### SambaとNFSで共有してみる

ZFSには、管理しているプールをSambaおよびNFSで共有する機能が備わっているっぽい。

```
$ sudo zfs create tank/share
$ sudo zfs get sharesmb tank/share
NAME        PROPERTY  VALUE     SOURCE
tank/share  sharesmb  on        local
$ sudo zfs set sharesmb=on tank/share
```

これで一応は表示されるようになったけど、なんか書き込めない。`vfs objects` の設定が全くされていないからなんじゃないかしら。

![](https://i.imgur.com/zUU0rBs.png)

う〜〜ん、わからんけど普通に `/etc/samba/smb.conf` で管理するんでいい気がしてきた。

### 「`ashift=12` や `compression=lz4` を設定しろ」という言説について

`ashift` は ZFS が内部で使うセクタの長さ。 `compression` は圧縮方式。 `ashift` を `12` にして、 `compression` を `lz4` にしろという情報がよく出てくるけど、よくわからなかった。

`asfhit` は内部で判定して自動的に適切な値を使うようになってくれているっぽく、今回は何もしなくても12になった。

```
neet@mitaka-nas:~$ sudo zdb -C tank | grep ashift
                ashift: 12
```

compressionもデフォルトでonになっているっぽい? onの値の意味がわからないけど。

```
neet@mitaka-nas:~$ zfs get all tank | grep compression
tank  compression           on                     default
```

`man zfsprops` を見ると各プロパティの意味と振る舞いが載っている。

> When set to on (the default), indicates that the current default compression algorithm should be used.  The default balances compression and decompression speed, with compression ratio and is expected to work well on a wide variety of workloads.  Unlike all other settings for this property, on does not select a fixed compression type.  As new compression algorithms are added to ZFS and enabled on a pool, the default compression algorithm may change.  The current default compression algorithm is either lzjb or, **if the lz4_compress feature is enabled, lz4**.

`man zpool-features` を見ると、プールのフィーチャーの確認方法が書いてある。

> The state of supported features is exposed through pool properties of the form `feature@short-name`.

というわけで `lz4_compress` フィーチャーが有効になっているか確かめる。

```
neet@mitaka-nas:/tank$ zpool get feature@lz4_compress
NAME  PROPERTY              VALUE                 SOURCE
tank  feature@lz4_compress  active                local
```

うむ、やはり規定でlz4で圧縮されているみたいだ。

### `zfs` と `zpool` の違い

- zpool: 実装寄り。プールの作成〜管理。
- zfs: クライアント寄り。データセットの作成〜管理。

## 参考文献

- [Setup a ZFS storage pool | Ubuntu](https://ubuntu.com/tutorials/setup-zfs-storage-pool#1-overview)
- [Ubuntu Manpage: zfs - configures ZFS file systems](https://manpages.ubuntu.com/manpages/resolute/en/man8/zfs.8.html)
- [Ubuntu Manpage: zpool - configures ZFS storage pools](https://manpages.ubuntu.com/manpages/resolute/en/man8/zpool.8.html)
- [zfs - Why are all the zpools named "tank"? - Server Fault](https://serverfault.com/questions/562564/why-are-all-the-zpools-named-tank)
