# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Generates Anki flashcard decks for Taiwanese Taigi (台灣台語) from MOE (教育部) open data at https://sutian.moe.edu.tw/zh-hant/siongkuantsuguan/.

## Environment

- **Ruby version**: 4.0.0 (managed via RVM with gemset `anki-taigi`)
- **Dependencies**: `csv`, `rake`, `rexml`, `rubyzip`, `sqlite3` (managed via Bundler)

## Run

```bash
bundle install
rake build     # full pipeline (default)
rake fetch     # download data from MOE
rake parse     # parse ODS → CSV cache
rake audio     # extract MP3s from zips
rake export    # rebuild deck from cached CSV + audio
rake test      # run tests
rake clean     # remove output/
rake clobber   # remove output/ + data/
```

`ruby generate.rb` also runs the full pipeline as a shortcut.

## Architecture

`Rakefile` defines the pipeline steps, `generate.rb` is a standalone shortcut. Modules:

1. **`lib/moe_fetcher.rb`** — Downloads `kautian.ods` + MP3 zip files from MOE via `net/http`
2. **`lib/ods_parser.rb`** — Parses ODS (ZIP+XML) using `rubyzip` + `rexml`. Handles `office:value` attributes for numeric IDs, `number-columns-repeated` / `number-rows-repeated` for cell/row spans
3. **`lib/taigi_dict.rb`** — Loads data via `Dictionary.from_ods` or `Dictionary.from_csv`. Joins Entry→Definition(義項)→Example into Card structs. Can also `export_csv` for caching
4. **`lib/audio_extractor.rb`** — Extracts MP3s from zip archives with prefixes (`sutiau-`, `leku-`)
5. Export:
   - **`lib/anki_exporter.rb`** — Exports Cards to tab-separated text with `[sound:]` tags
   - **`lib/apkg_exporter.rb`** — Exports Cards to `.apkg` (Anki Package) with embedded audio. Builds SQLite DB (`collection.anki2`) with note type, card template, and media map, then packages as ZIP. Deduplicates audio files.

## Data Model

Entry (詞目) → has many Definition (義項) → has many Example (例句). Cross-referenced via `詞目id` and `義項id`. Audio filenames encode the relationship: entry `N(1)`, example `N-M-K`.

## Key Data Files (not committed)

- `data/kautian.ods` — raw MOE dictionary (4MB, 19 sheets)
- `data/sutiau-mp3.zip` / `data/leku-mp3.zip` — audio archives (~285MB / ~490MB)
- `data/csv/` — cached CSV exports from ODS
- `output/taigi.apkg` — self-contained Anki package (~694MB, includes audio)
- `output/taigi_deck.txt` — tab-separated Anki import file (23K notes)
- `output/audio/` — extracted MP3s (~34K unique files)
