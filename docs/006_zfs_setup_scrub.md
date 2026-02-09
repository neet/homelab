# scrubとやらを設定してみよう

ZFSの整合性チェックの機能は、scrubというのを呼んで初めて成立するらしい

## やろうとしてみたこと（徒労だった）

### とりあえず実行してみる

`zpool scrub プール名` で行けた。

```
neet@mitaka-nas:~$ sudo zpool scrub tank
[sudo] password for neet:
neet@mitaka-nas:~$ zpool status tank
  pool: tank
 state: ONLINE
  scan: scrub in progress since Sun Feb  8 14:01:30 2026
        44.4G / 44.4G scanned, 4.27G / 44.4G issued at 547M/s
        0B repaired, 9.62% done, 00:01:15 to go
config:

        NAME                                 STATE     READ WRITE CKSUM
        tank                                 ONLINE       0     0     0
          raidz1-0                           ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2XR8L  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z7XF  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z8EL  ONLINE       0     0     0
```

しばらく放置していたら終わった。なるほど。

```
neet@mitaka-nas:~$ zpool status tank
  pool: tank
 state: ONLINE
  scan: scrub repaired 0B in 00:01:26 with 0 errors on Sun Feb  8 14:02:56 2026
config:

        NAME                                 STATE     READ WRITE CKSUM
        tank                                 ONLINE       0     0     0
          raidz1-0                           ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2XR8L  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z7XF  ONLINE       0     0     0
            ata-ST8000VN002-2ZM188_WPV2Z8EL  ONLINE       0     0     0

errors: No known data errors
```

### 定期実行させてみる

crontabしかしらなかったけど、systemdにタイマー機能があるのでそれを使うといいらしい。なんならsystemdの構成ファイルを規定で作ってくれているっぽい。

```
neet@mitaka-nas:~$ systemctl status zfs-scrub-monthly@tank.timer
○ zfs-scrub-monthly@tank.timer - Monthly zpool scrub timer for tank
     Loaded: loaded (/usr/lib/systemd/system/zfs-scrub-monthly@.timer; disabled; preset: enabled)
     Active: inactive (dead)
    Trigger: n/a
   Triggers: ● zfs-scrub@tank.service
       Docs: man:zpool-scrub(8)
```

これどういうこと？なんか `%i` の部分が変数になっているっぽい？

```
neet@mitaka-nas:~$ cat /usr/lib/systemd/system/zfs-scrub-monthly@.timer
[Unit]
Description=Monthly zpool scrub timer for %i
Documentation=man:zpool-scrub(8)

[Timer]
OnCalendar=monthly
Persistent=true
RandomizedDelaySec=1h
Unit=zfs-scrub@%i.service

[Install]
WantedBy=timers.target
```

### 実は何もしなくてもscrubしてくれているぽかった

`/etc/cron.d` を見ると、zfsutils-linuxが勝手に定義したcrontabが書いてある。これを読むと、毎月第2日曜日に勝手にscrubしてくれるので、別に自分で定義する必要はなかったっぽい。そしたらこれ二重に実行されてしまうんじゃないかな？気になってきた。

```
neet@mitaka-nas:~$ cat /etc/cron.d/zfsutils-linux
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# TRIM the first Sunday of every month.
24 0 1-7 * * root if [ $(date +\%w) -eq 0 ] && [ -x /usr/lib/zfs-linux/trim ]; then /usr/lib/zfs-linux/trim; fi

# Scrub the second Sunday of every month.
24 0 8-14 * * root if [ $(date +\%w) -eq 0 ] && [ -x /usr/lib/zfs-linux/scrub ]; then /usr/lib/zfs-linux/scrub; fi
```

『[addition of zfsutils-linux scrub every 2nd sunday · Issue #9858 · openzfs/zfs](https://github.com/openzfs/zfs/issues/9858)』によると、どうやらUbuntuのメンテナが気を利かせて勝手にcronを作るようにしてしまったらしい。それならそれでいいんだけど、じゃあ `/usr/lib/systemd/system/zfs-scrub-monthly@.timer` はどうなんだ？

あ〜なるほど、[zfs/etc/systemd/system/zfs-scrub-monthly@.timer.in at master · openzfs/zfs](https://github.com/openzfs/zfs/blob/master/etc/systemd/system/zfs-scrub-monthly%40.timer.in)を見た感じ、systemdのほうはOpenZFSが提供しているもので、cronのほうはUbuntuが提供しているものということらしかった。とりあえずはUbuntuのものを使うことにするか……。ZEDを使うようになれば、もし今後Ubuntuが気まぐれでcronを消しても気がつけるようになるはず。

## 参考

- [ZFS ファイルシステムの整合性をチェックする - Oracle Solaris ZFS 管理ガイド](https://docs.oracle.com/cd/E24845_01/html/819-6260/gbbwa.html)
- [Ubuntu Manpage: zpool-scrub — begin or resume scrub of ZFS storage pools](https://manpages.ubuntu.com/manpages/jammy/man8/zpool-scrub.8.html)
