# Hermes Agent - Portable Cross-Platform Setup

A **truly portable** Hermes Agent that runs from a single folder on **Windows, macOS, and Linux** without installing anything on the host computer. All your data, config, chat history, and API keys stay inside the `data/` folder.

> **Hermes Agent** is an open-source, self-improving AI agent by [Nous Research](https://github.com/NousResearch/hermes-agent). It remembers conversations, builds reusable skills, and connects to Telegram, Discord, Slack, WhatsApp, and more.

---

## Quick Start

### Windows
Double-click **`launch.bat`**

### macOS & Linux
Open a terminal in this folder and run:
```bash
./launch.sh
```

> **macOS tip:** If you want to double-click in Finder, rename `launch.sh` to `launch.command`.
> macOS recognizes `.command` files and opens them in Terminal automatically.

---

## What Happens on First Run?

1. The launcher detects your operating system and CPU architecture.
2. It downloads ~600 MB of portable runtime files:
   - Python 3.11
   - Node.js 22 LTS
   - `uv` (Python package manager)
   - `ripgrep` (fast file search)
   - Hermes Agent source code
3. It creates a local virtual environment.
4. It installs Hermes Python dependencies.
5. It launches Hermes.

**After the first run, launching is instant** — no more downloads needed.

---

## Folder Structure

```
hermes-portable/
├── launch.bat              # Windows launcher
├── launch.sh               # macOS / Linux launcher
├── scripts/
│   ├── setup-windows.ps1   # Windows first-run setup
│   └── setup-unix.sh       # macOS / Linux first-run setup
├── data/                   # YOUR DATA — moves with you
│   ├── config.yaml         # Hermes configuration
│   ├── .env                # API keys & secrets
│   ├── sessions/           # Chat history
│   ├── memories/           # Persistent memory
│   ├── skills/             # Learned skills
│   ├── cron/               # Scheduled jobs
│   └── ...
├── src/                    # Hermes source code (auto-downloaded)
│   └── hermes-agent/
├── .cache/                 # Downloaded runtimes (per-platform)
│   └── runtimes/
│       ├── windows-x64/
│       ├── macos-arm64/
│       ├── macos-x64/
│       ├── linux-x64/
│       └── linux-arm64/
└── README.md               # This file
```

---

## Portability in Action

**Scenario:** You set up Hermes on a Windows PC at home. You chat, teach it skills, and save memories.

**Next day:** You copy the entire `hermes-portable` folder to a USB drive and plug it into a Mac at work.

**Result:** Run `./launch.sh` (or double-click `launch.command` if you renamed it). Hermes launches instantly with **all your conversations, skills, and memory intact**. Nothing was ever saved on the Windows PC or the Mac.

---

## Adding Your API Keys

Edit `data/.env` and add your keys:

```bash
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxx
# or
OPENAI_API_KEY=sk-xxxxxxxx
# or
ANTHROPIC_API_KEY=sk-ant-xxxxxxxx
```

You can also run `hermes model` inside the launcher to configure providers interactively.

---

## Supported Platforms

| Platform | Architecture | Status |
|----------|-------------|--------|
| Windows 10/11 | x86_64 | ✅ Supported |
| macOS 13+ | Apple Silicon (ARM64) | ✅ Supported |
| macOS 13+ | Intel (x86_64) | ✅ Supported |
| Linux (Ubuntu/Fedora/Arch/Debian) | x86_64 | ✅ Supported |
| Linux | ARM64 | ✅ Supported |

---

## How It Works

### The Problem
Hermes Agent normally installs to `~/.hermes/` and `~/.local/bin/`, writes to the host home directory, and requires system-level Python/Node.js. This makes it impossible to run from a USB drive on a random computer.

### The Solution
1. **Custom Data Directory**: Every launcher sets `HERMES_HOME` to the local `data/` folder. Hermes reads/writes all config, sessions, and memory there.
2. **Portable Runtimes**: Instead of using the host's Python/Node.js, we download self-contained binaries into `.cache/runtimes/<platform>/`.
3. **Isolated Virtual Environment**: Each platform gets its own Python venv with Hermes and all dependencies installed.
4. **Zero Host Pollution**: No registry entries, no shell profile changes, no files outside this folder.

---

## Updating Hermes

To update to the latest Hermes Agent version:

```bash
# Inside a running Hermes session:
/hermes update

# Or manually:
# 1. Delete .cache/runtimes/<your-platform> and src/hermes-agent
# 2. Re-run the launcher — it will re-download the latest source and rebuild the venv.
```

---

## Troubleshooting

### "Setup failed" on first run
- Check your internet connection.
- Some corporate/school networks block GitHub or Node.js CDN. Try a different network or VPN.
- Delete `.cache/` and try again.

### macOS says "cannot be opened because the developer cannot be verified"
- Right-click `launch.sh` → **Open With** → **Terminal**.
- Or run `xattr -dr com.apple.quarantine .` in this folder via Terminal.

### Windows Defender flags the launcher
- This is a false positive due to PowerShell downloading files. Click **More info** → **Run anyway**.
- The scripts are open-source — inspect `scripts/setup-windows.ps1` if you have concerns.

### Slow performance from USB
- USB 2.0 flash drives are very slow for running Python applications. Use a **USB 3.0/3.1 flash drive** or an **external SSD** for best performance.

### Playwright / browser tools don't work
- Browser automation tools may refuse to run from removable drives due to OS sandboxing.
- If you need web browsing, copy the folder to the local SSD and run from there.

---

## Security Warning

> **Your USB drive = your identity.**

Since `data/.env` contains your API keys and `data/sessions/` contains your full chat history, **losing the USB drive means losing your privacy**. Consider:
- Encrypting the USB drive with BitLocker (Windows) or VeraCrypt (cross-platform).
- Not storing ultra-sensitive keys on a portable drive.

---

## Size Estimate

| Component | Size |
|-----------|------|
| Launchers + scripts | ~50 KB |
| Per-platform runtime (after first run) | ~600–900 MB |
| Hermes source code | ~50 MB |
| Your data (grows over time) | ~10 MB → several GB |

If you use Hermes on **3 different platforms**, the `.cache/` folder will contain ~1.8 GB total.

---

## License

This portable wrapper is provided as-is for convenience. Hermes Agent itself is licensed under the **MIT License** by Nous Research.

---

## Credits

- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Nous Research
- [python-build-standalone](https://github.com/indygreg/python-build-standalone) — indygreg
- [uv](https://github.com/astral-sh/uv) — Astral
