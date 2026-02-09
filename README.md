# homelab

自宅NASサーバーの構築ログとAnsible Playbookを管理するリポジトリです。

## 📝 構築ログ

構築時の作業ログは`logs/`ディレクトリに保存されています：

- `001_init_hdd.md` - HDDの初期化とマウント
- `002_install_smb.md` - Sambaのインストール
- `003_avahi.md` - avahi-daemonの設定
- `004_change_hostname.md` - ホスト名の変更
- `005_zfs_install.md` - ZFSへの移行
- `006_zfs_setup_scrub.md` - ZFS scrubの設定
- `007_zfs_zed.md` - ZFS Event Daemonの設定

## 🚀 セットアップ方法

### 前提条件

- Ansible 2.9以上がインストールされていること
- NASサーバーにSSH接続できること
- NASサーバーにsudo権限を持つユーザーが存在すること

### 1. インベントリの設定

`inventory/hosts.yml`を編集して、NASサーバーのIPアドレスとユーザー名を設定します：

```yaml
nas_servers:
  hosts:
    mitaka-storage-01:
      ansible_host: 192.168.1.100  # 実際のIPアドレスに変更
      ansible_user: neet             # 実際のユーザー名に変更
```

### 2. 変数の設定

`group_vars/all.yml`を確認し、必要に応じて変数を変更します。

### 3. 機密情報の設定

`group_vars/vault.yml`を編集して、機密情報を設定します：

```bash
# vault.ymlを編集
vim group_vars/vault.yml

# 以下の情報を実際の値に変更：
# - samba_password: Sambaユーザーのパスワード
# - msmtp_from: 送信元メールアドレス
# - msmtp_user: Gmailのメールアドレス
# - msmtp_password: GmailのApp Password
# - zed_email_addr: ZED通知先メールアドレス
```

### 4. vault.ymlを暗号化

```bash
ansible-vault encrypt group_vars/vault.yml
# パスワードを入力
```

編集する場合：

```bash
ansible-vault edit group_vars/vault.yml
```

### 5. ZFSプールの作成（手動）

Playbookを実行する前に、ZFSプールを手動で作成する必要があります：

```bash
# NASサーバーにSSH接続
ssh neet@192.168.1.100

# ディスクIDを確認
ls -l /dev/disk/by-id

# ZFSプールを作成（RAIDZ1）
sudo zpool create tank raidz1 \
  /dev/disk/by-id/ディスク1 \
  /dev/disk/by-id/ディスク2 \
  /dev/disk/by-id/ディスク3

# プールの状態を確認
sudo zpool status
```

### 6. Playbookの実行

```bash
# すべてのroleを実行
ansible-playbook playbooks/nas-setup.yml --ask-vault-pass

# 特定のroleのみ実行
ansible-playbook playbooks/nas-setup.yml --ask-vault-pass --tags samba

# ドライラン（実際には変更しない）
ansible-playbook playbooks/nas-setup.yml --ask-vault-pass --check
```

## 📁 ディレクトリ構成

```
.
├── ansible.cfg           # Ansible設定ファイル
├── inventory/
│   └── hosts.yml        # インベントリファイル
├── group_vars/
│   ├── all.yml          # 共通変数
│   └── vault.yml        # 機密情報（暗号化）
├── playbooks/
│   └── nas-setup.yml    # メインPlaybook
├── roles/
│   ├── hostname/        # ホスト名設定
│   ├── zfs/             # ZFS設定
│   ├── samba/           # Samba設定
│   ├── avahi/           # avahi-daemon設定
│   ├── msmtp/           # msmtp設定
│   └── zed/             # ZFS Event Daemon設定
└── logs/                # 構築ログ
```

## 🏷️ 利用可能なタグ

- `hostname` - ホスト名の設定のみ
- `zfs` - ZFSの設定のみ
- `samba` - Sambaの設定のみ
- `avahi` - avahi-daemonの設定のみ
- `msmtp` - msmtpの設定のみ
- `zed` - ZEDの設定のみ

## 🔧 トラブルシューティング

### vault.ymlのパスワードを忘れた場合

1. `group_vars/vault.yml`を削除
2. 新しい`vault.yml`を作成
3. 再度暗号化

### ZFSプールが見つからない

ZFS roleを実行すると、プールが存在しない場合は警告が表示されます。上記の「ZFSプールの作成」を参照してください。

## 📄 ライセンス

MIT
