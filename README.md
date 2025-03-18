# Local Docker Registry

このリポジトリでは、Docker の公式レジストリイメージ registry:2 を用いてローカルの Docker レジストリを起動するための簡易的な設定を提供します。

## 1. セットアップ方法

### 1.1. リポジトリのクローン

```bash 
git clone <REPOSITORY_URL> cd <REPOSITORY_NAME> 
```

### 1.2. docker-compose の起動

```bash 
docker-compose up -d 
```

初回起動時はイメージのダウンロードが走るので少し時間がかかる場合があります。

## 1.3. 動作確認

起動に成功すると、デフォルト設定では ホストのポート 5000 でレジストリが稼働しているはずです。以下コマンドで応答を確認できます。

```bash
curl http://localhost:5000/v2/_catalog
```

`{"repositories":[]}` のような JSON が返ってくればレジストリが動作しています。

## 2. イメージの Push / Pull

### 2.1. タグを付けてプッシュ

ローカルレジストリに格納したいイメージを localhost:5000（ポートは compose の設定に応じて変更）へタグ付けした上で push します。

例: memollection_app:latest というイメージをプッシュする場合

```bash
# タグを付ける
docker tag memollection_app:latest localhost:5000/memollection_app:latest

# プッシュする
docker push localhost:5000/memollection_app:latest
```

### 2.2. pull で取得

ほかの開発者や別の環境からレジストリを利用する場合は、同様にタグ付きリポジトリ名を指定して pull します。

```bash 
docker pull localhost:5000/memollection_app:latest 
```

## 3. k0s など containerd 環境から利用する場合

k0s (または単体の containerd) が稼働する環境で、このレジストリを直接参照したい場合は、containerd の設定で Insecure Registry を指定する必要があります。

たとえば `/var/lib/k0s/containerd-config.toml` (環境によって異なることがあります) に以下のように追記して、レジストリへ HTTP 接続できるようにします。

```toml
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:5000"]
  endpoint = ["http://localhost:5000"]
```

設定後、k0s や containerd を再起動すると、Kubernetes のマニフェストで image: localhost:5000/memollection_app:latest のように指定してもイメージを取得できるようになります。

## 4. よくある質問 (FAQ)

### HTTPS でアクセスするには？

自己署名証明書または正式な証明書を準備し、REGISTRY_HTTP_TLS_CERTIFICATE と REGISTRY_HTTP_TLS_KEY を環境変数で設定する必要があります。社内などで Insecure Registry の利用が許可されていない場合は、TLS を有効化してください。
レジストリ上の不要なイメージを削除したい

REGISTRY_STORAGE_DELETE_ENABLED=true を設定すると、レジストリ API を用いてイメージを削除できるようになります。ただし、レジストリのクリーンアップ手順が必要になるため、運用時には注意が必要です。

### ポート番号を変えたい

docker-compose.yml の ports セクションを修正し、5000:5000 を任意のポートに置き換えてください。たとえば 6000:5000 にするとホスト側のポート 6000 でレジストリにアクセスできます。

## 6. ライセンス
このリポジトリの設定ファイル自体には特定のライセンスはありませんが、Docker Registry (registry:2) は Apache License 2.0 が適用されています。詳細は Docker Registry のリポジトリを参照してください。 