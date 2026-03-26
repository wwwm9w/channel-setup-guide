---
name: telegram-setup
description: Telegram Botの作成からClaude Codeとの接続まで全自動セットアップ。Playwright（ブラウザ操作）でBotFatherからBot作成・トークン取得を行い、access.json設定まで完了する。トリガー例「Telegramセットアップ」「Telegram使いたい」「Telegram Bot作りたい」「TelegramでClaude使いたい」
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(mkdir *)
  - Bash(chmod *)
  - Bash(ls *)
  - Bash(bun *)
  - Bash(claude *)
  - mcp__playwright__browser_navigate
  - mcp__playwright__browser_snapshot
  - mcp__playwright__browser_click
  - mcp__playwright__browser_fill_form
  - mcp__playwright__browser_take_screenshot
  - mcp__playwright__browser_press_key
  - mcp__playwright__browser_type
---

# /telegram-setup -- Telegram Bot 全自動セットアップ

Claude Code の Telegram チャネルプラグインを、ゼロからセットアップするスキル。
ブラウザ操作（Playwright）で BotFather からBot作成・トークン取得を行い、access.json 設定まで一気通貫で行う。

引数: `$ARGUMENTS`

---

## 前提条件チェック

セットアップ開始前に以下を確認する。1つでも欠けていたら案内して止める。

```
1. Bun がインストールされているか
   → `bun --version` で確認。なければ `curl -fsSL https://bun.sh/install | bash` を案内

2. Claude Code v2.1.80 以上か
   → `claude --version` で確認

3. Telegram プラグインがインストール済みか
   → `~/.claude/plugins/` 配下に telegram があるか確認
   → なければユーザーに `/plugin install telegram@claude-plugins-official` を案内
```

---

## セットアップフロー

### ステップ1: Telegram Web でBotFatherにアクセス

1. `mcp__playwright__browser_navigate` で `https://web.telegram.org/` を開く
2. ログインが必要な場合 → ユーザーに電話番号とコード入力を依頼して待つ
3. snapshot を取得してページ状態を確認
4. 検索バーで「BotFather」を検索してチャットを開く

**BotFatherが見つからない場合:**
- 直接URLでアクセス: `https://web.telegram.org/k/#@BotFather`
- または検索バーに `@BotFather` と入力

### ステップ2: Botの作成

1. BotFatherのチャットで `/newbot` と送信
2. BotFatherが「Alright, a new bot. How are we going to call it?」と聞いてくる
3. Bot名を入力（表示名）
   - ユーザーに好みの名前を聞く。デフォルトは「CC Bot」
4. BotFatherが「Good. Now let's choose a username for your bot.」と聞いてくる
5. ユーザー名を入力（末尾に `bot` または `Bot` が必要）
   - 例: `my_cc_bot`, `nana_assistant_bot`
   - 既に使われている場合は別の名前を提案

**注意点:**
- Bot名（表示名）は自由に設定可能
- ユーザー名は一意で、末尾に `bot` が必須
- ユーザー名が取られている場合、BotFatherが教えてくれる

### ステップ3: トークンの取得

1. Bot作成が成功すると、BotFatherがトークンを含むメッセージを返す
2. メッセージの中から `Use this token to access the HTTP API:` の下にあるトークンを取得

**トークンの形式:** `123456789:AAH-xxxxxxxxxxxxxxxxxxxxxxxxxxxx`（数字:英数字の文字列）

3. トークンをコピー

### ステップ4: トークンの保存

```bash
mkdir -p ~/.claude/channels/telegram
```

`~/.claude/channels/telegram/.env` に以下を書き込む:
```
TELEGRAM_BOT_TOKEN=<取得したトークン>
```

```bash
chmod 600 ~/.claude/channels/telegram/.env
```

### ステップ5: access.json の設定

ユーザーの Telegram ユーザーIDが必要。取得方法:

**方法1: ブラウザ操作で取得**
1. Telegram Web で `@userinfobot` を検索してチャットを開く
2. 何かメッセージを送ると、自分のユーザーIDが返ってくる

**方法2: ユーザーに手動で取得してもらう**
1. Telegramアプリで `@userinfobot` にメッセージを送る
2. 返ってきた `Id:` の数字がユーザーID

`~/.claude/channels/telegram/access.json` を作成:

```json
{
  "dmPolicy": "allowlist",
  "allowFrom": ["<ユーザーID>"],
  "groups": {},
  "pending": {}
}
```

**設定項目の説明:**
- `dmPolicy: "allowlist"` → allowFromに登録されたユーザーのDMのみ受け付ける
- `allowFrom` → 許可するユーザーIDのリスト（文字列として格納）
- `groups` → グループチャットの設定（必要に応じて後で追加）

### ステップ6: 起動モードの選択

Claude Code の起動時に、権限モードを選ぶ必要がある。
**ユーザーに以下の3つのモードを提示し、選んでもらう。**

---

**モード1: 通常モード（推奨・初心者向け）**
```bash
claude --channels plugin:telegram@claude-plugins-official
```
- ファイル編集やコマンド実行のたびに確認が入る
- 安全性が最も高い。初めて使う方はこちら
- 少し手間はかかるが、意図しない操作を防げる

**モード2: 自動承認モード（中級者向け）**
```bash
claude --channels plugin:telegram@claude-plugins-official --allowedTools 'Bash(*)' 'Read' 'Write' 'Edit'
```
- よく使うツール（ファイル読み書き・コマンド実行）を自動承認
- 確認の手間が減りつつ、一定の安全性を維持

**モード3: 全権限スキップモード（上級者向け）**
```bash
claude --channels plugin:telegram@claude-plugins-official  # 権限モードは上記ステップ6を参照
```
- 全ての権限確認をスキップ。完全に自動で動作する
- **注意: ファイルの削除やシステム操作も確認なしで実行される**
- セキュリティを理解した上で使用すること

---

**どのモードでも後から変更可能。** まずはモード1で試して、慣れてきたらモード2や3に切り替えるのがおすすめ。

### ステップ7: 起動確認

選んだモードのコマンドで起動した後、以下を確認:
- 「Listening for channel messages from: plugin:telegram@claude-plugins-official」が表示されること
- `/mcp` で `plugin:telegram:telegram · connected` になっていること

TelegramのDMでBotにテストメッセージを送り、Botが返答することを確認。

---

## Discord との同時運用

**重要: Discord プラグインも有効な場合、MCPの競合が発生する。**

### 問題
settings.json で telegram と discord の両方が有効だと、全セッションで両方のMCPが起動し、接続が2セッション間で競合する。

### 解決策
各セッション起動後、ターミナルで `/mcp` を実行し、不要なチャネルのMCPサーバーを手動で停止する。

### 起動手順（Discord併用時）

**ターミナル1（Telegram用）:**
```bash
claude --channels plugin:telegram@claude-plugins-official  # 権限モードは上記ステップ6を参照
```

**ターミナル2（Discord用）:**
```bash
claude --channels plugin:discord@claude-plugins-official --dangerously-skip-permissions
```
→ 起動後に `/mcp` で Telegram を停止

### うまくいかなかったアプローチ（使わないこと）
- `--settings '{"plugins":{"telegram@claude-plugins-official":false}}'` → 効果なし
- `--bare` → 機能が制限されすぎてBotとして動かない
- `--strict-mcp-config --mcp-config '{}'` → MCPが全て無効になる

---

## トラブルシューティング

### Botが無反応（DMで）
1. access.json の `dmPolicy` が `"allowlist"` になっているか確認
2. `allowFrom` にユーザーIDが正しく入っているか確認
3. トークンが正しいか確認（.envファイル）
4. Claude Code のターミナルにエラーが出ていないか確認

### Telegram MCPが繰り返し切断される
1. 別セッションでTelegramが同時接続していないか確認（競合問題）
2. `/mcp` で再接続を試す
3. それでもダメならプラグインキャッシュをクリア:
   ```bash
   rm -rf ~/.claude/plugins/cache/claude-plugins-official/telegram
   ```
   その後 `/mcp` で再接続（プラグインが再ダウンロードされる）

### /telegram:configure が認識されない
→ `/reload-plugins` を実行してから再試行。それでもダメなら手動で.envファイルを作成

### BotFatherでユーザー名が取れない
→ 末尾に `bot` が必要。既に使われている場合はランダムな数字を追加（例: `cc_bot_12345`）

---

## クイックリファレンス

| 項目 | パス/コマンド |
|------|-------------|
| トークン保存先 | `~/.claude/channels/telegram/.env` |
| アクセス設定 | `~/.claude/channels/telegram/access.json` |
| 起動コマンド | `claude --channels plugin:telegram@claude-plugins-official  # 権限モードは上記ステップ6を参照` |
| アクセス管理 | `/telegram:access` （ターミナルで実行） |
| 設定確認 | `/telegram:configure` （ターミナルで実行） |
| MCP状態確認 | `/mcp` |
| プラグイン再読込 | `/reload-plugins` |
| ユーザーID確認 | `@userinfobot` にメッセージを送る |
