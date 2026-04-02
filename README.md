# MBZ Trimmer

A tool that trims bloated Moodle `.mbz` backup files by removing questions, files, and course data not used by the quizzes in the backup.

**Live app:** [aburomoh.github.io/mbz-trimmer](https://aburomoh.github.io/mbz-trimmer/)

## The Problem

When you export a Moodle quiz as a backup, Moodle drags in the **entire question bank** from the course — every question from every semester, every category, every embedded image — even if the quiz only uses a handful of them. A quiz with 16 questions can balloon into a 20 MB backup file.

Restoring these bloated backups into another course pollutes it with hundreds of unused questions, unwanted groups, and stale course data.

## What It Does

1. Extracts the `.mbz` archive (which is a `.tar.gz`)
2. Discovers all quiz activities and traces their question references
3. Keeps only the categories and questions actually used by the quizzes
4. Filters embedded media (images, drag-drop assets) to match kept questions
5. Optionally strips course-specific data (groups, enrolments, non-quiz activities) for clean sharing
6. Repackages as a new `.mbz` ready for Moodle restore

### What's Preserved

- Quiz settings, slot order, and grade distribution (maxmarks per slot)
- Randomization configuration (which categories to draw from)
- All questions in random-draw pools
- Embedded media (images, drag-drop assets) for kept questions
- Gradebook structure

### What's Removed

- Unused question bank categories and entries
- File records and content files for removed questions
- Non-quiz activities and course structure (when "Strip for sharing" is enabled)
- Groups, enrolments, roles (when stripping)

## Usage

### Web App (recommended)

Visit [aburomoh.github.io/mbz-trimmer](https://aburomoh.github.io/mbz-trimmer/) — runs entirely in your browser. No installation, no server, files never leave your machine.

1. Drop your `.mbz` file
2. Check "Strip for sharing" if desired
3. Click "Trim Backup"
4. Download the trimmed `.mbz`

### Desktop (Node.js)

Requires [Node.js](https://nodejs.org) installed.

**GUI:** Double-click `App/MBZ Trimmer.bat` or run:
```
node App/trim_mbz_gui.js
```

**CLI:**
```
node App/trim_mbz_cli.js input.mbz output.mbz
node App/trim_mbz_cli.js --strip input.mbz output.mbz
```

## Supported Moodle Versions

Handles question reference formats across Moodle 3.x through 4.5+:

| Format | Moodle Version |
|--------|---------------|
| `qtype_random` entries | 3.x |
| `"questioncategoryid": NNN` | 4.0 - 4.2 |
| `"filter":{"category":{"values":[NNN]}}` | 4.3+ |
| `"cat":"NNN,contextid"` | 4.3+ |

## Project Structure

```
App/                     The Node.js desktop app
  mbz_engine.js            Shared trim engine
  trim_mbz_gui.js          Web GUI (localhost:3456)
  trim_mbz_cli.js          Command-line interface
  MBZ Trimmer.bat          Windows launcher
  README.txt               Desktop app instructions

web/                     The static web app (deployed to GitHub Pages)
  index.html               Single-file client-side app (zero dependencies)

Docs/                    Documentation
  MBZ_Quiz_Structure_Memo.md   Detailed notes on Moodle .mbz internals

Test_Files/              Test data
  Source_Backups/            Original untrimmed .mbz files
  Trimmed_Gold/              Known-good trimmed outputs for validation
  XML_Bank/                  Sample Moodle XML question files
```

## How It Works Under the Hood

A Moodle `.mbz` backup contains:

- **`quiz.xml`** — quiz slots referencing questions by `questionbankentryid` (direct) or `filtercondition` JSON (random draws)
- **`questions.xml`** — the full question bank with categories, entries, and question versions
- **`files.xml`** — index mapping question IDs to content file hashes
- **`files/`** — binary content files (images, attachments) stored by SHA1 hash

The trimmer traces the reference chain from quiz slots through entries, categories, and file records — keeping only what the quizzes actually need and discarding everything else.

## License

Free to use and share.
