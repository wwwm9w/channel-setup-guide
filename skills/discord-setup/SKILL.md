---
name: discord-setup
description: Discord Botの作成からClaude Codeとの接続まで全自動セットアップ。Playwright（ブラウザ操作）でBot作成・トークン取得・サーバー招待を行い、access.json設定とチャンネル登録まで完了する。トリガー例「Discordセットアップ」「Discord使いたい」「Discord Bot作りたい」「DiscordでClaude使いたい」
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

# /discord-setup -- Discord Bot 全自動セットアップ

Claude Code の Discord チャネルプラグインを、ゼロからセットアップするスキル。
ブラウザ操作（Playwright）で Bot 作成からトークン取得、サーバー招待、access.json 設定まで一気通貫で行う。

引数: `$ARGUMENTS`

---

## 前提条件チェック

セットアップ開始前に以下を確認する。1つでも欠けていたら案内して止める。

```
1. Bun がインストールされているか
   → `bun --version` で確認。なければ `curl -fsSL https://bun.sh/install | bash` を案内

2. Claude Code v2.1.80 以上か
   → `claude --version` で確認

3. Discord プラグインがインストール済みか
   → `~/.claude/plugins/` 配下に discord があるか確認
   → なければユーザーに `/plugin install discord@claude-plugins-official` を案内
```

---

## セットアップフロー

### ステップ1: Discord Developer Portal でBot作成

1. `mcp__playwright__browser_navigate` で `https://discord.com/developers/applications` を開く
2. ログインが必要な場合 → ユーザーにログインを依頼して待つ
3. snapshot を取得してページ状態を確認
4. 「New Application」ボタンをクリック

**Bot名の注意点:**
- 「Claude」はDiscordの禁止ワード。使えない
- 推奨名: 「CC Bot」「My Assistant」など
- ユーザーに好みの名前を聞く。デフォルトは「CC Bot」

5. アプリ名を入力 → 利用規約チェックボックスをON → 作成
6. **CAPTCHAが表示されたら** → ユーザーに手動で解いてもらうよう案内し、完了を待つ

### ステップ2: Bot設定とトークン取得

1. 左メニューから「Bot」セクションに移動
2. 以下のIntentを全て有効化:
   - **MESSAGE CONTENT INTENT** → ON
   - **PRESENCE INTENT** → ON（あれば）
   - **SERVER MEMBERS INTENT** → ON（あれば）
3. 「Save Changes」で保存
4. 「Reset Token」をクリック
5. **MFA/パスワード入力が求められたら** → ユーザーに入力を依頼
6. 表示されたトークンをコピー（**一度しか表示されない**ので必ず保存）

**トークンの形式:** `MTxxxxxxxxxx.xxxxxx.xxxxxxxxxxxxxxxxxxxxxxxxxxx`（Base64風の長い文字列）

### ステップ3: サーバーへの招待

1. 左メニューから「OAuth2」→「URL Generator」に移動
2. SCOPES で「bot」にチェック
3. BOT PERMISSIONS で以下にチェック:
   - Send Messages
   - Read Message History
   - Read Messages/View Channels
   - Add Reactions
   - Attach Files
   - Embed Links
4. 生成されたURLをコピーしてブラウザで開く
5. サーバーを選択 → 認証
6. **CAPTCHAが表示されたら** → ユーザーに手動で解いてもらう

**サーバーが表示されない場合:** ページをリロードしてから再度ドロップダウンを開く

### ステップ4: トークンの保存

```bash
mkdir -p ~/.claude/channels/discord
```

`~/.claude/channels/discord/.env` に以下を書き込む:
```
DISCORD_BOT_TOKEN=<取得したトークン>
```

```bash
chmod 600 ~/.claude/channels/discord/.env
```

### ステップ5: access.json の設定

ユーザーの Discord ユーザーIDが必要。取得方法:

**ブラウザ操作で取得する場合:**
1. Discord Web でサーバーに移動
2. URLからサーバーIDとチャンネルIDを取得（`/channels/<サーバーID>/<チャンネルID>`）
3. メンバーリストからユーザープロフィールを確認

**ユーザーに手動で取得してもらう場合:**
1. Discordアプリ → 設定 → 詳細設定 → 開発者モードをON
2. サーバー内の自分のアイコンをタップ → プロフィールカード → 三点メニュー → 「IDをコピー」

`~/.claude/channels/discord/access.json` を作成:

```json
{
  "dmPolicy": "allowlist",
  "allowFrom": ["<ユーザーID>"],
  "groups": {
    "<チャンネルID>": {
      "allowFrom": ["<ユーザーID>"],
      "requireMention": false
    }
  },
  "pending": {}
}
```

**設定項目の説明:**
- `dmPolicy: "allowlist"` → allowFromに登録されたユーザーのDMのみ受け付ける
- `allowFrom` → 許可するユーザーIDのリスト
- `groups` → サーバーチャンネルの設定。キーはチャンネルID
- `requireMention: false` → @メンションなしでも反応する（trueなら@メンション必須）

### ステップ6: 起動モードの選択

Claude Code の起動時に、権限モードを選ぶ必要がある。
**ユーザーに以下の3つのモードを提示し、選んでもらう。**

---

**モード1: 通常モード（推奨・初心者向け）**
```bash
claude --channels plugin:discord@claude-plugins-official
```
- ファイル編集やコマンド実行のたびに確認が入る
- 安全性が最も高い。初めて使う方はこちら
- 少し手間はかかるが、意図しない操作を防げる

**モード2: 自動承認モード（中級者向け）**
```bash
claude --channels plugin:discord@claude-plugins-official --allowedTools 'Bash(*)' 'Read' 'Write' 'Edit'
```
- よく使うツール（ファイル読み書き・コマンド実行）を自動承認
- 確認の手間が減りつつ、一定の安全性を維持

**モード3: 全権限スキップモード（上級者向け）**
```bash
claude --channels plugin:discord@claude-plugins-official  # 権限モードは上記ステップ6を参照
```
- 全ての権限確認をスキップ。完全に自動で動作する
- **注意: ファイルの削除やシステム操作も確認なしで実行される**
- セキュリティを理解した上で使用すること

---

**どのモードでも後から変更可能。** まずはモード1で試して、慣れてきたらモード2や3に切り替えるのがおすすめ。

### ステップ7: 起動確認

選んだモードのコマンドで起動した後、以下を確認:
- 「Listening for channel messages from: plugin:discord@claude-plugins-official」が表示されること
- `/mcp` で `plugin:discord:discord · connected` になっていること

DiscordのDMまたはサーバーチャンネルでテストメッセージを送り、Botが返答することを確認。

---

## Telegram との同時運用

**重要: Telegram プラグインも有効な場合、MCPの競合が発生する。**

### 問題
settings.json で telegram と discord の両方が有効だと、全セッションで両方のMCPが起動し、Telegram接続が2セッション間で競合する。

### 解決策
Discord セッション起動後、ターミナルで `/mcp` を実行し、Telegram MCPサーバーを手動で停止する。

### 起動手順（Telegram併用時）

**ターミナル1（Telegram用）:**
```bash
claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions
```

**ターミナル2（Discord用）:**
```bash
claude --channels plugin:discord@claude-plugins-official  # 権限モードは上記ステップ6を参照
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

### Botが無反応（サーバーチャンネルで）
1. access.json の `groups` にチャンネルIDが登録されているか確認
2. チャンネルIDが正しいか確認（URLの末尾の数字）
3. `requireMention` が `false` になっているか確認（trueなら@メンション必須）

### Telegram が切断される
→ 上記「Telegramとの同時運用」セクションを参照

### SessionStart:startup hook error
→ 他プラグインのhookエラー。Discord自体の動作には影響しない場合が多い

### /discord:configure が認識されない
→ `/reload-plugins` を実行してから再試行。それでもダメなら手動で.envファイルを作成

### Bot名に「Claude」が使えない
→ Discordの禁止ワード。「CC Bot」「My Bot」など別の名前を使う

---

## クイックリファレンス

| 項目 | パス/コマンド |
|------|-------------|
| トークン保存先 | `~/.claude/channels/discord/.env` |
| アクセス設定 | `~/.claude/channels/discord/access.json` |
| 起動コマンド | `claude --channels plugin:discord@claude-plugins-official  # 権限モードは上記ステップ6を参照` |
| アクセス管理 | `/discord:access` （ターミナルで実行） |
| 設定確認 | `/discord:configure` （ターミナルで実行） |
| MCP状態確認 | `/mcp` |
| プラグイン再読込 | `/reload-plugins` |
