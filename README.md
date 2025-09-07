# todo-contract（OpenAPI コントラクト専用リポジトリ）

Todo API（OpenAPI 3.0）の単一の仕様書（SSOT: Single Source of Truth）を管理します。CI では Spectral による Lint を実施し、PR に問題がある場合はマージをブロックします。ローカルでは Redocly CLI を使ってドキュメントをプレビューできます。

（背景として、openapiからバックエンド・フロントエンドを自動生成して、スキーマ駆動開発がどれくらい良いものなのかその一端を確かめるために作っている）

## 目的・方針

- APIコントラクト（OpenAPI）の管理
  - バックエンド・フロントエンドはこのリポからopenapiをダウンロードする
  - また、openapiとアプリケーションの実装の差分検知は各リポジトリでCIを回し担保する
- openapiの品質を Lint (CI実行) で担保
- redoc cliによるローカルでの読みやすいプレビュー

## なぜコントラクトを独立リポジトリにするのか

- リリースを実装から分離できる
  - 仕様はタグで配布（Raw URL を各クライアントが固定参照）。サーバ・クライアントの実装進度に依存しない。
- 複数クライアント／多言語への配布が容易
  - どの技術スタックでも同じ URL を取得してコード生成・テストに利用できる。
- セキュリティ境界が明確
  - 実装コードや秘密情報を含まず、公開可否の判断や社外共有がしやすい。
- ツールチェーンが軽量
  - 実装依存を持たず、Lint とプレビューだけの最小セットで運用可能。

補足（トレードオフ）

- 破壊的変更の検知・契約テストはクライアント側 CI でも補完する


## ディレクトリ構成（抜粋）

- `openapi/todo.yaml`: 仕様本体
- `.spectral.json`: Spectral ルール
- `.github/workflows/spectral.yml`: CI（PR で Lint ＋ PR に注釈）
- `docker-compose.yml`: Lint 用（contract）・プレビュー用（redoc）
- `Dockerfile`: Node 20 の軽量環境

## 使い方

### 1) 仕様の取得（クライアントから参照）

- 最新（main）
  - `https://raw.githubusercontent.com/yaharu711/todo-contract/main/openapi/todo.yaml`

### 2) ローカルで Lint

Docker で実行:

```bash
docker compose run --rm contract npm run lint
```

### 3) ローカルでドキュメントをプレビュー

```bash
docker compose up -d redoc
# ブラウザ: http://localhost:8082
```

## 使用しているライブラリ／ツールと採用理由

- `@stoplight/spectral-cli`

  - 役割: OpenAPI Lint（CI ゲート）
  - 理由: 軽量・設定がシンプルで、最低限の品質担保に最適。デフォルトが redocly ほど厳しくない

- `reviewdog`

  - 役割: Lint 結果を PR に注釈（reporter=github-pr-review）
  - 理由: 差分行にピンポイントでコメントでき、開発者体験が良い。

- `@redocly/cli`（preview-docs 相当機能／v1 系）

  - 役割: ローカル環境に OpenAPI ドキュメントのプレビューを表示
    - https://redocly.com/docs/cli/v1/commands/preview-docs
  - 理由: 旧 `redoc-cli` は非推奨。保存 → 自動反映ができ、運用が安定。
    - https://www.npmjs.com/package/redoc-cli
  - 備考: docker-compose の `redoc` サービスで実行します。

- GitHub Actions
  - 役割: PR 作成・更新時の自動 Lint 実行
  - 理由: 「落ちたらマージ不可」を自動化して担保。

---
