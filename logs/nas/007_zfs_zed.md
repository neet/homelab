# ZEDを設定してみよう

zfsのプールを監視するためのツール郡みたいな感じ？

## やったこと

### 設定ファイルを確認する

ZEDのmanpageを読むと `/etc/zfs/zed.d/zed.rc` に設定があるとある。

```
##
# Name or path of executable responsible for sending notifications via email;
#   the mail program must be capable of reading a message body from stdin.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
#
#ZED_EMAIL_PROG="mail"

##
# Command-line options for ZED_EMAIL_PROG.
# The string @ADDRESS@ will be replaced with the recipient email address(es).
# The string @SUBJECT@ will be replaced with the notification subject;
#   this should be protected with quotes to prevent word-splitting.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
# If @SUBJECT@ was omited here, a "Subject: ..." header will be added to notification
#
#ZED_EMAIL_OPTS="-s '@SUBJECT@' @ADDRESS@"
```

このコメントを読んだ感じだと、以下のようなコマンドでメールを送れるプログラムがあればいいっぽい。

```
mail -s タイトル example@example.com < 本文
```

できれば設定をあんまりいじりたくないので、この `mail` をインストールしたいな。

いろいろ調べてみたところ、ZEDでは `msmtp` というツールを使っている人が多そうなので、それを使おうかなあ。メール鯖をローカルに立てたらちゃんとしたものを使おう。

### msmtpを入れる

Gmailとかの外部のSMTPサーバーを叩くクライアント？みたいな認識。とりあえず入れてみる。

```
sudo apt install msmtp
```

SSHなのに画面を書き換えられてダイアログが出てきた。なにこれ。よくわからんし調べても何も出てこなかったので、Yesにしておいた。

![](https://i.imgur.com/ZdCxMR4.png)

ZEDは実行ユーザーがrootになるので、rootに対して設定を書く必要がある。まず、`msmtp --verion`でシステム設定のディレクトリを確認する。

```
neet@mitaka-nas:~$ msmtp --version
msmtp version 1.8.24
Platform: x86_64-pc-linux-gnu
TLS/SSL library: GnuTLS
Authentication library: GNU SASL; oauthbearer and xoauth2: built-in
Supported authentication methods:
plain scram-sha-1 scram-sha-256 external gssapi cram-md5 digest-md5 login ntlm oauthbearer xoauth2
IDN support: disabled
NLS: enabled, LOCALEDIR is /usr/share/locale
Keyring support: Gnome
System configuration file name: /etc/msmtprc
User configuration file name: /home/neet/.msmtprc

Copyright (C) 2023 Martin Lambers and others.
This is free software.  You may redistribute copies of it under the terms of
the GNU General Public License <http://www.gnu.org/licenses/gpl.html>.
There is NO WARRANTY, to the extent permitted by law.
```

`/etc/msmtprc` であることがわかる。msmtp(1)を読むと、この下にある `msmtprc` を読むと言ってくれている。

> If it exists and is readable, a system wide configuration file SYSCONFDIR/msmtprc will be loaded, where SYSCONFDIR depends on your platform.  Use --version to find out which directory is used.

とりあえずいろいろなウェブサイトに書いてあった以下の情報を書いてみる。`/etc/msmtprc`を開いて：

```
# Set default values for all following accounts.
defaults
port 587
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile ~/.msmtp.log

account gmail
host smtp.gmail.com
from n33t5hin@gmail.com
auth on
user n33t5hin@gmail.com
password <GmailのApp Passwordをここに入れる>

# Set a default account
account default : gmail
```

GmailのApp Passwordはアカウントの設定画面的なところから作れた

![](https://i.imgur.com/1aee99K.png)

しかしこれいかにもインセキュアっぽいなあ。これが漏洩したら簡単に侵害されてしまう。

一応Debianのページを読んだ感じ、gpgで暗号化したパスワードを書くってやりかたもあるっぽいけど、めんどくさい。

できるだけ早くメール鯖を建てよう。

### メールを送ってみる

以下のコマンドで試してみる

```
echo "これはテストです" | msmtp n33t5hin@gmail.com
```

うむ。ちゃんと届いた。

![](https://i.imgur.com/YhUCnYG.png)

### msmtp-mta と mail コマンドを入れる

msmtp-mtaも入れれば、MTAサーバーがローカルに立って、`mail`コマンドでも送れるようになる。

```
sudo apt install msmtp-mta
```

msmtpdを有効にする。このデーモンがローカルの25番にMTAサーバーを立ててくれるらしい。

```
sudo systemctl enable msmtpd
```

bsd-mailxを入れる。これが `mail` コマンドを提供してくれる。他にも `mailutils` というのもあって、どっちでもいいらしい。

```
sudo pat install bsd-mailx
```

これで `mail` コマンドを使ってメールを送ってみる

```
echo テストです | mail -s mailコマンドを使ったテスト n33t5hin@gmail.com
```

うむ。割とすんなりと送れた。

![](https://i.imgur.com/aLbX3TM.png)

### ZEDを有効化する

`/etc/zfs/zed.d/zed.rc` の以下をいじった

```diff
##
# Email address of the zpool administrator for receipt of notifications;
#   multiple addresses can be specified if they are delimited by whitespace.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
# Enabled by default; comment to disable.
#
- ZED_EMAIL_ADDR="root"
+ ZED_EMAIL_ADDR="n33t5hin@gmail.com"

...

##
# Notification verbosity.
#   If set to 0, suppress notification if the pool is healthy.
#   If set to 1, send notification regardless of pool health.
#
- #ZED_NOTIFY_VERBOSE=0
+ ZED_NOTIFY_VERBOSE=1
```

`zed` は最初から有効になっているみたい？だけど、一応再起動しておこう。

```
sudo systemctl restart zed
```

これで scrub してみる。どうだ？

```
sudo zpool scrub tank
```

きた。すげ〜〜〜。

![](https://i.imgur.com/9QEu6IE.png)

## 参考

- [Ubuntu Manpage: ZED — ZFS Event Daemon](https://manpages.ubuntu.com/manpages/jammy/man8/zed.8.html)
- [msmtp - Debian Wiki](https://wiki.debian.org/msmtp)
