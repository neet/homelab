# Niri を入れる

さすがに GUI で Firefox が開きたかった。

## やったこと

### Niri を入れる

最近のレビュー動画とかを見ていると Niri という WM がイケているらしい。タイル型のやついいなと思っていたので、ちょうどよかった。

```
$ sudo pacman -Syu niri xwayland-satellite xdg-desktop-portal-gnome xdg-desktop-portal-gtk alacritty dms-shell-niri matugen cava qt6-multimedia-ffmpeg
$ systemctl --user add-wants niri.service dms
```

途中で jack2 と pipewire-jack どっちって訊かれたので pipewire-jack にした。[Which Is Better jack2 or pipewire-jack? : r/archlinux](https://www.reddit.com/r/archlinux/comments/1b3k635/which_is_better_jack2_or_pipewirejack/)

これで起動する

```
$ niri-session -l
```

自動ログインとかいうやつを有効にすれば、上のコマンドを入れなくても普通に Niri の状態で始まるらしい？あとで調べる

### Firefox を入れる

```
$ sudo pacman -S firefox
```

`ttf-font` は `gnu-free-fonts` を選んだ。違ったらあとで消そう。

### 1Password を入れる

え〜 1Password は AUR らしいので Pacman では入らないっぽい。どうすればいいんだ？

公式の案内があった：[Get the 1Password for Linux app | 1Password Support](https://support.1password.com/install-linux/#arch-linux)

案内通りにしたら `fakeroot` と `debugedit` がないって言われた。なんだ？

```
$ sudo pacman -S base-devel
```

これでいけた

```
$ makepkg -si
```

### 日本語フォントを入れたい


```
$ sudo pacman -S noto-fonts-cjk
```

これで入るには入ったんだが、中国語のほうが優先されてしまう。どうするんだ？

しょうがないから Niri の設定ウィンドウ（タイポグラフィとモーション>タイポグラフィ>通常フォント）でフォントをNoto Sans JPに固定した。［システム＞ロケール＞現在のロケール］も日本語に変えておこう。


### 使用感をマシにする

- remap で PageDown / PageUp を割り当てた
- fuzzel を入れた。 Mac でいうところの Spotlight 検索
