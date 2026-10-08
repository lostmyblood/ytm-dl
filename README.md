# ytm-dl

**A minimal terminal UI for bulk-downloading YouTube Music albums, EPs and singles with yt-dlp.**

ytm-dl is a single, lightweight bash script that wraps [yt-dlp](https://github.com/yt-dlp/yt-dlp). Give it a text file full of YouTube Music links, a comma-separated list, or one link, and it queues everything, downloads several links in parallel, and shows the status of each album, EP and single in a quiet gray-scale TUI.

```
 ytm-dl  0.4.0
 ──────────────────────────────────────────────────────────────
 [Link]  links.txt

 [Download status]
 › album  XXX                                   8/10  ━━━━━━━━────   80%
 › ep     XXX                                   3/12  ━━━─────────   25%
 · single XXX                                         ────────────
 ✓ track  XXX                                         ━━━━━━━━━━━━  100%

 XXX  8/10  ·  XXX  41%  2.3MiB/s  eta 00:04
 XXX  3/12  ·  XXX  12%  1.9MiB/s  eta 00:11
 ──────────────────────────────────────────────────────────────
 1/4 done · 0 failed · 2 active   mp3 · q cancel
```
---

## Features

- **Flexible input.** A `.txt` file, comma-separated links, or a single link. Inside a file, links can be separated by commas, new lines, or both. Duplicates are dropped automatically.
- **Minimal TUI.** A `[Link]` line for what you gave it and a `[Download status]` list with one row per item: type (`album`, `ep`, `single`, `track`, `list`), track count, progress bar and percentage.
- **Live detail line.** For every running link: the current track, its percentage, speed and ETA, so you can tell the difference between slow and stuck.
- **Format picker.** Choose `mp3` (default), `m4a`, `opus` or `flac` from a short menu, or skip it with `-f`.
- **Parallel downloads.** Several links at once (2 by default).
- **Stall protection.** If a link receives no new data for 60 seconds, it is stopped and retried automatically (2 retries by default). Finished tracks are skipped on retry, and half-downloaded files resume.
- **Resumable.** A download archive means re-running the same file only fetches what is missing.
- **Tagged output.** Embedded metadata and square-cropped cover art.
- **Debug friendly.** `--plain` shows raw yt-dlp output, and `-v` keeps verbose logs of failed links.
- **Terminal friendly.** It only redraws when something changed, restores your terminal on exit, and falls back to plain mode automatically when there is no terminal (pipes, cron).
- **Lightweight.** One shell script, no runtime beyond yt-dlp and ffmpeg, no background daemon.

---

## Requirements

| Type | Package | Why |
|---|---|---|
| Required | `bash` | the script itself |
| Required | `yt-dlp` | does the downloading |
| Required | `ffmpeg` | audio conversion, tagging, cover art |
| Required | `procps-ng` | `pkill`, used to stop stalled downloads cleanly |
| Optional | `deno` | JavaScript runtime that yt-dlp needs for YouTube (**recommended**) |
| Optional | `nodejs` | alternative JavaScript runtime if `deno` is not installed |
| Optional | `python-mutagen` | cover art embedding for `opus` files |

Without a JavaScript runtime, YouTube downloads may be slow or fail. On Arch:

```bash
sudo pacman -S yt-dlp ffmpeg deno
```

---

## Installation

### Arch Linux (package)

```bash
tar xzf ytm-dl-pkg.tar.gz
cd ytm-dl-pkg
makepkg -si
```

Uninstall with `sudo pacman -R ytm-dl`.

### Without installing

```bash
bash ytm-dl links.txt
```

### Manual install

```bash
sudo install -Dm755 ytm-dl /usr/local/bin/ytm-dl
sudo install -Dm644 ytm-dl.1 /usr/local/share/man/man1/ytm-dl.1
```

---

## Usage

```bash
ytm-dl                          # prompt: file path, comma-separated links, or one link
ytm-dl links.txt                # a file of links
ytm-dl 'LINK1,LINK2'            # several links
ytm-dl 'LINK'                   # a single link
ytm-dl -f flac links.txt        # choose a format and skip the picker
ytm-dl -j 3 links.txt           # three links in parallel
ytm-dl -o ~/Music/x links.txt   # custom output directory
ytm-dl -c firefox links.txt     # use browser cookies
ytm-dl -v links.txt             # verbose, keep logs of failures
ytm-dl --plain links.txt        # no TUI, raw yt-dlp output
ytm-dl --example                # print an example links file
ytm-dl -V                       # version
ytm-dl -h                       # help
man ytm-dl                      # manual page
```

### Options

| Option | Description | Default |
|---|---|---|
| `-o, --out DIR` | output directory | `~/Music/ytm-dl` |
| `-f, --format FMT` | `mp3`, `m4a`, `opus` or `flac`; skips the format picker | `mp3` |
| `-j, --jobs N` | links downloaded in parallel | `2` |
| `-c, --cookies BROWSER` | take cookies from a browser, e.g. `firefox` | off |
| `-v, --verbose` | pass `-v` to yt-dlp and keep logs of failed links | off |
| `--plain` | no TUI, one link at a time, raw yt-dlp output | off |
| `--example` | print an example links file | |
| `-V, --version` | print the version | |
| `-h, --help` | print usage | |

### Environment variables

| Variable | Meaning | Default |
|---|---|---|
| `YTM_DL_DIR` | output directory | `~/Music/ytm-dl` |
| `YTM_DL_FORMAT` | default format (skips the picker when set) | `mp3` |
| `YTM_DL_JOBS` | parallel links | `2` |
| `YTM_DL_STALL` | seconds without new data before a link is stopped and retried | `60` |
| `YTM_DL_RETRIES` | automatic retries per link | `2` |
| `YTM_DL_COOKIES` | browser to take cookies from | none |
| `YTM_DL_VERBOSE` | set to `1` for verbose output and kept logs | off |

---

## Link file format

Links are separated by commas, new lines, or both:

```
https://music.youtube.com/playlist?list=OLAK5uy_AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA,
https://music.youtube.com/playlist?list=OLAK5uy_BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB,
https://music.youtube.com/watch?v=XXXXXXXXXXX,
https://music.youtube.com/watch?v=YYYYYYYYYYY
```

- **Albums, EPs and singles:** use the playlist link (`playlist?list=OLAK5uy_…`), which downloads the whole release.
- **Individual tracks:** use `watch?v=…` or `youtu.be/…` links. These download only that track, even if the link also carries a playlist or radio ID.
- Lines that are not YouTube links are ignored. Quotes around a path or link are removed, so drag-and-drop paths work.

---

## Exit status

| Code | Meaning |
|---|---|
| `0` | every link succeeded |
| `1` | at least one link failed |
| `130` | cancelled with `q` or Ctrl-C |

