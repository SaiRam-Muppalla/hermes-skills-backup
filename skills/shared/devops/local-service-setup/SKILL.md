---
name: local-service-setup
description: Use when standing up a local self-hosted service on this PC.
version: 1.0.0
license: MIT
metadata:
  hermes:
    tags: [devops, local-service, setup, verification, windows]
    related_skills: [systematic-debugging]
---

# Local service setup (self-hosted daemons on this PC)

## When to Use

Any task that installs and configures a daemon the user runs on their own machine: local
AI gateways, LLM proxies/routers, self-hosted APIs, localhost dashboards. Trigger on
"set up X on my PC", "install and run X locally", "get X serving on localhost". Also covers
verifying an already-installed daemon actually authenticates and serves traffic.

Class-level procedure: install it, initialize it, prove it actually serves traffic, and
leave the user a working URL plus readable credentials.

## Rules that apply every time

- **Verify with real requests, never with "it printed a banner".** A service that says
  "running" can still 401 or 404. Always hit an unauthenticated page (expect 200/307),
  then an authenticated endpoint (expect 200), then confirm a bad credential fails
  (expect 401). Report those three numbers.
- **Set credentials to a value the user chose or can read.** If the tool generates a
  secret silently, immediately set a known one and write both values to plain files in
  the data dir (`<data-dir>/ADMIN_PASSWORD.txt`, `<data-dir>/API_KEY.txt`). Report where
  they live; never leave the only copy inside the tool's DB.
- **When the user asks for a weak secret the tool rejects, do not patch the tool's
  validation.** Propose the shortest compliant variant, state the tool's actual minimum,
  and let them pick. Weakening a check on a service that binds `0.0.0.0` is a silent
  security downgrade.
- **Do not connect the user's third-party accounts (OAuth providers, paid APIs).** Wire up
  everything that needs no credentials, verify it, then hand off the login step with the
  exact menu path. Same for any secret the user hasn't volunteered.
- **Report what you did not verify.** An installed gateway with zero providers connected
  is installed, not working — say so plainly.
- **Honor the requested service lifetime.** Before choosing a service manager, determine
  whether the daemon should follow machine boot, user logon, an application window, or a
  user session; a periodic watchdog can violate an app-scoped lifetime by resurrecting the
  daemon after the app closes.
- **Make ownership explicit and shutdown conditional.** For app-scoped daemons, have one
  supervisor observe the actual active app/session state, start or recover the daemon only
  while that state is active, and stop only a positively identified owned listener when it
  becomes inactive; never kill all runtime processes or an arbitrary port owner.
- **Keep lifecycle observers alive across transitions.** If a supervisor is started at
  logon but owns an application-scoped daemon, keep one observer waiting for the exact app
  process/window to appear, monitor it while active, clean up after a confirmed close, then
  return to waiting; an initial startup grace timeout or exiting after first shutdown misses
  delayed launches and later relaunches.
- **Diagnose before replacing supervision.** Inspect the registered scheduler trigger,
  action, instance policy, active supervisor processes, service PID/command line, health
  endpoint, and logs before editing startup scripts; the fault may be a controller that
  exited early or a stale running process that never loaded the edited script.
- **Restart the controller after editing its script.** A live PowerShell/shell process
  continues executing its loaded code, and a single-instance scheduler may ignore a new
  start; stop the existing identified controller and confirm exactly one replacement is
  running the intended script.
- **Verify both lifecycle edges.** Test recovery while the owner is active, then test owner
  shutdown, daemon termination, and persistence of the stopped state beyond any former
  watchdog interval. Inspect the registered task/action/trigger/settings and supervisor log;
  a successful scheduler "Run" response alone does not prove the service policy is correct.
- **A restart clears in-memory state.** Rate-limit counters, brute-force lockouts, and
  token caches live in RAM, not the DB; a restart is the fix, not DB surgery.
- **Distinguish user ambiguity early.** When a request could mean two different targets
  (a provider vs. the app's own password), ask or state your reading in one line rather
  than acting on the wrong one.

## Install procedure

1. Read the project's own docs before installing (web_search/web_extract the docs site) so
   you use the supported path, not a guessed one.
2. Install globally. On this host npm >= 11 blocks lifecycle scripts by default, so native
   deps (better-sqlite3, @swc/core, esbuild, keytar, onnxruntime-node) need explicit
   allow-listing or the binary never builds:
   `npm install -g --allow-scripts=<pkg>,better-sqlite3,@swc/core,esbuild,keytar <pkg>`
   Add `--allow-build=` equivalents on pnpm.
3. Confirm the binary resolves (`which <cmd>`, `<cmd> --version`) — on Windows a global npm
   bin lands under the Node install dir, not `%APPDATA%/npm`.
4. For an application with a managed runtime or optional feature installer, use its supported
   post-setup command before invoking the system interpreter; for Hermes voice features this
   is `hermes tools post-setup faster_whisper`, followed by `hermes config set stt.provider local`
   and the desired model (`tiny`, `base`, `small`, `medium`, or `large-v3`). The package may be
   absent from the shell's `python` while correctly installed in the application's managed venv.
5. For a local STT engine that Hermes does not natively implement, prefer its command-provider
   escape hatch: configure `HERMES_LOCAL_STT_COMMAND` with `{input_path}` and set
   `stt.provider` to `local_command`. Verify the exact executable path and model path first,
   because Hermes passes the rendered command to a subprocess and does not repair stale paths.
6. Choose the accelerator-specific binary before downloading large weights: on Windows with an
   NVIDIA GPU, a Vulkan build is a practical portable default; confirm the binary reports the
   expected GPU during a real transcription probe. Keep the executable and model under a stable
   Hermes data directory rather than a temporary extraction folder.
7. Verify the dependency with the application's runtime or a real feature probe, not only the
   system interpreter, because managed environments intentionally isolate optional packages.
8. Initialize non-interactively (`setup --non-interactive`, `--password ...`). Interactive
   setup in a non-TTY shell hangs or silently skips steps.
9. Start the daemon as a background process, then poll its HTTP port until it answers
   instead of sleeping a fixed amount.

## Reading secrets out of tools that mask them

Masked-looking values in tool output are frequently **Hermes redacting the output**, not the
app masking it. Do not chase the mask — disambiguate:

- Have the app itself return the value (its `/reveal` endpoint, CLI `reveal` subcommand).
- Or write it to a file with node and test the file: count length and `*` occurrences,
  assert `masked === false`, then use it in a request. Assert the property; never eyeball a
  redacted string in chat output.

A masked display is often `slice(0,8) + "****" + slice(-4)`, so seeing `sk-a33...c017` in
output is consistent with a perfectly valid 35-char key.

## Reading a live SQLite DB

- **Copy `storage.sqlite-wal` and `-shm` alongside `storage.sqlite`,** or you read a stale
  snapshot: rows you just created appear missing and you will wrongly conclude the write
  failed. Better: stop the daemon, or just use the app's own API/CLI.
- Open with the app's own bundled driver (e.g. its own `node_modules/better-sqlite3`) via
  absolute path — a globally installed app's native modules are not resolvable from cwd.
- Write findings to files and print only lengths/booleans, so secrets never land in the
  transcript.

## Handoff format

Close with: what is installed and verified (with status codes), where credentials live,
what is deliberately left undone and why, the exact command to start it next session, and
the security flags worth setting (e.g. bind loopback / require API key when the daemon
listens on `0.0.0.0`).

## Docker Desktop MCP Toolkit + Hermes

When wiring Docker MCP Toolkit into Hermes on Windows:

1. Confirm Docker Desktop and the MCP plugin first: `docker --version`, `docker info`, `docker mcp --help`, `docker mcp feature ls`, and `hermes mcp --help`.
2. Build a small loopback-only smoke-test container before touching Hermes config. Verify `docker compose config`, `docker compose up --build -d`, `docker compose ps`, container health, `curl http://127.0.0.1:<port>/healthz`, runtime user (`docker compose exec -T <svc> id`), and hardening flags (`docker inspect`). Stop the container after verification unless the user asked to leave it running.
3. Create a Docker MCP profile with the minimum catalog servers needed and scope filesystem mounts to one trusted project folder. Disable write-capable filesystem tools unless the user explicitly needs editing through MCP: `docker mcp profile tools <profile> --disable filesystem.create_directory --disable filesystem.edit_file --disable filesystem.move_file --disable filesystem.write_file`.
4. On Windows, configure Hermes to run Docker's real `docker.exe` as a stdio MCP server with a long connect timeout and Docker-required environment variables. Use `hermes config set`, not hand-edited YAML.
5. For Docker MCP filesystem paths, prefer MSYS-style `/c/Users/...` paths in the Docker MCP profile and set `MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS` to the same value in Hermes MCP server env. Native `C:/...` paths can be transformed into invalid `host:container:mode` volume strings by the gateway on Windows.
6. Prove the integration with `docker mcp gateway run --profile <profile> --dry-run --verbose` and `hermes mcp test <server>`. A successful Hermes test should discover both Docker dynamic tools and any profile tools; if the live session is already running, tell the user to use `/reload-mcp` or restart Hermes.

## References

- `references/omniroute.md` — OmniRoute AI gateway: exact install/verify commands, data
  dir layout, key creation + reveal, login lockout behavior, provider connect paths.
- `references/docker-mcp-hermes-windows.md` — exact Windows commands for Docker Desktop MCP Toolkit profiles and Hermes MCP integration.