# 「一般的な推奨事項」を確認

## やったこと

記憶があるうちにメモ

### systemd-networkd を設定

[systemd-networkd - ArchWiki](https://wiki.archlinux.jp/index.php/Systemd-networkd#%E6%9C%89%E7%B7%9A%E3%82%A2%E3%83%80%E3%83%97%E3%82%BF%E3%81%A7_DHCP_%E3%82%92%E4%BD%BF%E7%94%A8)

```
# systemctl enable systemd-networkd
# systemctl start systemd-networkd
# cat > /etc/systemd/network/20-wired.network <<EOF
[Match]
Name=eno1

[Network]
DHCP=yes
EOF
```

### systemd-resolved を設定

[systemd-resolved - ArchWiki](https://wiki.archlinux.jp/index.php/Systemd-resolved#DNS)

```
# systemctl enable systemd-resolved
# systemctl start systemd-resolved
# ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

### iwd を設定

[iwd - ArchWiki](https://wiki.archlinux.jp/index.php/Iwd)

```
# pacman -S iwd
# iwctl
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect Neetlab-WPA3
[iwd]# exit
# cat > /etc/systemd/network/25-wireless.network << EOF
[Match]
Name=wlan0

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
EOF
# systemctl restart systemd-networkd
# systemctl restart systemd-resolved
````

### ユーザを作成

[ユーザーとグループ - ArchWiki](https://wiki.archlinux.jp/index.php/%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%81%A8%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97#%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E7%AE%A1%E7%90%86)

```
# useradd -m neet
# passwd neet
```

### sudo を入れる

sudo-rs もあるけど Arch Wiki には sudo のほうが紹介されていたので。意識したことがなかったけど sudo ができるグループというものがあるらしくて、通常 wheel というグループを使うらしい。

```
# pacman -S sudo
# gpasswd -a neet wheel
# pacman -S vi
# visudo
```

以下のように変更

```diff
+ # %wheel ALL=(ALL:ALL) ALL
- # %wheel ALL=(ALL:ALL) ALL
```

通常ユーザとして入る

```
$ sudo echo "hello"
[sudo] password for neet:
hello
```
