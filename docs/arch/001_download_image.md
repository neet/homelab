# Arch Linux インストール

## イメージのダウンロード

[Arch Linux - Downloads](https://archlinux.org/download/#bittorrent-download) で推奨されている方法に従います。


BitTorrent からイメージをダウンロードする。 BitTorrent とは P2P プロトコルの一種。ホムペにあるTorrentのファイルを落としてくる

![](https://i.imgur.com/Yy3l1Kg.png)

ググっていたら [Transmission](https://transmissionbt.com/) というクライアントを見つけたのでこれを使う。Balena Etcher で ISO ファイルを USB に焼く。

## セキュアブートを一時的に無効化

なんか Ubuntu のときは USB を差すだけで USB からブートできたのに、 Arch はそうはいかなかった。 BIOS の設定を書き換える

- F7 で詳細設定モードに入る
- [UEFIモード]>[非UEFIモード]にする


再起動後に『Arch Linux Install Medium [x86-64]』を選ぶと、インストール用のOSが始まる

![](https://i.imgur.com/cfA1mVf.jpeg)

これでいろいろ確認する

