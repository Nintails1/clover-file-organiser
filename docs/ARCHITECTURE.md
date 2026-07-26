# Architecture

This document describes the project's file structure and the responsibility
of each module. It's meant as a quick reference while writing or extending
the code — not user-facing documentation (see `README.md` for that).

## File structure

```
clover-file-organiser/
├── README.md
├── LICENSE
├── .gitignore
├── config/
│   ├── config.example.json    # committed template
│   └── config.json            # user's real config — gitignored
│  
├── config/
│   ├── ARCHITECTURE.md        # notes file/project structure
│   ├── CHANGELOG.md           # documents changes 
│  
├── src/
│   ├── __init__.py
│   ├── main.py                # entry point — startup sequencing only
│   ├── watcher.py             # detects finished downloads
│   ├── sorter.py              # decides where a file goes, moves it
│   ├── prompt.py              # asks the user a question, returns an answer
│   ├── config.py              # loads/validates config.json
│   └── logger.py              # records what was moved, where, when
│
├── os-scripts/windows/        # Create a seperate OS dir for Linux/iOS as needed
│   ├── enable-sorter.bat      # Enabling the script to launch during login
│   ├── disable-sorter.bat     # Disabling the script to launch during login
│   ├── install-sorter.ps1     # Configuring the script as a Task Scheduler Instance.
│   └── stop.bat               # Stopping the script.
│
└── tests/                     # Testing is key since files are modified and moved.
    ├── test_sorter.py         # modified.
    ├── test_config.py
    └── fixtures/              # Stores a sandboxed version of Download/Desktop for                          testing.
```

## Control flow

Modules are arranged as a small hierarchy, not a flat sequence. `main.py`
sets things up once; `watcher.py` fires an event; `sorter.py` makes the
decision and reaches for whatever services it needs.

```
main.py
  → loads config
  → sets up logger
  → starts watcher, passing it a callback

watcher.py
  → detects a finished download
  → calls sorter.on_new_file(filepath)

sorter.py
  → checks filename for a prefix
  → if missing, calls prompt.py
  → moves the file
  → calls logger.py to record it
```

## Module responsibilities

### `main.py`
The conductor. Does not watch, sort, or prompt itself. Responsible only for
startup order: load config → set up logger → start the watcher. Exists so
that startup sequencing lives in exactly one place.

### `watcher.py`
Detects when a file has *finished* downloading — not just appeared.
Filters out browser temp files (`.crdownload`, `.part`, `.tmp`) and waits
for a stable final file before acting. Knows nothing about prefixes,
folders, or config — its only output is calling `sorter.on_new_file(path)`
when a real file is ready. Kept deliberately "dumb" so it never needs to
change when sorting logic changes.

### `sorter.py`
The decision-maker. Given a filepath:
- Checks for a two-digit numeric prefix (e.g. `01-`).
- If found, looks up the matching folder in config and moves the file.
- If not found, calls `prompt.py` to ask the user, then moves the file
  based on the answer.
- Calls `logger.py` after every move.

This is the module most future versions extend — e.g. v3's smart
categorization adds a `classifier.py` call here, before the fallback to
`prompt.py`, without touching `watcher.py`, `config.py`, or `logger.py`.

### `prompt.py`
Asks the user "where does this go?" and returns their answer. Has no
filesystem access and no persistent state — a pure input/output utility
called by `sorter.py`.

### `config.py`
Loads and validates `config/config.json` (category → folder path mapping).
Loaded once at startup by `main.py`; other modules receive the parsed
config rather than reading the file themselves.

### `logger.py`
Records every move (source, destination, timestamp) so actions are
traceable and reversible. Exists from v1.1.0 onward — a precondition for
safely adding smarter, more autonomous sorting later.

## Why this shape

Splitting by responsibility (rather than one large script) means each
future roadmap item slots into exactly one file without disturbing the
rest:

| Addition                      | Touches           |
|-------------------------------|--------------------|
| Collision handling            | `sorter.py`        |
| Undo command                  | `logger.py` (+ new `undo.py`) |
| Setup wizard                  | `config.py`        |
| macOS/Linux support           | `scripts/` only — `src/` stays OS-agnostic |
| Smart categorization (v2.0.0) | new `classifier.py`, called from `sorter.py` |