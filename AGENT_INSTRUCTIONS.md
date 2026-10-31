# Agent Instructions: OpenCode + GMI Cloud + DeepSeek V4.1 Flash

Copy everything below this line into another AI agent when you want it to install, configure, and verify OpenCode with GMI Cloud using `deepseek-ai/DeepSeek-V4.1-Flash`.

---

## Goal

Install OpenCode (if missing), connect it to **GMI Cloud**, and set the default model to:

```text
gmicloud/deepseek-ai/DeepSeek-V4.1-Flash
```

Do not ask the user which stack to use. Complete setup end-to-end. Prefer non-interactive commands. Do not create a pull request unless asked.

## Exact model ID

Use this exact string (case-sensitive):

| Field | Value |
|---|---|
| Provider ID | `gmicloud` |
| Model ID | `deepseek-ai/DeepSeek-V4.1-Flash` |
| OpenCode model ref | `gmicloud/deepseek-ai/DeepSeek-V4.1-Flash` |
| API base URL | `https://api.gmi-serving.com/v1` |
| Env var for API key | `GMI_API_KEY` |

Related catalog IDs (fallback only if V4.1 is missing):

- `gmicloud/deepseek-ai/DeepSeek-V4-Flash`
- `gmicloud/deepseek-ai/DeepSeek-V4-Pro`

## Prerequisites the agent must gather

1. A GMI Cloud API key from https://console.gmicloud.ai (API Keys).
2. If the user already exported `GMI_API_KEY` or `GMI_CLOUD_API_KEY`, reuse it.
3. Never hardcode API keys into git-tracked files. Use env vars or OpenCode auth storage only.

If no key is available, still write the config files, then stop at the auth step and tell the user exactly what to set.

## Step 1 — Detect or install OpenCode

```bash
export PATH="$HOME/.opencode/bin:$HOME/.local/bin:$PATH"
command -v opencode || curl -fsSL https://opencode.ai/install | bash
export PATH="$HOME/.opencode/bin:$PATH"
opencode --version
```

Expected: a version like `1.19.4` or newer. OpenCode **1.19.4+** already lists `DeepSeek-V4.1-Flash` under `gmicloud`.

Ensure PATH persists:

```bash
grep -q '.opencode/bin' "$HOME/.bashrc" 2>/dev/null || \
  echo 'export PATH="$HOME/.opencode/bin:$PATH"' >> "$HOME/.bashrc"
```

## Step 2 — Write global OpenCode config

Create or overwrite:

`~/.config/opencode/opencode.json`

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "gmicloud/deepseek-ai/DeepSeek-V4.1-Flash",
  "provider": {
    "gmicloud": {
      "options": {
        "baseURL": "https://api.gmi-serving.com/v1",
        "apiKey": "{env:GMI_API_KEY}"
      }
    }
  }
}
```

Optional project-level copy (same contents) at:

`<project>/opencode.json`

## Step 3 — Authenticate GMI Cloud

Prefer env var (non-interactive):

```bash
export GMI_API_KEY="<USER_GMI_API_KEY>"
# persist for future shells if appropriate for the environment
grep -q 'GMI_API_KEY' "$HOME/.bashrc" 2>/dev/null || \
  echo 'export GMI_API_KEY="<USER_GMI_API_KEY>"' >> "$HOME/.bashrc"
```

Interactive alternative (only if a TTY is available):

```bash
opencode providers login
# Select: GMI Cloud
# Paste: the GMI API key
```

Check credentials:

```bash
opencode providers list
```

## Step 4 — Verify the model exists

```bash
opencode models gmicloud | grep -F 'DeepSeek-V4.1-Flash'
```

Expected line:

```text
gmicloud/deepseek-ai/DeepSeek-V4.1-Flash
```

If missing, extend the provider models block in `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "gmicloud/deepseek-ai/DeepSeek-V4.1-Flash",
  "provider": {
    "gmicloud": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "GMI Cloud",
      "options": {
        "baseURL": "https://api.gmi-serving.com/v1",
        "apiKey": "{env:GMI_API_KEY}"
      },
      "models": {
        "deepseek-ai/DeepSeek-V4.1-Flash": {
          "name": "DeepSeek V4.1 Flash"
        }
      }
    }
  }
}
```

Confirm the live GMI API ID:

```bash
curl -sS https://api.gmi-serving.com/v1/models \
  -H "Authorization: Bearer $GMI_API_KEY" \
  | jq -r '.data[].id' | grep -i deepseek
```

Use the exact ID returned by that endpoint if it differs.

## Step 5 — Confirm resolved config

```bash
opencode debug config
opencode debug paths
```

Confirm:

- `model` is `gmicloud/deepseek-ai/DeepSeek-V4.1-Flash`
- `provider.gmicloud.options.baseURL` is `https://api.gmi-serving.com/v1`
- config path resolves under `~/.config/opencode`

## Step 6 — Smoke test (non-TUI)

```bash
opencode run -m gmicloud/deepseek-ai/DeepSeek-V4.1-Flash "Reply with exactly: OK"
```

Success = a short reply containing `OK` and no auth/model errors.

TUI usage for humans:

```bash
cd <project>
opencode
# then: /models  → select DeepSeek V4.1 Flash if needed
```

## Important behavior notes for agents

1. **Native support first.** On OpenCode ≥ 1.19.4, `DeepSeek-V4.1-Flash` is already in the `gmicloud` catalog. Setting `"model"` is enough; do not invent a second provider unless the model is missing.
2. **ID spelling matters.** `DeepSeek-V4.1-Flash` ≠ `DeepSeek-V4-Flash`. Do not “normalize” the ID.
3. **Secrets.** Never commit API keys. Never paste keys into README/PR bodies. Prefer `{env:GMI_API_KEY}`.
4. **Custom provider fallback.** If `/connect` GMI Cloud catalog is stale, use `@ai-sdk/openai-compatible` with `baseURL: https://api.gmi-serving.com/v1` and an explicit `models` entry for `deepseek-ai/DeepSeek-V4.1-Flash`.
5. **PATH.** After install, always export `$HOME/.opencode/bin` before calling `opencode`.
6. **Do not confuse providers.** DeepSeek’s own provider ≠ GMI Cloud. This setup must use `gmicloud/...`.

## Minimal checklist (agent should tick these)

- [ ] `opencode --version` works
- [ ] `~/.config/opencode/opencode.json` exists with model `gmicloud/deepseek-ai/DeepSeek-V4.1-Flash`
- [ ] `GMI_API_KEY` is set OR `opencode providers list` shows gmicloud credentials
- [ ] `opencode models gmicloud` includes `DeepSeek-V4.1-Flash`
- [ ] `opencode debug config` shows the intended model
- [ ] `opencode run -m gmicloud/deepseek-ai/DeepSeek-V4.1-Flash "Reply with exactly: OK"` succeeds

## Troubleshooting

| Symptom | Fix |
|---|---|
| `opencode: command not found` | Re-run installer; `export PATH="$HOME/.opencode/bin:$PATH"` |
| Model not listed | Add explicit `provider.gmicloud.models["deepseek-ai/DeepSeek-V4.1-Flash"]` block |
| 401 / unauthorized | Refresh `GMI_API_KEY`; re-run `opencode providers login` |
| 404 model not found | Compare against `GET /v1/models`; update model ID to the exact API id |
| Wrong model used | Set `"model": "gmicloud/deepseek-ai/DeepSeek-V4.1-Flash"` in config; restart OpenCode |
| Config ignored | Check `opencode debug paths` — edit the reported config dir, not a random file |

## One-shot setup script (for agents)

```bash
#!/usr/bin/env bash
set -euo pipefail

export PATH="$HOME/.opencode/bin:$HOME/.local/bin:$PATH"

if ! command -v opencode >/dev/null 2>&1; then
  curl -fsSL https://opencode.ai/install | bash
  export PATH="$HOME/.opencode/bin:$PATH"
fi

mkdir -p "$HOME/.config/opencode"
cat > "$HOME/.config/opencode/opencode.json" <<'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "model": "gmicloud/deepseek-ai/DeepSeek-V4.1-Flash",
  "provider": {
    "gmicloud": {
      "options": {
        "baseURL": "https://api.gmi-serving.com/v1",
        "apiKey": "{env:GMI_API_KEY}"
      }
    }
  }
}
EOF

if [[ -z "${GMI_API_KEY:-}" ]]; then
  echo "CONFIG WRITTEN. Set GMI_API_KEY then re-run verification."
  echo "export GMI_API_KEY=..."
  exit 0
fi

opencode models gmicloud | grep -F 'DeepSeek-V4.1-Flash'
opencode debug config | grep -F 'DeepSeek-V4.1-Flash'
opencode run -m gmicloud/deepseek-ai/DeepSeek-V4.1-Flash "Reply with exactly: OK"
```

## Done criteria

Report back to the user with:

1. OpenCode version
2. Config file path used
3. Whether `GMI_API_KEY` / provider auth is present (not the key itself)
4. Whether `gmicloud/deepseek-ai/DeepSeek-V4.1-Flash` appears in `opencode models gmicloud`
5. Smoke-test result
