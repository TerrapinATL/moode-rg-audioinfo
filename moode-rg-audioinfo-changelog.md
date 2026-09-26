# moode-rg-audioinfo — Change Log

All version changes are appended to this file, newest last, one `## vX` section per version.

**Update rule:** before changing the main script or README, save the current
content as a versioned copy (e.g. `moode-rg-audioinfo-v1`) so every published
version stays retrievable, then bump the version header inside the script.

**Current version: v1** — initial release; adds ReplayGain rows to moOde's
Audio info Track tab.

Main repo: [TerrapinATL/moode-rg-audioinfo](https://github.com/TerrapinATL/moode-rg-audioinfo)

---

## v1 Change Log (2026-09-26)

* Initial release. Patches `/var/www/command/audioinfo.php` so the
  **M → Audio info → Track tab** also displays the file's embedded
  ReplayGain tags (RG Track gain/peak, RG Album gain/peak,
  RG Reference loudness) via MPD's `readcomments` command, which returns
  raw file tags that `lsinfo` does not expose.
* Rows appear only for files that carry the tags; radio stations and
  untagged files are unaffected. Playback tab untouched (it already shows
  moOde's MPD replaygain mode setting).
* **Version-safe patching** — validates both anchor strings before any
  edit; auto-detects which path-quoting style the installed moOde uses
  (`escapeDblQuotes()` exists only in newer versions); aborts untouched
  if the installed layout matches no known version.
* **Safety net** — timestamped backup to `/var/backups/rg-audioinfo/`
  before patching, `php -l` syntax check after patching with automatic
  backup restore on failure, and `--status` / `--restore` commands.
  Re-running `install` on a patched file is a no-op.
* **Version compatibility** — anchors verified on moOde r942, 10.2.4, and
  develop. Live-verified on moOde 10.2.4 / MPD 0.24.12 (Raspberry Pi 5,
  nginx + php8.4-fpm): Track tab API returns the five RG rows for
  loudgain-tagged FLAC files.
* **Post-release fix (same day)** — removed an unbound `FUNC_ANCHOR`
  variable reference in the installer (caught by `set -u` on the first
  live run; no file was modified by the aborted attempt). Patch logic
  re-verified on r942 and develop copies after the fix.
