# zlib-download-skill

[中文文档](README.zh.md)

Forked from and inspired by [psylch/zlib-download-skill](https://github.com/psylch/zlib-download-skill).

An agent skill for searching and downloading books. Compatible with any AI coding assistant that supports the [AgentSkills](https://skills.sh) format (e.g. Claude Code, OpenClaw, etc.). Supports multiple backends with a unified CLI — full workflow from search to download in one skill.

| Backend | Source | Auth | Best For |
|---------|--------|------|----------|
| **Z-Library** | EAPI (reverse-engineered Android API) | Email + Password | Largest catalog, direct download, Chinese books |
| **Anna's Archive** | annas-mcp Go binary | API Key (donation) | Aggregated sources, multiple mirrors |

## Installation

### OpenClaw

```bash
# Via ClawHub
clawhub install zlib-download

# Or manually
git clone https://github.com/sleepingF0x/zlib-download-skill.git
cp -r zlib-download-skill/skills/zlib-download ~/.openclaw/skills/
```

### Claude Code

#### Install this skill only

```bash
npx skills add sleepingF0x/zlib-download-skill -g -y
```

#### Via Claude Code Plugin Marketplace

```shell
/plugin marketplace add sleepingF0x/zlib-download-skill
/plugin install zlib-download@sleepingF0x-zlib-download-skill
```

Restart Claude Code after installation.

## Prerequisites

- **Python 3** with `requests` library (`pip install requests`)
- **Z-Library account** (email + password) for search and download
- **annas-mcp binary** (optional) — for Anna's Archive backend

## Setup

### 1. Configure Credentials

```bash
mkdir -p ~/.config/book-tools
cp ${SKILL_PATH}/scripts/.env.example ~/.config/book-tools/.env
```

`${SKILL_PATH}` is automatically set by the agent runtime and points to the skill's installation directory.

Edit `~/.config/book-tools/.env` with your Z-Library email and password:

```
ZLIB_EMAIL=your_email@example.com
ZLIB_PASSWORD=your_password_here
```

> **Important**: Do not share credentials in chat. The skill reads them from the `.env` file only.

### 2. (Optional) Install Anna's Archive

```bash
bash ${SKILL_PATH}/scripts/setup.sh install-annas
```

Then add your API key (obtained via donation to Anna's Archive) to `.env`:

```
ANNAS_SECRET_KEY=your_api_key_here
```

### 3. Verify

```bash
python3 ${SKILL_PATH}/scripts/book.py setup
```

## Usage

Use any of these trigger phrases in your AI assistant:

```
find book about deep learning
search for machine learning textbooks
找一本关于强化学习的书
帮我搜几本莱姆的科幻小说
下载这本书
```

The skill handles the full workflow: **search → smart-pick (or ask) → download**.

## How It Works

1. **Search** — queries the selected backend (or auto-detects) with filters (language, format, year)
2. **Smart-pick** — checks top 3 results. If all 3 `author` values are identical, auto-picks #1.
3. **Ask when ambiguous** — if top 3 authors differ, presents a numbered table and asks the user to choose.
4. **Download** — fetches the file to `~/Downloads/` and reports path + size

### CLI Reference

```bash
# Search (auto-detect backend)
python3 book.py search "deep learning" --limit 10

# Search with filters
python3 book.py search "machine learning" --source zlib --lang english --ext pdf --limit 5

# Download from Z-Library
python3 book.py download --source zlib --id <id> --hash <hash> -o ~/Downloads/

# Download from Anna's Archive
python3 book.py download --source annas --hash <md5> --filename book.pdf

# Book details
python3 book.py info --source zlib --id <id> --hash <hash>

# Config management
python3 book.py config show
python3 book.py config set --zlib-email <email> --zlib-password <pw>
python3 book.py config set --zlib-domain <domain>   # if Z-Library domain changes
python3 book.py setup
```

### Credential Storage

| Source | Path | Priority |
|--------|------|----------|
| `.env` file | `~/.config/book-tools/.env` | Higher (overrides JSON) |
| Config JSON | `~/.config/book-tools/config.json` | Lower (auto-managed) |

On first successful Z-Library login, remix tokens are cached in `config.json` — subsequent calls use tokens directly, skipping email/password login.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| "Z-Library not configured" | Edit `~/.config/book-tools/.env` with credentials |
| "Z-Library login failed" | Verify credentials and run `book.py config reset`. Check DNS/network to domain. Avoid quoting `ZLIB_EMAIL`/`ZLIB_PASSWORD` values in `.env`. |
| "Z-Library download requires --id when --source zlib" | Re-run search and pass both `--id` and `--hash` from the same search result. |
| "Z-Library download failed: no file returned." | Usually `id/hash` mismatch, unavailable book, quota exhaustion, or transient network issue. Re-run search and retry with matching `id/hash`. |
| "annas-mcp binary not found" | Run `setup.sh install-annas` |
| "No backend available" | Configure at least one backend in `.env` |

## File Structure

```
zlib-download-skill/
├── skills/
│   └── zlib-download/
│       ├── SKILL.md                 # Main skill definition
│       ├── scripts/
│       │   ├── book.py              # Unified CLI wrapper
│       │   ├── Zlibrary.py          # Vendored Z-Library API (MIT)
│       │   ├── setup.sh             # Dependency check & install
│       │   └── .env.example         # Credential template
│       └── references/
│           └── api_reference.md     # API quick reference
├── README.md
├── README.zh.md
└── .gitignore
```

## Credits

- [Zlibrary-API](https://github.com/bipinkrish/Zlibrary-API) by bipinkrish (MIT) — vendored Z-Library Python wrapper
- [annas-mcp](https://github.com/iosifache/annas-mcp) by iosifache — Anna's Archive CLI + MCP server

## License

MIT
