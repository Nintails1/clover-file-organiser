# Roadmap: v0 → v1.0.0

This tracks the work needed to go from nothing to a stable `1.0.0` release —
the point where the core sorter is trustworthy enough to run unattended.

## 0.1.0 — Core sorting logic (no watcher yet)

- [ ] Set up project structure (`src/`, `config/`, `tests/`, `scripts/`)
- [ ] Write `config.example.json` with the 5-category prefix scheme
- [ ] `config.py` — load and validate `config.json`
- [ ] `sorter.py` — detect a two-digit numeric prefix in a filename
- [ ] `sorter.py` — look up destination folder from config and move the file
- [ ] `logger.py` — record each move (source, destination, timestamp)
- [ ] Basic tests for prefix detection and move logic, using fixtures

**Milestone:** given a file and a config, the script can correctly sort one
file by hand-calling the function — no automation yet.

## 0.2.0 — Interactive prompt fallback

- [ ] `prompt.py` — display category options and capture a response
- [ ] `sorter.py` — fall back to `prompt.py` when no prefix is found
- [ ] Tests covering the no-prefix path

**Milestone:** any file, prefixed or not, can be correctly filed.

## 0.3.0 — The watcher

- [ ] `watcher.py` — detect new files in Downloads via filesystem events
- [ ] Filter out `.crdownload` / `.part` / `.tmp` temp files
- [ ] Wait for file stability before treating a download as complete
- [ ] `watcher.py` calls `sorter.on_new_file()` on a real, finished file
- [ ] `main.py` — startup sequencing (load config → set up logger → start watcher)
- [ ] Manual end-to-end test: drop a file in a test folder, confirm it's sorted

**Milestone:** the script runs continuously and sorts files automatically
without being triggered by hand.

## 0.4.0 — Running unattended on Windows

- [ ] `scripts/install-task.ps1` — creates the Task Scheduler entry
- [ ] `scripts/enable-sorter.bat` / `disable-sorter.bat`
- [ ] Stop-file mechanism in `main.py` / `watcher.py`
- [ ] `scripts/stop.bat` — creates the stop-file for graceful shutdown
- [ ] Confirm the script survives log-off/log-on via Task Scheduler

**Milestone:** the script starts at login, runs invisibly, and can be
cleanly stopped and disabled without hunting through Task Manager.

## 0.5.0 — Hardening before calling it stable

- [ ] Handle filename collisions in the destination folder
- [ ] Handle a missing/invalid config gracefully (clear error, not a crash)
- [ ] Handle the destination folder not existing (create it or fail clearly)
- [ ] Full test pass across sorter, config, and watcher modules
- [ ] Manual soak test — leave it running for a real day of normal use

**Milestone:** the script fails safely and predictably in edge cases
instead of silently doing the wrong thing.

## 1.0.0 — Stable release

- [ ] `README.md` finished and accurate to actual behavior
- [ ] `ARCHITECTURE.md` finished and accurate to actual structure
- [ ] `CHANGELOG.md` started, backfilled with 0.1.0–0.5.0 entries
- [ ] Tag the release in git (`v1.0.0`)
- [ ] Confirm a clean install (fresh clone → config → install-task → running) works start to finish

**Milestone:** the tool can be trusted to run unattended, sorting real
downloads, without supervision — the baseline everything in the v2/v3
roadmap builds on top of.