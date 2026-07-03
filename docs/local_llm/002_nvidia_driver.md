# NVIDIA のドライバーをインストールする

## やったこと

### カーネルヘッダーのインストール

たぶんLinuxのカーネルにアクセスするための `.h` のことだと思われる。 `uname -r` はKernelのバージョンを取得するコマンドっぽい。

```
apt install linux-headers-$(uname -r)
```

### NVIDIA の Keyring パッケージを追加

ディストロ名は[Index of /compute/cuda/repos](https://developer.download.nvidia.com/compute/cuda/repos/)から確認した。

```
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2604/x86_64/cuda-keyring_1.1-1_all.deb
# dpkg -i cuda-keyring_1.1-1_all.deb
# apt update
```

### ドライバーのバージョンの固定

`/etc/apt/preferences.d/nvidia` に設定ファイルを書く方法（ `apt_preferences(5)` ）と、固定用のパッケージを入れる方法があるらしい。固定用のパッケージを入れる方法のほうが新しくて推奨されているっぽい。固定用のパッケージがやってくれていることが、けっきょくは apt_preferences(5) らしいのだが。

`apt search`を使って最新版で安定版っぽいものを探した。

```
neet@compute-mitaka-02:~$ apt search nvidia-driver-pinning
nvidia-driver-pinning-595.58.03/unknown 595.58.03-1ubuntu1 all
  APT driver pinning file for driver version 595.58.03

nvidia-driver-pinning-595.71.05/unknown 595.71.05-1ubuntu1 all
  APT driver pinning file for driver version 595.71.05

nvidia-driver-pinning-610/unknown 610-2ubuntu1 all
  APT driver pinning file for driver branch 610

nvidia-driver-pinning-610.43.02/unknown 610.43.02-1ubuntu1 all
  APT driver pinning file for driver version 610.43.02
```

この場合は `595.71.05` っぽいので、一旦これで。

```
sudo apt install nvidia-driver-pinning-595.71.05
```

### ドライバーのインストール

Vulkanとかその類のものは不要なので、headlessのほうをインストールする。ちなみに `nvidia-dkms-open` と `nvidia-dkms` には意味のある差はないらしい。

```
sudo apt install libnvidia-compute nvidia-dkms-open
```

このあと再起動する。セキュアブートまわりについては [セキュアブート有効環境でUbuntu 22.04にNVIDIAドライバをインストールする方法 #Ubuntu22.04 - Qiita](https://qiita.com/ntrlmt/items/4fdf9ba917a2c58076f9) を参照。

```
sudo reboot
```

### 動作確認

これでGPUが出るかを見る。

```
cat /proc/driver/nvidia/version
nvidia-smi
```

## 参考

- [Ubuntu — NVIDIA Driver Installation Guide](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/ubuntu.html)
- [Supported Drivers and CUDA Toolkit Versions — NVIDIA Data Center Drivers](https://docs.nvidia.com/datacenter/tesla/drivers/supported-drivers-and-cuda-toolkit-versions.html)
- [Ubuntu/Debian環境におけるNVIDIAドライバのバージョン固定が簡単になっていた話](https://zenn.dev/yuyakato/articles/8885f10b1f0f1d)
- [nvidia - GPU driver not loaded when secure boot is enabled - Ask Ubuntu](https://askubuntu.com/questions/1438024/gpu-driver-not-loaded-when-secure-boot-is-enabled)
- [セキュアブート有効環境でUbuntu 22.04にNVIDIAドライバをインストールする方法 #Ubuntu22.04 - Qiita](https://qiita.com/ntrlmt/items/4fdf9ba917a2c58076f9)
