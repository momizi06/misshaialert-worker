# ミス廃アラート works with Cloudflare Workers

## 設定方法

### 1. リポジトリをクローン

[momizi06/misshaialert-worker](https://github.com/momizi06/misshaialert-worker) をクローンします。

### 2. 依存パッケージをインストール

`pnpm i` で依存パッケージを入れます。

### 3. KVを作成

[Cloudflare Dashboard](https://dash.cloudflare.com/)に行き、デプロイしたいアカウントを選択し、ストレージとデータベース→KVを選択します。

Create Instanceをクリックして、好きな名前を入力し、Addをクリックして作成します。

作成後NamespaceのリストにIDが表示されますので、これを使用します。

### 4. wrangler.tomlを編集

wrangler.tomlを開き、`kv_namespaces` のコメントアウトを解除します。

`id` に先ほど作成したKVのIDを貼り付けます。

```toml
# binding = "KV" は変更しないこと
kv_namespaces = [
    { binding = "KV", id = "作成したKVのID" }
]
```

さらに、下の行にMisskeyの設定があります。

```toml
[vars]
MISSKEY_HOST = "投稿先サーバーのドメイン"
POST_VISIBILITY = "home" # public (パブリック), home (ホーム), followers (フォロワーのみ), specified (DM)
POST_TAGS = ["misshaialert"] # 投稿時に使用するタグ。複数指定可。#は書かない。misshaialertはミュート使用者のために推奨
DEBOBI = true # 今日のデボビゲゴを投稿するかどうか
```

投稿する時間も変更することができます。初期状態は 0:00 JST (15:00 UTC)です。  
書式はLinuxのcronと同じです。また、時間はUTCとして解釈されるため、日本で使用する場合は投稿したい時間から 9時間 引いたものを入力してください。

```toml
# 日本時間 0:00 (15:00 UTC)に実行
crons = [ "0 15 * * *" ]
```

### 5. Cloudflareにデプロイ

まず最初に`pnpm wrangler login` を実行して、Cloudflareにログインします。

```shell
$ pnpm wrangler login                                                                                  
 ⛅️ wrangler 4.29.1
───────────────────
Attempting to login via OAuth...
Opening a link in your default browser: https://dash.cloudflare.com/oauth2/auth...
```

自動的にブラウザが開き、Allow Wrangler to make changes to your Cloudflare account?というページが表示されるので下の Allow をクリックして承認します。

You have granted authorization to Wrangler! と表示されればOKです。以降、ログイン操作をする必要はありません。

次に、 `pnpm wrangler deploy` を実行します。  
チームアカウントなど複数ある場合は選択するように求められますので、個人であれば個人用のアカウントを選択します。

```shell
$ pnpm wrangler deploy
 ⛅️ wrangler 4.29.1
───────────────────
Total Upload: 3.54 KiB / gzip: 1.41 KiB
Your Worker has access to the following bindings:
Binding                                                     Resource
env.KV (0d7bc81ac3ba365498dfee9a01acbe994836287)            KV Namespace
env.MISSKEY_HOST ("HOST")                                   Environment Variable
env.POST_VISIBILITY ("home")                                Environment Variable
env.POST_TAGS (["misshaialert"])                            Environment Variable
env.DEBOBI (true)                                           Environment Variable

Uploaded misshaialert-worker (2.91 sec)
Deployed misshaialert-worker triggers (0.78 sec)
  schedule:  0 15 * * *
Current Version ID: 65c30dc4-bcb2-4804-a0cd-53388d6e45c8
```

### 6. トークンをCloudflareに登録する

Misskeyの設定で作成したトークンをCloudflare側に登録します。  
`pnpm wrangler secret put MISSKEY_TOKEN` を実行します。その後トークンを入力(コピペ)してください。

```shell
$ pnpm wrangler secret put MISSKEY_TOKEN
 ⛅️ wrangler 4.29.1
───────────────────
? Enter a secret value: » ******************************** (設定で作成したトークン)
```

Enterを押すと完了です。

```shell
√ Enter a secret value: ... ********************************
🌀 Creating the secret for the Worker "misshaialert-worker"
✨ Success! Uploaded secret MISSKEY_TOKEN
```

## Copyright

&copy; 2023-2024 CyberRex  
MIT License
