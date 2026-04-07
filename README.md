# Telegram + Whisper

Fork of the official [Claude Code Telegram plugin](https://github.com/anthropics/claude-plugins-official) with **automatic voice message transcription** via OpenAI Whisper API.

When someone sends a voice message, the bot:
1. Downloads the audio from Telegram
2. Sends it to OpenAI's `whisper-1` model for transcription
3. Delivers the transcribed text to Claude (instead of `(voice message)`)
4. Echoes the transcription back in Telegram as a reply

## Prerequisites

- [Bun](https://bun.sh) — install with `curl -fsSL https://bun.sh/install | bash`
- A Telegram bot token (from [@BotFather](https://t.me/BotFather))
- An [OpenAI API key](https://platform.openai.com/api-keys) for voice transcription

## Installation

### 1. Create a Telegram bot

Open [@BotFather](https://t.me/BotFather), send `/newbot`, pick a name and username. Copy the token (`123456789:AAH...`).

### 2. Install the official Telegram plugin first

In Claude Code:
```
/plugin install telegram@claude-plugins-official
```

### 3. Replace with the fork

Clone this repo and replace the official plugin files with the fork:

```bash
# Clone the fork
git clone https://github.com/dastanko/claude-telegram-whisper.git ~/Projects/claude-telegram-whisper

# Replace the marketplace copy with a symlink to the fork
PLUGIN_DIR=~/.claude/plugins/marketplaces/claude-plugins-official/external_plugins/telegram
mv "$PLUGIN_DIR" "${PLUGIN_DIR}.backup"
ln -s ~/Projects/claude-telegram-whisper "$PLUGIN_DIR"

# Install dependencies
cd "$PLUGIN_DIR" && bun install
```

> To revert to the official plugin: `rm "$PLUGIN_DIR" && mv "${PLUGIN_DIR}.backup" "$PLUGIN_DIR"`

### 4. Configure tokens

```
/telegram:configure 123456789:AAHfiqksKZ8...
```

This writes `TELEGRAM_BOT_TOKEN=...` to `~/.claude/channels/telegram/.env`.

Now add your OpenAI API key to the same file:

```bash
echo "OPENAI_API_KEY=sk-..." >> ~/.claude/channels/telegram/.env
```

Or edit `~/.claude/channels/telegram/.env` directly:

```
TELEGRAM_BOT_TOKEN=123456789:AAHfiqksKZ8...
OPENAI_API_KEY=sk-proj-...
```

> Without `OPENAI_API_KEY`, voice messages will still arrive but won't be transcribed.

### 5. Launch Claude Code

```sh
claude --channels plugin:telegram@claude-plugins-official
```

The symlink makes Claude Code load the fork transparently.

### 6. Pair your Telegram account

DM your bot on Telegram — it replies with a 6-character code. In Claude Code:

```
/telegram:access pair <code>
```

### 7. Lock it down

Once paired, switch to allowlist mode so strangers don't get pairing replies:

```
/telegram:access policy allowlist
```

## Updating

```bash
cd ~/Projects/claude-telegram-whisper && git pull
```

The symlink means Claude Code picks up changes on next restart — no reinstall needed.

## Voice transcription

Voice messages are automatically transcribed when `OPENAI_API_KEY` is set. The bot:
- Replies to the voice message in Telegram with the full transcription text
- Sends the transcription to Claude as the message content

If transcription fails (network error, API issue, expired file), a fallback message is delivered instead — no crashes.

Supported: Telegram voice messages (`.oga` / Opus codec). Audio files sent as documents use the standard attachment flow.

## Access control

See **[ACCESS.md](./ACCESS.md)** for DM policies, groups, mention detection, delivery config, and the `access.json` schema.

## Tools exposed to the assistant

| Tool | Purpose |
| --- | --- |
| `reply` | Send to a chat. Supports `reply_to` for threading and `files` for attachments. Auto-chunks long text. |
| `react` | Add an emoji reaction (Telegram's fixed whitelist only). |
| `edit_message` | Edit a bot's own message. Useful for progress updates. |
| `download_attachment` | Download any attachment by `file_id` on demand. |

## Photos

Inbound photos are downloaded to `~/.claude/channels/telegram/inbox/` and included in the notification so Claude can `Read` them.

## No history or search

Telegram's Bot API has no message history or search. The bot only sees messages as they arrive.
