# chardb-dist

Public distribution repo for the char-database Dalamud plugin: the fleet remote-config origin
(now) and the Dalamud custom plugin repository — `repo.json` + release zips — once the plugin
ships. Source code lives in a separate private repo.

`client-config.json` is fetched by every plugin install on a jittered interval. It is served
from GitHub raw (a separate origin from the ingest API by design — the config that protects
the API must not live behind the API). Clients cache last-known-good and degrade to minimal
send rates if this file is unreachable for >24h (fail-safe, never fail-open).

Fields:

| Field | Meaning |
|---|---|
| `schema_version` | Config schema version; clients treat unknown versions as unreachable |
| `kill_switch` | `true` stops all collection and uploads fleet-wide |
| `min_send_interval_seconds` | Floor between ingest batches |
| `activity_interval_seconds` | Fast-tier collection interval |
| `slow_interval_seconds` | Slow-tier collection interval |
| `config_poll_interval_seconds` | How often clients re-fetch this file |

Changes here reach the fleet within one poll interval (~15 min). Raise intervals or flip the
kill switch to shed load during incidents.
