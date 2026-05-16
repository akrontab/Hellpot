# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **log analysis repository** for a HellPot HTTP honeypot. It is **not** the HellPot application source code. The actual HellPot Go server runs remotely on `alpine-linode` at `/home/www/.config/HellPot/` and listens on port 8080.

This repo contains:
- `logs/` — JSON-formatted honeypot log files collected from the remote server
- `LogStats.ipynb` — Jupyter notebook for analyzing attack patterns
- `Get-Logs.ps1` — PowerShell script to SCP logs from the remote server

## Common Commands

**Fetch new logs from remote server:**
```powershell
.\Get-Logs.ps1
```

**Run analysis (Jupyter):**
```powershell
.venv\Scripts\jupyter notebook LogStats.ipynb
```

**Activate the Python virtual environment:**
```powershell
.venv\Scripts\Activate.ps1
```

## Log Format

Each log line is a JSON object:
```json
{
  "level": "info|debug|warn",
  "time": "2022-08-23T04:37:31Z",
  "message": "NEW|FINISH|END_ON_ERR",
  "REMOTE_ADDR": "IP address",
  "URL": "requested path",
  "USERAGENT": "user agent string",
  "BYTES": "bytes transferred (as string)",
  "DURATION": "connection duration in ms (as string)"
}
```

`NEW` events mark connection start; `FINISH` marks clean connection end; `END_ON_ERR` indicates the client disconnected early (common with bots that bail once they realize it's a honeypot).

## Analysis Architecture

`LogStats.ipynb` loads all log files from `logs/`, concatenates them into a single pandas DataFrame, and performs:
- Time-series connection volume analysis
- Monthly data transfer distribution (boxplots)
- URL/parameter pattern extraction to identify exploit attempts (LFI, RFI, WordPress attacks)
- Base64 payload decoding from User-Agent headers to reveal attacker tooling (crypto miners, shell downloaders)

The notebook reads logs with `pd.read_json(..., lines=True)` since each file is newline-delimited JSON.
