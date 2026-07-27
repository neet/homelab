# Arch Linux インストール

## やったこと

### イメージのダウンロード

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

### パーティションを切る

#### EFIシステムパーティション

> 複数のカーネルや複数の unified カーネルイメージ、ブートローダー、ファームウェアのアップデートファイル、他のオペレーティングシステムのファイル、OEM のファイルを格納するのに十分なスペースを確保するために、ESP のサイズは 1 GiB にすることが推奨されます。これでも十分であるか疑問である場合は、4 GiB を割り当てておけばいかなる場合でも十分であるはずです。

[EFI システムパーティション - ArchWiki](https://wiki.archlinux.jp/index.php/EFI_%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%82%B7%E3%83%A7%E3%83%B3) より引用

#### スワップ

> 私がお勧めするのは 2-3GB かそれ以上のスワップを持ついくつかのテストシステムをセットアップし、ささまざまな（メモリの）負荷状況の元で 1 週間程度の連続稼働をして何が起こるかをモニタリングすることです。

[スワップの弁護：よくある誤解を解く](https://chrisdown.name/ja/2018/01/02/in-defence-of-swap.html) より引用

