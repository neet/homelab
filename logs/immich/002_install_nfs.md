# NFSを入れる

## やったこと

### NFSサーバーのインストール

いつもの

```
sudo apt install nfs-kernel-server
sudo systemctl start nfs-kernel-server.service
```

### 設定を変更

/etc/exportsをいじる。めちゃくちゃ贅沢な名前だ。

とりあえずこんなんでどうだろうか。

```
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)

/tank/immich    compute-mitaka-01.local(rw,sync,no_subtree_check)
```

意味はこういう感じっぽい

| オプション | 意味 |
| -- | -- |
| rw | 読み書きできる |
| sync | 同期か非同期か。|
| no_subtree_check | パフォーマンスよくするあれっぽい |

ディレクトリ作る

```
sudo mkdir /tank/immich
```

適用

```
sudo exportfs -a
```

### クライアントにNFSをインストール

ほい

```
sudo apt install nfs-common
```

fstabにこれを書く

```
store-mitaka-01.local:/tank/immich       /mnt/immich     nfs
```

マウント

```
sudo mkdir /mnt/immich
sudo mount -a
```

いけた！

```
root@store-mitaka-01:/home/neet# echo "hello world!" > /tank/immich/hello.txt
neet@compute-mitaka-01:~$ cat /mnt/immich/hello.txt hello world!
```

## 参考

- [Network File System (NFS) - Ubuntu Server documentation](https://ubuntu.com/server/docs/how-to/networking/install-nfs/)
