# 🎓 MS Teams Auto-Join Attendance Agent

![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)
![Playwright](https://img.shields.io/badge/powered_by-Playwright-2EAD33.svg)

A Python agent that automatically joins your MS Teams classes on schedule so your attendance is always marked. Built with **Playwright** browser automation.

You configure your weekly timetable once, and the agent handles everything: navigating to the right team/channel, waiting for the meeting to start, muting your mic, turning off your camera, joining, staying for the duration, and leaving.

---

## 🚀 How It Works

1. 🕒 **Scheduler fires** 3 minutes before class time
2. 🧭 **Navigates** to the correct Team > Channel in MS Teams web
3. 🔄 **Polls** for an active meeting (checks every 15 seconds, waits up to 15 minutes)
4. 🔇 **Joins** the meeting with mic muted and camera off
5. ⏳ **Stays** in the meeting for the class duration
6. 🚪 **Leaves** automatically when the class ends (or early if next class is starting)
7. 🔔 **Sends notifications** (desktop toast + optional Discord) on join, leave, failure, or cancellation

---

## ⚡ Quick Start

### Prerequisites

- **Python 3.10+**
- **Microsoft Edge** (pre-installed on Windows)
- **uv** (Python package manager) - [Install uv](https://docs.astral.sh/uv/getting-started/installation/)

### 1. Install uv (if you don't have it)

```bash
# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Linux / macOS
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Clone and Install

```bash
git clone <repo-url>
cd attendance_agent
uv sync
uv run playwright install msedge
```

*Note: `uv sync` creates a virtual environment and installs all dependencies with exact pinned versions.*

### 3. Configure Your Timetable

```bash
cp config/config.example.yaml config/config.yaml
```

Open `config/config.yaml` and update the email to your roll number (or account email):

```yaml
student:
  email: "your.email@example.com"
```

*The timetable structure is pre-configured in the template. Add or modify your classes accordingly. **Note: Your `config.yaml` is gitignored.** Your data stays private.*

### 4. Login (One-Time Setup)

```bash
uv run attendance-agent --login
```
This opens Edge and navigates to Teams. Log in manually. Your session is saved in `browser_data/` and persists across runs—you only need to do this once.

### 5. Start the Agent

```bash
uv run attendance-agent
```
This starts the scheduler. It runs in the foreground and auto-joins classes based on the times in your timetable. 

*Want to test it out? Use `--join-now` to immediately join a specific class:*
```bash
uv run attendance-agent --join-now "Data Structures (Mon)"
```

---

## 💻 CLI Commands Reference

| Command | Description |
|---------|-------------|
| `uv run attendance-agent --login` | One-time browser login (saves session) |
| `uv run attendance-agent --join-now "Class Name"` | Immediately join a specific class |
| `uv run attendance-agent --pause` | Open Playwright Inspector to discover/debug selectors |
| `uv run attendance-agent` | Start the scheduled auto-join agent |
| `uv run attendance-agent -v` | Run with verbose/debug logging |
| `uv run attendance-agent --config path/to/config.yaml` | Use a custom config file |

*(You can also use `python -m src.main` instead of `uv run attendance-agent`)*

---

## 🔔 Notifications

The agent sends desktop notifications for important events:

- ✅ **Joined class:** "Joined Class: Data Structures (staying ~120 min)"
- ❌ **Join failed:** "Join Failed: Data Structures (Could not click join buttons)"
- ⚠️ **Class cancelled:** "No Meeting Found: Data Structures. Class may be cancelled."
- 🚪 **Left meeting:** "Class Ended: Left meeting: Data Structures"
- 💥 **Browser crashed:** "Browser Crashed: Lost connection during: Data Structures"
- 🔐 **Session expired:** "Session Expired: Teams login expired. Run --login"

*(Set `notifications.discord_webhook_url` in config to also get alerts on your phone via Discord!)*

---

## ⚙️ Configuration Details (`config.yaml`)

| Setting | Default | Description |
|---------|---------|-------------|
| `join_early_minutes` | 3 | Join N minutes before class start time |
| `stay_extra_minutes` | 5 | Stay N minutes after class is supposed to end |
| `max_wait_for_meeting` | 15 | Wait up to N minutes for meeting to appear |
| `poll_interval_seconds` | 15 | Check for meeting every N seconds |
| `retry_join_attempts` | 3 | Retry joining if it fails |
| `browser.headless` | false | Run browser without visible window |
| `browser.channel` | msedge | Browser to use (msedge, chromium, chrome) |
| `notifications.desktop_toast` | true | Show desktop notifications |
| `notifications.discord_webhook_url` | "" | Discord webhook for phone alerts |

---

## 🛠️ Auto-Start on Windows Login

Create a Windows Task Scheduler entry so the agent starts automatically when you log in:

```bash
python scripts/install_task.py          # Install scheduled task
python scripts/install_task.py --remove # Remove it
```

---

## 📁 Project Structure

```text
attendance_agent/
├── config/
│   ├── config.yaml              # Your timetable (gitignored)
│   └── config.example.yaml      # Template for new users
├── src/
│   ├── main.py                  # CLI entry point
│   ├── config_loader.py         # YAML parsing + validation
│   ├── scheduler.py             # Main-thread timetable scheduler
│   ├── browser.py               # Playwright persistent browser context
│   ├── teams_navigator.py       # Teams UI interactions
│   ├── meeting_joiner.py        # Full join lifecycle orchestrator
│   ├── selectors.py             # All CSS/aria selectors (update when Teams UI changes)
│   ├── notifier.py              # Desktop + Discord notifications
│   └── utils.py                 # Retry decorators, logging setup
├── scripts/
│   └── install_task.py          # Windows Task Scheduler auto-start
├── browser_data/                # Saved browser session (gitignored)
├── logs/                        # Rotating log files (gitignored)
├── pyproject.toml               # Project config + dependencies
├── uv.lock                      # Pinned dependency versions
└── .gitignore
```

---

## ❓ Troubleshooting

- **"Not logged in" error:** Your session expired. Run `uv run attendance-agent --login` again.
- **Meeting not detected:** The agent polls for up to 15 minutes. If the meeting starts very late, increase `max_wait_for_meeting` in your config.
- **Selectors not working (can't click buttons):** Teams UI updates can break selectors. Run `uv run attendance-agent --pause` to open Playwright Inspector, find the new selectors, and update `src/selectors.py`.
- **No audio in meeting:** Make sure `browser.channel` is set to `msedge` in your config. Edge has the best Teams compatibility.
- **Mic not muted:** The agent tries both UI button clicks and keyboard shortcuts (Ctrl+Shift+M). If your Teams uses a different layout, update the selectors in `src/selectors.py`.

---

## 📜 License

[MIT](LICENSE)
