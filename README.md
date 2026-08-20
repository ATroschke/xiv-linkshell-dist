# chardb-dist

Public distribution repo for **Linkshell Sync**, the Dalamud plugin behind
[linkshell.fanfuse.app](https://linkshell.fanfuse.app). It carries two independent things: the
Dalamud custom plugin repository (`repo.json` + release zips) and the fleet remote-config origin
(`client-config.json`). Source code lives in a separate private repo.

## Installing the plugin

In game: `/xlplugins` → Settings → Experimental → Custom Plugin Repositories → add

```
https://raw.githubusercontent.com/ATroschke/chardb-dist/main/repo.json
```

Save, then find **Linkshell Sync** in the plugin list. It is early access; nothing is collected or
uploaded until you switch upload on in its settings.

## Layout

| Path | Purpose |
|---|---|
| `repo.json` | Dalamud third-party repository index: a JSON array of one manifest |
| `plugins/CharDatabase/latest.zip` | The packaged plugin, built `Release` by DalamudPackager |
| `plugins/CharDatabase/icon.png` | 512×512 plugin icon, served as `IconUrl` |
| `client-config.json` | Fleet tuning file, fetched by every install (see below) |

`CharDatabase` is the plugin's `InternalName` and never changes — config paths and existing
installs key off it. The display name is cosmetic and lives in `Name`.

`RepoUrl` points here rather than at the source repo because the source repo is private, and a
manifest link that 404s for every user is worse than one that points at the distribution.

### Releasing

Build `Release`, then copy `plugin/CharDatabase/bin/x64/Release/CharDatabase/latest.zip` here and
mirror the built `CharDatabase.json` values into `repo.json`, adding the download links, `IconUrl`
and a fresh `LastUpdate` (unix seconds). `DalamudApiLevel` and `AssemblyVersion` come from the
build; never hand-edit them.

The download links are stable paths, not per-version ones, so an update is a new zip at the same
URL plus a bumped `AssemblyVersion` and `LastUpdate`.

## Remote config

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
