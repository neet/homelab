# 日本語入力したい

## やったこと

### Discord を入れる

Discord を入れる。 Syu しないとアップデートが配信されたときに拾えない

```
$ sudo pacman -Syu discord
```

### Fcitx5 と Mozc を入れる

Fcitx5 というのとIBus というのがあるらしいがどっちを使えばいいのかさっぱりわからん。とりあえず Fcitx にしてみる。

```
$ sudo pacman -Syu fcitx5 fcitx5-configtool fcitx5-gtk fcitx5-qt
$ cat >> $HOME/.bashrc << EOF
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
EOF
```

Fcitx っていうのは入力メソッドを選択できるガワみたいな感じで、入力メソッド自体は Mozc で入れるっぽい。なんもわからねえ。

```
$ sudo pacman -S fcitx5-mozc
```

いけたか？

![](https://i.imgur.com/EZbi1PH.png)
