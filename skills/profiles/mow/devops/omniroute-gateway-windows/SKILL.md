---
name: omniroute-gateway-windows
description: "Use when running OmniRoute or wiring Hermes to it."
version: 1.0.0
metadata:
  hermes:
    tags: [omniroute, gateway, windows, llm-router, openai-compatible]
---

# OmniRoute on Windows + Hermes integration

OmniRoute = local OpenAI-compatible AI gateway (`/v1`) over many providers, incl. OAuth ones (Codex/ChatGPT, Claude Code, Copilot, Gemini).

## Install (Windows, Hermes-managed Node)

```bash
npm install -g omniroute
# npm 11 blocks install scripts — native deps then fail. Re-run with allow-list:
npm install -g --allow-scripts=omniroute,keytar,onnxruntime-node,@parcel/watcher,@swc/core,protobufjs,esbuild omniroute
omniroute setup --non-interactive   # writes ~/.omniroute/{.env,storage.sqlite}
omniroute                            # dashboard :20128, API base :20128/v1
```

## Pitfalls (each one cost real time)

- **Passwords must be >= 8 chars.** `omniroute setup --password "123456"` is refused. Verify with `POST /api/auth/login`.
- **Login lockout is in-memory:** 5 failed attempts => HTTP 429 for 15 min. A restart (`omniroute restart`) clears it. Don't brute-force-check credentials.
- **Secret masking is misleading.** `omniroute keys reveal <id>` and `GET /api/keys/<id>/reveal` appear to return `sk-a33...c017`; that's redaction in the *agent's* tool-output layer, not the app. Read the key to a file instead of expecting to see it inline.
- **Reading `storage.sqlite` directly gives stale rows** unless you copy the `-wal` sidecar too, and the `key` column holds only a masked value anyway. Use the reveal endpoint.
- **Key reveal is gated** by env `ALLOW_API_KEY_REVEAL` in `~/.omniroute/.env` (not the one inside the installed package — that one is ignored and wiped on update). Without it the endpoint 403s.
- **Multiple API keys**: creating one does not delete the others; delete the spares with `DELETE /api/keys/<id>` (dashboard session cookie required). The CLI `delete-api-keys-id-` takes no body and silently no-ops — use the HTTP route.
- **Warnings about `node_modules/omniroute/.env` are expected**; user config in `~/.omniroute/.env` wins.
- **Server binds 0.0.0.0 with no API-key requirement by default.** On an untrusted network set `REQUIRE_API_KEY=true` or `OMNIROUTE_SERVER_HOST=127.0.0.1`.
- `omniroute autostart` is Linux/systemd only — no Windows boot service.

## ChatGPT/Codex OAuth models

`omniroute oauth status` lists live connections (e.g. `codex`). Model ids carry a connection prefix — `codex/gpt-5.6-luna`, `cx/...`, plus router pseudo-models `auto/gpt`, `auto/best-coding`.

Not every catalogued model is entitled: `codex/gpt-5.6-sol*` returns `The 'gpt-5.6-sol' model is not supported when using Codex with a ChatGPT account`. Probe candidates with a cheap `/v1/chat/completions` call before wiring one in.

## Point Hermes at it

```bash
# key into Hermes .env (secrets only in .env, never config.yaml)
OMNIROUTE_API_KEY=<endpoint key>

hermes config set providers.omniroute.base_url "http://localhost:20128/v1"
hermes config set providers.omniroute.key_env "OMNIROUTE_API_KEY"
hermes config set providers.omniroute.api_mode "chat_completions"
hermes config set model.provider "omniroute"
hermes config set model.base_url "http://localhost:20128/v1"
hermes config set model.api_mode "chat_completions"
hermes config set model.default "auto/gpt"

# optional aliases (short form "provider/model")
hermes config set model.aliases.gptauto "omniroute/auto/gpt"
hermes config set model.aliases.best   "omniroute/auto/best-coding"
```

On Windows, do not rely on `nohup omniroute &` from a Hermes terminal hook: Windows job-object cleanup can kill the child when the terminal backend recycles. Use a Task Scheduler watchdog plus a session-start hook:

- `run-omniroute.cmd`: health-check `http://127.0.0.1:20128/api/health`; if down, start `omniroute.cmd` with `OMNIROUTE_SERVER_HOST=127.0.0.1`.
- `hermes-self-heal.cmd`: health-check; kill a stale port-20128 listener if present; `Start-Process run-omniroute.cmd`; poll until healthy.
- Scheduled task: `schtasks /Create /TN "HermesSelfHeal" /TR "C:\\Users\\<you>\\AppData\\Local\\hermes\\hooks\\hermes-self-heal.cmd" /SC MINUTE /MO 1 /F`.
- Hook: `hermes config set hooks.on_session_start '[{"command":"C:/Users/<you>/AppData/Local/hermes/hooks/start-omniroute.sh","timeout":15}]'`; that hook just runs `schtasks /Run /TN "HermesSelfHeal"` when health is down.

After editing a hook, refresh its allowlist: `hermes hooks revoke "C:/Users/<you>/AppData/Local/hermes/hooks/start-omniroute.sh" && hermes --accept-hooks chat -q "Reply exactly OK" && hermes hooks doctor`. On this Windows setup, `/SC ONLOGON` returned Access denied, but `/SC MINUTE /MO 1` worked and repaired a killed OmniRoute in ~23s during verification.

Setting `model.provider` clears the previous provider's `base_url`/`api_mode` — re-set `base_url` after. `hermes doctor` flags a vendor-prefixed default under a non-openrouter provider; both forms work at runtime, warning is cosmetic.

Verify in two steps — a text reply proves nothing about tool use:

```bash
hermes chat -q "Reply with exactly: PONG"
hermes chat -q "Use the terminal tool to run: echo HERMES_TOOL_OK"
```