# baseTransactionBroadcasterRuntime

Runs `dallonby/basebroadcast:latest` and checks for updates every 30 seconds.

The updater requires the `basebroadcast` container name, the `com.centurylinklabs.watchtower.scope=basebroadcast` label and `com.centurylinklabs.watchtower.enable=true`. Its own scope label isolates it from other scoped Watchtower instances; its enable label is false so it does not update itself. Other Watchtower instances on the same host must also have distinct scopes, because an unscoped instance can stop other updaters.

The updater restarts with Docker unless explicitly stopped. The broadcaster keeps its existing restart policy and rotated logs (three files of up to 50 MB each). Docker Hub credentials come from the existing `~/.docker/config.json` mount.

Preserve and reconcile local edits before updating. The existing 50m/3-file logging block is already included here; keep existing backups. Run the following from this checkout. It refuses tracked local changes and refuses an unscoped configuration:

```sh
(
set -eu
test -z "$(git status --porcelain --untracked-files=no)" || { echo "Resolve tracked local changes first" >&2; exit 1; }
git pull --ff-only
docker compose config --format json | python3 -c '
import json, sys
s = json.load(sys.stdin)["services"]
w = s["watchtower"]
assert w["command"] == ["--interval", "30", "--scope", "basebroadcast", "--label-enable", "basebroadcast"]
for name, enabled in [("basebroadcast", "true"), ("watchtower", "false")]:
    assert s[name]["labels"]["com.centurylinklabs.watchtower.scope"] == "basebroadcast"
    assert s[name]["labels"]["com.centurylinklabs.watchtower.enable"] == enabled
assert w["restart"] == "unless-stopped"
'
docker compose pull basebroadcast
docker compose up -d --no-deps basebroadcast watchtower
)
```

Check `docker logs --since 2m watchtower` for successful scans of exactly one container, and check the broadcaster image revision.
