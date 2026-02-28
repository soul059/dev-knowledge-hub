# Chapter 4: Background & Foreground Jobs

[← Previous: Process Control](03-process-control.md) | [Back to README](../README.md) | [Next: Systemd Service Management →](05-systemd-service-management.md)

---

## Overview

Job control allows you to run commands in the background, bring them to the foreground, suspend them, and resume them. For DevOps, this is vital when running long tasks (builds, deployments, data migrations) over SSH sessions that might disconnect.

---

## 4.1 Foreground vs Background

```
┌───────────────────────────────────────────────────────────────┐
│                  JOB CONTROL MODEL                             │
│                                                                 │
│  Terminal (TTY)                                                 │
│  ┌───────────────────────────────────────────────────────┐     │
│  │                                                       │     │
│  │  Foreground Job (1 at a time)                         │     │
│  │  ┌─────────────────────────────────────────────┐     │     │
│  │  │  ./deploy.sh                                │     │     │
│  │  │  (receives keyboard input, Ctrl+C, Ctrl+Z)  │     │     │
│  │  └─────────────────────────────────────────────┘     │     │
│  │                                                       │     │
│  │  Background Jobs (multiple allowed)                   │     │
│  │  ┌───────────────┐ ┌───────────────┐                 │     │
│  │  │ [1] backup.sh │ │ [2] build.sh  │ ...             │     │
│  │  │   (running)   │ │  (stopped)    │                 │     │
│  │  └───────────────┘ └───────────────┘                 │     │
│  │                                                       │     │
│  └───────────────────────────────────────────────────────┘     │
│                                                                 │
│  Key: Foreground → gets terminal I/O                           │
│       Background → runs silently, no terminal input            │
└───────────────────────────────────────────────────────────────┘
```

```bash
# Run in foreground (default)
./long_task.sh

# Run in background (append &)
./long_task.sh &
# [1] 12345          ← job number [1], PID 12345
```

---

## 4.2 Job Control Commands

```bash
# Suspend foreground job → Ctrl+Z
./deploy.sh
# ^Z
# [1]+  Stopped   ./deploy.sh

# List jobs
jobs                     # Current shell's jobs
jobs -l                  # Include PIDs
# [1]+  Stopped   ./deploy.sh
# [2]-  Running   ./backup.sh &

# Resume in background
bg                       # Resume most recent stopped job
bg %1                    # Resume job [1] in background

# Bring to foreground
fg                       # Most recent job
fg %1                    # Specific job
fg %2

# Job specifiers
%1                       # Job number 1
%+                       # Current job (most recent)
%-                       # Previous job
%string                  # Job starting with "string"
%?string                 # Job containing "string"
```

```
┌────────────────────────────────────────────────────────┐
│              JOB STATE TRANSITIONS                      │
│                                                          │
│  Start              ┌──────────────┐                    │
│   │                 │   RUNNING    │                    │
│   ├── cmd &  ──────→│ (background) │←── bg %N          │
│   │                 └──────┬───────┘                    │
│   │                   Ctrl+Z │  ↑ fg %N                │
│   │                        ↓  │                         │
│   │                 ┌──────────────┐                    │
│   ├── cmd    ──────→│   RUNNING    │                    │
│   │                 │ (foreground) │                    │
│   │                 └──────┬───────┘                    │
│   │                   Ctrl+Z│                           │
│   │                        ↓                            │
│   │                 ┌──────────────┐                    │
│   │                 │   STOPPED    │                    │
│   │                 └──────────────┘                    │
│   │                   bg → background                   │
│   │                   fg → foreground                   │
└────────────────────────────────────────────────────────┘
```

---

## 4.3 `nohup` — Survive SSH Disconnection

When you close an SSH session, SIGHUP is sent to all child processes. `nohup` ignores SIGHUP so the process continues running.

```bash
# Basic nohup
nohup ./long_build.sh &
# Output goes to nohup.out by default

# Redirect output
nohup ./long_build.sh > build.log 2>&1 &

# Disown — detach already-running job
./deploy.sh &
disown %1                # Remove from job table
disown -a                # Disown all background jobs
disown -h %1             # Keep in table but ignore SIGHUP
```

```
┌──────────────────────────────────────────────────────────┐
│  SSH Disconnect Survival Methods                          │
│                                                            │
│  Method 1: nohup                                          │
│  $ nohup ./script.sh > output.log 2>&1 &                 │
│  └─ Simple, always available                              │
│                                                            │
│  Method 2: disown                                         │
│  $ ./script.sh &                                          │
│  $ disown %1                                              │
│  └─ For already-running jobs                              │
│                                                            │
│  Method 3: tmux / screen (RECOMMENDED)                    │
│  $ tmux new -s deploy                                     │
│  $ ./script.sh                                            │
│  (Ctrl+B, D to detach)                                    │
│  $ tmux attach -t deploy                                  │
│  └─ Full session persistence, reattachable                │
│                                                            │
│  Method 4: systemd service                                │
│  └─ For long-running daemons (see next chapter)           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.4 `tmux` — Terminal Multiplexer

```bash
# Install
sudo apt install tmux       # Ubuntu
sudo dnf install tmux       # RHEL

# Sessions
tmux                         # New session
tmux new -s deploy           # Named session
tmux ls                      # List sessions
tmux attach -t deploy        # Reattach
tmux kill-session -t deploy  # Kill session

# Key bindings (Ctrl+B is the prefix)
# Ctrl+B, D    → Detach
# Ctrl+B, C    → New window
# Ctrl+B, N    → Next window
# Ctrl+B, P    → Previous window
# Ctrl+B, %    → Split vertically
# Ctrl+B, "    → Split horizontally
# Ctrl+B, arrow → Switch pane
```

---

## 4.5 Real-World Scenario: Long-Running Deployment Over SSH

```bash
# Scenario: You need to run a 2-hour database migration over SSH

# Option A: tmux (Best)
ssh production-server
tmux new -s db-migration
./migrate_database.sh 2>&1 | tee migration.log
# Ctrl+B, D   ← safely detach

# Check back later
ssh production-server
tmux attach -t db-migration

# Option B: nohup (Quick & Simple)
ssh production-server
nohup ./migrate_database.sh > migration.log 2>&1 &
echo $!                      # Note the PID
exit                          # Safe to disconnect

# Check later
ssh production-server
tail -f migration.log
ps -p 12345                   # Check if still running

# Option C: Script with logging
ssh production-server << 'EOF'
nohup bash -c '
    echo "Migration started at $(date)" >> /var/log/migration.log
    ./migrate_database.sh >> /var/log/migration.log 2>&1
    echo "Migration completed at $(date)" >> /var/log/migration.log
' &
echo "Migration launched as PID $!"
EOF
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Background job stops when reading input | Needs terminal input (SIGTTIN) | Use input redirection `< /dev/null` |
| Job dies when SSH disconnects | Not protected from SIGHUP | Use `nohup`, `disown`, or `tmux` |
| `nohup.out` fills disk | Output not managed | Redirect: `> logfile 2>&1` |
| `fg` says "no current job" | No stopped/background jobs | Check `jobs -l` |
| Can't Ctrl+C a background job | Not in foreground | `fg %N` first, then Ctrl+C; or `kill %N` |
| tmux session lost after reboot | tmux doesn't survive reboot | Use `tmux-resurrect` plugin or systemd |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `cmd &` | Run in background | `./build.sh &` |
| `Ctrl+Z` | Suspend foreground job | (keyboard shortcut) |
| `jobs` | List shell jobs | `jobs -l` |
| `fg %N` | Bring job to foreground | `fg %1` |
| `bg %N` | Resume job in background | `bg %1` |
| `nohup` | Survive hangup signal | `nohup ./cmd > log 2>&1 &` |
| `disown` | Detach running job | `disown %1` |
| `tmux` | Terminal multiplexer | `tmux new -s session_name` |

---

## Quick Revision Questions

1. **What happens to background jobs when you close an SSH session?** How do you prevent this?
2. **What is the difference between `nohup` and `disown`?**
3. **How do you move a running foreground process to the background without restarting it?**
4. **What does the `+` and `-` next to job numbers in `jobs` output mean?**
5. **Why is `tmux` preferred over `nohup` for long-running deployments?**
6. **A background job keeps stopping with message "Stopped (tty input)". Why?**

---

[← Previous: Process Control](03-process-control.md) | [Back to README](../README.md) | [Next: Systemd Service Management →](05-systemd-service-management.md)
