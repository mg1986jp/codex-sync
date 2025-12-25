# codex-sync

複数のMac間でCodex CLIの設定を共有するためのリポジトリです。

## このリポジトリで何ができるか

- **複数Mac間での設定共有**: 別のMacでも同じCodex環境をすぐに構築できます
- **権限設定の自動化**: 危険なコマンド（`rm -rf /`、`git push --force`等）を自動的にブロックします
- **確認プロンプト無効化**: 信頼済みプロジェクトでの確認プロンプトをスキップできます
- **ブラウザ操作機能**: MCP Puppeteerにより、Codexがブラウザを操作できるようになります
- **AI指示の統一**: コミットメッセージ形式や開発方針をCodex全体で統一できます

## セットアップ

### 前提条件

以下のソフトウェアが必要です。未インストールの場合は先にインストールしてください。

#### 必須
- **Codex CLI** - このリポジトリの設定を適用するために必要です
  - インストール: [公式ガイド](https://developers.openai.com/codex/cli)
  - 確認方法: `codex --version`

#### オプション（ブラウザ操作機能を使う場合）
- **Node.js / npm** - MCP Puppeteerに必要です
  - インストール: [公式サイト](https://nodejs.org/)
  - 確認方法: `npm --version`

### 1. 既存設定のバックアップ

既に`~/.codex`ディレクトリが存在する場合、上書きされてしまうため先にバックアップします。

```bash
# ~/.codexが存在する場合のみ実行してください
if [ -d ~/.codex ]; then
  mv ~/.codex ~/.codex_backup_$(date +"%Y%m%d%H%M%S")
  echo "既存の~/.codexをバックアップしました"
fi
```

### 2. リポジトリのクローン

このリポジトリを`~/.codex`にクローンします。

```bash
git clone git@github.com:mg1986jp/codex-sync.git ~/.codex
cd ~/.codex
```

**結果**: `~/.codex`に設定ファイル（config.toml、rules/default.rules、AGENTS.md等）が配置されます。

### 3. Codex CLIの初回起動

Codexを起動して初回認証を完了します。

```bash
codex
```

**実行内容**: ChatGPTアカウントでの認証が求められます。ブラウザが開くので認証を完了してください。

**結果**: 認証が完了すると、Codexが使用可能になります。

### 4. 動作確認

Codexを起動して設定が反映されているか確認します。

```bash
codex
```

**確認ポイント**:
- 簡単なコマンド実行（例: `ls -la を実行して`）が確認プロンプトなしで実行される
- 危険なコマンド（例: `rm -rf /tmp/test を実行して`）がブロックされる

**MCP Puppeteerの確認**:
```bash
/mcp
```

**期待される表示**:
```text
🔌  MCP Tools

  • puppeteer
    • Status: enabled
```
`puppeteer - Status: enabled`と表示されればブラウザ操作機能が使えます。

## このリポジトリに含まれるファイル

- **config.toml** - approval_policy、sandbox_mode、network_access、MCP Puppeteer設定
- **rules/default.rules** - コマンド実行ポリシー（危険なコマンドをブロック）
- **AGENTS.md** - Codexへの指示（開発方針、Git規約等）
- **.gitignore** - 個人データ（認証情報、セッション履歴等）を除外

## 設定の適用範囲

このリポジトリの設定（`~/.codex/config.toml`、`~/.codex/rules/default.rules`）は全プロジェクトで有効です。プロジェクト毎に個別設定を作成・管理する手間を省くことができます。

## カスタマイズ

### 確認プロンプトを一時的に有効化する

特定の作業で確認プロンプトを出したい場合、`~/.codex/config.toml`の`approval_policy`を変更します。

```toml
# approval_policy = "never"  ← コメントアウト
approval_policy = "on-request"  # 確認プロンプトを有効化
```

**影響**: Codexが重要な操作前に確認を求めるようになります。

### ブロックされるコマンドをカスタマイズする

現在ブロックされるコマンド:
- `rm -rf` - ファイルシステムの破壊
- `dd if=/dev/zero`, `dd if=/dev/random` - ディスクの破壊
- `mkfs`, `fdisk`, `parted` - パーティション操作
- `shutdown`, `reboot`, `poweroff`, `halt` - システム停止
- `git push --force`, `git push -f`, `git push --force-with-lease` - Git強制push

**変更方法**: `~/.codex/rules/default.rules`を編集してください。

## Tips

### 全自動モード（--full-auto）

実行前の確認や対話を極力挟まず、AIが自動で作業を進める全自動モードを使用できます。

```bash
codex --full-auto "最もおしゃれなToDoリストアプリを作成して"
```

**実行される内容**: Codexが対話なしで設計・実装・テストを自動的に進めます。

**結果**: 指示したタスクが完全自動で完了します。進捗は画面に表示されますが、途中で確認を求められません。

**注意**: 全自動モードでは予期しない変更が行われる可能性があります。重要なプロジェクトでは慎重に使用してください。

## トラブルシューティング

### 確認プロンプトが表示される

**確認方法**: コマンド実行時に確認プロンプトが出るか確認

**対処方法**:
1. プロジェクトディレクトリで`codex`起動時に「1. Yes, allow Codex to work in this folder without asking for approval」を選択
2. `~/.codex/config.toml`で`approval_policy = "never"`になっているか確認
3. Codexを再起動（新規セッション: `codex`）

### 危険なコマンドが実行されてしまう

**確認方法**: 危険なコマンドがブロックされるか試す（例: `rm -rf /tmp/test を実行して`）

**対処方法**:
1. `~/.codex/rules/default.rules`が存在するか確認
2. ファイル内容が正しいか確認（`prefix_rule(pattern=["rm", "-rf"], decision="forbidden")`等）
3. Codexを再起動（新規セッション: `codex`）

### MCP Puppeteerが動作しない

**確認方法**:
```bash
codex
/mcp
```

**対処方法**:
1. Node.js/npmがインストールされているか確認（`npm --version`）
2. `~/.codex/config.toml`で以下の設定があるか確認:
   ```toml
   sandbox_mode = "workspace-write"

   [sandbox_workspace_write]
   network_access = true

   [mcp_servers.puppeteer]
   type = "stdio"
   command = "npx"
   args = ["-y", "@modelcontextprotocol/server-puppeteer"]
   ```
3. 新規セッションでCodexを起動（`codex resume`ではなく`codex`）

### 設定が反映されない

**確認方法**: 設定変更後にCodexの挙動を確認

**対処方法**:
- 新規セッションを起動: `codex`（`codex resume`ではなく）
- config.tomlやrules/default.rulesの変更は、既存セッションには反映されません

## 別のMacでのセットアップ

同じ手順で`~/.codex`にクローンするだけで、同じ環境が構築されます。

```bash
# 既存設定のバックアップ
if [ -d ~/.codex ]; then
  mv ~/.codex ~/.codex_backup_$(date +"%Y%m%d%H%M%S")
fi

# クローン
git clone git@github.com:mg1986jp/codex-sync.git ~/.codex

# Codexを起動（初回認証）
codex
```
