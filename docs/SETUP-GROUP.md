# Setting Up a Discord Group (Channel)

NanoClaw only responds in **registered** Discord channels. Until you register at least one channel, messages are ignored (you’ll see `groupCount: 0` in the logs).

## 1. Get your Discord channel ID

1. In Discord: **User Settings** → **Advanced** → enable **Developer Mode**.
2. Right‑click the channel where you want !nano to respond.
3. Click **Copy Channel ID**. You’ll get a long number like `1234567890123456789`.

## 2. Create the group config

Create or edit `data/registered_groups.json` in the project root:

```json
{
  "discord:YOUR_CHANNEL_ID_HERE": {
    "name": "main",
    "folder": "main",
    "trigger": "!nano",
    "added_at": "2026-02-08T00:00:00.000Z"
  }
}
```

- Replace `YOUR_CHANNEL_ID_HERE` with the channel ID you copied (no spaces).
- The key **must** be `discord:` followed by the channel ID.
- **`folder`** is the name of the folder under `groups/` (e.g. `main` → `groups/main/`). Use lowercase and hyphens for new channels (e.g. `general-chat`).
- **`trigger`** is the prefix that activates the bot in that channel (default `!nano`).

## 3. Create the group folder

Create a folder for that group and add a `CLAUDE.md` so the agent has context:

```bash
mkdir -p groups/main/logs
```

If you use a different `folder` (e.g. `general-chat`), create that instead:

```bash
mkdir -p groups/general-chat/logs
```

You can copy and customize from `groups/main/CLAUDE.md`.

## 4. Restart NanoClaw

Restart the app so it reloads `registered_groups.json`:

```bash
npm run start
```

You should see `groupCount: 1` (or more) in the logs. Messages in that channel that start with your trigger (e.g. `!nano whats good`) will be handled by the agent.

## Multiple channels

Add more entries to `data/registered_groups.json`:

```json
{
  "discord:1111111111111111111": {
    "name": "main",
    "folder": "main",
    "trigger": "!nano",
    "added_at": "2026-02-08T00:00:00.000Z"
  },
  "discord:2222222222222222222": {
    "name": "General",
    "folder": "general-chat",
    "trigger": "!nano",
    "added_at": "2026-02-08T00:00:00.000Z"
  }
}
```

Create the matching folders: `groups/main/`, `groups/general-chat/`, etc.

## Main channel

The channel whose `folder` is **`main`** is the “main” channel: it can register other channels, schedule tasks for any group, and manage tasks. Only one group should have `"folder": "main"`.

## Optional: Choosing a model

By default the agent uses whatever model the Claude Agent SDK picks. To force a specific model, add to `.env`:

```bash
CLAUDE_MODEL=claude-sonnet-4-20250514
```

Use any valid [Anthropic model id](https://docs.anthropic.com/en/docs/about-claude/models). Restart NanoClaw and rebuild the container so the env is passed through (`./container/build.sh` then `npm run start`).
