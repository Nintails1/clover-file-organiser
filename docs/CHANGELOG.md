# Changelog

All notable changes to this project are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Planned — 1.0.0
- Filesystem watcher for the Downloads folder.
- Temp-download detection (`.crdownload`, `.part`, `.tmp`) so files are only
  acted on once they're stable.
- Two-digit prefix detection and auto-move to the matching category folder.
- Interactive prompt fallback for files without a prefix.
- Windows Task Scheduler setup for run-at-login.

### Planned — 1.1.0
- Filename collision handling on move.
- Move logging for traceability.
- "Undo last move" command.
