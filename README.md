# Portainer Edge Agent

Docker Compose setup for running a [Portainer Edge Agent](https://docs.portainer.io/admin/environments/add/docker/edge) that connects a Docker host to a Portainer server.

## Requirements

- Docker Engine with the Compose plugin (`docker compose`)
- A Portainer server with an Edge environment created for this host

## Setup

1. In Portainer, go to **Environments > Add environment > Docker Standalone > Edge Agent** and create the environment. Copy the generated **Edge ID** and **Edge key**.

2. Create your `.env` file:

   ```sh
   cp .env.example .env
   ```

3. Fill in `EDGE_ID` and `EDGE_KEY` in `.env`.

4. Start the agent:

   ```sh
   docker compose up -d
   ```

The agent will poll the Portainer server and the environment should show as connected shortly after.

## LAN / local DNS override

If the Portainer server's hostname doesn't resolve from this host (e.g. `portainer.local` on a LAN without DNS), use `docker-compose.lan.yml` to add a static host entry inside the container.

Set in `.env`:

```sh
COMPOSE_FILE=docker-compose.yml:docker-compose.lan.yml
PORTAINER_DOMAIN=portainer.local
PORTAINER_IP=192.168.1.10
```

Or pass the files explicitly:

```sh
docker compose -f docker-compose.yml -f docker-compose.lan.yml up -d
```

Remove `COMPOSE_FILE` from `.env` if you don't need the override.

## Configuration

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `EDGE_ID` | yes | — | Edge ID from the Portainer server |
| `EDGE_KEY` | yes | — | Edge key from the Portainer server |
| `PORTAINER_DOMAIN` | LAN only | `portainer.local` | Hostname of the Portainer server |
| `PORTAINER_IP` | LAN only | `127.0.0.1` | IP that `PORTAINER_DOMAIN` resolves to |
| `PORTAINER_AGENT_TAG` | no | `2.45.1` | `portainer/agent` image tag — keep it in line with the server version |
| `TZ` | no | `Europe/Athens` | Container timezone |
| `PORTAINER_AGENT_CPU_LIMIT` | no | `0.5` | CPU limit |
| `PORTAINER_AGENT_MEMORY_LIMIT` | no | `128m` | Memory limit |
| `PORTAINER_AGENT_MEMORY_RESERVATION` | no | `32m` | Memory reservation |
| `PORTAINER_AGENT_PIDS_LIMIT` | no | `200` | Max processes in the container |

## What the container gets

- `/var/run/docker.sock` — to manage the local Docker engine
- `/var/lib/docker/volumes` — to browse volume contents
- `/` mounted read-only at `/host` — for host filesystem browsing
- `portainer-edge-agent-data` volume at `/data` — persistent agent state

It also runs with `no-new-privileges`, rotated JSON logs (5 × 10 MB, compressed), and the resource limits above. Note that Docker socket access is effectively root on the host, so only connect agents to a Portainer server you trust.

## Operations

```sh
docker compose logs -f                   # follow agent logs
docker compose pull && docker compose up -d   # upgrade after changing PORTAINER_AGENT_TAG
docker compose down                      # stop (keeps the data volume)
docker compose down -v                   # stop and remove agent state
```
