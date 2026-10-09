# Roadmap

Last updated: 2026-10-07. Estimates are rough guesses at 1 to 2 hours a day.

## End goal

A reusable workspace where you say "build this", a local model works through a queue of tested tasks unattended, and you review the result in the morning. Every project uses the same structure: memory files, verification, a sandbox, and no commit power for the worker.

## What the end result looks like

- One command creates a new project folder from the kit.
- You give a goal. A stronger model plans it into a spec and a queue of small tasks, each with tests. You approve the plan once.
- One launcher starts the overnight loop. Qwen does one task per fresh session. The loop runs the tests itself, commits only on an agent branch, and stops on a STOP file, a time limit, or repeated failures.
- In the morning you read a short summary, review the branch diff, merge what is good, and hand escalated tasks to the stronger model.
- Documentation (for example Roblox and Lua) is fetched ahead of time into local notes by a separate tool. The worker reads local copies only.
- The worker cannot commit, push, edit its own rules, or read outside the project.
- A fresh session or a new model can resume from the files in minutes.
- Three numbers show whether it is working: first-try acceptance, resume time, corrections per task.

## Order of work

M0 > M1 > M2 > M3 and M4 together > M5 > M6 > M7 > M8 > M9 > M10

|Milestone|What|Estimate|Who|
|---|---|---|---|
|M0|Foundation|done|you|
|M1|Stable baseline|about 1 day|you|
|M2|Facts the design depends on|1 to 2 days|you with Fable|
|M3|Runner|2 to 4 days|Fable builds, you test|
|M4|Sandbox|2 to 4 days|alongside M3|
|M5|First unattended run|1 to 2 days|you|
|M6|Planner and a real project|about 1 week|Fable and you|
|M7|Docs and review|3 to 5 days|Fable|
|M8|Reusable kit|2 to 3 days|Fable|
|M9|Scale up|ongoing|all|
|M10|Three machines|after M6|all|

## M0: Foundation (done)

- [x] Project folder, memory files, git
- [x] Ollama with qwen3.5:9b at 16k context
- [x] Smoke test: tool calling 5 of 5
- [x] OpenCode connected, commit and push blocked in interactive mode
- [x] First task started (vault_check.py)

## M1: Stable baseline (about 1 day, you)

- [x] Review the overnight work: tests, `vault_check.py`, the diff of AGENTS.md and core/state.md
- [x] Merge or discard it, then tag `baseline-before-fable`
- [x] Add `__pycache__/` to .gitignore, clean stray files (in.txt, README folder)
- [x] Record in core/decisions.md: the worker never edits state.md, the loop does not either, you copy the numbers

**Exit:** clean `git status`, tests pass, tag exists.

## M2: Facts the design depends on (1 to 2 days, you with Fable)

- [x] GPU memory is 8GB (the 16GB is system RAM)
- [x] Check the model fit with `ollama ps` and `nvidia-smi`, and record tokens per second from the smoke test
- [x] If it is split between CPU and GPU, lower the context or free GPU memory
- [x] Confirm drive A: is NTFS
- [x] Record paths: `where python`, `where opencode`, `where node`
- [x] Run the deny-rule evidence procedure: run-mode denies, ask behavior with no human, auto-approve behavior
- [x] Fable Gate 1: sandbox option approved. Gate 2: final permission block approved

**Exit:** evidence pasted and accepted. If denies do not hold in run mode, stop and redesign here.

## M3: Runner (2 to 4 days, Fable builds, you test)

- [x] `tools/run_queue.py`, tests, launcher, dry-run mode
- [x] Task files carry status and attempts

**Exit:** `python -m unittest` passes. Toy dry run: 2 tasks done, 1 escalated, STOP works, a hung task is killed at the timeout (45 minutes by default), master unchanged.

## M4: Sandbox (2 to 4 days, can run alongside M3)

- [x] Worker launch mechanism chosen and working
- [x] Control files cannot be edited by the worker (.git, opencode.json, AGENTS.md, tools, core, templates)
- [x] Worker cannot read a canary file outside the project
- [x] Windows sleep disabled for overnight runs

**Exit:** each test shown with output.

## M5: First unattended run (1 to 2 days)

- [x] 3-task toy project run overnight with real Qwen
- [x] Morning review and metrics recorded

**Exit:** at least 2 of 3 pass, no out-of-scope changes, master untouched, log and summary written.

## M6: Planner and a real project (about 1 week)

- [ ] Automatic planner: the runner calls a strong model through its API with a planner prompt, and gets back `core/spec.md`, `core/interfaces.md` and task files
- [ ] The API key lives only in the runner environment on one coordinator machine, with a spending cap and usage log
- [ ] Plan lint (plain code): every task has files in scope and verify commands, no overlapping files, no dependency cycles, size limits
- [ ] Red check: each task's verify commands must fail on the untouched tree before the worker starts, which proves the tests mean something
- [ ] You approve the short spec once. Task breakdowns that pass lint run without asking
- [ ] Pick one real small project. Suggested: a Luau logic module with tests, or a small web page with a build check
- [ ] Run 10 or more tasks. Track acceptance and corrections
- [ ] Add every repeated mistake to GOTCHAS in AGENTS.md

**Exit:** a working result you tested yourself, and three weeks of metrics trending the right way.

## M7: Docs and review (3 to 5 days)

- [ ] `tools/fetch_docs.py`: domain allowlist, size caps, saves markdown to `notes/docs/<topic>/`
- [ ] Fetch Roblox and Luau documentation
- [ ] Optional reviewer role on a trial. Keep it only if it flags things the tests missed

**Exit:** docs available locally, reviewer flag statistics recorded.

## M8: Reusable kit (2 to 3 days)

- [ ] `tools/new_project.cmd` creates a project folder from the template, with the proven permission block, no credentials
- [ ] Generalize AGENTS.md, USER_GUIDE and the roadmap template
- [ ] Optional: propose mode (the worker suggests next tasks, you approve)

**Exit:** a new project is created and its first task runs, from scratch, in under 10 minutes.

## M9: Scale up (ongoing)

- [ ] Larger projects, for example a Roblox game or a trading bot
- [ ] Escalation to the stronger model becomes routine
- [ ] Revisit the sandbox (container) if you want literal isolation

## M10: Three machines (after M6, once one worker is reliable)

- [ ] Task files gain `owner:` and `depends_on:` lines. The planner splits work into independent waves with no overlapping files
- [ ] Runner flags: machine name, owner filter, model and context per machine. One branch per machine: `agent/<machine>/<date>`
- [ ] Private git remote. The runner, not the worker, pushes only its own branch. Master stays yours. Needs your approval first
- [ ] `tools/integrate.py` merges machine branches in dependency order onto an integration branch, runs the full tests, reports conflicts, never touches master
- [ ] Escalation ladder: first attempt on an 8GB machine, retry on the 16GB machine at higher precision and longer context, then the stronger model
- [ ] Reviewer runs on the 16GB machine, reading diffs from branches
- [ ] Message bus in git (`messages/` folder): typed messages (question, blocked, handoff, result). The runner delivers them into task prompts as quoted data
- [ ] Workers can ask the planner questions. The runner calls the planner model and writes the answer back
- [ ] Replan between waves: the planner reads results and failures, then writes the next wave. A task that fails twice is split into smaller tasks before it is escalated to you
- [ ] Caps: tasks created per replan, total tasks, and spend. The planner cannot change the spec without your approval

**Exit:** three machines finish one wave of independent tasks overnight, the integration branch passes its tests, and your review time per task stays flat or drops.

## Guardrails

- **Trading bot:** backtest and paper trading only. No exchange keys anywhere the worker can reach. Any real-money step is run by you.
- **Roblox:** the worker cannot run Studio. Test pure Luau logic automatically, play-test yourself.
- **Ollama** stays on 127.0.0.1. No ports opened.
- **One worker per machine.** 8GB of GPU memory leaves no room for parallel model sessions on the same machine.

## Rules for the whole project

- After M5, always work on a real deliverable. The system is not the project.
- Each milestone ends with evidence, not a feeling.
- If the metrics do not improve after a couple of real projects, change the setup.