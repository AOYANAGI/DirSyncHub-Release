# DirSyncHub（Windows Release）

DirSyncHub は、**Active Directory（AD）専用**の認証・検索 API サーバーです。  
通常の LDAP フィルター検索に加えて、**自然言語による検索（Ollama 連携・任意）**にも対応しています。Windows 環境で動作し、設定ファイル（`config`）で運用できます。

---

## 今後の予定

- LDAP 対応を将来的に追加予定
- Linux 対応を将来的に追加予定
- API 経由で AD のユーザー/グループ作成機能を将来的に追加予定

---

## ダウンロード

最新の ZIP ファイルは **[Releases ページ](../../releases)** からダウンロードできます。

1. 上記の「Releases ページ」を開く
2. 最新バージョンの「Assets」欄にある ZIP ファイルをダウンロード
3. ダウンロードした ZIP ファイルを右クリックし、「すべて展開（解凍）」を実行

---

## 動作環境

- **OS**: Windows 10 / 11 (64bit)、または .NET 10 が動作する Windows Server OS（64bit）
- **必要なソフト**: .NET 10 Runtime
- **接続先**: Active Directory サーバー
- **任意**: Ollama サーバー（自然言語検索利用時）

---

## 配布物構成（例）

    DirSyncHub/
    ├─ DirSyncHub.exe
    ├─ DirSyncHub.dll
    ├─ *.dll
    └─ config/
       ├─ server.json.sample
       ├─ auth.json.sample
       ├─ directory.json.sample
       ├─ nlog.config.sample
       └─ ollama.json.sample   （任意）

---

## 事前準備（Active Directory）

本システムは **グループベース認証** です。事前に以下が必要です。

1. 認証対象ユーザーを作成
2. 認可用グループを作成
3. ユーザーを該当グループへ所属
4. `config/auth.json` の `directoryRoleMappings` にグループ DN とロールの対応を設定

> どのロールにもマッピングされないユーザーは `POST /api/dshub/v1/login` で拒否されます。

---

## 初回セットアップ

1. `.sample` をコピーして実ファイルを作成  
   - `config/server.json.sample` → `config/server.json`  
   - `config/auth.json.sample` → `config/auth.json`  
   - `config/directory.json.sample` → `config/directory.json`  
   - `config/nlog.config.sample` → `config/nlog.config`  
   - `config/ollama.json.sample` → `config/ollama.json`（任意）
2. `config/*.json` を環境に合わせて編集
3. `DirSyncHub.exe` を実行
4. `GET /health` で起動確認

> 設定内のパスワード項目（証明書/AD バインド）は、起動時に自動暗号化され設定ファイルへ書き戻されます。

---

## 設定ファイル詳細

### `config/server.json`
- `server.binding`: 待ち受け IP/ホスト
- `server.port`: 待ち受けポート
- `server.ssl.enabled`: TLS 有効/無効
- `server.ssl.certificatePath`: 証明書パス
- `server.ssl.certificatePassword`: 証明書パスワード

### `config/auth.json`
- `signingKey`: JWT 署名鍵
- `tokenLifetimeMinutes`: JWT 有効期間（分）
- `roles[]`: ロールと権限
- `directoryRoleMappings[]`: AD グループ DN とロール対応

### `config/directory.json`
- `provider`: 現行は `ActiveDirectory`
- `host` / `port` / `useSsl`
- `baseDn`: 検索起点 DN
- `bindDn` / `bindPassword`: バインド情報
- `domain`: AD ドメイン名

### `config/ollama.json`（任意）
- `host`: Ollama URL
- `model`: 使用モデル名（例: `llama3.1:8b`）

### `config/nlog.config`
- `targets`: ログ出力先
- `rules`: レベル別ルール

---

## 実行オプション

- `DirSyncHub.exe -h` / `--help` / `-?`  
  ヘルプを表示して終了
- `DirSyncHub.exe -v` / `--version`  
  バージョンを表示して終了
- `DirSyncHub.exe -i` / `--install-service`  
  Windows サービス登録（管理者権限）
- `DirSyncHub.exe -u` / `--uninstall-service`  
  Windows サービス削除（管理者権限）
- `DirSyncHub.exe`  
  通常起動

---
  
## API ドキュメント

- OpenAPI HTML（ReDoc）: https://aoyanagi.github.io/DirSyncHub-Release/api/)

---

## 主なエンドポイント

- `POST /api/dshub/v1/login`（認証）
- `GET /api/dshub/v1/health`（ヘルスチェック）
- `POST /api/dshub/v1/directory/users/search`（ユーザー検索）
- `POST /api/dshub/v1/directory/groups/search`（グループ検索）
- `POST /api/dshub/v1/directory/users/get`（ユーザー取得）
- `POST /api/dshub/v1/directory/groups/get`（グループ取得）
- `POST /api/dshub/v1/directory/search/nl`（自然言語検索・任意）

---

## 認証・認可

- JWT は `Authorization: Bearer <token>` で指定
- ディレクトリ検索系 API の必要操作権限:
  - `directory.search`: ディレクトリの検索・取得を許可

---

## 起動確認チェックリスト

1. プロセス/サービスが起動している
2. `GET /api/dshub/v1/health` が応答する
3. `POST /api/dshub/v1/login` で JWT を取得できる
4. JWT 付きで `POST /api/dshub/v1/directory/users/search` が成功する
5. （任意）`POST /api/dshub/v1/directory/search/nl` が成功する

---

## トラブルシュート

### `/api/dshub/v1/directory/search/nl` が `503` を返す
- Ollama 側障害時のフォールバック応答です
- Ollama の起動状態、モデル配置、URL を確認してください

### 起動時に設定ファイル不足エラーで終了する
- 必須ファイル  
  `config/server.json` / `config/auth.json` / `config/directory.json` / `config/nlog.config`  
  の存在を確認してください

---

## アンインストール方法

1. サービス運用している場合は `DirSyncHub.exe -u` を実行
2. 解凍したフォルダを削除

---

## 不具合報告・お問い合わせ

- バグ報告や機能要望: このリポジトリの **[Issues](../../issues)**
- メールでのお問い合わせ: **iam.aoyanagi@gmail.com**

## ライセンス・免責事項

本ソフトウェアは [MIT License](LICENSE) のもとで公開されています。  
本ソフトウェアの使用によって生じたいかなる損害についても、開発者は一切の責任を負いません。
