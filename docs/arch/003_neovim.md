# Neovim を入れる

## やったこと

### Neovim を入れる

```
$ sudo pacman -S neovim
$ sudo pacman -S git
```

これがないとくるしい……

```
$ git clone https://github.com/neet/dotfiles.git
$ ln -s $PWD/dotfiles/.config/nvim $HOME/.config/nvim
```

Treesitter がないって怒られたので入れる

```
$ sudo pacman -S treesitter-cli
```

Cコンパイラが無いって言われるので gcc を入れてみる

```
$ sudo pacman -S gcc
```

色こそ写っていないが普通に使えているように見える

![](https://i.imgur.com/vsKR5c9.jpeg)
