# Portable Hermes Agent — Cross-Platform Setup Plan

## 1. Research Summary: What is Hermes Agent?

**Hermes Agent** is an open-source, self-improving AI agent built by Nous Research (MIT License). It is NOT a chatbot — it is a persistent autonomous agent that learns from experience, creates reusable skills, and maintains memory across sessions.

### What It Can Do
- **Persistent Memory**: Remembers preferences, projects, environment across every session
- **Automated Skill Creation**: When it solves a hard problem, it writes a reusable skill document
- **Multi-Platform Gateway**: Connect Telegram, Discord, Slack, WhatsApp, Signal, Email, CLI
- **Scheduled Automations**: Built-in cron scheduler for daily reports, backups, audits
- **Parallel Sub-Agents**: Spawn isolated sub-agents for parallel workstreams
- **Full Browser & Web Control**: Web search, page extraction, browser automation, vision analysis
- **200+ Model Support**: OpenRouter, Nous Portal, OpenAI, Anthropic, Ollama, LM Studio, local endpoints
- **MCP Integration**: Connect any Model Context Protocol server
- **Voice Mode**: Speech-to-text via faster-whisper

### How It Works
- Python 3.11 application using `uv` package manager
- Node.js dependency for browser automation tools
- Data stored in `~/.hermes/` directory containing:
  - `config.yaml` — configuration
  - `.env` — API keys and secrets
  - `sessions/` — conversation history
  - `memories/` — persistent memory
  - `skills/` — learned skills
  - `cron/` — scheduled jobs
  - `logs/`, `pairing/`, `hooks/`, `image_cache/`, `audio_cache/`
- Entry point: `hermes` CLI command (symlinked to `~/.local/bin/`)

### Standard Installation
- **Linux/macOS/WSL2**: `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash`
- **Windows (native, early beta)**: `irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex`
- Installer auto-downloads: `uv`, Python 3.11, Node.js, `ripgrep`, `ffmpeg`, Git (MinGit on Windows)
- Windows native support is explicitly flagged "early beta" by Nous Research

---

## 2. Feasibility: Is True Cross-Platform Portability Possible?

**Short Answer: YES, but with important caveats.**

A single folder on a USB drive CAN work across Mac, Linux, and Windows **IF** it contains pre-built runtime environments for each target platform and uses a smart launcher.

### Why This Is Hard (Technical Barriers)
1. **Different Binary Formats**: macOS uses Mach-O, Linux uses ELF, Windows uses PE — these are mutually incompatible
2. **CPU Architectures**: Apple Silicon (ARM64) vs Intel/AMD (x86_64) need different native binaries
3. **Compiled Python Packages**: Many dependencies (e.g., `psutil`, `cryptography`, `faster-whisper`) contain C extensions that must match the exact OS + architecture
4. **Node.js Native Modules**: Browser automation tools compile Node addons per-platform
5. **Path Hardcoding**: Hermes defaults to `~/.hermes/` and `~/.local/bin/hermes` — must be overridden via env vars
6. **System Dependencies**: `ripgrep`, `ffmpeg`, `git`, Playwright Chromium — each needs OS-specific builds
7. **File Permissions**: Unix executable bits don’t exist on Windows; ACLs don’t transfer to Unix
8. **Security Gatekeeping**: macOS may quarantine unsigned binaries; Windows Defender may flag portable executables
9. **Browser Sandbox**: Playwright/Chromium may refuse to launch from removable drives due to sandbox policies

### Critical Finding: Hermes Already Supports Custom Data Directory
Hermes respects the `HERMES_HOME` environment variable:
```bash
HERMES_HOME=/path/to/usb/hermes-data
```
This means **data portability is already solved** — we just need to bundle the runtimes.

---

## 3. Recommended Architecture

### Option A: Pre-Baked Multi-Platform Runtimes (Recommended)
A single USB drive containing platform-specific subfolders. The launcher detects OS/architecture at runtime and selects the correct Python/Node/venv.

```
HermesPortable/
├── launcher.sh           # Unix launcher (macOS/Linux)
├── launcher.bat          # Windows launcher
├── launcher.command      # macOS double-clickable
├── data/                 # ALL user data (shared across platforms)
│   ├── config.yaml
│   ├── .env
│   ├── sessions/
│   ├── memories/
│   ├── skills/
│   ├── cron/
│   ├── logs/
│   └── ...
└── runtimes/
    ├── windows-x64/
    │   ├── python/       # Embedded Python 3.11
    │   ├── node/         # Portable Node.js 22
    │   ├── uv/           # uv binary
    │   ├── git/          # MinGit portable
    │   ├── bin/          # ripgrep, ffmpeg
    │   └── hermes-agent/ # Source code + venv
    ├── macos-arm64/
    │   ├── python/       # python-build-standalone (indygreg)
    │   ├── node/         # Prebuilt Node.js macOS arm64
    │   ├── uv/
    │   ├── bin/          # ripgrep, ffmpeg (Homebrew bottles)
    │   └── hermes-agent/ # Source + venv
    ├── macos-x64/
    │   └── (same structure as macos-arm64)
    ├── linux-x64/
    │   └── (same structure)
    └── linux-arm64/
        └── (same structure)
```

**How the launcher works:**
1. Detect OS (`uname -s` / `VER`)
2. Detect architecture (`uname -m` / `PROCESSOR_ARCHITECTURE`)
3. Set `HERMES_HOME` to `../data` (relative)
4. Set `PATH` to include `../runtimes/<platform>/bin`
5. Activate the platform-specific venv
6. Launch `hermes`

### Option B: Shared Source + Download Runtimes On-Demand
Keep one copy of the source code and download/cache platform-specific runtimes on first launch.

```
HermesPortable/
├── launcher.sh / launcher.bat
├── data/
├── src/                  # Shared Hermes source (git clone)
└── .cache/
    └── runtimes/
        ├── windows-x64/  # Downloaded on first Windows run
        ├── macos-arm64/  # Downloaded on first Mac run
        └── ...
```

**Pros of Option B**: Smaller initial size (~100MB vs 2-5GB)
**Cons of Option B**: Requires internet on first use per platform; slower first launch

---

## 4. Implementation Steps

### Step 1: Prepare Shared Data Structure
- Create `data/` directory with Hermes directory tree
- Copy `cli-config.yaml.example` → `data/config.yaml`
- Create `data/.env`, `data/SOUL.md`
- Ensure ALL paths in `data/config.yaml` use relative or `$HERMES_HOME` references

### Step 2: Build Platform Runtimes
For each target platform (windows-x64, macos-arm64, macos-x64, linux-x64):

1. **Python**: Use platform-specific portable Python
   - Windows: `python-3.11.x-embed-amd64.zip` or WinPython
   - macOS/Linux: `python-build-standalone` from github.com/indygreg

2. **Node.js**: Download official prebuilt binaries from nodejs.org

3. **uv**: Download from astral.sh (platform-specific releases)

4. **Git / ripgrep / ffmpeg**:
   - Windows: MinGit, ripgrep releases, ffmpeg builds
   - macOS: Extract from Homebrew bottles or build statically
   - Linux: Statically linked binaries (musl)

5. **Hermes Source + Venv**:
   - `git clone https://github.com/NousResearch/hermes-agent.git`
   - `uv venv venv --python <portable-python>`
   - `uv pip install -e ".[all]"`
   - Note: venv MUST be built on the target platform (or at least matching OS/arch)

### Step 3: Create Smart Launchers
- `launcher.sh` (Unix): Bash script detecting OS/arch, setting env vars, running Hermes
- `launcher.bat` (Windows): Batch/PowerShell script doing the same
- `launcher.command` (macOS): Double-clickable wrapper around `launcher.sh`

Key env vars to set:
```bash
export HERMES_HOME="$(cd "$(dirname "$0")/data" && pwd)"
export PATH="$RUNTIME/bin:$PATH"
export UV_PYTHON="$RUNTIME/python/bin/python3"
# Prevent writing to host home directory
export PYTHONNOUSERSITE=1
export PYTHONPATH=""
```

### Step 4: Patch Hermes for Full Portability
Hermes’ installer hardcodes some paths. May need patches:
- The `hermes` CLI shim references absolute venv paths — launcher bypasses this
- Some Python code may expand `~` directly — verify `HERMES_HOME` is respected everywhere
- Playwright browser cache defaults to `~/.cache/ms-playwright` — must set `PLAYWRIGHT_BROWSERS_PATH` relative

### Step 5: Test Matrix
Test on real hardware for each:
- Windows 10/11 x64
- macOS Sonoma/Sequoia (Apple Silicon)
- macOS (Intel) — if included
- Ubuntu 22.04/24.04 x64
- Verify data written on one OS is readable on another (YAML, JSON, SQLite are fine; file permissions may need normalization)

---

## 5. Pros and Cons

### ✅ Pros
| Benefit | Description |
|---------|-------------|
| **True Mobility** | One USB drive works on any machine without installation |
| **Zero Host Footprint** | No files, registry entries, or shell configs on the host computer |
| **Privacy & Security** | All API keys, chat history, memories stay on your drive |
| **Consistent Environment** | Same Hermes version, same skills, same config everywhere |
| **Easy Backup** | Copy one folder to back up your entire AI agent |
| **No Admin Rights** | Portable Python/Node need no system installation |
| **Offline Capable** | With local models (Ollama/LM Studio), works without internet |

### ❌ Cons
| Drawback | Description |
|----------|-------------|
| **Large Size** | ~2–5GB depending on how many platforms are bundled |
| **USB Performance** | Flash drive I/O is much slower than SSD; venv imports may lag |
| **USB Wear** | Persistent memory = frequent writes to flash = reduced drive lifespan |
| **Security Risk** | Lost/stolen USB = exposed API keys, chat history, memories |
| **macOS Gatekeeper** | Unsigned binaries may be quarantined; user must right-click → Open |
| **Windows Defender** | Portable executables sometimes trigger false positives |
| **Browser Tools** | Playwright/Chromium sandbox may block execution from removable drives |
| **Platform Updates** | Each platform runtime must be updated independently |
| **Permission Loss** | Moving from Windows → Unix may strip executable bits; launcher must `chmod +x` |
| **No Native Service** | Gateway (Telegram/Discord bot) can’t auto-start on host boot from USB |

---

## 6. Sizing Estimate

| Component | Size (per platform) |
|-----------|-------------------|
| Portable Python 3.11 | ~35MB (Windows embed) / ~150MB (standalone) |
| Node.js 22 LTS | ~60MB |
| uv + git + ripgrep + ffmpeg | ~80MB |
| Hermes source code | ~50MB |
| Python venv + deps (`[all]`) | ~400–800MB |
| **Subtotal per platform** | **~600MB–1.2GB** |
| Data folder (empty) | ~10MB |
| **Total (4 platforms)** | **~2.5–5GB** |

If limiting to only 2 platforms (Windows x64 + macOS ARM64), total ~1.5–2.5GB.

---

## 7. Final Verdict

**Is a truly portable Hermes Agent possible? YES.**

**Will it work by simply plugging a Windows-formatted USB into a Mac? YES — if the Mac folder contains a pre-built macOS runtime and a launcher script that detects macOS.**

The key insight is that you cannot run Windows binaries on macOS, but you CAN ship multiple platform runtimes in the same folder and let the launcher pick the right one. Hermes’ `HERMES_HOME` environment variable makes the data layer fully portable.

**Recommendation**: Build Option A (pre-baked runtimes) for offline, plug-and-play use. This is the only approach that satisfies the user’s requirement of "setup on Windows, plug into Mac, works immediately."
