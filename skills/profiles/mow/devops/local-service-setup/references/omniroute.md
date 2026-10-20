# OmniRoute (AI gateway, npm `omniroute`)

Local-first gateway putting many LLM providers behind one OpenAI-compatible endpoint.
Dashboard and API share port **20128** by default.

## Install

```bash
npm view omniroute version                       # check the real published version first
npm install -g --allow-scripts=omniroute,keytar,onnxruntime-node,@parcel/watcher,@swc/core,protobufjs,esbuild omniroute
omniroute --version
```

Without `--allow-scripts` the install completes but native modules are unbuilt and the DB
layer fails at runtime.

## Initialize

```bash
omniroute setup --non-interactive --password "<password>"
```

- Writes data dir `~/.omniroute` (`%USERPROFILE%\.omniroute`): `storage.sqlite`, `.env`
  with `STORAGE_ENCRYPTION_KEY`, `logs/`, `db_backups/`, `supervisor/`.
- **Password minimum is 8 characters** — shorter values are rejected outright.
- A `.env` also exists inside the installed package dir; the user-level one wins, and the
  package one is destroyed on every upgrade. Only edit `~/.omniroute/.env`.

## Start and verify

```bash
omniroute --port 20128            # foreground; background it to keep working
curl -o /dev/null -w '%{http_code}' http://localhost:20128/login          # 200
K=$(tr -d '\r\n' < ~/.omniroute/API_KEY.txt)
curl -o /dev/null -w '%{http_code}' -H "Authorization: Bearer $K" http://localhost:20128/v1/models  # 200
curl -o /dev/null -w '%{http_code}' http://localhost:20128/v1/models        # 401 without key
omniroute health                  # Status/Uptime/Version
omniroute doctor                  # 10 ok / 0 failures is the healthy baseline
```

`omniroute restart` from a foreground shell can appear to hang while the supervisor hands
off; run it detached and poll the port.

## API keys

```bash
omniroute api api-keys post-api-keys --body '{"name":"primary"}'   # returns id + MASKED key
```

Key display is masked (`sk-a33...a807`) unless `ALLOW_API_KEY_REVEAL=true` is set in
`~/.omniroute/.env` (requires restart). To capture the full value:

```bash
PW=$(cat ~/.omniroute/ADMIN_PASSWORD.txt); JAR="$LOCALAPPDATA/Temp/omni.jar"
curl -s -c "$JAR" -X POST http://localhost:20128/api/auth/login \
  -H 'Content-Type: application/json' -d "{\"password\":\"$PW\"}" >/dev/null
ID=$(curl -s -b "$JAR" http://localhost:20128/api/keys \
     | node -e "let d='';process.stdin.on('data',c=>d+=c).on('end',()=>{const a=(j=>j.keys||j.data||j)(JSON.parse(d));console.log(a[0].id)})")
curl -s -b "$JAR" "http://localhost:20128/api/keys/$ID/reveal"    # -> {"key":"sk-..."}
```

Capture the revealed value with a node one-liner that writes it to a file and asserts no
`*` is present — never by reading it out of chat output, which Hermes redacts.

Cleanup: `DELETE /api/keys/<id>` with the auth cookie. The `omniroute api ...` delete
subcommand takes no id flag — use the REST route.

## Login lockout

Failed dashboard logins: **5 attempts → locked 15 minutes**, tracked in process memory, so
`omniroute restart` clears it. Symptom: `{"error":"Too many failed attempts. Try again
later."}` with HTTP 429 even for the *correct* password. When a password change appears to
fail, read the status code before assuming the change didn't take — 429 means lockout, 401
means wrong password.

## Providers

`omniroute providers list` → empty until connected. Connect in the dashboard
(http://localhost:20128 → **Providers**) — OAuth flows (Claude Code, Codex, Gemini CLI,
Qoder) and API keys (OpenRouter, GLM, OpenAI-compatible base URL) both need the user's own
account. `omniroute providers available --search <name>` lists provider ids.

`/v1/models` returns `auto/*` combos even with no providers connected, so a 200 there does
**not** mean the gateway can serve a completion — test `/v1/chat/completions` for that.

## Hardening flags

The daemon defaults to `0.0.0.0` with no API-key requirement on `/v1`. On an untrusted
network set `REQUIRE_API_KEY=true` or `OMNIROUTE_SERVER_HOST=127.0.0.1` in
`~/.omniroute/.env` and restart. `omniroute autostart` is Linux/systemd only — on Windows
the user must start it manually or via Task Scheduler.