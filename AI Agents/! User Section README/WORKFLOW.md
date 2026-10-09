# Workflow Design

Two modes. Manual mode works today. Queue mode is the long-running version and still has to be built.

## Roles

| Role | Who | Job |
|---|---|---|
| Planner | Strongest model available (Fable, Claude, other). Used once per project or phase. | Turns "make this" into a spec and a queue of small tasks, each with tests. |
| Worker | Qwen 3.5 9B, local, through OpenCode. | Does one task per fresh session. |
| Loop script | Plain code, no model. | Starts sessions, runs the tests itself, commits on a branch only when they pass, enforces stop rules. |
| You | Human. | Approve the plan, review the branch, merge, handle escalations. |

Why a loop script and not a long prompt: the worker has a 16k context. It cannot hold hours of work in one session, so the work is cut into tasks, and each task starts from the memory files.

## Manual mode (today)

```
you write task -> agent plans -> you approve -> agent works -> agent runs tests
-> you verify tests + git status -> you commit -> agent updates state.md
```

## Queue mode (to build)

```
PLAN (once, human checkpoint)
  "make this"  ->  Planner  ->  core/spec.md + tasks/TASK-001..N.md   (you review)

LOOP (unattended)
  while time budget left and no STOP file and failures in a row < limit:
    pick lowest-numbered task with status: todo
    make sure we are on branch agent/<date>
    run a fresh session:  opencode run  (fixed prompt, one task)
    loop script runs the task's verify commands itself
    loop script compares changed files to the task's "files in scope"
    all pass and nothing out of scope:
        loop script commits "TASK-NNN: name", sets status: done
    otherwise:
        restore the branch to the last commit, attempts + 1
        attempts = 2  ->  status: escalate, move on
    append one line to sessions/run-<date>.log

MORNING (human)
  read the log, review the diff of agent/<date>, merge or discard
  give escalated tasks to the stronger model
```

## Task file fields the loop script reads

```
status: todo | done | escalate
attempts: 0
files in scope: (list)
verify with: (commands that exit 0 on success)
```

The rest of the task file follows `templates/task.md`.

## Stop rules

- Time budget reached.
- A file named `STOP` exists in the project root (kill switch).
- 3 failed tasks in a row.
- Any change outside the files in scope (task fails and is reverted).
- Any denied command attempted repeatedly.

## Memory in queue mode

- Each session reads `AGENTS.md`, `core/state.md`, and its one task file. Nothing else.
- The loop script, not the worker, writes the run log. The worker only edits files in scope.
- `core/state.md` is updated by the loop script from the results (done count, escalations), so a weak model cannot corrupt it.

## Safety in queue mode

- The agent never gets commit or push. Only the loop script commits, only to `agent/<date>`, never to main.
- Permission denies for `git commit` and `git push` stay in `opencode.json`.
- Auto-approval skips prompts for everything not explicitly denied, so every shell command runs without asking. Therefore the worker must run in a sandbox (container or VM) with no credentials and access only to the project folder, before any unattended run.
- Ollama stays bound to `127.0.0.1`. It is not exposed to make the container work (see the open question).

## Open design question

If the worker runs in a container, it must reach Ollama, which listens only on the host's `127.0.0.1`. Options to weigh:
1. Run Ollama inside the same container or VM, with GPU passthrough.
2. Bind Ollama to an internal-only interface and firewall it to the container.
3. Skip the container and restrict the worker with a separate low-privilege Windows user and folder permissions.

Pick the option with the least exposure that actually works on this machine, and test that the worker cannot read files outside the project folder.

## Escalation

An escalated task goes to the stronger model with: `AGENTS.md`, the task file, the failing test output, and the files in scope. You review its fix like any other diff.

## Measuring

After each run, record in `core/state.md`: tasks attempted, done, escalated, and how many needed a retry. Compare with doing the same tasks manually.
