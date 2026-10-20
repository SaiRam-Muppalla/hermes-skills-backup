# Docker Desktop MCP Toolkit + Hermes on Windows

Use this recipe when the user asks to connect Docker to Hermes through MCP, or to test a Dockerfile while setting up that integration.

## Verify Docker first

```sh
docker --version
docker info
docker compose version
docker mcp --help
docker mcp feature ls
hermes mcp --help
```

Docker Desktop 4.50+ with the Docker MCP plugin exposes `docker mcp gateway run`. If the plugin exists, prefer Docker MCP Toolkit over manually installing random MCP servers.

## Build and test a local container before MCP wiring

Create or use a tiny loopback-only service, then verify the Dockerfile and Compose file with real execution:

```sh
docker compose -f 'C:/path/to/project/compose.yaml' config
docker compose -f 'C:/path/to/project/compose.yaml' up --build -d
docker compose -f 'C:/path/to/project/compose.yaml' ps
docker inspect --format '{{json .State.Health}}' <container>
curl -i --max-time 10 http://127.0.0.1:<port>/healthz
docker compose -f 'C:/path/to/project/compose.yaml' exec -T <service> id
docker inspect --format 'read_only={{.HostConfig.ReadonlyRootfs}} memory={{.HostConfig.Memory}} cpus={{.HostConfig.NanoCpus}}' <container>
```

Prefer a non-root user, loopback-only port binding, healthcheck, read-only root filesystem, `cap_drop: [ALL]`, `no-new-privileges:true`, and resource limits for smoke-test services. Stop the service after verification unless the user asked to keep it running:

```sh
docker compose -f 'C:/path/to/project/compose.yaml' down
```

## Create a minimal Docker MCP profile

Use Docker's catalog profile mechanism and scope filesystem access to one trusted folder:

```sh
docker mcp profile create --name 'Hermes Docker' --id hermes-agent --server catalog://mcp/docker-mcp-catalog/filesystem
docker mcp profile config hermes-agent --set 'filesystem.paths=["/c/Users/Nulle/path/to/project"]'
docker mcp profile tools hermes-agent \
  --disable filesystem.create_directory \
  --disable filesystem.edit_file \
  --disable filesystem.move_file \
  --disable filesystem.write_file
docker mcp profile show hermes-agent --format yaml
```

Keep Docker Engine administration in Hermes terminal tools unless the user explicitly asks for a Docker-control MCP server. Do not mount `docker.sock` or broad home directories as the default integration.

## Windows path rule

For Docker MCP filesystem mounts on this host, use MSYS-style `/c/Users/...` paths in the Docker MCP profile and allow-list the same path through `MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS`.

Native `C:/Users/...` paths can make Docker MCP Gateway produce a volume like `C:/Users/...:C:/Users/...:ro`, which Docker parses as too many colon-separated fields. The successful pattern is:

```sh
MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS='/c/Users/Nulle/path/to/project' \
  docker mcp gateway run --profile hermes-agent --dry-run --verbose
```

The dry run should report the filesystem server started and list its tools.

## Add Docker MCP Gateway to Hermes

Use `hermes config set`, never manual YAML editing:

```sh
hermes config set 'mcp_servers.docker.command' 'C:/Users/Nulle/AppData/Local/Programs/DockerDesktop/resources/bin/docker.exe'
hermes config set 'mcp_servers.docker.args' '["mcp","gateway","run","--profile","hermes-agent"]'
hermes config set 'mcp_servers.docker.connect_timeout' 60
hermes config set 'mcp_servers.docker.env.PROGRAMFILES' 'C:\Program Files'
hermes config set 'mcp_servers.docker.env.PROGRAMDATA' 'C:\ProgramData'
hermes config set 'mcp_servers.docker.env.MCP_GATEWAY_DOCKER_BIND_ALLOWED_PATHS' '/c/Users/Nulle/path/to/project'
hermes mcp test docker
```

A good `hermes mcp test docker` result connects over stdio and discovers Docker dynamic tools plus the scoped filesystem tools. If the current Hermes session was already running, tell the user to run `/reload-mcp` or restart Hermes before expecting the tools in chat.

## Report format

Close with:

- Docker version, Compose version, and MCP plugin presence.
- Smoke-test artifact path and exact rerun command.
- Real verification results: Compose config, build/run, container health, HTTP status/body, user and hardening flags.
- Hermes MCP result: server name, profile name, discovered tool count, and any tools intentionally disabled.
- Whether the container was stopped or left running.
