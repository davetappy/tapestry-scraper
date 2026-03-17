# Tapestry Journal Scraper

A command-line tool that downloads all observations and media assets from a [Tapestry Journal](https://tapestryjournal.com) account, organises them into a tidy folder structure, and embeds metadata into each file so dates and descriptions are preserved.

This project was built entirely with [Claude Code](https://claude.ai/claude-code) by Anthropic.

## What it does

- Logs in to your Tapestry account and fetches all observations via the internal API
- Downloads photos, videos and documents attached to each observation
- Organises files into folders by date and child, e.g. `export/2024-06/sonny/2024-06-12_sports_day/`
- Renames files sequentially: `2024-06-12_001.jpg`, `2024-06-12_002.mp4`, etc.
- Embeds the observation date into file timestamps (filesystem creation date, EXIF, and MP4 movie header)
- Embeds the title, description, child name and keywords as EXIF, IPTC and XMP metadata
- Skips files that have already been downloaded, so it is safe to re-run

## Requirements

Python 3.9 or later. Install dependencies with:

```
pip install -r requirements.txt
```

## Usage

```
python tapestry_scraper.py -e EMAIL -p PASSWORD [options]
```

### Options

| Flag | Description |
|------|-------------|
| `-e`, `--email` | Tapestry login email address |
| `-p`, `--password` | Tapestry login password |
| `-o`, `--output DIR` | Output directory (default: `./tapestry_export`) |
| `--list-children` | Print available children and their IDs, then exit |
| `--child CHILD_ID` | Only download observations for one child |
| `--limit N` | Only process the first N observations (useful for testing) |
| `-v`, `--verbose` | Enable debug logging |

### Examples

Download everything:

```
python tapestry_scraper.py -e you@example.com -p yourpassword -o ./export
```

Test with a small batch first:

```
python tapestry_scraper.py -e you@example.com -p yourpassword -o ./export --limit 5
```

Find child IDs, then download for one child only:

```
python tapestry_scraper.py -e you@example.com -p yourpassword --list-children
python tapestry_scraper.py -e you@example.com -p yourpassword -o ./export --child 12345
```

> **Note:** If your password contains special characters such as `!`, wrap it in single quotes on the command line: `-p 'your!password'`

## Output structure

```
export/
  2024-06/
    child_name/
      2024-06-12_sports_day/
        2024-06-12_001.jpg
        2024-06-12_002.jpg
        2024-06-12_003.mp4
      2024-06-20_painting/
        2024-06-20_001.jpg
```

## Metadata written

**JPEG images**
- EXIF: `DateTimeOriginal`, `ImageDescription`, `Artist`
- IPTC: `Caption-Abstract` (2:120), `By-line` (2:80)
- XMP: `dc:title`, `dc:description`, `dc:creator`, `dc:subject`, `xmp:CreateDate`

**MP4/MOV videos**
- QuickTime atoms: `©day`, `©nam`, `©cmt`, `©ART`
- Movie header (`mvhd`) creation and modification timestamps patched directly in the binary, which is what most media players and Google Photos use for the file date
- XMP uuid box with `dc:description`

**All files**
- Filesystem modification and creation timestamps set to the observation date (on Windows, uses the Win32 API to set the true creation date)

## Notes

- Tapestry Journal has no public API. This tool uses the same internal API endpoints that the web application uses, authenticated with your own credentials.
- Only tested against Tapestry 3 (the current version as of 2025).
- This is a personal archiving tool. Use it responsibly and in accordance with your school's data policies.
