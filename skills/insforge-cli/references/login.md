# npx @insforge/cli login

Authenticate with the InsForge platform.

## Syntax

```bash
npx @insforge/cli login [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--user-api-key <key>` | Authenticate directly with a `uak_` user API key (no browser, no prompt) — best for headless / agent / CI use |
| `--email` | Use email/password login instead of OAuth |
| `--no-browser` | Print the sign-in URL and exit; finish later with `--callback-url`. For sandboxes where the browser cannot reach the CLI's local callback server |
| `--callback-url <url>` | Complete a `--no-browser` login by pasting the URL the browser was redirected to |
| `--client-id <id>` | Custom OAuth client ID |

## Authentication Methods

### OAuth (Default)

Opens your browser for OAuth 2.0 authentication with PKCE:

```bash
npx @insforge/cli login
```

The CLI starts a local callback server, opens the browser, and waits up to 5 minutes for you to authorize.

### Sandboxed OAuth (two-step) — when the browser can't reach the CLI

In sandboxed environments (the ChatGPT app, remote/SSH sessions, containers), the browser runs on the host but the CLI's `127.0.0.1` callback server is inside the sandbox, so the default flow hangs forever. Use the two-step flow instead (requires `@insforge/cli` ≥ 0.2):

```bash
# Step 1 — print the sign-in URL (PKCE state is saved to ~/.insforge/pending-login.json, valid ~10 min)
npx @insforge/cli login --no-browser --json
```

Have the user open `auth_url` and sign in. Their browser will land on a `http://127.0.0.1:.../callback?...` page that **cannot connect — this is expected**. Ask them to copy the full URL from the address bar, then:

```bash
# Step 2 — redeem the pasted callback URL
npx @insforge/cli login --callback-url "http://127.0.0.1:PORT/callback?code=...&state=..." --json
```

Both steps must run with the same `$HOME` so the pending login file is shared.

### User API Key (direct) — recommended for headless / agent / CI

No browser, no interactive prompt. Create a key in the dashboard (Profile → API Keys):

```bash
npx @insforge/cli login --user-api-key "$INSFORGE_USER_API_KEY"
```

The key is stored and sent directly as the bearer credential on every request — it authenticates as your account with full access. There is no token exchange or refresh: if the key is revoked or expires, the CLI asks you to log in again.

### Email/Password

```bash
npx @insforge/cli login --email
```

Prompts for email and password interactively. For non-interactive use (CI/CD), set environment variables:

```bash
INSFORGE_EMAIL=user@example.com INSFORGE_PASSWORD=secret npx @insforge/cli login --email
```

## Credential Storage

Credentials are saved to `~/.insforge/credentials.json` with restricted file permissions (0600). The shape depends on the login method:
- OAuth / email — `access_token` (JWT) + `refresh_token`
- User API key — `user_api_key` (the `uak_`, used directly as the bearer)

Plus user info (id, name, email). OAuth/email sessions refresh their JWT automatically on 401; a user-API-key session isn't refreshed — an invalid key prompts a re-login.

## Examples

```bash
# Interactive OAuth login (recommended for humans)
npx @insforge/cli login

# Headless / agent / CI: user API key login (no browser)
npx @insforge/cli login --user-api-key "$INSFORGE_USER_API_KEY" --json

# Sandbox (e.g. ChatGPT app): two-step OAuth — user signs in on the host browser
npx @insforge/cli login --no-browser --json
npx @insforge/cli login --callback-url "<url copied from the browser address bar>" --json

# Email/password login
npx @insforge/cli login --email

# CI/CD non-interactive login via email/password
INSFORGE_EMAIL=$EMAIL INSFORGE_PASSWORD=$PASSWORD npx @insforge/cli login --email --json
```
