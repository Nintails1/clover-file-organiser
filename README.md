# Clover File Organiser

A background script that watches your Downloads folder and automatically files
new downloads into category folders, based on a simple naming convention.

## The problem

Files pile up in Downloads with no consistent organization. Manually sorting
them is easy to forget and tedious to batch through later.

## Categories

| Prefix | Category      |
|--------|---------------|
| `01`   | Personal      |
| `02`   | Reading       |
| `03`   | Admin         |
| `04`   | Project Paper |
| `05`   | Other         |

## How it works (`v1`/`0.x.0 -> 1.0.0`)

1. **Watch** — the script monitors the Downloads folder for new files using a
   filesystem watcher (not a timed re-scan), so it reacts the moment a
   download finishes rather than polling on a delay.

2. **Detect completion, not just creation** — browsers write temporary files
   (`.crdownload`, `.part`, `.tmp`) while a download is in progress and only
   produce the final filename once it's done. The script ignores these
   temp files and waits for the real, stable file before acting on it.

3. **Check for a prefix** — once a finished file appears, the script looks at
   the filename for a two-digit numeric prefix followed by a hyphen
   (e.g. `01-personal-report.pdf`).

   - **Prefix found** → the file is moved straight into the matching
     category folder. No interruption, no prompt.
   - **No prefix found** → the script falls back to asking you directly:

     ```
     Where would you like this file to go?
     1. Personal
     2. Reading
     3. Admin
     4. Project Paper
     5. Other
     ```

     Your answer sorts the file and (optionally) the script can rename it
     with the matching prefix for consistency going forward.

## Two ways to use it

- **Self-tag as you save** — name the file yourself with the right prefix
  at save time (`02-reading-heavy-metals-report.pdf`). The script files it
  automatically with zero interaction.
- **Let the prompt handle it** — save files with their normal names and let
  the script ask you where each one goes, either as downloads complete or
  in a single batch pass at the end of the day.

Both paths can be used interchangeably — the script always checks for a
prefix first and only prompts when one isn't there.

## First-time setup (Windows)

1. **Install Python** (if not already installed) and confirm `pythonw.exe` is
   available on the system.
2. **Place the script and config file** somewhere permanent, e.g.
   `C:\Scripts\sorter.py` and `C:\Scripts\config.json`.
3. **Edit `config.json`** to point each category at the real folder on your
   system (see the config section above). No code editing required.
4. **Create the scheduled task**:
   - Open **Task Scheduler** → *Create Task*.
   - **General**: name it `DownloadsAutoSorter`.
   - **Triggers**: New → *At log on* (your user account).
   - **Actions**: New → *Start a program* → Program: `pythonw.exe` →
     Arguments: full path to `sorter.py`.
   - **Settings**: untick "Stop the task if it runs longer than 3 days."
   - Save.
5. **Log out and back in** (or right-click the task → *Run*) to confirm it
   starts correctly.

From this point on, the script starts automatically every time you log in —
no further action needed.

## Turning it on/off

Because it runs invisibly (`pythonw.exe` has no console window), stopping it
needs a deliberate step rather than closing a window:

- **Disable** (stop it auto-starting at future log-ins):
  Task Scheduler → right-click `DownloadsAutoSorter` → *Disable*.
- **End the current instance** (stop it running right now):
  Task Scheduler → right-click the task → *End*, or use the script's
  stop-file mechanism for a cleaner shutdown.
- **Re-enable**: right-click the task → *Enable*. It resumes at next log-in.

Optional convenience: two small `.bat` files (`enable-sorter.bat` /
`disable-sorter.bat`) can wrap the `schtasks` commands so this is a
double-click instead of a trip through the Task Scheduler UI.

## v2 considerations (`v2`/`1.x.0`)

**Reliability and safety** (`1.1.0`)
- Filename collision handling — what happens if the destination folder
  already has a file with the same name (rename with a suffix, skip, ask).
- Logging — a record of what was moved where and when, so mistakes are
  traceable.
    - Logging should be stored within `%APPDATA%` or alternate OS equivalents.
- An "undo last move" command, as a quick escape hatch for automated moves.

**Setup and configuration** (`1.2.0`)
- First-run setup wizard instead of hand-editing `config.json`.
- Support for nested/sub-categories (e.g. Project Paper → per-project
  subfolders) as the number of files grows.
- A simple settings UI instead of raw JSON editing, if this ever needs to
  be handed to a less technical user.

**Cross-platform** (`1.3.0`)
- macOS/Linux equivalents of the Task Scheduler setup (`launchd` on macOS,
  `systemd`/cron on Linux). The watcher logic itself is already
  OS-agnostic — only the "run at login" mechanism is Windows-specific.

**Polish** (`1.4.0`)
- Compiling to a standalone exe via `PyInstaller`, so that python isn't
  needed on a users machine.
- Tray icon to enable easier quitting. (Similar to Discord, Teams, etc.)

## v3+ considerations (`v3`/`>2.0.0`)

**Smarter categorization**
- Keyword/filename matching (e.g. anything containing "invoice" → Admin) as
  a lightweight first pass before full content analysis.
- Metadata-based sorting — reading file metadata (author, publisher,
  creation app) to guess a category.
- Content-based sorting — scanning file contents (or the first page of a
  PDF) for signal, with a suggested category shown to the user for
  confirmation rather than auto-filing blind.
- A confidence threshold — auto-file only when the script is highly
  confident, otherwise fall back to the prompt. Avoids jumping straight
  from "always ask" to "always guess."

