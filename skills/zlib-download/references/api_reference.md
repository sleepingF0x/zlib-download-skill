# API Reference

## book.py CLI

All commands output JSON to stdout. Errors output JSON to stderr with non-zero exit code.

### search

```
book.py search <query> [options]
```

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `query` | string | required | Search query |
| `--source` | zlib/annas/auto | auto | Backend selection |
| `--limit` | int | - | Max results |
| `--lang` | string | - | Language filter (e.g. english, chinese) |
| `--ext` | string | - | File extension (e.g. pdf, epub) |
| `--year-from` | int | - | Publication year from |
| `--year-to` | int | - | Publication year to |

### download

```
book.py download --source <zlib|annas> --hash <hash> [options]
```

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--source` | zlib/annas | required | Backend |
| `--id` | string | - | Book ID (required for zlib) |
| `--hash` | string | required | Book hash (zlib) or MD5 (annas) |
| `--filename` | string | - | Output filename (annas only) |
| `-o, --output` | path | ~/Downloads | Output directory |

### info

```
book.py info --id <id> --hash <hash> [--source zlib]
```

### config

```
book.py config show                         # Display config (secrets masked)
book.py config set [options]                # Set values
book.py config reset                        # Delete all config
```

Config set options:
- `--zlib-email` / `--zlib-password` — Z-Library credentials
- `--zlib-domain` — Z-Library domain (default: `1lib.sk`, change if domain rotates)
  - You can also set `ZLIB_DOMAIN` in `~/.config/book-tools/.env` (env value overrides config JSON).
- `--annas-key` — Anna's Archive API key
- `--annas-binary` — Path to annas-mcp binary
- `--annas-download-path` — Download directory for Anna's Archive
- `--annas-mirror` — Alternative mirror URL
- `--download-dir` — Default download directory

### setup

```
book.py setup                               # Check all dependencies
```

Returns JSON with `ready` boolean, dependency status, and backend availability.

## Z-Library EAPI Endpoints (via Zlibrary.py)

The vendored `Zlibrary.py` communicates with Z-Library EAPI (default domain: `1lib.sk`, configurable via `config set --zlib-domain` or `.env` `ZLIB_DOMAIN`):

| Method | Endpoint | Purpose |
|--------|----------|---------|
| search | POST /eapi/book/search | Search books |
| getBookInfo | GET /eapi/book/{id}/{hash} | Book metadata |
| downloadBook | GET /eapi/book/{id}/{hash}/file | Download file (via book dict) |
| downloadBookById | GET /eapi/book/{id}/{hash}/file | Download file (via id + hash) |
| getProfile | GET /eapi/user/profile | User info + download limits |
| getMostPopular | GET /eapi/book/most-popular | Popular books |
| getRecently | GET /eapi/book/recently | Recently added |
| getUserRecommended | GET /eapi/user/book/recommended | Personalized |
| getSimilar | GET /eapi/book/{id}/{hash}/similar | Similar books |
| getBookForamt | GET /eapi/book/{id}/{hash}/formats | Available formats |

## Anna's Archive CLI (annas-mcp)

| Command | Auth Required | Description |
|---------|---------------|-------------|
| `annas-mcp search <query>` | No | Search (plain text output) |
| `annas-mcp download <md5> <filename>` | Yes (ANNAS_SECRET_KEY) | Download book |

Active mirrors: `annas-archive.li`, `annas-archive.pm`, `annas-archive.in`
