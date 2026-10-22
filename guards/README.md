# Operational guards

Two small systemd guards that keep the Docker-hosted datastores actually serving, rather than
trusting Docker's restart policy to do it. Installed on production 2026-10-21.

| file | host | installed as |
|---|---|---|
| `arango-guard`, `arango-guard.service`, `arango-guard.timer` | `rl-arangodb` | `/usr/local/bin/arango-guard`, `/etc/systemd/system/arango-guard.{service,timer}` |
| `postgres-guard`, `postgres-guard.service`, `postgres-guard.timer` | `rl-postgres-main` | `/usr/local/bin/postgres-guard`, `/etc/systemd/system/postgres-guard.{service,timer}` |

Both run every 2 minutes.

## Why these exist

On 2026-10-13 the nightly `yum -y update` on `rl-arangodb` upgraded docker/containerd while the
daemon was restarting. Docker logged `ShouldRestart failed, container will not be restarted ...
daemonShuttingDown=true error="restart canceled"` and then failed to launch the container:
`failed to create TTRPC connection: unsupported protocol`. Because the restart was *cancelled*
rather than attempted, `--restart always` gave up permanently. Nothing retried it.

ArangoDB stayed dead for **nine days**. Every database-backed API route returned 500 after a
constant ~7.4 s, because `ratelimit_helper` keeps its rate-limit tokens in Arango and
`arango_crud` retries five times with backoff before giving up. Postgres was perfectly healthy
the whole time; the outage looked like "the API is broken" and was nothing of the sort.

`rl-postgres-main` was in a worse position structurally: `restart=no` on the container,
`docker.service` **disabled**, and recovery depending on a single `@reboot` cron line. A Docker
daemon restart or a reboot would have left the database down with nothing to bring it back, and
unlike Arango, every route needs it.

## The rule these encode

**A restart policy is not a supervisor.** `--restart always` is a policy that a daemon shutdown
can silently void, so a guard has to probe the *service* and not the state flag:

1. probe the datastore;
2. require two consecutive failures before acting, so a slow moment cannot become a restart loop;
3. escalate: `docker start` (stopped) -> `docker restart` (running but not answering) ->
   `systemctl restart docker` and retry (the shim/TTRPC repair, i.e. the actual 2026-10-13 failure);
4. log every step to the journal, and exit non-zero when it cannot recover.

## Probes: the two details that were each learned the hard way

**The published address comes from the container, never from an assumption.**
`rl-arangodb` publishes on `10.0.0.61:8529` only. A first version of `arango-guard` probed
`127.0.0.1`, got `000` from a perfectly healthy server, and restart-looped ArangoDB every two
minutes -- strictly worse than having no guard. Both scripts now derive the endpoint from
`docker port <container> <port>/tcp` (falling back to `ARANGOHOST` on Arango), substituting
`127.0.0.1` only when the container publishes on `0.0.0.0`.

**A health check must exercise the protocol where it can.**
- `postgres-guard` uses `pg_isready`, which completes a real handshake. A bare TCP connect cannot
  distinguish "accepting connections" from "accepting the socket and then never answering", and the
  second is a failure mode this fleet has produced.
- `arango-guard` treats *any* HTTP status as healthy. `/_api/version` is behind auth, so a serving
  ArangoDB answers `401`, and `curl` prints `000` when nothing is listening.

## Never create the Postgres container

`postgres-guard` will start or restart the `postgres` container, but if the container is **missing**
it refuses and exits 1.

The database lives in a Docker volume
(`900233cb5cc0897a8c5489e73a19beed15b88d096017cfa457a96c42f9721b72`, an anonymous volume created by
the original `docker run`, mounted at `/var/lib/postgresql/data`). A `docker run` without `-v` would
attach a *brand new empty* volume. The API would then answer `200` against an empty database and the
site would look healthy while showing nobody's loans. Automatic recovery must never be able to
produce an empty database, so that step is left to a human; the guard logs the volume id and the
exact `docker run -v ...` needed.

## Installing on a new instance

```bash
sudo install -m 0755 arango-guard /usr/local/bin/arango-guard
sudo install -m 0644 arango-guard.service arango-guard.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now arango-guard.timer
```

Write the file to a temporary path and `mv` it into place rather than `tee`-ing it in place: a
two-minute timer will read a half-written script, and bash will happily execute the middle of a
comment (`because the daemons: No such file or directory`) as a command.

## Verifying, and the two traps

```bash
systemctl list-timers arango-guard.timer      # check NEXT is in the near future
journalctl -u arango-guard.service -n 20      # what it decided and why
```

**`OnCalendar`, not `OnBootSec`.** These boxes have uptimes measured in years
(`rl-arangodb` booted 2022-06-21, `rl-postgres-main` 2024-06-08). `OnBootSec=60` combined with
`OnUnitActiveSec=120` computes against the last activation and produced a `NEXT` in **2022** -- an
enabled timer that would never fire. Always read `NEXT` after installing a timer.

**A healthy tick must be a no-op.** Confirm it by comparing the container's start time across a tick:

```bash
B=$(sudo docker inspect -f '{{.State.StartedAt}}' postgres); sleep 130
[ "$B" = "$(sudo docker inspect -f '{{.State.StartedAt}}' postgres)" ] && echo "no-op, good"
```

Both guards were verified this way, and each was also verified against a real failure: stopping the
container and letting the timer recover it. `arango-guard` recovered in 43 s; `postgres-guard`
recovered in 61 s with the data verified byte-for-byte intact
(1703 MB / 44 tables / 199385 loans / 52621 users, identical before and after).

## Known drift from `setup-notes/`

`setup-notes/postgres.md`'s cron block installs
`@daily yum update -y; sleep 60; service docker start; docker start postgres`. The live root crontab
on `rl-postgres-main` does **not** contain that line; it has only `create_backup.sh` (daily),
`clean_expire_tables.sh` (hourly), `clean_logs.sh` (monthly) and
`@reboot service docker start; docker start postgres`.

**Do not re-add the unattended `yum update`.** It is what broke ArangoDB. These guards make an
upgrade survivable; they do not make an unattended upgrade safe. Move patching into a deliberate
window, and check both guards afterwards.

Also note `docker.service` was disabled on `rl-postgres-main` (it is enabled now). On a rebuild,
enable it rather than relying on an `@reboot` cron line.
