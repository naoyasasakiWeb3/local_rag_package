# Local AI Package - グローバル環境とDocker環境を併用したRAGシステム構築ガイド

## システム概要

このプロジェクトは、複合的なデータ（テキスト、PDF、CSVなど）を効率的にデータベース化し、既存のLLMモデルを強化できるRAG（Retrieval Augmented Generation）システムを構築するためのテンプレートです。

### 特徴
- グローバル環境のn8nとOllamaを活用
- Dockerコンテナによる各種サービス（Supabase、Qdrant等）の管理
- 複数のデータ形式に対応したRAGシステム
- ワークフロー自動化による効率的なデータ処理

## 前提条件

### 1. グローバル環境の準備

#### n8nのインストール
```bash
# npmを使用してn8nをグローバルにインストール
npm install n8n -g

# n8nの起動（デフォルトポート:5678）
n8n start
```
詳細は[n8n公式ドキュメント](https://docs.n8n.io/hosting/installation/npm/)を参照してください。

#### Ollamaのインストール
```bash
# macOSの場合
brew install ollama

# Linuxの場合
curl -fsSL https://ollama.com/install.sh | sh
```
詳細は[Ollama公式ドキュメント](https://ollama.com/download)を参照してください。

#### 必要なモデルのインストール
```bash
# RAG対応のLLMモデル
ollama pull mistral

# Embedding用モデル
ollama pull nomic-embed-text
```

### 2. 環境構築手順

1. リポジトリのクローン
```bash
git clone https://github.com/naoyasasakiWeb3/local_rag_package.git
cd local_rag_package
```

2. 環境変数の設定
```bash
# .env.exampleをコピーして.envを作成
cp .env.example .env

# .envファイルを編集
nano .env
```
環境変数の詳細な設定方法は[README.md](./README.md)を参照してください。

3. サービスの起動
```bash
# Pythonスクリプトによるサービス起動
python start_services.py
```

### 3. n8nクレデンシャルの設定

#### OpenAI (Ollama) の設定
- Base URL: `http://127.0.0.1:11434/v1`
- API Key: 任意の値（ローカル環境のため）

#### PostgreSQL (Supabase) の設定
- Host: `db`（Dockerネットワーク内での接続）
- Database: `postgres`
- User: `postgres.{POOLER_TENANT_ID}`
- Password: `.env`ファイルで設定した値
- Port: `5434`（Supavisor経由）

#### Google Drive の設定
Google DriveのOAuth認証設定については、[n8nの公式ガイド](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)を参照してください。

## n8nノードの設定

### 1. OpenAI (Ollama) ノード
- 作成したクレデンシャルを選択
- モデル選択: `mistral`（または他のRAG対応モデル）
- 必要に応じてシステムプロンプトを調整

### 2. PostgreSQLノード
- 作成したクレデンシャルを選択
- その他のパラメータはデフォルト設定で動作

### 3. Google Driveノード
- 作成したクレデンシャルを選択
- フォルダID設定が必要
  - フォルダIDの取得方法は[Google Drive公式ガイド](https://developers.google.com/drive/api/guides/about-files)を参照
  - Google DriveのフォルダURLから直接取得可能
    ```
    例: https://drive.google.com/drive/folders/[folder-id]
    ```

## データベースの初期設定

1. **メタデータテーブルの作成**
   - n8nワークフロー内の`Create Document Metadata Table`ノードを実行
   - ドキュメントのメタ情報を管理するテーブルが作成される

2. **ドキュメントデータテーブルの作成**
   - n8nワークフロー内の`Create Document Rows Table`ノードを実行
   - 実際のドキュメントデータを管理するテーブルが作成される

## RAGシステムの運用方法

### データの自動更新
- Google Drive指定フォルダ内のデータは1分間隔で監視
- 新規追加・更新されたファイルは自動的にRAGシステムに反映
- 更新プロセス：
  1. ファイル変更の検知
  2. データの抽出とチャンク分割
  3. Embeddings生成
  4. ベクトルDBへの保存

### システムの利用
1. n8nのチャットインターフェースにアクセス
2. 質問を入力
3. システムが以下の処理を実行：
   - 関連データのベクトル検索
   - コンテキストを考慮したLLMによる回答生成
   - ソース情報の提示

## 注意事項

このシステムは[n8n-io/self-hosted-ai-starter-kit](https://github.com/n8n-io/self-hosted-ai-starter-kit)をベースに、グローバル環境とDocker環境を併用するように最適化されています。

## 参照ドキュメント一覧

1. **n8n関連**
   - [n8nインストールガイド](https://docs.n8n.io/hosting/installation/npm/)
   - [Google認証設定](https://docs.n8n.io/integrations/builtin/credentials/google/)
   - [PostgreSQL接続ガイド](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.postgres/)

2. **Ollama関連**
   - [インストールガイド](https://ollama.com/download)
   - [APIリファレンス](https://ollama.com/docs/api)
   - [モデル管理](https://ollama.com/docs/models)

3. **Supabase関連**
   - [セルフホスティングガイド](https://supabase.com/docs/guides/self-hosting/docker)
   - [データベース設定](https://supabase.com/docs/guides/database)

4. **RAG開発関連**
   - [ベクトルデータベース概要](https://docs.n8n.io/advanced-ai/examples/understand-vector-databases/)
   - [AIエージェント開発ガイド](https://docs.n8n.io/advanced-ai/langchain/langchain-n8n/) 