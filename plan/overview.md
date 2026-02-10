# piko-pulse ロードマップ

各ステップで「何を学ぶか」「完了条件」を明確にし、順番に進める。

## Step 0: 開発環境構築
**学ぶこと**: Docker Compose, devcontainer, モノレポのプロジェクト初期化
- [ ] Docker Compose で PostgreSQL を起動できる
- [ ] devcontainer の設定を作成し、コンテナ内で開発できる
- [ ] `backend/` と `frontend/` の package.json を作成し、TypeScript が動く状態にする
- [ ] ESLint + Prettier を導入する

**完了条件**: `docker compose up` でDBが立ち上がり、backend/frontend それぞれで `npm run dev` が通る

## Step 1: バックエンド基礎 - Hono + DB接続
**学ぶこと**: Hono の基本, Drizzle ORM, DBマイグレーション
- [ ] Hono で Hello World API を作る (`GET /api/health`)
- [ ] Drizzle でスキーマ定義 (まずは `targets` テーブルのみ)
- [ ] マイグレーション実行してテーブルが作られることを確認
- [ ] targets の CRUD API を実装 (`GET/POST/PUT/DELETE /api/targets`)
- [ ] Zod でリクエストバリデーションを入れる

**完了条件**: curl や REST クライアントで targets の CRUD 操作ができる

## Step 2: フロントエンド基礎 - React + Ant Design
**学ぶこと**: Vite + React セットアップ, Ant Design の使い方, TanStack Query
- [ ] Vite で React + TypeScript プロジェクトを作成
- [ ] Ant Design を導入し、レイアウト (サイドバー + メインエリア) を組む
- [ ] 監視対象の一覧ページを作る (テーブル表示)
- [ ] 監視対象の追加フォーム (モーダル) を作る
- [ ] TanStack Query で バックエンドAPI と接続

**完了条件**: UIから監視対象を追加・一覧表示・削除でき、DBに反映される

## Step 3: pingコマンドの実行と保存
**学ぶこと**: Node.js の child_process, ワーカープロセス, node-cron
- [ ] Node.js から ping コマンドを実行し、結果をパースするモジュールを作る
- [ ] `ping_results` テーブルのスキーマ定義とマイグレーション
- [ ] ワーカープロセスのエントリポイントを作る
- [ ] node-cron で定期的に ping を実行して結果を DB に保存
- [ ] Vitest でコマンドパース部分のユニットテストを書く

**完了条件**: ワーカーを起動すると、定期的にpingが実行され、DBにレコードが増えていく

## Step 4: グラフ表示
**学ぶこと**: @ant-design/charts, 時系列データの扱い, クエリパラメータ
- [ ] ping 結果取得 API を実装 (`GET /api/results/ping`)
- [ ] 期間指定 (from/to) のクエリパラメータに対応
- [ ] フロントエンドで @ant-design/charts の折れ線グラフを表示
- [ ] 期間選択 UI (直近1時間 / 24時間 / 7日間 など) を作る

**完了条件**: UIでpingのレイテンシを時系列グラフとして確認できる

## Step 5: 監視ジョブの管理UI
**学ぶこと**: 複数テーブルの連携, フォーム設計
- [ ] `monitor_jobs` テーブルのスキーマ定義とマイグレーション
- [ ] ジョブ CRUD API を実装
- [ ] UIからジョブを作成 (対象ホスト・コマンド種別・実行間隔を指定)
- [ ] ワーカーがジョブ定義に従って動的にスケジュール実行するように改修

**完了条件**: UIからジョブを作成・変更すると、ワーカーのスケジュールが反映される

## Step 6: speedtest-cli 対応
**学ぶこと**: 新しいコマンド追加のパターン, コマンド抽象化
- [ ] speedtest-cli の実行・パースモジュールを作る
- [ ] `speedtest_results` テーブルのスキーマ定義とマイグレーション
- [ ] speedtest 結果取得 API を実装
- [ ] フロントエンドにダウンロード/アップロード速度のグラフを追加

**完了条件**: UIでspeedtestの結果をグラフ確認できる

## Step 7: 設定機能とデータ保持期間
**学ぶこと**: アプリケーション設定の管理パターン
- [ ] `settings` テーブルのスキーマ定義とマイグレーション
- [ ] 設定 API を実装
- [ ] 設定画面 UI を作る (データ保持期間の変更)
- [ ] ワーカーに古いデータの自動削除処理を追加

**完了条件**: 設定画面から保持期間を変更でき、古いデータが自動削除される

## Step 8: 仕上げ
**学ぶこと**: E2Eテスト, Docker本番ビルド
- [ ] Playwright で主要フローの E2E テストを書く
- [ ] フロントエンドの本番ビルドをバックエンドから配信する構成にする
- [ ] Docker Compose で全サービス (DB + backend + frontend) を一発起動できるようにする
- [ ] README.md を整備する

**完了条件**: `docker compose up` だけで全機能が動く状態になる
