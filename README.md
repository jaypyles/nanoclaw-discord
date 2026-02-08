<p align="center">
  <img src="assets/nanoclaw-logo.png" alt="nanoclaw-discord" width="400">
</p>

<p align="center">
  <strong>nanoclaw-discord</strong> — NanoClaw preconfigured for <strong>Discord</strong> and <strong>Docker</strong>. One process, one channel stack, no config matrix.
</p>

<p align="center">
  Message Claude from Discord. Agents run in Docker. Small codebase, secure by isolation.
</p>

## What This Is

[NanoClaw](https://github.com/gavrielc/nanoclaw) is a minimal, understandable Claude assistant: one Node process, agents in containers, no framework bloat. **nanoclaw-discord** is that same stack, shipped for a single main use case:

- **Discord** as the only messaging channel.
- **Docker** as the only container runtime (macOS and Linux).

No “choose your runtime” or “add Discord via skill” — it’s Discord + Docker by default. You get a small, auditable codebase that does one thing clearly.

## Quick Start

```bash
git clone https://github.com/gavrielc/nanoclaw.git
cd nanoclaw
claude
```

Then run **`/setup`**. Claude Code will walk you through: dependencies, Discord bot token, Claude auth, Docker image, and registering your first channel.

## Philosophy

**One stack.** Discord for I/O, Docker for agents. No optional runtimes or channel providers to wire up.

**Small enough to understand.** One process, a handful of files. No microservices, no message queues. Have Claude Code walk you through it.

**Secure by isolation.** Agents run in Docker containers with explicit mounts. They only see what you mount. Bash inside the container is safe.

**Built for one user.** Fork it and have Claude Code adapt it to your workflow. Customization = code changes, not config files.

**AI-native.** No install wizard; Claude Code guides setup. No dashboard; ask Claude what’s happening. No debug UI; describe the problem, Claude fixes it.

**Best harness, best model.** Runs on Claude Agent SDK (Claude Code). The harness matters; this one is minimal and transparent.

## What It Supports

- **Discord I/O** — Register channels in `data/registered_groups.json`; only those channels trigger the agent. Trigger word (default `!nano`) in each channel.
- **Isolated group context** — Each channel has its own `CLAUDE.md` memory and filesystem; each run is in its own container with only that group’s data mounted.
- **Main channel** — One Discord channel is “main” (admin): register other channels, manage scheduled tasks, optional extra mounts.
- **Scheduled tasks** — Recurring or one-time jobs that run Claude and can message back via Discord.
- **Web access** — Search and fetch; agent can use browser automation (agent-browser) when you add the skill.
- **Optional skills** — Add Gmail (`/add-gmail`), more channels, or custom skills via `.md` in `container/skills/` and CLAUDE.md.

## Usage

In any registered Discord channel, use the trigger (e.g. `!nano`):

```
!nano send a summary of the sales pipeline every weekday at 9am
!nano every Monday at 8am, compile AI news from Hacker News and message me a briefing
!nano what skills do you have?
```

From the **main** channel you can also:

```
!nano list all scheduled tasks
!nano pause the Monday briefing task
!nano register the #general channel
```

See [docs/SETUP-GROUP.md](docs/SETUP-GROUP.md) for registering channels and [docs/SETUP-GROUP.md#optional-choosing-a-model](docs/SETUP-GROUP.md#optional-choosing-a-model) for setting `CLAUDE_MODEL` in `.env`.

## Customizing

Tell Claude Code what you want; the codebase is small enough to change safely:

- “Change the trigger word to !bob”
- “Remember to keep responses shorter”
- “Add a skill for Notion” (and add `container/skills/notion-api.md` + a bullet in `groups/main/CLAUDE.md`)

Or run **`/customize`** for guided changes.

## Requirements

- **macOS or Linux**
- **Node.js 20+**
- **[Claude Code](https://claude.ai/download)** (for setup and editing)
- **[Docker](https://docker.com/products/docker-desktop)**

## Architecture

```
Discord (discord.js) → MessageCreate events → Host (Node) → Docker container (Claude Agent SDK) → Response
```

Single Node process on the host: Discord connection, routing, scheduler, IPC. Each message (or scheduled task) spawns a short-lived Docker container with the group’s mounts; the container runs Claude Agent SDK and exits. No daemons, no queues.

Key files:

| File | Purpose |
|------|---------|
| `src/index.ts` | Discord connection, message routing, IPC |
| `src/container-runner.ts` | Builds mounts, spawns `docker run` |
| `src/task-scheduler.ts` | Scheduled tasks |
| `src/db.ts` | SQLite (tasks, run history) |
| `groups/*/CLAUDE.md` | Per-channel memory and instructions |
| `data/registered_groups.json` | Registered Discord channel IDs |

## FAQ

**Why Discord only?**  
This variant is Discord + Docker only. To add another channel, use a skill or fork and add it.

**Why Docker only?**  
So the same setup runs on macOS and Linux with one runtime. No Apple Container or other backends.

**Can I run this on Linux?**  
Yes. Install Docker and Node, clone, run `/setup` with Claude Code.

**Is this secure?**  
Agents run in containers with explicit mounts; they don’t see your full filesystem or `.env` (only a small allowlist of vars is passed in). See [docs/SECURITY.md](docs/SECURITY.md) for the security model.

**How do I debug issues?**  
Ask Claude Code: “Why didn’t the last message get a response?” “What’s in the latest container log?” Or run **`/debug`** for guided checks.

**What’s the difference from upstream NanoClaw?**  
Upstream may support other runtimes/channels via skills. This setup locks to Discord + Docker for a simpler, single-use-case.

## Contributing

**Prefer skills over in-repo features.**  
If you want Telegram or Slack, contribute a skill (e.g. `.claude/skills/add-telegram/SKILL.md`) that transforms a clone to add that channel. Keep the base as Discord + Docker.

**RFS (Request for Skills)**  
- `/add-telegram`, `/add-slack` — Additional channels alongside Discord  
- `/setup-windows` — Windows via WSL2 + Docker  
- `/add-clear` — `/clear` command to compact conversation context

## License

MIT
