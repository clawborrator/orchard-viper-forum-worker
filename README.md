# orchard-viper-forum-worker

Linux-VM-friendly Docker deployment for the
`@MRIIOT/orchard-viper-forum` specialist agent. Same shape as the
`worker_v1-example-reddit-engager-worker` repo: single long-lived
container, secrets mounted read-only, agent code pulled from a
sibling repo at boot.

## Quick start (Linux VM)

```sh
# On the VM:
git clone https://github.com/clawborrator/orchard-viper-forum-worker
cd orchard-viper-forum-worker

# 1. Capture VCA forum cookies on a desktop browser via the
#    Cookie-Editor extension (Export → Export as JSON), then scp
#    the file to the VM:
scp viperclub.cookies.json vm:./orchard-viper-forum-worker/secrets/

# 2. Copy and fill in .env
cp .env.example .env
$EDITOR .env       # set CLAWBORRATOR_PAT + REPO_URL etc.

# 3. Up
docker compose up -d
docker compose logs -f
```

The container will:

1. Pull `ladder99/clawborrator-worker-playwright:latest`.
2. Clone the sibling repo (`REPO_URL` in `.env`) into the
   container's workspace.
3. Start a Claude Code session with that repo's `CLAUDE.md` as
   the playbook.
4. Register with the hub via the channel token (`CLAWBORRATOR_PAT`)
   under the routing name (`ROUTING_NAME`).
5. Run `node specialists/viperclub.js auth-check` on the first
   prompt to confirm the cookies are alive.
6. Wait for inbound dispatches.

## Updating the playbook

Edits to `CLAUDE.md` or `specialists/viperclub.js` land in the
sibling `orchard-viper-forum-repo`. Push there, then on the VM:

```sh
docker compose restart   # pulls the repo fresh on next start
```

For env changes (e.g. cookies path, hub URL), use `docker compose
up -d viper-forum` — `restart` does not re-read `env_file`.

## Refreshing cookies

VCA cookies expire (vBulletin defaults are typically 30-90 days).
When `auth-check` starts returning `logged_in: false`:

1. Log into the forum again on your desktop.
2. Export fresh cookies via Cookie-Editor.
3. scp the new JSON to `./secrets/viperclub.cookies.json` on the VM.
4. `docker compose restart` (the cookies are mounted read-only;
   restart re-reads the file).

## Logs

```sh
docker compose logs -f                    # follow stdout/stderr
docker exec -it orchard-viper-forum sh    # poke around
```

The agent's session also surfaces in the SPA at
https://next.clawborrator.com/orchard/ under the routing name
(`@orchard-viper-forum`). Watching the terminal there is the
nicest debugging UX.

## Resources

- Agent playbook + Playwright wrapper:
  https://github.com/clawborrator/orchard-viper-forum-repo
- Base image:
  https://hub.docker.com/r/ladder99/clawborrator-worker-playwright
- Sibling example (reddit-engager):
  https://github.com/clawborrator/worker_v1-example-reddit-engager-worker
