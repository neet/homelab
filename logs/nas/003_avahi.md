# avahi-daemonを有効化して*.localでアクセスできるようにする

## 実行したコマンド

### avahi-daemonを入れる

なんか標準で入ってるみたいな情報があるけど、自分の場合は入っていなかった。

```
sudo apt install avahi-daemon
```

### avahi-daemonを有効化

普通にsystemdで有効化できる

```
sudo systemctl enable avahi-daemon
sudo reboot
```

### 使う

これで `ホスト名.local` を使ってアクセスできるようになる

![](https://i.imgur.com/fULqX5t.png)

## 参考

- [networking - Enabling mDNS on fresh Ubuntu 24.04 server in /run/systemd/network does not persist across reboots - Ask Ubuntu](https://askubuntu.com/q/1556977)
- [Ubuntu Manpage: avahi-daemon - The Avahi mDNS_DNS-SD daemon](https://manpages.ubuntu.com/manpages/focal/man8/avahi-daemon.8.html)
