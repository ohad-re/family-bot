# Family Bot — Project Context for Claude Code

## 🎯 Project Goal

Build a **family assistant chatbot** that integrates with **WhatsApp** to help manage day-to-day household needs. The bot serves multiple family members and acts as a shared helper.

### Core capabilities (MVP)
1. **Calendar & reminders** — create, read, and manage family events via natural language
2. **Shopping & task lists** — add, complete, and review items across shared lists

### Future possibilities
- Recipe suggestions based on what's in the fridge
- Kids' homework help
- Household expense tracking
- Smart home integration

---

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  WhatsApp   │────▶│   Ubuntu VM  │────▶│ Claude API  │
│  (family)   │◀────│  (webhook)   │◀────│ (Anthropic) │
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Google APIs  │
                    │  Calendar +  │
                    │    Tasks     │
                    └──────────────┘
```

### Data flow
1. Family member sends WhatsApp message
2. Meta webhook hits our VM (HTTPS endpoint)
3. Backend loads conversation context from SQLite
4. Claude API decides intent and extracts parameters
5. Backend calls Google Calendar / Tasks API as needed
6. Response sent back via WhatsApp Cloud API

---

## 🛠️ Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Host** | Oracle Cloud (Always Free) | Free tier, no cost |
| **OS** | Ubuntu 22.04 (AMD x86) | Stable, widely supported |
| **Language** | Python 3.11+ | Good Anthropic + Google SDK support |
| **Web framework** | FastAPI | Fast, modern, great for webhooks |
| **AI** | Claude API (`claude-sonnet-4-5` or newer) | Core reasoning |
| **Messaging** | WhatsApp Cloud API (Meta) | Official, free tier for low volume |
| **Calendar** | Google Calendar API | Most families already use it |
| **Tasks/Lists** | Google Tasks API | Free, simple, syncs to phones |
| **Storage** | SQLite | Sufficient for family-scale session state |
| **HTTPS** | Cloudflare Tunnel *or* Let's Encrypt | WhatsApp requires HTTPS webhook |
| **Process manager** | systemd | Native, no extra deps |

---

## ✅ What's Done So Far

### Oracle Cloud infrastructure (completed)
- ✅ Oracle Cloud account set up in **Jerusalem region** (`il-jerusalem-1`)
- ✅ OCI CLI installed and configured locally (`~/.oci/config`)
- ✅ VCN created: `family-bot-vcn` (`10.0.0.0/16`)
- ✅ Internet Gateway created and attached
- ✅ Default route table: `0.0.0.0/0` → IGW
- ✅ Security List: ingress 22/80/443 open from `0.0.0.0/0`, egress all allowed
- ✅ Public subnet: `family-bot-subnet` (`10.0.1.0/24`)
- ✅ SSH key pair: `~/.ssh/oracle_vm` + `~/.ssh/oracle_vm.pub`
- ✅ VM launched: `family-bot-vm`
  - Shape: `VM.Standard.E2.1.Micro` (Always Free, AMD EPYC)
  - 1 OCPU, 1 GB RAM, Ubuntu 22.04
  - Public IP assigned
  - Availability domain: `Muip:IL-JERUSALEM-1-AD-1`

### Infrastructure IDs — stored in `.env.oci`

All OCI resource IDs live in **`.env.oci`** (gitignored, template in `.env.oci.example`).

```bash
# Load the infra vars into your shell before running oci CLI commands
source .env.oci

# Then you can use them directly
echo $COMP_ID
oci compute instance list --compartment-id $COMP_ID
```

**Variables in `.env.oci`:**

| Variable    | What it is                                    |
|-------------|-----------------------------------------------|
| `COMP_ID`   | Root compartment (= tenancy OCID)             |
| `VCN_ID`    | Virtual Cloud Network OCID                    |
| `IGW_ID`    | Internet Gateway OCID                         |
| `SUBNET_ID` | Public subnet OCID                            |
| `VM_IP`     | Public IP of the VM (update when it changes)  |

Region: `il-jerusalem-1`, Availability Domain: `Muip:IL-JERUSALEM-1-AD-1`

> Additional IDs that are stable and rarely referenced (instance OCID, image OCID) can be retrieved on-demand via `oci compute instance list --compartment-id $COMP_ID`.

> **Note:** ARM (A1.Flex) free capacity in Jerusalem is exhausted. If more RAM is needed later, retry ARM provisioning periodically or add swap.

---

## 📋 What's Next (Roadmap)

### Phase 1: VM finalization (in progress)
- [ ] SSH into VM and verify access
- [ ] `apt update && apt upgrade`
- [ ] Install Python 3.11, pip, venv, nginx, git
- [ ] Configure UFW firewall
- [ ] Add 2GB swap (1GB RAM is tight)
- [ ] Create `~/family-bot` project directory on the VM
- [ ] Clone this GitHub repo onto the VM

### Phase 2: HTTPS endpoint
- [ ] Choose: Cloudflare Tunnel (no domain needed) *or* Let's Encrypt (requires domain)
- [ ] Configure nginx reverse proxy → FastAPI
- [ ] Verify HTTPS is reachable from public internet

### Phase 3: WhatsApp Cloud API
- [ ] Create Meta Developer account
- [ ] Create a WhatsApp Business App in Meta Developer Portal
- [ ] Get temporary access token + Phone Number ID
- [ ] Implement `/webhook` endpoint (GET for verification, POST for messages)
- [ ] Verify webhook in Meta dashboard
- [ ] Send a test message end-to-end

### Phase 4: Claude integration
- [ ] Get Anthropic API key
- [ ] Design system prompt for family assistant persona
- [ ] Implement tool-use loop (Claude decides which Google API to call)
- [ ] Store per-sender conversation history in SQLite

### Phase 5: Google APIs
- [ ] Create Google Cloud project
- [ ] Enable Calendar API + Tasks API
- [ ] OAuth setup (family members authorize once)
- [ ] Implement `create_event`, `list_events`, `add_task`, `list_tasks` tools

### Phase 6: Polish
- [ ] Family member identification (map WhatsApp number → family role)
- [ ] Hebrew + English support
- [ ] Error handling, logging, monitoring
- [ ] Systemd service for auto-start on reboot
- [ ] Backup strategy for SQLite

---

## 📁 Target Project Structure

```
family-bot/
├── CLAUDE.md                  # This file — project context
├── README.md                  # Public-facing overview
├── .gitignore
├── .env.example               # Template for secrets
├── requirements.txt
├── pyproject.toml
│
├── app/
│   ├── __init__.py
│   ├── main.py                # FastAPI entry point
│   ├── config.py              # Settings via pydantic-settings
│   ├── webhook.py             # WhatsApp webhook handler
│   ├── whatsapp.py            # WhatsApp Cloud API client
│   ├── claude_client.py       # Anthropic API wrapper
│   ├── tools/                 # Claude tool definitions
│   │   ├── __init__.py
│   │   ├── calendar.py        # Google Calendar tools
│   │   └── tasks.py           # Google Tasks tools
│   ├── storage/
│   │   ├── __init__.py
│   │   ├── db.py              # SQLite connection
│   │   └── sessions.py        # Conversation history
│   └── google/
│       ├── __init__.py
│       ├── auth.py            # OAuth handling
│       ├── calendar.py        # Calendar API wrapper
│       └── tasks.py           # Tasks API wrapper
│
├── scripts/
│   ├── setup_vm.sh            # One-shot VM setup
│   └── deploy.sh              # Deploy from laptop to VM
│
├── systemd/
│   └── family-bot.service     # systemd unit file
│
└── tests/
    └── test_webhook.py
```

---

## 🔐 Secrets (never commit!)

All secrets live in `.env` on the VM. Template in `.env.example`.

```
# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# WhatsApp Cloud API (Meta)
WHATSAPP_ACCESS_TOKEN=...
WHATSAPP_PHONE_NUMBER_ID=...
WHATSAPP_VERIFY_TOKEN=<any_random_string_you_choose>
WHATSAPP_APP_SECRET=...

# Google OAuth
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_REDIRECT_URI=https://your-domain.com/oauth/callback

# App
APP_ENV=production
DATABASE_URL=sqlite:///./family_bot.db
LOG_LEVEL=INFO
```

---

## 🧑‍💻 Local Dev Workflow

1. Clone repo on laptop
2. `cp .env.example .env` and fill in dev values
3. `python -m venv .venv && source .venv/bin/activate`
4. `pip install -r requirements.txt`
5. Run locally with `uvicorn app.main:app --reload`
6. Use [ngrok](https://ngrok.com/) to expose to WhatsApp for dev testing

---

## 🚀 Deployment Workflow

```bash
# From laptop
git push origin main

# SSH into VM
ssh -i ~/.ssh/oracle_vm ubuntu@<VM_IP>

# On VM
cd ~/family-bot
git pull
source .venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart family-bot
sudo systemctl status family-bot
```

---

## 📝 Conventions for Claude Code

When working on this project:

- **Language:** code + comments in English. Hebrew is fine for UX-facing strings.
- **Style:** follow PEP 8, use type hints everywhere.
- **Commits:** conventional commits (`feat:`, `fix:`, `docs:`, `chore:`).
- **Testing:** add a test whenever adding a new tool or endpoint.
- **Secrets:** never hardcode — always via `config.py` reading from env.
- **Async:** prefer `async`/`await` for I/O (FastAPI, httpx).
- **No global state:** pass dependencies explicitly (FastAPI's `Depends`).
- **Logging over print:** use `logging` module, structured where possible.

---

## 🔗 Useful Links

- [WhatsApp Cloud API docs](https://developers.facebook.com/docs/whatsapp/cloud-api)
- [Anthropic API docs](https://docs.claude.com)
- [Google Calendar API](https://developers.google.com/calendar/api)
- [Google Tasks API](https://developers.google.com/tasks)
- [FastAPI docs](https://fastapi.tiangolo.com/)
- [Oracle Cloud Free Tier limits](https://www.oracle.com/cloud/free/)
