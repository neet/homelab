# Samba (smb) をインストールしてみる

とりあえずMacとかiPhoneからNASにアクセスしたい。

## 使ったコマンド

### パッケージを更新

`apt` と `apt-install` はけっきょく、ユーザー向けかスクリプト向けかという違いらしい。

```
sudo apt update
```

### インストール

はい

```
sudo apt install samba
```

### 共有

たぶん、HDDをマウントした中にさらにサブディレクトリを切ったほうが使いやすいんじゃないかな？そうでもない？

```
sudo mkdir /nas/sambashare
```

`/etc/samba/smb.conf` の末尾に以下を追加した。

```conf
[sambashare]
    comment = Samba on Ubuntu
    path = /nas/sambashare
    read only = no
    browsable = yes
    vfs objects = catia fruit streams_xattr
```

再起動

```
sudo service smbd restart
```

ファイアウォールを更新。たぶんSELinux関連？

```
sudo ufw allow samba
```

### /nas/sambashareの所有者を変える

これでいいのか？

```
neet@mitaka-nas:/nas$ sudo chown neet sambashare
neet@mitaka-nas:/nas$ ls -al
total 28
drwxr-xr-x  4 root root  4096 Feb  2 07:35 .
drwxr-xr-x 24 root root  4096 Feb  2 07:24 ..
drwx------  2 root root 16384 Feb  2 07:09 lost+found
drwxr-xr-x  2 neet root  4096 Feb  2 07:45 sambashare
```

### Samba用パスワード作成

これをやった。

```
neet@mitaka-nas:/nas$ sudo smbpasswd -a neet
New SMB password:
Retype new SMB password:
Added user neet.
```

これでSambaとして使えるユーザーを一覧できる

```
sudo pdbedit -L -v
```

## 使おう

ワロタ。急に使えるようになるんかい。

![Finderのnetworkタブにmitaka-nasと出ている様子](https://i.imgur.com/fwZxPXA.png)

iOSからでも使えた

![iOSのFilesアプリのスクショ](https://i.imgur.com/B4GTGMg.png)

## 失敗事例

### 共有用フォルダの所有者をrootにしてしまった

マウントするだけだと `/nas` は root になっちゃう。サブディレクトリは別ユーザーにしないとだめぽ。

### iOSからreadonlyになってしまう

![](https://i.imgur.com/btFZaCb.png)

[Samba share is readonly from iOS, but writable from other OSes - Ask Ubuntu](https://askubuntu.com/a/1556067)にあった。以下を設定すればいけるらしい。

```
[sambashare]
    ...
    vfs objects = catia fruit streams_xattr
```

オプションの値（`catia`, `fruit`, `streams_xattr`）は全部Apple関係のものらしい。

## 参考

- [Install and Configure Samba | Ubuntu](https://ubuntu.com/tutorials/install-and-configure-samba#1-overview)
- [3.13. MacOS クライアント向けの Samba の設定 | さまざまな種類のサーバーのデプロイメント | Red Hat Enterprise Linux | 8 | Red Hat Documentation](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_configuring-samba-for-macos-clients_assembly_using-samba-as-a-server)
