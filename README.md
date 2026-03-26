# channel-setup-guide

Claude Code の Discord / Telegram チャネルセットアップを全自動化するプラグイン。

ブラウザ操作（Playwright MCP）で Bot 作成からトークン取得、アクセス設定まで一気通貫で行います。

## インストール

Claude Code のターミナルで以下を実行:

```
/plugin install-local ~/Desktop/channel-setup-guide
```

または GitHub からインストール:

```
/plugin install wwwm9w/channel-setup-guide
```

## 使い方

### Discord セットアップ

```
/discord-setup
```

Bot 作成 → トークン取得 → サーバー招待 → access.json 設定 → 起動確認まで全自動でガイドします。

### Telegram セットアップ

```
/telegram-setup
```

BotFather で Bot 作成 → トークン取得 → access.json 設定 → 起動確認まで全自動でガイドします。

## 必要なもの

- Claude Code v2.1.80 以上
- Bun（`curl -fsSL https://bun.sh/install | bash` でインストール）
- Playwright MCP（ブラウザ操作に使用）
- Discord / Telegram のアカウント

## 同時運用について

Discord と Telegram を別セッションで同時に使う場合、MCP の競合が発生します。
各スキル内に解決策が記載されています。

基本: 片方のセッション起動後、`/mcp` で不要なチャネルを手動停止してください。

## トラブルシューティング

各スキルの末尾にトラブルシューティングセクションがあります。よくある問題:

- Bot が無反応 → access.json の設定を確認
- MCP が切断される → 競合問題（別セッションとの同時接続）
- CAPTCHA → 手動で解く必要あり（自動化不可）
- 「Claude」は Discord の禁止ワード → 別の Bot 名を使用

## ライセンス

MIT
