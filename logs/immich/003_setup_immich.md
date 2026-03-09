# immichインストール

## やったこと

### 設定ファイルのコピー

ユーザーディレクトリに作業用のディレクトリをつくる

```
mkdir ./immich-app
cd ./immich-app
```

composeとenvを取ってくる

```
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### 環境変数の上書き

```diff
# You can find documentation for all the supported env variables at https://docs.immich.app/install/environment-variables

# The location where your uploaded files are stored
-UPLOAD_LOCATION=./library
+UPLOAD_LOCATION=/mnt/immich

# The location where your database files are stored. Network shares are not supported for the database
DB_DATA_LOCATION=./postgres
```

### docker入れる

公式リポジトリからインストールしないとdocker composeなどが使えなかった。長いので省略するが、参考記事のとおりにした。

### 権限のつじつまを合わせる

NFSってデフォだとrootで入ろうとしたら弾かれるっぽい（なんも権限がないアカウントにマッピングされるっぽい）。それを無効にする必要があった。 `no_root_squash` をつける。

```
/tank/immich    compute-mitaka-01.local(rw,sync,no_subtree_check,no_root_squash)
```

### systemdにする

```
neet@compute-mitaka-01:~/immich-app$ cat /etc/systemd/system/immich.service
[Unit]
Description=Immich
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/neet/immich-app
ExecStart=docker compose up
ExecStop=docker compose down
Restart=always

[Install]
WantedBy=multi-user.target
```

Go!!!!

```
sudo systemctl daemon-reload
sudo systemctl enable --now immich
```

## 参考

- [Quick start | Immich](https://docs.immich.app/overview/quick-start/)
- [Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
