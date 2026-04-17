# 🏠 Family Bot

A WhatsApp-based family assistant powered by Claude. Helps manage calendars, reminders, shopping lists, and tasks across the whole household.

## Features (planned)

- 📅 Natural-language calendar management ("תזכיר לי מחר ב-8 לקחת את הילדים")
- 🛒 Shared shopping lists ("תוסיף חלב לרשימה")
- ✅ Task management for the family
- 🌐 Hebrew + English support

## Status

🚧 **In active development.** See [CLAUDE.md](./CLAUDE.md) for full project context and roadmap.

## Stack

- **Host:** Oracle Cloud Free Tier (Ubuntu 22.04)
- **Backend:** Python / FastAPI
- **AI:** Claude API (Anthropic)
- **Messaging:** WhatsApp Cloud API (Meta)
- **Integrations:** Google Calendar, Google Tasks
- **Storage:** SQLite

## Quick start (dev)

```bash
git clone https://github.com/<your-username>/family-bot.git
cd family-bot
cp .env.example .env           # fill in your keys
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

See [CLAUDE.md](./CLAUDE.md) for architecture, deployment, and detailed setup.

## License

MIT
