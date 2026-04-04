# Cloudflare Tunnel でインターネットに公開

## やったこと

### CloudflaredをインストールしてTunnelを作る

1. **Networks > Connectors** を開き、「Add Tunnel」を押す
2. Tunnel Type として Cloudflared を選ぶ
3. Name として `mitaka` を入れる。LAN上の他のホストも指定できるので、一つで充分
4. 出てくるコマンドを入力

#### 出てくるコマンド

インストール

```
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | sudo tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null

# Add this repo to your apt repositories
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# install cloudflared
sudo apt-get update && sudo apt-get install cloudflared
```

トンネルの指定

```
sudo cloudflared service install [TOKEN HERE]
```

### Tunnelの設定

これでやった

![](https://i.imgur.com/mPib6UJ.png)

## 参考

- [Create a tunnel (dashboard) · Cloudflare One docs](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel/)
