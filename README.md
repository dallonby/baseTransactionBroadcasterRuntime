# baseTransactionBroadcasterRuntime

Runs `dallonby/basebroadcast:latest` and checks for updates every 30 seconds.

The updater requires the `basebroadcast` container name, the `com.centurylinklabs.watchtower.scope=basebroadcast` label and `com.centurylinklabs.watchtower.enable=true`. Its own scope label isolates it from other scoped Watchtower instances; its enable label is false so it does not update itself. Other Watchtower instances on the same host must also have distinct scopes, because an unscoped instance can stop other updaters.

The updater restarts with Docker unless explicitly stopped. The broadcaster keeps its existing restart policy and rotated logs (three files of up to 50 MB each). Docker Hub credentials come from the existing `~/.docker/config.json` mount.

After reviewing any local changes, pull the merged revision and recreate only these two services:

```sh
git pull --ff-only
docker compose pull basebroadcast
docker compose up -d --no-deps basebroadcast watchtower
```

Check `docker logs --since 2m watchtower` for successful scans of exactly one container, and check the broadcaster image revision. Preserve local configuration and backups when updating an existing checkout.
