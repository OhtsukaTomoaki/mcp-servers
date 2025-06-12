# mcp-servers

MCPサーバーチュートリアルプロジェクト

このプロジェクトは、Model Context Protocol (MCP) サーバーを試すためのチュートリアル環境です。MCPを使用することで、AIモデルと外部サービスを連携させ、AIアシスタントの能力を大幅に向上させることができます。

## MCPとは

Model Context Protocol（MCP）は、データソースとAIアシスタントツールの間で安全な双方向通信を実現するためのオープンスタンダードです。開発者はMCPサーバーを公開し、これらのサーバーに接続するAIアプリケーション（MCPクライアント）を作成できます。

MCPを使用することで、以下のようなことが可能になります：
- 外部データソースへのアクセス（Confluence、GitHub、データベースなど）
- APIを通じた外部サービスとの連携
- AIアシスタントの能力拡張

## 含まれるMCPサーバー

このプロジェクトには、以下の3つのMCPサーバーが含まれています：

### 1. mcp-atlassian

Atlassian製品（ConfluenceとJira）と連携するためのMCPサーバーです。

**主な機能：**
- Confluenceページの取得と検索
- Confluenceページのコメント取得
- Confluenceページのラベル取得
- Jiraイシューの取得と検索

### 2. mcp-github

GitHubリポジトリと連携するためのMCPサーバーです。

**主な機能：**
- リポジトリ内のファイル内容取得
- イシュー一覧の取得
- 特定のイシューの詳細取得
- コード検索
- プルリクエスト管理

### 3. mcp-slack

Slackワークスペースと連携するためのMCPサーバーです。

**主な機能：**
- ワークスペース内のチャンネル一覧取得
- チャンネルのメッセージ履歴取得
- スレッドの返信取得
- チャンネルへのメッセージ投稿
- ユーザー情報の取得

## セットアップ手順

### 1. 環境変数の設定

各MCPサーバーに必要な環境変数を設定します：

1. `.env`ファイルをテンプレートとして使用
   ```bash
   # .env.localファイルを作成
   cp .env .env.local
   ```

2. `.env.local`ファイルを編集して、実際のAPI情報を設定
   ```bash
   # エディタで.env.localを開く
   vim .env.local
   ```

3. `.env.local`ファイルの内容例：
   ```bash
   # Atlassian API設定
   CONFLUENCE_URL=https://your-domain.atlassian.net/wiki
   CONFLUENCE_API_TOKEN=YOUR_TOKEN
   CONFLUENCE_USERNAME=YOUR_EMAIL
   JIRA_URL=https://your-domain.atlassian.net
   JIRA_API_TOKEN=YOUR_TOKEN
   JIRA_USERNAME=YOUR_EMAIL

   # GitHub API設定
   GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_TOKEN

   # Slack
   SLACK_TEAM_ID=YOUR_TEAM_ID
   SLACK_CHANNEL_IDS=YOUR_CHANNEL_IDS
   SLACK_BOT_TOKEN=YOUR_BOT_TOKEN
   ```

> **注意**: `.env`ファイルはデフォルト値やテンプレートとしてバージョン管理に含め、実際のシークレット情報は`.env.local`に保存します。これにより、機密情報をリポジトリにコミットするリスクを避けることができます。

### 2. Dockerを使用したMCPサーバーの実行

```bash
# Dockerコンテナを起動
docker-compose up -d

# 特定のサーバーのみ起動する場合
docker-compose up -d mcp-atlassian
docker-compose up -d mcp-github
docker-compose up -d mcp-slack

# ログの確認
docker-compose logs -f

# サーバーの停止
docker-compose down
```

### 3. Cursor MCP設定ファイルの作成

Cursorエディタで使用するためのMCP設定ファイルを作成します。`~/.cursor/mcp.json`ファイルに以下の設定を追加します：

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--net=host",
        "-e",
        "CONFLUENCE_URL=${CONFLUENCE_URL}",
        "-e",
        "CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME}",
        "-e",
        "CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN}",
        "ghcr.io/sooperset/mcp-atlassian:latest"
      ],
      "alwaysAllow": [
        "confluence_get_page_children",
        "confluence_get_comments",
        "confluence_get_labels",
        "confluence_get_page",
        "confluence_add_comment"
      ]
    },
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN=${GITHUB_PERSONAL_ACCESS_TOKEN}",
        "ghcr.io/github/github-mcp-server:latest"
      ],
      "alwaysAllow": [
        "add_issue_comment",
        "add_pull_request_review_comment_to_pending_review",
        "create_branch",
        "create_issue",
        "create_pending_pull_request_review",
        "create_pull_request",
        "get_code_scanning_alert",
        "get_commit",
        "get_file_contents",
        "get_issue",
        "get_issue_comments",
        "get_me",
        "get_notification_details",
        "get_pull_request",
        "get_pull_request_comments",
        "get_pull_request_diff",
        "get_pull_request_files",
        "get_pull_request_reviews",
        "get_pull_request_status",
        "get_secret_scanning_alert",
        "get_tag",
        "list_branches",
        "list_code_scanning_alerts",
        "list_commits",
        "list_issues",
        "list_notifications",
        "list_pull_requests",
        "list_secret_scanning_alerts",
        "list_tags",
        "search_code",
        "search_issues",
        "search_repositories",
        "search_users",
        "update_issue",
        "update_pull_request",
        "update_pull_request_branch"
      ]
    },
    "mcp-slack": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "SLACK_TEAM_ID",
        "-e",
        "SLACK_CHANNEL_IDS",
        "-e",
        "SLACK_BOT_TOKEN",
        "mcp/slack"
      ],
      "env": {
        "SLACK_TEAM_ID": "YOUR_TEAM_ID",
        "SLACK_CHANNEL_IDS": "CHANNEL_ID_1,CHANNEL_ID_2,CHANNEL_ID_3",
        "SLACK_BOT_TOKEN": "YOUR_BOT_TOKEN"
      },
      "alwaysAllow": [
        "slack_list_channels"
      ]
    }
  }
}
```

> **重要**:
> - 設定ファイルは `~/.cursor/mcp.json` (ユーザーホームディレクトリ) に配置してください
> - 設定変更後はCursorを完全に再起動する必要があります (`Cmd + Q` で終了後、再起動)
> - Dockerが起動していることを確認してください

## 各MCPサーバーの設定方法

### Atlassian MCP

#### 必要な認証情報

1. **Confluenceアクセストークンの取得:**
   - Atlassianアカウントの設定画面にアクセス (https://id.atlassian.com/manage-profile/security/api-tokens)
   - API tokens → Create API token
   - 生成されたトークンを `.env.local` の `CONFLUENCE_API_TOKEN` に設定

2. **設定値の例:**
   - `CONFLUENCE_URL`: `https://your-domain.atlassian.net/wiki`
   - `CONFLUENCE_USERNAME`: あなたのメールアドレス
   - `CONFLUENCE_API_TOKEN`: 上記で生成したAPIトークン
   - `JIRA_URL`: `https://your-domain.atlassian.net`
   - `JIRA_API_TOKEN`: Confluenceと同じトークンを使用可能

#### 使用例

```
# 例: 特定のConfluenceページから情報を取得
@confluence-page-id:12345 の記事をもとに実行可能なコードを生成してください

# 例: 技術仕様書の内容確認
@confluence の技術仕様書でユーザーフローについて確認してください
```

### GitHub MCP

#### 必要な認証情報

1. **GitHubパーソナルアクセストークンの取得:**
   - GitHubの Settings → Developer settings → Personal access tokens → Tokens (classic) (https://github.com/settings/tokens)
   - 必要なスコープ: `repo`, `read:org`, `read:user`
   - 生成されたトークンを `.env.local` の `GITHUB_PERSONAL_ACCESS_TOKEN` に設定

#### 使用例

```
# 例: リポジトリ内のファイル検索
このリポジトリ内でAPIエンドポイントを実装しているファイルを探してください

# 例: コードレビュー支援
このプルリクエストのコードをレビューして、改善点を指摘してください

# 例: イシュー分析
最近のバグ報告イシューを分析して、共通の原因を特定してください
```

### Slack MCP

#### 必要な認証情報

1. **Slack Appの作成:**
   - Slack API の管理画面にアクセス (https://api.slack.com/apps)
   - "Create New App" → "From scratch" を選択
   - App名とワークスペースを指定して作成

2. **Bot Tokenの取得:**
   - 作成したAppの "OAuth & Permissions" セクションに移動
   - "Scopes" → "User OAuth Token" で必要な権限を追加:
     - `channels:read` - パブリックチャンネル情報の読み取り
     - `channels:history` - パブリックチャンネル履歴の読み取り
     - `groups:read` - プライベートチャンネル情報の読み取り
     - `groups:history` - プライベートチャンネル履歴の読み取り
     - `chat:write` - メッセージの投稿
     - `reactions:write` - リアクションの追加
     - `users:read` - ユーザー情報の読み取り
   - "Install to Workspace" でワークスペースにインストール
   - "Bot User OAuth Token" をコピー（`xoxb-` で始まるトークン）

3. **Team IDの取得:**
   - ワークスペースの設定画面で確認
   - または Slack API の `auth.test` メソッドで取得可能

4. **Channel IDの取得:**
   - Slackアプリで対象チャンネルを右クリック → "View channel details"
   - 最下部にChannel IDが表示される
   - または、チャンネルURLの末尾部分（例：`/messages/C12345678`の `C12345678`）

#### 使用例

```
# 例: 特定チャンネルの最新の議論を要約
dev_teamチャンネルの最近のスレッドを要約してください

# 例: プロジェクトの進捗確認
developmentチャンネルの今週の活動を分析して、プロジェクトの進捗状況をまとめてください

# 例: チームメンバーの活動確認
全体チャンネルの最新メッセージから、チームの現在の作業状況を把握してください

# 例: インシデント対応の追跡
incidentsチャンネルの最新のインシデント情報と対応状況を確認してください
```

## トラブルシューティング

### Docker MCPサーバーのトラブルシューティング

```bash
# Dockerコンテナの状態確認
cd mcp-servers
docker-compose ps

# コンテナのログ確認
docker-compose logs -f mcp-atlassian
docker-compose logs -f mcp-github
docker-compose logs -f mcp-slack

# コンテナの再起動
docker-compose restart mcp-atlassian
docker-compose restart mcp-github
docker-compose restart mcp-slack

# 環境変数の確認
cat .env.local

# Dockerコンテナの削除と再作成
docker-compose down
docker-compose up -d
```

### Docker自体の問題

```bash
# Dockerデーモンの状態確認
docker info

# Dockerデーモンの再起動
# macOS
killall Docker && open /Applications/Docker.app

# Linux
sudo systemctl restart docker
```

## 参考リソース

- [Model Context Protocol (MCP) 公式ドキュメント](https://github.com/anthropics/anthropic-cookbook/tree/main/mcp)
- [Cursor エディタ](https://cursor.sh/)
- [Atlassian API ドキュメント](https://developer.atlassian.com/cloud/confluence/rest/v1/intro/)
- [GitHub API ドキュメント](https://docs.github.com/en/rest)
- [Slack API ドキュメント](https://api.slack.com/docs)
