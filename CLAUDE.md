# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Aziza Transfer Caisse** — A Flask web application for checking file existence across network shares (SMB) and transferring files to multiple servers. Packaged as a standalone Windows executable via PyInstaller. UI is in French.

## Build & Run

```bash
# Install dependencies
pip install -r requirements.txt

# Run in development
python app.py
# Opens browser at http://127.0.0.1:5001

# Build Windows executable
build.bat
# Output: dist/Aziza Transfer Caisse.exe

# Build manually with PyInstaller
python -m PyInstaller --clean FileChecker.spec
```

## Architecture

**Single-file Flask backend** (`app.py`, ~1170 lines) serving three HTML pages with embedded CSS/JS:

- `/` → `index.html` — Upload Excel with CodeMag/ipaddress columns, check if a named file exists on each network path. Generates downloadable Excel report.
- `/auth` → `auth.html` — Test single or bulk network connections (SMB credentials). Manages active connections.
- `/transfer` → `transfer.html` — Transfer files to multiple servers from Excel list. Generates transfer report.

**Network operations** use Windows `net use` subprocess calls for SMB authentication. File transfers use `shutil.copy2`.

**PyInstaller dual-mode**: `app.py` detects `sys.frozen` to resolve paths correctly both in development (script) and as a compiled EXE. Templates and assets are bundled via `FileChecker.spec`.

### Key directories

- `templates/` — Jinja2 HTML templates (each page is self-contained with inline styles/scripts)
- `assets/` — Static files (logo.png), served at `/assets`
- `data/` — Excel files (e.g. `liste_mag.xlsx`) selectable from the UI
- `reports/` — Auto-generated Excel reports (timestamped)
- `uploads/` — Temporary file storage for uploads

### API routes (all return JSON)

| Endpoint | Purpose |
|---|---|
| `POST /check-files` | Batch file existence check from Excel |
| `POST /transfer-file` | Transfer single file to servers |
| `POST /transfer-files` | Transfer multiple files to servers |
| `POST /test-connection` | Test single SMB connection |
| `POST /test-bulk-connections` | Test connections from Excel |
| `POST /disconnect-all` | Drop all active SMB connections |
| `GET /active-connections` | List current connections |
| `GET /get-excel-files` | List Excel files in data folder |
| `POST /shutdown` | Graceful server shutdown |

## Key Conventions

- Excel files must contain `CodeMag` and `ipaddress` columns (case-insensitive detection)
- Default SMB credentials: `trf`/`trf` (configured via `DEFAULT_USERNAME`/`DEFAULT_PASSWORD` in `app.py`)
- Reports saved as `reports/report_YYYYMMDD_HHMMSS.xlsx` with auto-sized columns and summary rows
- Max upload size: 16MB
- Flask runs on port 5001, localhost only, debug off
