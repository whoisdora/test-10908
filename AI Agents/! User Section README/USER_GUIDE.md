# User Guide

A project folder where AI agents keep memory in files, prove their work with tests, and cannot commit anything without you.

## Folder map

```
project folder\
  00_INDEX.md              front page: what this project is
  AGENTS.md                rules the agent reads first (and the verification commands)
  NOTICE_TO_OTHER_AGENTS.md  terms for any outside agent that reads the files
  opencode.json            which model to use + the permission rules
  core\state.md            where the project is now (rewritten every session, under 60 lines)
  core\decisions.md        log of decisions (only add lines, never rewrite)
  templates\task.md        the format for a task
  tasks\                   one file per task (TASK-001.md, TASK-002.md ...)
  tools\                   scripts (smoke test, later the loop script)
  notes\  sessions\        messy notes and handoffs. Agents skip them unless asked.
  inbox\                   requests from other agents. You read it, the model does not.
```

## Daily routine (manual mode)

1. Open the project folder in File Explorer, click the address bar, type `cmd`, press Enter.
2. Run `git status`. It should say the tree is clean.
3. Run `opencode`.
4. Give the agent a task file and ask for a plan only: "Read AGENTS.md, core/state.md and tasks/TASK-001.md. Write a short plan. Do not edit any file until I approve."
5. Read the plan. If it stays inside the files in scope, say "Plan approved. Implement it."
6. Approve its test runs. Reject anything involving git.
7. In your own terminal: run the tests, then `git status`. Only the expected files should have changed.
8. Commit it yourself: `git add .` then `git commit -m "what changed"`.
9. Ask the agent to update `core/state.md` and `core/decisions.md`. Commit that too.

## Rules of the road

- The agent never commits or pushes. You do.
- Check `git status` after every run. Anything unexpected means it drifted.
- No passwords, keys or tokens in this folder, ever.
- Ollama stays local-only (`127.0.0.1`). Never set it to `0.0.0.0` or forward its port.
- Tasks need checkable "done" criteria. "Make it nice" is not a task.
- Every time the agent repeats a mistake, add one line under GOTCHAS in `AGENTS.md`.

## Cheat sheet (Windows, use Command Prompt)

| Do this | Command |
|---|---|
| Go to the folder (other drive, spaces) | `cd /d "A:\path\to\project"` |
| See what changed | `git status` and `git diff` |
| See history | `git log --oneline` |
| Commit (you only) | `git add .` then `git commit -m "message"` |
| Throw away uncommitted changes | `git restore .` (preview new files first with `git clean -n`) |
| Is the model on GPU, and what context? | `ollama ps` |
| Installed models | `ollama list` |
| Run the tests | `python -m unittest` |
| Check tool calling still works | `python smoke_test.py` |
| Start the agent | `opencode` |
| One-shot, no interface | `opencode run "prompt"` |
| Show a file | `type opencode.json` |
| Show the folder tree | `tree /f /a` |

## When something breaks

| Symptom | Fix |
|---|---|
| `git` not recognized | Install git, then open a new terminal |
| `npm.ps1 cannot be loaded` | Use Command Prompt, or `npm.cmd` in PowerShell |
| `cd` does not change drive | `cd /d "path"`, with quotes around paths that have spaces |
| Command fails with a stray `'` at the end | The quote mark was copied with the command |
| Config not applied | File may be saved as `.json.txt`. Turn on file extensions in Explorer |
| Agent prints a tool call as text, or does nothing | Run `python smoke_test.py`. Check Ollama version and that `ollama ps` shows 16k context |
| Very slow | `ollama ps`: if the processor column says CPU, the model is not on the GPU |
| Agent cannot find files | Wrong folder, or files are not at the project root. Run `tree /f /a` |
| Agent did extra things | `git status`, then `git restore .` to undo uncommitted changes |

## The three numbers to track in `core/state.md`

- First-try acceptance: tasks accepted without rework.
- Resume time: how long a fresh session takes to be useful.
- Corrections per task: how often you had to redirect.

If these are not improving after a couple of weeks, the setup is not paying off. Change it.
