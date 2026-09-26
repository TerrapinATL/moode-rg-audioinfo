# moode-rg-audioinfo

Patches moOde audio player so the **M → Audio info → Track tab** also displays
the ReplayGain tags embedded in the selected file.

## What it does

moOde's Track tab is built by `/var/www/command/audioinfo.php` from MPD's
`lsinfo` output — which never includes `REPLAYGAIN_*` tags, even when loudgain
has written them into the files. This script patches `parseTrackInfo()` to also
issue MPD's `readcomments` command, which returns the **raw** file tags
(including ReplayGain) and appends up to five rows to the Track tab:

* RG Track gain (e.g. `-7.23 dB`)
* RG Track peak
* RG Album gain
* RG Album peak
* RG Reference loudness

Rows appear only for files that actually carry the tags (loudgain-tagged
library); radio stations, untagged files, and non-audio selections are
unaffected. The Playback tab is not modified — it already shows moOde's MPD
replaygain mode setting (Off / Track / Album), which is the engine-side
complement of these file-side tags.

Verified against MPD v0.23.13 source: `readcomments` emits raw tag pairs for
both ID3v2.4 TXXX frames (MP3) and Vorbis comments (FLAC/OGG), so all
loudgain-tagged formats are covered.

## Compatibility

* Patch anchors verified byte-identical on moOde **r942** (current release) and
  the develop branch. The script auto-detects which path-quoting style the
  installed version uses (`escapeDblQuotes()` exists only in newer versions)
  and generates the matching code.
* Before editing, the script validates both anchor strings; if the installed
  moOde has drifted from both known layouts, it aborts without touching
  anything.
* After patching it runs `php -l` and auto-restores the backup if the syntax
  check fails.

## Install (run on the moOde Pi)

```bash
scp moode-rg-audioinfo pi@<moode-host>:~/
ssh pi@<moode-host>
sudo ./moode-rg-audioinfo install
```

Then refresh the browser (F5) and open **M → Audio info → Track** on a
loudgain-tagged track. The RG rows appear after the Comment row.

## Status / restore

```bash
sudo ./moode-rg-audioinfo --status    # installed or not
sudo ./moode-rg-audioinfo --restore   # revert to most recent backup
```

Backups are kept at `/var/backups/rg-audioinfo/audioinfo.php.<timestamp>.orig`
(original file is never deleted). Idempotent: re-running `install` on a patched
file does nothing.

## Notes

* The patch survives moOde upgrades only until the upgrade overwrites
  `/var/www/command/audioinfo.php` — after a moOde update, re-run
  `install` (it re-validates anchors against the new file).
* Tests the same data source for every launch path of the Track tab: the "M"
  menu Audio info, the playbar info button, and Library song-row info.
* No files outside `/var/www/command/audioinfo.php` are modified.
