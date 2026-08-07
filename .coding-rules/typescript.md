# TypeScript言語設定

- Reactを使用しアプリのフロントエンドを作成します。

---

## 必須事項

- **strict モード**を必ず有効にする
- `any` は使わない。不明な型は `unknown` を使う
- `interface` を優先、union/intersection には `type` を使う
- 非同期処理は `async/await`（コールバックは避ける）

---

## 技術スタック

- ルーティング: TanStack Router
- バンドラ: `Vite`
- パッケージマネージャー: `pnpm`（なければ `npm`）
- UIコンポーネント / CSSフレームワーク: Mantine
- 状態管理: Context API
- APIクライアント: TanStack Query
- スキーマバリデーション: Zod
- フォーマッター: Prettier
- リンター: ESLint / Biome（プロジェクトに合わせる）
- テスト: Vitest / Jest

---

## APIレスポンス形式

すべてのAPIレスポンスは以下のJSON構造に統一されている。

```typescript
type ApiResponse<T> = {
  status: "ok" | "ng";
  data: T;
};
```

- `status` が `"ok"` のとき正常、`"ng"` のときエラー
- エラー時の `data` は `{ message: string }` 形式
- API呼び出し時は必ず `status` を確認してからデータを使用する

---

## 運用ルール

- コンポーネント名はPascalケースを採用する
- 関数コンポーネントをexportするファイルは以下のルールを適用する。
  - ファイル名をコンポーネントと同名にする。
  - Propsなど外部で利用する際に必要な情報をコンポーネントと同名の名前空間でアクセスできるようexportする。

---

## ディレクトリ構成

`src/` 以下を以下の構成で管理する。

```
src/
├── layouts/     # ヘッダー・フッターなど共通レイアウトコンポーネント
├── components/  # ツールチップなど汎用的なオリジナルコンポーネント
└── pages/       # ページ単位のディレクトリ
    └── {PageName}/
        ├── index.tsx        # ページのレイアウト
        └── {ComponentName}.tsx  # そのページ固有のコンポーネント
```

- `layouts/` と `components/` は複数ページで再利用されるものを置く
- 特定のページにしか使わないコンポーネントは `pages/{PageName}/` 内に置く

---

## 環境変数

- 環境変数はプロジェクトルートの `.env` ファイルで管理する
- フロントエンドは `frontend/` ディレクトリに配置されるため、`vite.config.ts` に `envDir` を設定してプロジェクトルートを参照する

```typescript
// vite.config.ts
export default defineConfig({
  envDir: "../", // プロジェクトルートの .env を読み込む
  // ...
});
```

- Viteの環境変数はプレフィックス `VITE_` を付ける（例: `VITE_API_URL`）
- コード内では `import.meta.env.VITE_XXX` でアクセスする

---

## 認証

- 認証にはセッション方式を使用する
- セッションIDはCookieに保存する（サーバから `Set-Cookie` で受け取る）
- APIリクエスト時は `credentials: "include"` を指定し、Cookieを自動送信する

---

## TanStack Query

queryKeyはURLのパスを1階層ごとに文字列として渡し、パラメータがある場合は最後にオブジェクトとして渡す。

```typescript
// パラメータなし
useQuery({ queryKey: ["tasks"] })
useQuery({ queryKey: ["tasks", taskId] })

// パラメータあり
useQuery({ queryKey: ["tasks", { status: "active" }] })
useQuery({ queryKey: ["tasks", taskId, "comments", { page: 1 }] })
```

---

## フォーム

- フォームの管理には Mantine の `useForm` を使用する
- バリデーションエラーも `useForm` のエラーハンドリング機能で管理する
- 送信処理は `async/await` で直接非同期通信を行う
