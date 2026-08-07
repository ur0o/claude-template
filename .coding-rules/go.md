# Go 言語設定

## 技術スタック

- ホットリロードにはAirを使用
- WebフレームワークにはGinを使用
- ORM: GORM
- マイグレーション: golang-migrate
- 環境変数ローダー: godotenv
- ロガー: logrus
- フォーマッター: `gofmt` / `goimports`（自動適用）
- リンター: `golangci-lint`
- テスト: `go test ./...`
- セッション管理: github.com/gin-contrib/sessions / github.com/gin-contrib/sessions/cookie

---

## 必須事項
- エラーは必ずハンドリングする（`_` で握り潰さない）
- `fmt.Println` によるデバッグ出力をコミットに含めない
- パッケージ名はシンプルに（`util` `common` `helper` は避ける）
- goroutine を使うときは必ず終了条件を明確にする
- APIサーバとして利用する時は、コントローラ層、サービス層、リポジトリ層、モデル層のレイヤードアーキテクチャで構成する

---

## ディレクトリ構成

プロジェクトルート直下を以下の構成で管理する。

```
.
├── initializers/    # サーバ起動時に実行する処理（ルーティング定義、依存性注入など）
├── controllers/     # コントローラ層
├── services/        # サービス層
├── repositories/    # リポジトリ層
├── models/          # モデル層
└── db/
    └── migrations/  # マイグレーションファイル
```

### 依存関係のルール

各層は以下の方向にのみ依存する。外側の層から内側の層への依存は禁止。

```
controllers → services → repositories → models
```

- `controllers/`: HTTPリクエストの受け取りとレスポンス返却のみ担当。ビジネスロジックを書かない
- `services/`: ビジネスロジックを担当。`controllers/` にのみ呼び出される
- `repositories/`: DB操作を担当。`services/` にのみ呼び出される
- `models/`: データ構造（GORMモデル）を定義。`repositories/` にのみ呼び出される
- `initializers/`: サーバ起動時の初期化処理（ルーティング、DI、DB接続など）をまとめる

---

## APIレスポンス形式

すべてのAPIレスポンスは以下のJSON構造に統一する。

```json
{
  "status": "ok",
  "data": { ... },
  "error": "エラーメッセージ",
}
```

- `status`: 正常終了は `"ok"`、エラー発生時は `"ng"`
- `data`: 各エンドポイントの処理結果オブジェクト。エラー発生時は存在しない。
- `error`: エラー発生時のみ含める。エラーの内容をstringで返す。
- HTTPステータスコードも適切に設定する（200, 400, 404, 500 等）

---

## 認証

- 認証方式はセッション方式を採用する
- セッションIDはサーバ側で生成し、レスポンスのSet-CookieヘッダーでクライアントにCookieとして渡す
- セッション情報はサーバ側で管理する

---

## CORS設定
`github.com/gin-contrib/cors`を利用し、以下の内容で設定すること
```
Access-Control-Allow-Origin: (環境変数ACCEPT_ORIGINの値を入れる)
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Accept, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization
Access-Control-Allow-Credentials: true
```

---

## 運用ルール
- logsディレクトリ以下に、アクセスログ、エラーログをそれぞれ環境別にファイルを分けて出力する(例: access.development.log, error.production.log)
- 開発環境ではSQLiteを利用する。本番環境ではmariaDBを利用し、環境変数(DB_HOST, DB_USER, DB_NAME, DB_PASS)を使い接続する。
- GORMの `AutoMigrate` は使用しない。スキーマ変更はすべて golang-migrate で管理する。
- マイグレーションファイルは `db/migrations/` ディレクトリに配置する。
- マイグレーションファイルの命名規則: `{version}_{description}.up.sql` / `{version}_{description}.down.sql`
  - バージョン番号は6桁のゼロ埋め連番（例: `000001_create_tasks.up.sql`）
  - `up.sql` にはスキーマ変更の適用、`down.sql` にはロールバック処理を記述する

---

## 環境変数

- 環境変数は godotenv を使用し、プロジェクトルートの `.env` ファイルから読み込む。
- 以下の環境変数を利用する
  - `APP_ENV`: アプリケーションの起動環境。`development`、`production`、`test`など
  - `PORT`: APIサーバのリクエストを受け付けるポート番号。
  - `ACCEPT_ORIGIN`: CORS設定でAccept-Originに適用する
