# Chapter 3: Process Control

[← Previous: Viewing Processes](02-viewing-processes.md) | [Back to README](../README.md) | [Next: Background & Foreground Jobs →](04-background-foreground-jobs.md)

---

## Overview

Process control lets you stop, resume, terminate, and adjust the priority of running processes. Understanding Linux signals, `kill`, `pkill`, `nice`, and `renice` is essential for managing applications and troubleshooting unresponsive services in production.

---

## 3.1 Linux Signals

Signals are software interrupts sent to processes to trigger specific actions.

```
┌──────────────────────────────────────────────────────────────────┐
│                     COMMON LINUX SIGNALS                         │
├──────────┬────────┬──────────────┬───────────────────────────────┤
│ Signal   │ Number │ Default      │ Purpose                       │
├──────────┼────────┼──────────────┼───────────────────────────────┤
│ SIGHUP   │   1    │ Terminate    │ Hangup / reload config        │
│ SIGINT   │   2    │ Terminate    │ Interrupt (Ctrl+C)            │
│ SIGQUIT  │   3    │ Core dump    │ Quit (Ctrl+\)                 │
│ SIGKILL  │   9    │ Terminate    │ Force kill (cannot be caught) │
│ SIGTERM  │  15    │ Terminate    │ Graceful termination (default)│
│ SIGSTOP  │  19    │ Stop         │ Pause (cannot be caught)      │
│ SIGTSTP  │  20    │ Stop         │ Terminal stop (Ctrl+Z)        │
│ SIGCONT  │  18    │ Continue     │ Resume stopped process        │
│ SIGUSR1  │  10    │ Terminate    │ User-defined signal 1         │
│ SIGUSR2  │  12    │ Terminate    │ User-defined signal 2         │
└──────────┴────────┴──────────────┴───────────────────────────────┘

  Signal Delivery Flow:
  ┌──────────┐  signal   ┌───────────────────┐
  │  Sender  │ ───────── │   Target Process   │
  │ (kill)   │           │                     │
  └──────────┘           │ Signal Handler?     │
                         │  ├─ Yes → Run handler│
                         │  └─ No  → Default    │
                         └───────────────────────┘

  Note: SIGKILL (9) and SIGSTOP (19) CANNOT be caught or ignored!
```

```bash
# List all signals
kill -l

# Signal numbers and names
kill -l SIGTERM    # 15
kill -l 15         # TERM
```

---

## 3.2 `kill` — Send Signals to Processes

```bash
# Syntax: kill [-signal] PID

# Graceful termination (default SIGTERM = 15)
kill 1234
kill -15 1234
kill -SIGTERM 1234
kill -TERM 1234              # All equivalent

# Force kill (SIGKILL = 9)
kill -9 1234
kill -SIGKILL 1234

# Reload configuration (SIGHUP = 1)
kill -HUP $(pidof nginx)    # Nginx reloads config
kill -1 $(pidof sshd)       # SSHD re-reads config

# Pause and resume
kill -STOP 1234              # Pause process
kill -CONT 1234              # Resume process

# Kill multiple processes
kill 1234 5678 9012
```

### Correct Termination Order

```
┌─────────────────────────────────────────────────────────────┐
│          PROPER PROCESS TERMINATION SEQUENCE                 │
│                                                               │
│  Step 1:  kill PID              (SIGTERM — ask nicely)       │
│           ↓                                                   │
│           Wait 5-10 seconds                                  │
│           ↓                                                   │
│  Step 2:  kill -9 PID           (SIGKILL — force kill)       │
│                                                               │
│  WHY: SIGTERM lets the process:                               │
│  • Close open files                                          │
│  • Release locks                                              │
│  • Flush buffers                                              │
│  • Clean up child processes                                   │
│  • Write final log entries                                    │
│                                                               │
│  SIGKILL (9) skips ALL of this → data corruption risk!       │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.3 `pkill` and `killall` — Kill by Name

```bash
# pkill — kill by pattern match
pkill nginx              # SIGTERM to all nginx processes
pkill -9 java            # SIGKILL to all java processes
pkill -u devops          # Kill all processes of user devops
pkill -f "python app.py" # Match full command line
pkill -t pts/0           # Kill all on terminal pts/0
pkill -P 1234            # Kill children of PID 1234

# killall — kill by exact name
killall nginx            # SIGTERM to all nginx processes
killall -9 python3       # Force kill all python3
killall -u devops        # Kill all by user
killall -i nginx         # Interactive (confirm each)

# Differences
# pkill → matches PATTERN (partial match)
# killall → matches EXACT process name
```

---

## 3.4 `nice` and `renice` — Process Priority

```
┌──────────────────────────────────────────────────────────────┐
│               LINUX PROCESS PRIORITY                          │
│                                                                │
│  Nice value: -20 ────────── 0 ────────── +19                 │
│              Highest        Default       Lowest              │
│              Priority                     Priority            │
│                                                                │
│  Priority (PR) = 20 + nice                                    │
│  PR 0 (nice -20) = runs first                                │
│  PR 39 (nice +19) = runs last                                │
│                                                                │
│  WHO CAN SET WHAT:                                            │
│  ┌──────────────┬──────────────────────────────────┐         │
│  │ User         │ 0 to +19 (lower priority only)   │         │
│  │ Root         │ -20 to +19 (any priority)        │         │
│  └──────────────┴──────────────────────────────────┘         │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Start a process with a nice value
nice -n 10 ./heavy_report.sh          # Lower priority
nice -n -5 ./critical_backup.sh       # Higher priority (root only)
nice ./script.sh                       # Default nice = 10

# Check current nice values
ps -eo pid,ni,comm | grep nginx

# Change priority of running process
renice 10 -p 1234                     # Set nice to 10
renice -5 -p 1234                     # Set nice to -5 (root)
renice 15 -u devops                   # All processes of user
renice 0 -g developers                # All processes of group

# Verify the change
ps -p 1234 -o pid,ni,pri,comm
```

---

## 3.5 Real-World Scenario: Graceful Application Restart

```bash
#!/bin/bash
# graceful-restart.sh — Restart an application gracefully

APP_NAME="myapp"
PIDFILE="/var/run/${APP_NAME}.pid"

echo "=== Graceful Restart of ${APP_NAME} ==="

# Step 1: Get current PID
if [ ! -f "$PIDFILE" ]; then
    echo "PID file not found. Searching for process..."
    APP_PID=$(pgrep -f "$APP_NAME")
else
    APP_PID=$(cat "$PIDFILE")
fi

if [ -z "$APP_PID" ]; then
    echo "Process not running. Starting fresh..."
    /opt/${APP_NAME}/start.sh
    exit 0
fi

echo "Found ${APP_NAME} running as PID: ${APP_PID}"

# Step 2: Send SIGTERM
echo "Sending SIGTERM..."
kill -TERM "$APP_PID"

# Step 3: Wait for graceful shutdown (max 30 seconds)
TIMEOUT=30
while kill -0 "$APP_PID" 2>/dev/null && [ $TIMEOUT -gt 0 ]; do
    echo "Waiting for shutdown... (${TIMEOUT}s remaining)"
    sleep 1
    TIMEOUT=$((TIMEOUT - 1))
done

# Step 4: Force kill if still running
if kill -0 "$APP_PID" 2>/dev/null; then
    echo "WARNING: Process did not stop. Sending SIGKILL..."
    kill -9 "$APP_PID"
    sleep 2
fi

# Step 5: Verify termination
if kill -0 "$APP_PID" 2>/dev/null; then
    echo "ERROR: Failed to stop ${APP_NAME}"
    exit 1
fi

echo "Process stopped. Starting ${APP_NAME}..."
/opt/${APP_NAME}/start.sh
echo "Restart complete."
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `kill: No such process` | Process already exited | PID is stale; check with `ps` |
| `kill: Operation not permitted` | Not the owner/root | Use `sudo kill PID` |
| Process ignores `kill` | Caught SIGTERM | Use `kill -9` (SIGKILL) |
| `kill -9` doesn't work | Zombie or D-state process | Zombie: kill parent; D-state: check I/O / reboot |
| `pkill` kills wrong processes | Pattern too broad | Use `pgrep -a PATTERN` first to verify |
| `renice: Permission denied` | Non-root lowering nice | Only root can set negative nice values |
| Process respawns after kill | Supervised by systemd/init | Use `systemctl stop service` instead |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `kill PID` | Send SIGTERM (graceful stop) | `kill 1234` |
| `kill -9 PID` | Force terminate | `kill -9 1234` |
| `kill -HUP PID` | Reload configuration | `kill -HUP $(pidof nginx)` |
| `pkill pattern` | Kill by name pattern | `pkill -f "python app"` |
| `killall name` | Kill by exact name | `killall nginx` |
| `nice -n N cmd` | Start with priority | `nice -n 10 ./backup.sh` |
| `renice N -p PID` | Change running priority | `renice 5 -p 1234` |

---

## Quick Revision Questions

1. **What is the difference between SIGTERM and SIGKILL?** Why should you try SIGTERM first?
2. **Which two signals cannot be caught or ignored by a process?**
3. **How do you reload Nginx configuration without stopping it?**
4. **What is the range of nice values? What is the default?**
5. **What is the difference between `pkill` and `killall`?**
6. **A process keeps respawning after you kill it. What's likely happening and how do you properly stop it?**

---

[← Previous: Viewing Processes](02-viewing-processes.md) | [Back to README](../README.md) | [Next: Background & Foreground Jobs →](04-background-foreground-jobs.md)
