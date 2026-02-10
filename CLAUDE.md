# CLAUDE.md - piko-pulse ネットワーク監視ダッシュボード

## プロジェクト概要
ローカルネットワーク向けのネットワーク監視ダッシュボード。
バックエンドでping/speedtest-cliなどのコマンドを定期実行し、結果をPostgreSQLに保存、フロントエンドで時系列折れ線グラフとして可視化する。

## 開発方針
- **学習用プロジェクト**: 開発者本人が手を動かして学ぶことが目的
- **Claudeの役割**: コードを代わりに書くのではなく、以下に徹する
  - ロードマップに沿った進め方のガイド
  - 技術的な質問への回答
  - 書いたコードのレビュー
  - エラーやつまずきへのヒント提示
- **禁止事項**: ユーザーに頼まれない限り、Claudeが自らコードを書いて実装を進めてはならない
- **進め方**: `plan/overview.md` のロードマップに従い、ステップを順に進める。詰まったらClaudeに相談する

## 関連ドキュメント
- **ロードマップ**: [plan/overview.md](plan/overview.md) - 全ステップの詳細と完了条件

## 技術スタック

### バックエンド
- ランタイム: Node.js
- フレームワーク: Hono + TypeScript
- ORM: Drizzle
- データベース: PostgreSQL
- 定期実行: node-cron
- バリデーション: Zod
- 環境変数: dotenv
- テスト: Vitest + Playwright

### フロントエンド
- ビルドツール: Vite
- フレームワーク: React + TypeScript
- UIライブラリ: Ant Design
- グラフ: @ant-design/charts
- データフェッチング/状態管理: TanStack Query

### インフラ
- コンテナ: Docker Compose
- 開発環境: devcontainer

## アーキテクチャ

### 構成
モノレポ構成で、3つのプロセスが連携する:
1. **API サーバー** (Hono) - REST APIを提供
2. **ワーカー** (別プロセス) - node-cronで監視コマンドを定期実行し、結果をDBに保存
3. **フロントエンド** (React SPA) - APIからデータ取得しグラフ描画

### データフロー
```
ワーカー → コマンド実行(ping/speedtest) → PostgreSQL保存
フロントエンド → API問い合わせ → グラフ描画
```

## ディレクトリ構成
```
piko-pulse/
├── backend/
│   ├── src/
│   │   ├── api/           # Hono APIルート
│   │   │   └── routes/
│   │   ├── worker/        # 定期実行ワーカー
│   │   │   ├── commands/  # 各コマンドの実行ロジック (ping, speedtest等)
│   │   │   └── scheduler.ts
│   │   ├── db/
│   │   │   ├── schema.ts  # Drizzleスキーマ定義
│   │   │   └── migrate.ts
│   │   ├── shared/        # API/ワーカー共有の型・ユーティリティ
│   │   └── index.ts       # APIサーバーエントリポイント
│   ├── drizzle/           # マイグレーションファイル
│   ├── package.json
│   └── tsconfig.json
├── frontend/
│   ├── src/
│   │   ├── components/    # UIコンポーネント
│   │   ├── pages/         # ページコンポーネント
│   │   ├── hooks/         # カスタムフック (TanStack Query)
│   │   ├── api/           # APIクライアント
│   │   ├── types/         # 型定義
│   │   └── App.tsx
│   ├── package.json
│   └── tsconfig.json
├── docker-compose.yml
├── .devcontainer/
├── CLAUDE.md
└── architecture.md
```

## 機能要件

### MVP (Phase 1)
- [x] 監視対象コマンド: ping, speedtest-cli
- [x] 監視対象ホスト: UIから動的に追加・削除
- [x] 実行間隔: UIから設定可能
- [x] データ表示: 時系列折れ線グラフ (@ant-design/charts)
- [x] データ保持期間: UIまたは設定で変更可能
- [x] 認証: なし (ローカルネットワーク前提)
- [x] アラート: なし (グラフ表示のみ)

### 将来拡張 (Phase 2以降, 未定)
- 実行タイミングの詳細指定 (開始/終了時刻)
- 追加コマンドの実行対応
- WebSocketによるリアルタイム通知
- 外部通知 (Slack等)

## データモデル (概要)

### targets テーブル
監視対象ホストの管理
- id, name, host, is_active, created_at, updated_at

### monitor_jobs テーブル
監視ジョブの定義 (コマンド種別 + 対象 + 間隔)
- id, target_id, command_type (ping/speedtest), interval_seconds, is_active, created_at, updated_at

### ping_results テーブル
ping実行結果
- id, job_id, target_id, latency_ms, packet_loss, is_success, executed_at

### speedtest_results テーブル
speedtest実行結果
- id, job_id, download_mbps, upload_mbps, ping_ms, is_success, executed_at

### settings テーブル
アプリ全体の設定 (データ保持期間など)
- id, key, value, updated_at

## API設計 (概要)

### 監視対象
- `GET    /api/targets`        - 一覧取得
- `POST   /api/targets`        - 追加
- `PUT    /api/targets/:id`    - 更新
- `DELETE /api/targets/:id`    - 削除

### 監視ジョブ
- `GET    /api/jobs`           - 一覧取得
- `POST   /api/jobs`           - 作成
- `PUT    /api/jobs/:id`       - 更新
- `DELETE /api/jobs/:id`       - 削除

### 結果データ
- `GET /api/results/ping?targetId=&from=&to=`       - ping結果取得
- `GET /api/results/speedtest?from=&to=`             - speedtest結果取得

### 設定
- `GET /api/settings`          - 設定取得
- `PUT /api/settings`          - 設定更新

## 開発コマンド

```bash
# 起動 (Docker Compose)
docker compose up -d

# バックエンド開発
cd backend && npm run dev

# フロントエンド開発
cd frontend && npm run dev

# テスト
cd backend && npm test
cd frontend && npm test

# DBマイグレーション
cd backend && npm run db:migrate
```

## コーディング規約
- 言語: TypeScript (strict mode)
- フォーマッター/リンター: 導入時に決定 (ESLint + Prettier推奨)
- コミットメッセージ: 日本語OK
- コメント: 日本語OK
