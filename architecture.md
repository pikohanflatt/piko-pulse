# ネットワーク監視ダッシュボード - 技術スタック

## バックエンド
- **ランタイム**: Node.js
- **フレームワーク**: Hono + TypeScript
- **ORM**: Drizzle
- **データベース**: PostgreSQL
- **定期実行**: node-cron
- **バリデーション**: Zod
- **環境変数**: dotenv
- **リアルタイム通信**: WebSocket (Hono)
- **テスト**: Vitest + Playwright

## フロントエンド
- **ビルドツール**: Vite
- **フレームワーク**: React + TypeScript
- **UIライブラリ**: Ant Design
- **グラフ**: @ant-design/charts
- **データフェッチング**: TanStack Query
- **状態管理**: TanStack Query のみ

## インフラ
- **コンテナ**: Docker Compose
- **開発環境**: devcontainer

## アーキテクチャ概要
- **構成**: バックエンドAPI + ワーカー（別プロセス）+ フロントエンド（SPA）
- **データフロー**: 
  - ワーカーが定期的にping/speedtest実行 → PostgreSQLに保存
  - フロントエンドがAPIから取得 → グラフ表示
  - 異常検知時はWebSocketでリアルタイム通知
