# Dropbox KB Auto

**OpenClaw skill: Fully automated Dropbox knowledge base with interactive setup, delta-based sync, OCR, Office file extraction, and semantic search integration.**

## How It Works

```
Dropbox files → This skill extracts text → Markdown files → OpenClaw indexes & embeds → Searchable via agent
```

**What this skill does:**
- Monitors your Dropbox for new/changed files (delta-based sync)
- Extracts text from PDFs, images (OCR), Office files, documents
- Saves content as markdown in OpenClaw's knowledge base folder

**What OpenClaw handles automatically:**
- Generates vector embeddings (text-embedding-3-small via OpenAI)
- Stores in PostgreSQL with pgvector extension
- Provides semantic + keyword hybrid search (70% vector + 30% BM25)
- Makes content available to your agent for retrieval

**You get:** Ask your agent about any Dropbox document and it can answer based on the content.

## Features

🚀 **Delta-based sync** - Only processes new/changed files (10x-100x faster than full scans)  
📄 **Multi-format extraction** - PDF, Word, Excel, PowerPoint, images, text files  
🔍 **OCR support** - Extracts text from scanned PDFs and images (eng + deu)  
⚡ **Incremental indexing** - Efficient hash-set lookups, cursor persistence  
📊 **Office file support** - Excel (first 5 sheets), PowerPoint (first 30 slides), Word docs  
🎯 **Production-tested** - Handles 650K+ files, rate limits, retries, large file edge cases  

## What This Skill Includes

- **Delta-based Dropbox sync** - Only processes changed files (fast incremental updates)
- **Multi-format text extraction** - PDF (OCR fallback), images (Tesseract), Office files, documents
- **Cron scheduler integration** - Runs automatically every 6 hours (configurable)
- **Progress tracking** - Remembers indexed files, handles deletions

**Not included (provided by OpenClaw core):**
- Vector embeddings generation (OpenAI API)
- PostgreSQL + pgvector storage
- Semantic search engine
- Agent memory/retrieval system

## Requirements

- OpenClaw installed and running
- Dropbox account with API access
- Linux/macOS system (tested on Ubuntu 24.04, macOS 14+)

## Installation

### Prerequisites

```bash
# Install system dependencies (Debian/Ubuntu)
sudo apt-get update
sudo apt-get install -y tesseract-ocr tesseract-ocr-eng tesseract-ocr-deu poppler-utils

# Install Python dependencies
pip3 install pypdf openpyxl python-pptx python-docx
```

### Install via ClawHub

**Recommended:**
```bash
clawhub install dropbox-kb-auto
cd ~/.openclaw/workspace/skills/dropbox-kb-auto
./install.sh  # Interactive setup
```

**Manual installation:**
```bash
cd ~/.openclaw/workspace/skills
git clone https://github.com/ferreirafabio/dropbox-kb-auto.git
cd dropbox-kb-auto
./install.sh  # Interactive setup
```

> **Note:** ClawHub is OpenClaw's package manager for agent skills. Learn more at [clawhub.com](https://clawhub.com)

## Quick Start

**Interactive installer handles everything:**
```bash
./install.sh
```

The installer will:
1. ✅ Install system dependencies (tesseract, poppler)
2. ✅ Install Python packages (pypdf, openpyxl, etc.)
3. ✅ **Ask which Dropbox folders to index** (interactive)
4. ✅ **Ask which paths to exclude** (interactive)
5. ✅ Configure file types and OCR languages
6. ✅ Set up Dropbox API credentials
7. ✅ Create automatic cron job (schedule of your choice)
8. ✅ Run initial sync (optional)

## Manual Setup

If you prefer manual configuration:

### 1. Create Dropbox App

1. Go to https://www.dropbox.com/developers/apps
2. Create app → **Scoped access** → **Full Dropbox** (or App folder if preferred)
3. Permissions tab:
   - ✅ `files.metadata.read`
   - ✅ `files.content.read`
4. Copy **App key**, **App secret**
5. Generate **Refresh token**:
   ```bash
   # Use Dropbox OAuth 2 flow or their official SDK
   ```

### 2. Configure Credentials

Add to `~/.openclaw/.env`:
```bash
DROPBOX_FULL_APP_KEY=your_app_key
DROPBOX_FULL_APP_SECRET=your_app_secret
DROPBOX_FULL_REFRESH_TOKEN=your_refresh_token
```

### 3. Customize Config

Edit `config.json`:
```json
{
  "folders": [
    "/Documents",
    "/Work",
    "/Research"
  ],
  "skip_paths": [
    "/Documents/Archive",
    "/Downloads"
  ],
  "file_types": ["pdf", "docx", "xlsx", "pptx", "jpg", "png", "txt"],
  "max_file_size_mb": 20
}
```

### 4. Run Initial Sync

```bash
cd ~/.openclaw/workspace/skills/dropbox-kb-auto
python3 dropbox-sync.py
```

First run takes 5-10 minutes (builds delta cursors). Subsequent runs: <10 seconds.

### 5. Set Up Cron Job (Optional)

```bash
openclaw cron create \
  --name "Dropbox KB Sync" \
  --cron "0 */6 * * *" \
  --tz "Europe/Berlin" \
  --timeout-seconds 14400 \
  --session isolated \
  --message "Run Dropbox KB sync: cd ~/.openclaw/workspace/skills/dropbox-kb-auto && python3 dropbox-sync.py"
```

## Usage

### Manual Sync
```bash
python3 ~/.openclaw/workspace/skills/dropbox-kb-auto/dropbox-sync.py
```

### Search Your Files
Once indexed, ask your agent:
- "Show me my blood test results from 2026"
- "Find documents about tax deductions"
- "Search for presentations about machine learning"

Your agent searches using:
- **Vector similarity** (semantic understanding via embeddings)
- **Keyword matching** (BM25 full-text search)
- **Hybrid ranking** (combines both for best results)

All handled automatically by OpenClaw's memory system (PostgreSQL + pgvector).

## How It Works

### Delta-Based Sync
Uses Dropbox's `list_folder` with cursor persistence:
1. **First run:** Lists all files, saves cursor
2. **Subsequent runs:** Fetches only changes since cursor (new/modified/deleted)
3. **Result:** 10x-100x faster than re-scanning everything

### Text Extraction Pipeline
```
File detected → Check extension
  ├─ PDF → pypdf extraction → OCR fallback (tesseract)
  ├─ DOCX/DOC → python-docx
  ├─ XLSX/XLS → openpyxl (first 5 sheets, 100 rows each)
  ├─ PPTX/PPT → python-pptx (first 30 slides)
  ├─ JPG/PNG → tesseract OCR (eng+deu)
  └─ TXT/MD/CSV → direct UTF-8 read
    ↓
Save as markdown → OpenClaw auto-generates embeddings
```

## File Structure

```
dropbox-kb-auto/
├── README.md              # This file
├── SKILL.md               # OpenClaw skill manifest
├── dropbox-sync.py        # Main indexer (delta-based)
├── config.json            # User configuration
├── setup.sh               # Dependency installer
├── .env.example           # Credential template
└── LICENSE                # MIT License
```

## Supported File Types

| Type | Extensions | Extraction Method |
|------|-----------|------------------|
| PDF | `.pdf` | pypdf + OCR fallback |
| Word | `.docx`, `.doc` | python-docx |
| Excel | `.xlsx`, `.xls` | openpyxl |
| PowerPoint | `.pptx`, `.ppt` | python-pptx |
| Images | `.jpg`, `.jpeg`, `.png` | Tesseract OCR |
| Text | `.txt`, `.md`, `.csv`, `.json` | UTF-8 decode |

## Configuration

### Folders to Index
```json
"folders": ["/Documents", "/Work"]
```

### Skip Patterns
```json
"skip_paths": [
  "/Archive",
  "/Backups",
  "/Downloads"
]
```

### File Size Limit
```json
"max_file_size_mb": 20
```

## Troubleshooting

### "No module named 'pypdf'"
```bash
pip3 install pypdf openpyxl python-pptx python-docx
```

### "tesseract: command not found"
```bash
sudo apt-get install tesseract-ocr tesseract-ocr-eng tesseract-ocr-deu
```

### Rate Limiting
The indexer includes automatic retry logic with exponential backoff. If you hit Dropbox rate limits:
- Reduce folder scope
- Increase sleep intervals in `dropbox-sync.py`

### Timeout on First Run
Initial sync of large Dropboxes (500K+ files) can take 10-20 minutes. Increase cron timeout:
```bash
openclaw cron edit <job-id> --timeout-seconds 28800  # 8 hours
```

## Performance

**Test case:** 650,000 files, 1,840 indexable documents

| Metric | First Run | Subsequent Runs |
|--------|-----------|----------------|
| List time | 5-10 min | <10 sec |
| Files indexed | 1,840 | 5-20 (avg) |
| Total time | ~15 min | <30 sec |
| Disk usage | 45 MB | +100 KB |

## Privacy & Security

- **Read-only access recommended** - Use scoped app with only read permissions
- **Local processing** - All extraction happens on your machine
- **No data transmission** - Files stay in your Dropbox and local knowledge base
- **Credential storage** - Store tokens in `~/.openclaw/.env` (not in repo)

## Comparison to Alternatives

| Feature | dropbox-kb-auto | dropbox-api | dropbox-integration |
|---------|----------------|-------------|---------------------|
| Auto-indexing | ✅ | ❌ | ❌ |
| OCR support | ✅ | ❌ | ❌ |
| Office files | ✅ | ❌ | ❌ |
| Delta sync | ✅ | ❌ | ❌ |
| Semantic search | ✅ (via OpenClaw KB) | ❌ | ❌ |
| Use case | Knowledge base | File operations | Manual browsing |

## Contributing

Issues and PRs welcome at https://github.com/ferreirafabio/dropbox-kb-auto

## License

MIT License - see LICENSE file

## Related Links

- **OpenClaw Docs:** https://docs.openclaw.ai
- **OpenClaw GitHub:** https://github.com/openclaw/openclaw
- **ClawHub (Skill Marketplace):** https://clawhub.com
- **Community Discord:** https://discord.gg/clawd

## Author

Fabio Ferreira ([@ferreirafabio](https://github.com/ferreirafabio))

## Changelog

### v1.0.0 (2026-02-21)
- Initial release
- Delta-based Dropbox sync
- OCR support (eng + deu)
- Office file extraction (Excel, PowerPoint, Word)
- Production-tested on 650K+ files
