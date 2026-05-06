# ClaudeClaw Master

AI Master Agent: universal orchestrator that creates agents, manages skills, generates PDFs, handles Git, and does research. Based on [ClaudeClaw](https://github.com/earlyaidopters/claudeclaw).

## What's inside

- **ClaudeClaw core** -- Telegram bot that connects to Claude Code CLI
- **Master Agent** (`agents/master/`) -- pre-configured orchestrator that knows how to:
  - Create new agents (agent.yaml + CLAUDE.md + systemd)
  - Install and create skills via SkillKit
  - Generate professional PDFs from Markdown (minimax-pdf)
  - Git operations (commit, push, pull)
  - Web research with document generation
- **minimax-pdf skill** (`skills/minimax-pdf/`) -- 15 PDF design templates
- **Agent templates** -- ready-made: research, comms, content, ops

## Quick Start (20 min)

### Prerequisites
- VPS with Ubuntu 22+ (2+ GB RAM)
- Node.js 22+
- Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)
- Anthropic API key or Claude Max subscription

### 1. Clone and install

```bash
git clone https://github.com/pashkacurl/claudeclaw-master.git
cd claudeclaw-master
npm install
npm run build
```

### 2. Install SkillKit (skill manager)

```bash
npm install -g skillkit
skillkit init
```

### 3. Create Telegram bot

1. Open @BotFather in Telegram
2. `/newbot` -> pick a name and username
3. Copy the token

### 4. Configure

```bash
cp .env.example .env
nano .env
```

Fill in:
```env
TELEGRAM_BOT_TOKEN=your_bot_token
ALLOWED_CHAT_ID=              # get from step 6
ANTHROPIC_API_KEY=sk-ant-xxx  # or use claude login
DB_ENCRYPTION_KEY=            # generate below
```

Generate encryption key:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### 5. Set up master agent

```bash
cd agents/master
cp agent.yaml.example agent.yaml
nano agent.yaml
```

Add bot token to `.env`:
```env
MASTER_BOT_TOKEN=your_master_bot_token
```

### 6. Get your Chat ID

```bash
npm start
```

Send any message to the bot in Telegram. Check the logs for your chat ID.
Stop the bot (Ctrl+C), add the ID to `.env`:
```env
ALLOWED_CHAT_ID=123456789
```

### 7. Customize CLAUDE.md

Edit the root `CLAUDE.md` -- replace all `[BRACKETED]` placeholders with your info.

### 8. Run as service

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/claudeclaw-master.service << 'EOF'
[Unit]
Description=ClaudeClaw Master Bot
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/node /home/aibot/claudeclaw-master/dist/index.js --agent master
WorkingDirectory=/home/aibot/claudeclaw-master
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
EOF

sudo loginctl enable-linger $(whoami)
systemctl --user daemon-reload
systemctl --user enable claudeclaw-master
systemctl --user start claudeclaw-master
```

### 9. Test

Send to the bot in Telegram:

```
Who are you and what can you do?
```

Then try:

```
Create a new agent called "seo" with SEO audit capabilities
```

## Installing Skills

The master agent uses SkillKit to manage skills:

```bash
# Search available skills
skillkit find "seo audit"

# Install a skill
skillkit add anthropics/skills

# AI-recommended skills based on your project
skillkit recommend

# Create your own skill
skillkit create my-skill

# AI-generate a skill from description
skillkit generate

# Deploy to Claude Code
skillkit sync
```

## Creating More Agents

The master bot can create agents via Telegram, or you can do it manually:

```bash
# Use the built-in wizard
npm run agent:create

# Or manually:
mkdir agents/my-agent
# Create agent.yaml and CLAUDE.md (see agents/_template/)
# Add MY_AGENT_BOT_TOKEN to .env
# Start: npm start -- --agent my-agent
```

## Architecture

```
claudeclaw-master/
  |-- src/                  # Bot core (TypeScript)
  |-- agents/
  |    |-- _template/       # Blank template
  |    |-- master/          # Universal orchestrator
  |    |-- research/        # Research agent template
  |    |-- comms/           # Communications template
  |    |-- content/         # Content creation template
  |    +-- ops/             # Operations template
  |-- skills/
  |    |-- minimax-pdf/     # PDF generation (15 templates)
  |    |-- gmail/           # Gmail integration
  |    |-- google-calendar/ # Calendar integration
  |    |-- slack/           # Slack integration
  |    |-- timezone/        # Timezone utilities
  |    +-- tldr/            # Summarization
  |-- store/                # SQLite DB (auto-created, gitignored)
  |-- CLAUDE.md             # Main system prompt (customize this)
  |-- .env                  # Configuration (create from .env.example)
  +-- README.md             # You are here
```

## Two Access Channels

Both work simultaneously with one Claude subscription:

1. **Telegram bot** -- quick tasks, voice messages, on the go
2. **VS Code Remote SSH** -- complex tasks, file editing, direct CLI access

## Credits

Built on [ClaudeClaw](https://github.com/earlyaidopters/claudeclaw) by Early AI Dopters.
Skills management powered by [SkillKit](https://github.com/rohitg00/skillkit).
