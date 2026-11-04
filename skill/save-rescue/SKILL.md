---
name: save-rescue
description: Recover game saves that were overwritten, deleted, or replaced by a new or cloud save. Use when the user says their save is gone, overwritten, rolled back, replaced by a new game, or that Steam/Epic cloud sync destroyed progress. Also matches 存档被覆盖、存档没了、回档、新档顶掉旧档、云同步覆盖. Prioritizes evidence gathering, backup-before-write, and safe restore.
---

# Save Rescue

Find the best recoverable copy, protect it, restore it, and stop cloud sync from overwriting it again.

## Non-negotiables

- Do not write anything until the game and its launcher are fully closed (Steam, Epic, GOG, Xbox app).
- Before any restore, copy the **entire save directory** — not just one file — to a timestamped backup outside the game tree.
- Never delete or overwrite a candidate inside the backup tree. Work on copies.
- Confirm with the user before replacing the live save.
- Do not bypass anti-cheat, DRM, or server-side saves. If progress lives on a server, say so and stop.
- Do not invent paths. Verify each one exists before using it.

## 1. Intake

Ask only what is needed:

- Game, platform (Steam / Epic / GOG / Xbox / local / emulator), OS.
- Symptom: new save appears, save vanished, cloud conflict prompt, progress rolled back.
- Is the game or launcher still running? Has the user launched it since noticing?
- Shared account, second device, or account switch?
- What progress do they expect to see (level, playtime, chapter, character)?

If the user is still in the game, have them quit without saving first.

## 2. Locate the save root

Start from the game install and look for a save directory. Common locations:

Windows:

- `%APPDATA%`, `%LOCALAPPDATA%`, `%LOCALAPPDATA%Low`
- `%USERPROFILE%\Documents\My Games`, `%USERPROFILE%\Saved Games`
- Steam: `<library>\steamapps\common\<game>`; `userdata\<account-id>\<app-id>\remote`; `steamapps\compatdata\<app-id>\pfx\drive_c\users\steamuser`
- Xbox / Game Pass: `%LOCALAPPDATA%\Packages\<FamilyName>\SystemAppData\wgs`
- Epic: `%LOCALAPPDATA%\<Game>\Saved`

macOS:

- `~/Library/Application Support`, `~/Library/Containers`, `~/Library/Group Containers`

Linux / Proton:

- `~/.steam/steam/steamapps/compatdata/<app-id>/pfx/drive_c/users/steamuser`
- `~/.local/share`, `~/.config`

Strategy:

1. Search by game name (case-insensitive) under the likely roots.
2. Search for recently modified save-like files: `*.sav`, `*.save`, `*.dat`, `*.slot`, `*.profile`.
3. Prefer directories that contain rotating backups or multiple timestamped files.

Do not guess Steam app IDs. Look them up from the store page or the installed `appmanifest_*.acf`. The folder under `userdata` is the account ID, not SteamID64.

## 3. Gather evidence

Look for, in this order:

- Timestamped backups and rotations: `1RealTime`, `2Hourly`, `3Daily`, `backup*`, `*.bak`, `*.old`, `*.tmp`
- Cloud conflict copies and cache: `remotecache.vdf`, `localconfig.vdf`, `*conflict*`
- Sibling folders for other accounts, other library copies, or other Windows users
- Windows File History / previous versions / Recycle Bin
- Game-specific snapshot or slot directories

Important: **the newest filename is not necessarily the progress the user wants.** A new game also produces fresh backups. Compare in-save metadata when possible:

- in-save time, playtime, level/chapter, quest, character name
- if a decoder exists, use it read-only; do not modify files to inspect them

## 4. Build a candidate table

List every plausible copy:

`path | file mtime | in-save time | level/quest/playtime | confidence`

Choose the copy matching the user's described progress. If two or more are plausible, present them and ask before restoring.

## 5. Restore safely

1. Close the game and launcher completely; verify the process is gone.
2. Copy the whole save root to `<save-root>__backup_<YYYYMMDD-HHMMSS>` outside the game tree.
3. Copy the chosen candidate into a staging directory; hash source and staged copy.
4. Replace the live save from staging. Keep the pre-restore live file.
5. Hash the restored file again; parse it if a decoder is available.
6. Prevent immediate re-overwrite:
   - Launch the launcher in offline mode, or disable cloud sync for that game.
   - Verify the correct progress in game.
   - Only then re-enable cloud sync; if a conflict appears, choose **keep local**.

## 6. Verify and report

Have the user launch offline and confirm level, playtime, and current quest/checkpoint.

Report concisely:

- candidates found and why one was chosen
- exact file restored and its hash
- full backup location
- what remains uncertain
- how to avoid the same overwrite next time

## 7. Prevention

- Keep periodic whole-folder backups outside the game directory.
- Disable cloud sync for games played on shared accounts.
- Coordinate devices; do not launch before a restore is verified.
- Keep manual saves at milestones instead of relying on autosaves alone.

## Reference case

A real Steam Cloud overwrite and recovery is documented in `docs/case-2026-11-steam-cloud-overwrite.md`.
