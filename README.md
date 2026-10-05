# monacloud

CLI that adds MONA Cloud agent instructions and MCP configuration to a project, and deploys apps to MONA Cloud from the terminal.

The CLI runs on Node.js 20 or later, has no dependencies and makes no network calls during `init`.

## Install

Run it with `npx` from the project root:

```bash
npx monacloud init
```

Deploy and billing commands use [`monacloud-mcp`](https://github.com/mona-software/monacloud-mcp) (installed or in the npm cache) and its MONA Pass login.

## Quick start

```bash
npx monacloud init --yes          # write agent instructions and MCP config
npx -y monacloud-mcp login        # sign in with MONA Pass
npx monacloud deploy --sandbox    # estimate a deploy of the current directory
npx monacloud deploy              # show the cost, ask for approval, deploy and print the URL
```

## Usage

```text
monacloud init [--yes] [--dry-run] [--tool claude|codex|cursor|all] [--lang vi|en] [--recipe <slug>]
monacloud recipes [--lang vi|en]
monacloud doctor [--lang vi|en]
monacloud plans
monacloud vps create --plan <code> --monthly [--period month|year] [--name <name>] [--sandbox] [--dry-run] [--yes]
monacloud invoices [--pdf <id>]
monacloud deploy [--local | --git] [--name <name>] [--repo <url>] [--branch <name>] [--build dockerfile|nixpacks|static] [--domain <host>] [--app-host <id>] [--port <port>] [--dockerfile <path>] [--sandbox] [--dry-run] [--yes]
monacloud --help
monacloud --version
```

### `init`

| Flag | Default | Effect |
|---|---|---|
| `--yes`, `-y` | off | Skip the confirmation prompt |
| `--dry-run` | off | Print a unified diff without writing anything |
| `--tool` | `all` | Which integrations to write |
| `--lang` | `vi` | Language for templates and output |
| `--recipe` | none | Add instructions for one recipe |

Files managed by `init`:

| File | Purpose |
|---|---|
| `AGENTS.md` | Stack rules and spend guard for Codex and Cursor |
| `CLAUDE.md` | The same rules for Claude Code |
| `.mcp.json` | Claude Code project MCP config |
| `.cursor/mcp.json` | Cursor project MCP config |
| `.env.monacloud.example` | Endpoints and sample variables, no secrets |
| `.gitignore` | Adds the line `.env.monacloud` |

By `--tool`: `claude` writes `CLAUDE.md` and `.mcp.json`; `codex` writes `AGENTS.md` and prints `codex mcp add monacloud -- npx -y monacloud-mcp`; `cursor` writes `AGENTS.md` and `.cursor/mcp.json`; `all` does everything. `.env.monacloud.example` and `.gitignore` are always managed.

Existing content in `AGENTS.md` and `CLAUDE.md` is kept; the CLI only updates the block between `<!-- monacloud:start -->` and `<!-- monacloud:end -->` (and `<!-- monacloud:recipe:<slug>:start/end -->` for recipes). MCP files are merged at `mcpServers.monacloud`:

```json
{
  "mcpServers": {
    "monacloud": {
      "command": "npx",
      "args": ["-y", "monacloud-mcp"]
    }
  }
}
```

If an existing JSON file is invalid or a marker is missing or duplicated, the CLI stops before writing.

### Recipes

`monacloud recipes` lists them offline. Each recipe ships Vietnamese and English templates with the goal, the MCP tools to call in order, the steps that need the user (cost approval, OTP) and completion criteria.

| Slug | Purpose |
|---|---|
| `app-tu-git` | Put an app on the web (git or folder) |
| `phan-mem-noi-bo` | CRM, attendance, inventory and internal reporting |
| `web-ban-hang` | A store or sales app with automatic bank-transfer confirmation |
| `bot-cskh` | A customer-care and sales bot for Zalo or Telegram |
| `landing-form-lead` | A landing page, lead form and follow-up flow |
| `tro-ly-chu-ca` | An owner assistant for sales, tasks, drafts and cash flow |
| `gui-mail-otp` | OTP, order confirmation and notification email with MONA Mail |

### `doctor`

Checks Node.js 20+, that `monacloud-mcp` can be found and run, and that `~/.config/monacloud/token.json` exists as a regular file with mode `0600`. It reads only the token file metadata, never the token.

### `deploy`

By default `deploy` uploads the current directory when it has a `Dockerfile` or `package.json`; `--local` forces a directory upload, `--git` uses `git remote get-url origin` and the current branch (override with `--repo` and `--branch`). Git sources must be public HTTPS repositories; common SSH remotes are converted to HTTPS.

Without `--yes` the CLI prints the cost and asks before acting; `--dry-run` only returns the estimate and `--sandbox` simulates the deploy. On success it prints the app URL; on timeout it keeps the `job_id` so you can poll again. Upload, build or API errors exit with code 1.

| CLI step | MCP tool |
|---|---|
| Detect the project offline | `cloud_app_detect(local_dir)` |
| Read hosts and cost | `cloud_app_host_list`, `cloud_app_create(local_dir, sandbox=true)` |
| Check the wallet, upload, build, return the URL | `cloud_balance`, `cloud_app_create(local_dir)` |

Uploads are ZIP files up to 80 MiB, excluding `.env*`, `*.pem`, `.git`, `node_modules` and symlinks.

### Billing commands

```bash
monacloud plans
monacloud vps create --plan <code> --monthly
monacloud invoices
monacloud invoices --pdf <id>
```

Full command reference: [docs/commands.md](docs/commands.md).

## Development

```bash
node --test
npm pack --dry-run
```

The source uses only Node.js built-in modules.

## License

MIT

**MONA Cloud CLI is part of MONA Cloud by The MONA Group.**
