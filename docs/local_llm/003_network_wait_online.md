# 起動後にオンラインになるまで待たされる

systemdの設定を上書きする。

```
sudo systemctl edit systemd-networkd-wait-online.service
```

以下の内容を書く。 `--any` はどれか一つのネットワークインターフェースがオンラインになったら終了。 `ExecStart=` は、デフォルトの設定を無効にするみたいな意味らしい。

```
[Service]
ExecStart=
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --any
```
