# Claude Code Instructions

## Chat Bookends (MANDATORY — EVERY PROMPT)
- **First output — coding plan**: for every user prompt that will involve changes, the very first line written to chat must be `🚩🚩CODING PLAN🚩🚩` on its own line, followed by a brief bullet-point list of what will be done in this response, then a **blank line** followed by `⚡⚡CODING START⚡⚡` on its own line to signal work is beginning. The blank line is required to break out of the bullet list context so CODING START renders left-aligned. Keep the plan concise — one bullet per distinct action (e.g. "Edit CLAUDE.md to add coding plan rule", "Update README.md timestamp"). This is for transparency, not approval — do NOT wait for user confirmation before proceeding. If the response is purely informational with no changes to make, skip the plan and open with `⚡⚡CODING START⚡⚡` directly. **CODING PLAN and CODING START appear exactly once per response** — never repeat them mid-response. Use `🔄🔄NEXT PHASE🔄🔄` instead (see below)
- **Continuation after user interaction**: when `AskUserQuestion` or `ExitPlanMode` returns mid-response (the user answered a question or approved a plan), the response continues but must **NOT** repeat `🚩🚩CODING PLAN🚩🚩` or `⚡⚡CODING START⚡⚡`. Instead:
  - After `AskUserQuestion`: use `🔄🔄NEXT PHASE🔄🔄` with a description incorporating the user's choice (e.g. "User chose option A — proceeding with implementation")
  - After `ExitPlanMode` (plan approved): output `📋📋PLAN APPROVED📋📋` on its own line, followed by `🚩🚩CODING PLAN🚩🚩` with the execution plan bullets, then `⚡⚡CODING START⚡⚡`. This is the **only** scenario where CODING PLAN/CODING START may appear a second time — because plan approval is a distinct boundary between planning and execution, and the user needs to see the execution plan clearly. The `📋📋PLAN APPROVED📋📋` marker signals that this is a continuation, not a new prompt
- **Checklist running**: output `✔️✔️CHECKLIST✔️✔️` on its own line before executing any mandatory checklist (Session Start, Pre-Commit, Pre-Push), followed by the checklist name (e.g. `Session Start Checklist`). This separates checklist overhead from the user's actual task. Output once per checklist invocation
- **Researching**: output `🔍🔍RESEARCHING🔍🔍` on its own line when entering a research/exploration phase — reading files, searching the codebase, or understanding context before making changes. Skip if going straight to changes without research
- **Mid-response phase marker**: when work within a single response naturally divides into multiple distinct sub-tasks or phases (e.g. "Edit 1" then "Edit 1a: fix related issue"), output `🔄🔄NEXT PHASE🔄🔄` on its own line followed by a brief description of the new phase. **Never repeat** `🚩🚩CODING PLAN🚩🚩` or `⚡⚡CODING START⚡⚡` within the same response — those appear exactly once (at the very top). The mid-response marker keeps the top/bottom boundaries of each prompt/response turn unambiguous while still signaling transitions between sub-tasks
- **Blocked**: output `🚧🚧BLOCKED🚧🚧` on its own line when an obstacle is hit (permission denied, merge conflict, ambiguous requirement, failed push, hook check failure). Follow with a brief description of the blocker. This makes problems immediately visible rather than buried in tool output
- **Verifying**: output `🧪🧪VERIFYING🧪🧪` on its own line when entering a verification phase — running git hook checks, confirming no stale references, validating edits post-change. Separates "doing the work" from "checking the work"
- **Time estimate**: output `⏳⏳ESTIMATED TIME ≈ Xm⏳⏳` on its own line, followed by a brief reason (e.g. "~8 file edits + commit + push cycle"), in **two contexts**: (1) **Overall** — **always** appears immediately before `⚡⚡CODING START⚡⚡`, estimating the entire response from CODING PLAN to CODING COMPLETE. This gives the user an upfront sense of total wall-clock time. The overall estimate is **never skipped** — even for quick responses (e.g. `⏳⏳ESTIMATED TIME ≈ 1m⏳⏳`). Use `Xm` for minutes or `Xs` for estimates under 1 minute. (2) **Per-phase** — immediately before any subsequent phase bookend (NEXT PHASE, RESEARCHING, CHECKLIST, etc.) whose phase alone is expected to take longer than 2 minutes. **Only output per-phase estimates when the estimate exceeds 2 minutes** — skip for phases that will be quick. Use these rough heuristics to estimate: ~10s per tool call (read, edit, grep, glob), ~15s per bash command, ~30s per commit cycle (checklist + staging + commit), ~30s per push cycle (checklist + push + verify), ~1–2m per subagent spawn. Sum the expected tool calls for the phase and round to the nearest minute. The estimate does not need a timestamp or `date` call — it is an annotation, not a bookend that participates in `⏱️` duration tracking
- **Revised estimate**: once all initial file reads, greps, and exploration needed to understand the scope of work are complete — regardless of whether a RESEARCHING bookend was used — re-evaluate the overall estimate. If it now differs from the original by ≥1 minute, output `⏳⏳REVISED ESTIMATED TIME ≈ Xm⏳⏳` on its own line with a brief reason (e.g. "~12 files to edit, not 4 as initially expected"). Place it inline right after the last read/exploration tool result and before the next action or bookend. **Skip if the original estimate still looks accurate.** The revised estimate replaces the original as the baseline for the ACTUAL TIME comparison at the end
- **Actual time**: output `⏳⏳ACTUAL TIME: Xm Ys (estimated Xm)⏳⏳` on its own line immediately before `✅✅CODING COMPLETE✅✅`. This is the real elapsed wall-clock time from CODING START to CODING COMPLETE — computed by subtracting the CODING START timestamp from the CODING COMPLETE timestamp. The parenthetical shows the original overall estimate for easy comparison. If a REVISED ESTIMATED TIME was issued, use the revised value instead. **Always present** when CODING COMPLETE is written (never skipped). The `date` call for CODING COMPLETE (already required) provides the end time — no additional `date` call is needed
- **Estimate calibration**: after computing ACTUAL TIME, if the actual and estimated times differ by **more than 2 minutes**, perform a calibration step **before** writing CODING COMPLETE. This makes estimation accuracy self-improving over time:
  1. **Diagnose the miss** — identify what caused the gap. Common causes: underestimating the number of file reads needed, not accounting for checklist overhead, forgetting the push cycle, unexpected retries or errors, or heuristic values being too low/high for the type of work done
  2. **Update the heuristics** — edit the Time estimate bullet's heuristic values in this file (`CLAUDE.md`) to better reflect reality. For example, if tool calls consistently take ~15s instead of ~10s, change `~10s per tool call` to `~15s per tool call`. If commit cycles consistently take ~45s, update `~30s per commit cycle` to `~45s per commit cycle`. Only change values where the evidence from this response supports the adjustment — do not guess or over-correct
  3. **Output the calibration** — in the end-of-response block, output `🔧🔧ESTIMATE CALIBRATED🔧🔧` after SUMMARY and before ACTUAL TIME, followed by a brief description of what was adjusted and why (e.g. "Increased tool call estimate from ~10s to ~15s — actual tool calls averaged 14s this response"). This makes the self-correction visible to the user
  4. **Commit the update** — include the heuristic change in the current commit (or make a follow-up commit if the main commit already happened). The updated heuristics take effect starting from the next response
  **Skip calibration entirely if the difference is ≤2 minutes** — small variances are normal and not worth correcting
- **Hook anticipation**: before writing `✅✅CODING COMPLETE✅✅`, check whether the stop hook (`~/.claude/stop-hook-git-check.sh`) will fire. **This check must happen after all actions in the current response are complete** (including any `git push`) — do not predict the pre-action state; check the actual post-action state. **Actually run** the three git commands (do not evaluate mentally): (a) uncommitted changes — `git diff --quiet && git diff --cached --quiet`, (b) untracked files — `git ls-files --others --exclude-standard`, (c) unpushed commits — `git rev-list origin/<branch>..HEAD --count`. If any condition is true, **omit** `✅✅CODING COMPLETE✅✅` and instead write `🐟🐟AWAITING HOOK🐟🐟` as the last line of the current response — the hook will fire, and `✅✅CODING COMPLETE✅✅` should close the hook feedback response instead. **Do not forget the `⏱️` duration annotation** — AWAITING HOOK is a bookend like any other, so the previous phase's `⏱️` must appear immediately before it. After the hook anticipation git commands complete, call `date`, compute the duration since the previous bookend's timestamp, write the `⏱️` line, then write AWAITING HOOK
- **Hook feedback override**: if the triggering message is hook feedback (starts with "Stop hook feedback:", "hook feedback:", or contains `<user-prompt-submit-hook>`), use `⚓⚓HOOK FEEDBACK⚓⚓` as the first line instead of `🚩🚩CODING PLAN🚩🚩` or `⚡⚡CODING START⚡⚡`. The coding plan (if applicable) follows immediately after `⚓⚓HOOK FEEDBACK⚓⚓`, then `⚡⚡CODING START⚡⚡`
- **End-of-response sections**: after all work is done, output the following sections in this exact order. Skip the entire block only if the response was purely informational with no changes made. **The entire block — from the `━━━` divider through CODING COMPLETE — must be written as one continuous text output with no tool calls in between.** To achieve this, run the `date` command for CODING COMPLETE's timestamp **before** starting the block, then output: the last phase's `⏱️` duration, a `━━━━━━━━━━━━━━━━━━━━━━━━━━━━` divider on its own line (Unicode heavy horizontal line — visually separating work phases from the end-of-response block), then AGENTS USED through CODING COMPLETE using the pre-fetched timestamp:
  - **Agents used**: output `🕵🕵AGENTS USED🕵🕵` followed by a **numbered list** of all agents that contributed to this response — including Agent 0 (Main). Format: `1. Agent N (Type) — brief description of contribution`. Number each agent sequentially starting from 1. This appears in every response that performed work. Skip only if the response was purely informational with no actions taken
  - **Files changed**: output `📁📁FILES CHANGED📁📁` followed by a list of every file modified in the response, each tagged with the type of change: `(edited)`, `(created)`, or `(deleted)`. This gives a clean at-a-glance file manifest. Skip if no files were changed in the response
  - **Commit log**: output `🔗🔗COMMIT LOG🔗🔗` followed by a list of every commit made in the response, formatted as `SHORT_SHA — commit message`. Skip if no commits were made in the response
  - **Worth noting**: output `🔖🔖WORTH NOTING🔖🔖` followed by a list of anything that deserves attention but isn't a blocker (e.g. "Push-once already used — did not push again", "Template repo guard skipped version bumps", "Pre-commit hook modified files — re-staged"). Skip if there are nothing worth noting
  - **Summary of changes**: output `📝📝SUMMARY📝📝` on its own line followed by a concise bullet-point summary of all changes applied in the current response. Each bullet must indicate which file(s) were edited (e.g. "Updated build-version in `live-site-pages/index.html`"). If a bullet describes a non-file action (e.g. "Pushed to remote"), no file path is needed
  - **Estimate calibration** (conditional): if ACTUAL TIME differs from the estimate by >2 minutes, output `🔧🔧ESTIMATE CALIBRATED🔧🔧` followed by what was adjusted. This is the **one exception** to the "no tool calls in the end-of-response block" rule — the calibration edits CLAUDE.md's heuristic values via an Edit tool call between SUMMARY and ACTUAL TIME. See the Estimate calibration bullet above for the full procedure
- **Last output**: for every user prompt, the very last line written to chat after all work is done must be exactly: `✅✅CODING COMPLETE✅✅`
- These apply to **every single user message**, not just once per session
- These bookend lines are standalone — do not combine them with other text on the same line
- **Timestamps on bookends** — every bookend marker must include a real EST timestamp on the same line, placed after the marker text in square brackets. **Three bookends get time+date** (format: `[HH:MM:SS AM/PM EST YYYY-MM-DD]`): CODING PLAN, CODING START, and CODING COMPLETE. **All other bookends get time-only** (format: `[HH:MM:SS AM/PM EST]`). **You must run `date` via the Bash tool and get the result BEFORE writing the bookend line** — you have no internal clock, so any timestamp written without calling `date` first is fabricated. Use `TZ=America/New_York date '+%I:%M:%S %p EST %Y-%m-%d'` for the time+date bookends and `TZ=America/New_York date '+%I:%M:%S %p EST'` for time-only bookends. Do not guess, estimate, or anchor on times mentioned in the user's message. The small delay before text appears is an acceptable tradeoff for accuracy. For the opening pair (CODING PLAN + CODING START), a single `date` call is sufficient — run it once before any text output and reuse the same timestamp for both markers. For subsequent bookends mid-response, call `date` inline before writing the marker. End-of-response section headers (AGENTS USED, FILES CHANGED, COMMIT LOG, WORTH NOTING, SUMMARY) do not get timestamps. **CODING COMPLETE's `date` call must happen before AGENTS USED** — fetch the timestamp, then write the entire end-of-response block (AGENTS USED → FILES CHANGED → COMMIT LOG → WORTH NOTING → SUMMARY → CODING COMPLETE) as one uninterrupted text output using the pre-fetched timestamp
- **Duration annotations** — a `⏱️` annotation appears between **every** consecutive pair of bookends (and before the end-of-response block). No exceptions — if two bookends appear in sequence, there must be a `⏱️` line between them. Format: `⏱️ Xs` (or `Xm Ys` for durations over 60 seconds). The duration is calculated by subtracting the previous bookend's timestamp from the current time. **You must run `date` to get the current time and compute the difference** — never estimate durations mentally. If a phase lasted less than 1 second, write `⏱️ <1s`. **The last working phase always gets a `⏱️`** — its annotation appears immediately before AGENTS USED (as part of the pre-fetched end-of-response block). This includes the gap between CODING START and the next bookend, the gap between AWAITING HOOK and HOOK FEEDBACK, and every other transition

### Bookend Summary

| Bookend | When | Position | Timestamp | Duration |
|---------|------|----------|-----------|----------|
| `🚩🚩CODING PLAN🚩🚩 [HH:MM:SS AM EST YYYY-MM-DD]` | Response will make changes | Very first line of response (skip if purely informational) | Required | — |
| `⚡⚡CODING START⚡⚡ [HH:MM:SS AM EST YYYY-MM-DD]` | Work is beginning | After coding plan bullets (or first line if no plan) | Required | `⏱️` before next bookend |
| `⏳⏳ESTIMATED TIME ≈ Xm⏳⏳` (overall) | Every response with changes | Immediately before CODING START (never skipped) | — | — |
| `⏳⏳ESTIMATED TIME ≈ Xm⏳⏳` (per-phase) | Next phase estimated >2 min | Immediately before the phase's bookend marker | — | — |
| `⏳⏳REVISED ESTIMATED TIME ≈ Xm⏳⏳` | Estimate changed ≥1m after reads | After initial reads/exploration complete, before next action | — | — |
| `📋📋PLAN APPROVED📋📋 [HH:MM:SS AM EST]` | User approved a plan via ExitPlanMode | Before execution begins; followed by CODING PLAN + CODING START (only allowed repeat) | Required | — |
| `✔️✔️CHECKLIST✔️✔️ [HH:MM:SS AM EST]` | A mandatory checklist is executing | Before the checklist name, during work | Required | `⏱️` before next bookend |
| `🔍🔍RESEARCHING🔍🔍 [HH:MM:SS AM EST]` | Entering a research/exploration phase | During work, before edits begin (skip if going straight to changes) | Required | `⏱️` before next bookend |
| `🔄🔄NEXT PHASE🔄🔄 [HH:MM:SS AM EST]` | Work pivots to a new sub-task | During work, between phases (never repeats CODING PLAN/CODING START) | Required | `⏱️` before next bookend |
| `🚧🚧BLOCKED🚧🚧 [HH:MM:SS AM EST]` | An obstacle was hit | During work, when the problem is encountered | Required | `⏱️` before next bookend |
| `🧪🧪VERIFYING🧪🧪 [HH:MM:SS AM EST]` | Entering a verification phase | During work, after edits are applied | Required | `⏱️` before next bookend |
| `🐟🐟AWAITING HOOK🐟🐟 [HH:MM:SS AM EST]` | Hook conditions true after all actions | After verifying; replaces CODING COMPLETE when hook will fire | Required | `⏱️` before HOOK FEEDBACK |
| `⚓⚓HOOK FEEDBACK⚓⚓ [HH:MM:SS AM EST]` | Hook feedback triggers a follow-up | First line of hook response (replaces CODING PLAN as opener) | Required | `⏱️` before end-of-response block |
| `⏱️ Xs` | Phase just ended | Immediately before the next bookend marker | — | Computed |
| `━━━━━━━━━━━━━━━━━━━━━━━━━━━━` | End-of-response block begins | After last `⏱️`, before AGENTS USED | — | — |
| `🕵🕵AGENTS USED🕵🕵` | Response performed work | First end-of-response section | — | — |
| `📁📁FILES CHANGED📁📁` | Files were modified/created/deleted | After AGENTS USED (skip if no files changed) | — | — |
| `🔗🔗COMMIT LOG🔗🔗` | Commits were made | After FILES CHANGED (skip if no commits made) | — | — |
| `🔖🔖WORTH NOTING🔖🔖` | Something deserves attention | After COMMIT LOG (skip if nothing worth noting) | — | — |
| `📝📝SUMMARY📝📝` | Changes were made in the response | After WORTH NOTING | — | — |
| `🔧🔧ESTIMATE CALIBRATED🔧🔧` | Estimate missed by >2 min | After SUMMARY, before ACTUAL TIME (skip if ≤2 min gap) | — | — |
| `⏳⏳ACTUAL TIME: Xm Ys (estimated Xm)⏳⏳` | Every response with CODING COMPLETE | Immediately before CODING COMPLETE (never skipped) | — | Computed from CODING START → CODING COMPLETE |
| `✅✅CODING COMPLETE✅✅ [HH:MM:SS AM EST YYYY-MM-DD]` | All work done | Always the very last line of response | Required | — |

### Flow Examples

**Normal flow (with revised estimate):**
```
🚩🚩CODING PLAN🚩🚩 [01:15:00 AM EST 2026-01-15]
  - brief bullet plan of intended changes

⏳⏳ESTIMATED TIME ≈ 2m⏳⏳ — ~3 file reads + ~4 edits + commit + push cycle
⚡⚡CODING START⚡⚡ [01:15:01 AM EST 2026-01-15]
  ... reading files, searching codebase ...
⏳⏳REVISED ESTIMATED TIME ≈ 4m⏳⏳ — found 12 files to edit, not 4
  ... applying changes ...
  ⏱️ 2m 29s
✔️✔️CHECKLIST✔️✔️ [01:17:30 AM EST]
  Pre-Commit Checklist
  ... checklist items ...
  ⏱️ 30s
🧪🧪VERIFYING🧪🧪 [01:18:00 AM EST]
  ... validating edits, running hook checks ...
  ⏱️ 15s
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🕵🕵AGENTS USED🕵🕵
  1. Agent 0 (Main) — applied changes, ran checklists
📁📁FILES CHANGED📁📁
  `file.md` (edited)
  `new-file.js` (created)
🔗🔗COMMIT LOG🔗🔗
  abc1234 — Add feature X
📝📝SUMMARY📝📝
  - Updated X in `file.md` (edited)
  - Created `new-file.js` (created)
⏳⏳ACTUAL TIME: 3m 14s (estimated 4m)⏳⏳
✅✅CODING COMPLETE✅✅ [01:18:15 AM EST 2026-01-15]
```

**Hook anticipated flow:**
```
🚩🚩CODING PLAN🚩🚩 [01:15:00 AM EST 2026-01-15]
  - brief bullet plan of intended changes

⏳⏳ESTIMATED TIME ≈ 3m⏳⏳ — ~4 file edits + commit + push cycle
⚡⚡CODING START⚡⚡ [01:15:01 AM EST 2026-01-15]
  ... work (commit without push) ...
  ⏱️ 1m 44s
🐟🐟AWAITING HOOK🐟🐟 [01:16:45 AM EST]
  ← hook fires →
  ⏱️ 5s
⚓⚓HOOK FEEDBACK⚓⚓ [01:16:50 AM EST]
  ... push ...
  ⏱️ 20s
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🕵🕵AGENTS USED🕵🕵
  1. Agent 0 (Main) — applied changes, pushed
📁📁FILES CHANGED📁📁
  `file.md` (edited)
🔗🔗COMMIT LOG🔗🔗
  abc1234 — Add feature X
📝📝SUMMARY📝📝
  - Updated X in `file.md`
  - Pushed to remote
⏳⏳ACTUAL TIME: 2m 9s (estimated 3m)⏳⏳
✅✅CODING COMPLETE✅✅ [01:17:10 AM EST 2026-01-15]
```

### Hook anticipation — bug context
**The failure pattern:** if the hook conditions are evaluated *before* a `git push` completes (or evaluated mentally instead of actually running the git commands), the prediction can be wrong — e.g. concluding there are unpushed commits when the push already succeeded. Writing `🐟🐟AWAITING HOOK🐟🐟` in that case means the hook never fires (because all conditions are actually false), and the conversation gets stuck with no `✅✅CODING COMPLETE✅✅`.

**What to watch for:** any scenario where actions (especially `git push`) complete in the same response as the hook check. The temptation is to predict the outcome rather than wait and verify.

**The fix:** (1) always evaluate *after* all actions in the response are complete, and (2) *actually run* the three git commands — never reason about their output mentally.

### Token Budget Reference
*See `repository-information/TOKEN-BUDGETS.md` — section "Chat Bookends"*

---
> **--- END OF CHAT BOOKENDS ---**
---

## Session Start Checklist (MANDATORY — RUN FIRST)
> **MANDATORY FIRST ACTION — DO NOT SKIP**
> Complete ALL checks below and commit any fixes BEFORE reading or acting on the user's request.
> If checklist items produce changes, commit them as a separate commit with message:
> `Session start: fix template drift`
> **The user's task is NOT urgent enough to skip this. Do it first. Every time.**

### Always Run (every repo, every session — NEVER skip)
These rules apply universally — they are **NOT** skipped by the template repo short-circuit.

**Session isolation** — before acting on ANY instructions, verify this session is not contaminated by stale context from a prior session or a different repo. Run these checks in order:
- **Repo identity**: run `git remote -v` and extract the full `org/repo` identity (e.g. `github.com/MyOrg/my-project` → `MyOrg/my-project`). Compare this against any repo-specific references carried over in the session context — branch names (e.g. `claude/...`), commit SHAs, remote URLs, or org/repo identifiers from prior instructions. If the current repo's `org/repo` does **not** match references from prior session context, those references are stale cross-repo contamination — **discard all prior commit, push, and branch instructions entirely** and act only on the user's explicit current request. Do NOT replay commits, cherry-pick changes, push to branches, or continue work that originated in a different repo
- **Clean working state**: run `git status`. If there are uncommitted changes (staged or unstaged) that were NOT made by this session, they are leftovers from a prior session — do NOT commit them. Ask the user how to handle them (stash, discard, or incorporate). If there are local `claude/*` branches that don't match this session's branch, they are stale — ignore them (do NOT push to or continue work on a stale branch)
- **Auto memory cross-check**: if auto memory (`~/.claude/projects/*/memory/MEMORY.md`) contains references to a different `org/repo` than the current one, those entries are stale — ignore them entirely when making decisions about commits, branches, or repo structure
- **Session continuity**: if this session was started via `--continue` or `--fork-session` from a prior session that worked on a **different repo**, treat all inherited conversation context as informational only — do NOT execute any commits, pushes, or file changes carried over from the prior session's repo. Verify the current repo identity before every destructive or write action

**Branch hygiene** — run `git remote set-head origin main` to ensure `origin/HEAD` points to `main`. If a local `master` branch exists and points to the same commit as `origin/main`, delete it with `git branch -D master`. This prevents the auto-merge workflow from failing with exit code 128 due to branch misconfiguration.

**Deployment Flow**
- Never push directly to `main`
- Push to `claude/*` branches only
- `.github/workflows/auto-merge-claude.yml` handles everything automatically:
  1. Merges the claude branch into main
  2. Deletes the claude branch
  3. Deploys to GitHub Pages
- The "Create a pull request" message in push output is just GitHub boilerplate — ignore it, the workflow handles merging automatically
- **Push only once per branch** — do NOT push multiple times to the same `claude/*` branch in a single session. The workflow uses a shared concurrency group (`"pages"`) with `cancel-in-progress: false`, so each push queues a separate workflow run. If an earlier run merges and deletes the branch, subsequent queued runs fail with exit code 128 because the branch no longer exists. **This includes sequential user requests** — if the user asks for task A and then task B in the same session, commit both locally and push once after all work is done. Do NOT push after task A and then push again after task B. The only exception is if a re-push is needed to recover from a failed workflow (e.g. the branch still exists on the remote but the merge didn't happen)
- **Pre-push verification** — before executing any `git push`, run the Pre-Push Checklist (see below). This is mandatory even when the Deployment Flow rules are satisfied

**Template repo short-circuit** — check the `IS_TEMPLATE_REPO` variable in the Template Variables table. If its value matches the actual repo name (extracted from `git remote -v`), this is the template repo itself — skip the Template Drift Checks below and proceed directly to the user's request. If the value is `No` or does not match the actual repo name, continue to the next check.

**Initialized repo short-circuit** — check if `README.md` contains the placeholder text `You are currently using the **`. If it does NOT, the repo has already been initialized — skip the Template Drift Checks below and proceed directly to the user's request. If it DOES, the repo is a fresh fork that needs initialization — continue to the Template Drift Checks.

### Template Drift Checks (forks/clones only)
These checks catch template drift that accumulates when the repo is cloned/forked into a new name. They do **not** apply to the template repo itself.

> **Token budget:** *See `repository-information/TOKEN-BUDGETS.md` — section "Template Drift Checks"*

0. **Set `IS_TEMPLATE_REPO` to `No`** — in the Template Variables table, change `IS_TEMPLATE_REPO` from its current value to `No`. Reaching the drift checks means the short-circuit already confirmed this is NOT the template repo
1. **Org name auto-detect** — run `git remote -v` and extract the org/owner from the remote URL (e.g. `github.com/NewOrg/myrepo` → `NewOrg`). Compare it to the `YOUR_ORG_NAME` value in the Template Variables table. If they differ, update the table value and propagate to every file in the "Where it appears" column by **finding and replacing** the old org name (`ShadowAISolutions`) with the new org name in all occurrences (URLs, text, branding). Also update `DEVELOPER_NAME` to match the new org name (unless the user has explicitly set `DEVELOPER_NAME` to a different value)
2. **Repo name auto-detect** — compare the actual repo name (from the same remote URL) to the `YOUR_REPO_NAME` value in the Template Variables table. If they differ, update the table value and propagate it to every file in the "Where it appears" column by **finding and replacing** the old repo name (`autoupdatehtmltemplate`) with the new repo name in all occurrences (URLs, text, structure references)
3. **Relative links (already dynamic — do NOT modify)** — certain markdown files use relative paths that automatically resolve to the correct repo via GitHub's blob-view URL structure (see *Relative Path Resolution on GitHub* reference section for how this works). These links work on any fork/clone without initialization and must **never** be converted to absolute URLs or modified during drift checks. Files with relative links:
   - `SECURITY.md` — private security advisory link (`../../security/advisories/new`)
   - `repository-information/SUPPORT.md` — issue creation links (`../../../issues/new`)
   - `README.md` — repository settings links in "Initialize This Template" section (`../../settings/pages`, `../../settings/environments`). These are removed by step #6 but must not be converted to absolute URLs if encountered before removal
4. **Absolute URL propagation** — some files contain absolute URLs with the org and repo name that cannot use relative paths (YAML metadata fields, GitHub Pages URLs on a different domain, Mermaid diagram text). After steps 1–2, find and replace the template repo's values (`ShadowAISolutions`/`autoupdatehtmltemplate`) with the fork's actual values in these files. **For each file, verify every URL listed below — not just the first one you find:**
   - `.github/ISSUE_TEMPLATE/config.yml` — YAML `url` fields require absolute URLs:
     - `url: https://github.com/ShadowAISolutions/autoupdatehtmltemplate/blob/main/repository-information/SUPPORT.md` (support link)
     - `url: https://github.com/ShadowAISolutions/autoupdatehtmltemplate/security/advisories/new` (security advisory link)
   - `CITATION.cff` — citation metadata (not rendered markdown):
     - `repository-code:` URL (`https://github.com/ShadowAISolutions/autoupdatehtmltemplate`)
     - `url:` field (`https://ShadowAISolutions.github.io/autoupdatehtmltemplate`)
   - `repository-information/STATUS.md` — placeholder in Hosted Pages table (`github.io` is a different domain, can't use relative paths):
     - If the Live URL column still contains `*(deploy to activate)*`, replace it with `[View](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME/)` (resolved values)
   - `repository-information/ARCHITECTURE.md` — Mermaid diagram text (not a clickable link):
     - `LIVE["Live Site\nShadowAISolutions.github.io/autoupdatehtmltemplate"]`
   - `README.md` — live site link and source repo URL:
     - Live site `[View](https://...)` or `[...github.io/...](https://...)` link
     - Source repo URL in "Copy This Repository" section (gets removed in step #6, but update if still present)
   When replacing, change **only the org and repo name portions** of each URL — preserve the rest of the URL path and structure intact (e.g. `/security/advisories/new` stays the same, only `ShadowAISolutions/autoupdatehtmltemplate` changes to `NewOrg/newrepo`)
   **Verification:** after replacements, run `grep -r 'ShadowAISolutions\|autoupdatehtmltemplate' --include='*.md' --include='*.yml' --include='*.cff' --include='*.html'` (excluding CLAUDE.md) and confirm zero hits remain outside of `Developed by:` branding lines and provenance markers
5. **README title** — replace the `# Auto Update HTML Template` heading at the top of `README.md` with `# ReadMe - YOUR_REPO_NAME` (resolved value, preserving any invisible characters before the title text). This heading format is specific to the README and is independent of `YOUR_PROJECT_TITLE` — do NOT update `YOUR_PROJECT_TITLE` in the Template Variables table (it stays as `Auto Update HTML Template` until the user explicitly changes it)
6. **README live site link** — check if `README.md` still contains the placeholder text (`You are currently using the **YOUR_REPO_NAME**...`). If so, replace it with: `**Live site:** [YOUR_ORG_NAME.github.io/YOUR_REPO_NAME](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME)` (resolved values)
7. **Remove "Copy This Repository" and "Initialize This Template" sections** — if `README.md` contains the `## Copy This Repository` or `## Initialize This Template` sections, delete each entirely (from the `##` heading through to the line immediately before the next `##` heading). These sections are only useful on the template repo itself; forks/clones should not keep them
8. **Unresolved placeholders** — scan for any literal `YOUR_ORG_NAME`, `YOUR_REPO_NAME`, `YOUR_PROJECT_TITLE`, or `DEVELOPER_NAME` strings in code files (not CLAUDE.md) and replace them with resolved values
9. **Variable propagation** — if any value in the Template Variables table was changed (in this or a prior session), verify the new value has been propagated to every file listed in the "Where it appears" column
10. **Confirm completion** — after all checks pass, briefly state to the user: "Session start checklist complete — no issues found" (or list what was fixed). Then proceed to their request

### Token Budget Reference
*See `repository-information/TOKEN-BUDGETS.md` — section "Session Start Checklist"*

---
> **--- END OF SESSION START CHECKLIST ---**
---

## Initialize Command
If the user's prompt is just **"initialize"** (after the Session Start Checklist has completed):
1. **Verify placeholders are resolved** — confirm that `repository-information/STATUS.md` no longer contains `*(deploy to activate)*` (drift check step #4 should have replaced it). If it's still there, replace it now with `[View](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME/)` (resolved values)
2. Update the `Last updated:` timestamp in `README.md` to the real current time
3. Commit with message `Initialize deployment`
4. Push to the `claude/*` branch (Pre-Push Checklist applies)

**No version bumps** — initialization never bumps `build-version`, `version.txt`, or any version-tracking files. It deploys whatever versions already exist. This applies on both the template repo and forks.

This triggers the auto-merge workflow, which merges into `main` and deploys to GitHub Pages — populating the live site for the first time. No other changes are needed.

---
> **--- END OF INITIALIZE COMMAND ---**
---

## Template Repo Guard
> When the actual repo name (from `git remote -v`) matches the `IS_TEMPLATE_REPO` value in the Template Variables table (i.e. this is the template repo itself, not a fork/clone):
> - **Session Start Checklist template drift checks are skipped** — the "Template repo short-circuit" in the Always Run section skips the entire numbered checklist. The "Always Run" section (branch hygiene and deployment flow) still applies every session
> - **All version bumps are skipped** — Pre-Commit Checklist items #1 (`.gs` version bump), #2 (HTML build-version), #3 (version.txt sync), #5 (STATUS.md), **#7 (CHANGELOG.md)**, and #9 (version prefix in commit message) are all skipped unless the user explicitly requests them. **DO NOT add CHANGELOG entries on the template repo** — the CHANGELOG must stay clean with `*(No changes yet)*` so that forks start with a blank history
> - **GitHub Pages deployment is skipped** — the workflow's `deploy` job reads `IS_TEMPLATE_REPO` from `CLAUDE.md` and compares it against the repo name; deployment is skipped when they match
> - **`YOUR_ORG_NAME` and `YOUR_REPO_NAME` are frozen as placeholders** — in the Template Variables table, these values must stay as `YourOrgName` and `YourRepoName` (generic placeholders). Do NOT update them to match the actual org/repo (`ShadowAISolutions`/`autoupdatehtmltemplate`). The code files throughout the repo use the real `ShadowAISolutions/autoupdatehtmltemplate` values so that links are functional. On forks, the Session Start drift checks detect the mismatch between the placeholder table values and the actual `git remote -v` values, then find and replace the template repo's real values (`ShadowAISolutions`/`autoupdatehtmltemplate`) in the listed files with the fork's actual org/repo
> - Pre-Commit items #0, #4, #6, #8, #10, #11, #12 still apply normally
> - **Pre-Push Checklist is never skipped** — all 5 items apply on every repo including the template repo

---
> **--- END OF TEMPLATE REPO GUARD ---**
---

## Pre-Commit Checklist
**Before every commit, verify ALL of the following:**

> **TEMPLATE REPO GATE** — before running any numbered item, check: does the actual repo name (from `git remote -v`) match `IS_TEMPLATE_REPO` in the Template Variables table? If **yes**, items #1, #2, #3, #5, #6 (version-bump portion), #7, and #9 are **all skipped** — do NOT bump versions, update version-tracking files, add CHANGELOG entries, or use version prefixes in commit messages. Proceed directly to the items that still apply (#0, #4, #8, #10, #11, #12, #13). This gate also applies during `initialize` — initialization never bumps versions on any repo

0. **Commit belongs to this repo and task** — before staging or committing ANY changes, verify: (a) `git remote -v` still matches the repo you are working on — if it doesn't, STOP and do not commit; (b) every file being staged was modified by THIS session's task, not inherited from a prior session or a different repo; (c) the commit message describes work you actually performed in this session — never commit with a message copied from a prior session's commit. If any of these checks fail, discard the stale changes and proceed only with the user's current request. **This item is never skipped** — it applies on every repo including the template repo
1. **Version bump (.gs)** — if any `.gs` file was modified, increment its `VERSION` variable by 0.01 (e.g. `"01.13g"` → `"01.14g"`)
2. **Version bump (HTML)** — if any embedding HTML page in `live-site-pages/` was modified, increment its `<meta name="build-version">` by 0.01 (e.g. `"01.01w"` → `"01.02w"`). **Skip if Template Repo Guard applies (see above)**
3. **Version.txt sync** — if a `build-version` was bumped, update the corresponding `<page-name>.version.txt` to the same value. **Skip if Template Repo Guard applies**
4. **Template version freeze** — never bump `live-site-templates/AutoUpdateOnlyHtmlTemplate.html` — its version must always stay at `01.00w`
5. **STATUS.md** — if any version was bumped, update the matching version in `repository-information/STATUS.md`. **Skip if Template Repo Guard applies**
6. **ARCHITECTURE.md** — if any version was bumped or the project structure changed, update the diagram in `repository-information/ARCHITECTURE.md`. **Version-bump portion: skip if Template Repo Guard applies.** Structure changes still apply on the template repo. **When versions are bumped, update every Mermaid node that displays a version string** — not just the HTML node. Specifically check: `INDEX["index.html\n(build-version: XX.XXw)"]`, `VERTXT["index.version.txt\n(XX.XXw)"]`, and any future page/GAS nodes with version text. The VERTXT version must always match the INDEX build-version (since version.txt tracks the HTML page). The TPL node (`01.00w`) is frozen and never changes
7. **CHANGELOG.md** — every user-facing change must have an entry under `## [Unreleased]` in `repository-information/CHANGELOG.md`. Each entry must include an EST timestamp down to the second (format: `` `YYYY-MM-DD HH:MM:SS EST` — Description``). The `[Unreleased]` section header must also show the date/time of the most recent entry. **Timestamps must be real** — run `TZ=America/New_York date '+%Y-%m-%d %H:%M:%S EST'` to get the actual current time; never fabricate or increment timestamps. **Skip if Template Repo Guard applies (see above)**
8. **README.md structure tree** — if files or directories were added, moved, or deleted, update the ASCII tree in `README.md`
9. **Commit message format** — if versions were bumped, the commit message must start with the version prefix(es): `v{VERSION}` for `.gs`, `v{BUILD_VERSION}` for HTML (e.g. `v01.14g v01.02w Fix bug`)
10. **Developer branding** — any newly created file must have `Developed by: DEVELOPER_NAME` as the last line (using the appropriate comment syntax for the file type), where `DEVELOPER_NAME` is resolved from the Template Variables table
11. **README.md `Last updated:` timestamp** — on every commit, update the `Last updated:` timestamp near the top of `README.md` to the real current time (run `TZ=America/New_York date '+%Y-%m-%d %I:%M:%S %p EST'`). **This rule always applies — it is NOT skipped by the Template Repo Guard**
12. **Internal link integrity** — if any markdown file is added, moved, or renamed, verify that all internal links (`[text](path)`) in the repo still resolve to existing files. Pay special attention to cross-directory links — see the Internal Link Reference section for the correct relative paths
13. **README section link tips** — every `##` section in `README.md` that contains (or will contain) any clickable links must have this blockquote as the first line after the heading (before any other content): `> **Tip:** Links below navigate away from this page. **Ctrl + click** (or right-click → *Open in new tab*) to keep this ReadMe visible while you work.` — Sections with no links (e.g. a section with only a code block or plain text) do not need the tip

### Maintaining these checklists
- The Session Start, Pre-Commit, and Pre-Push checklists are the **single source of truth** for all actionable rules. Detailed sections below provide reference context only
- When adding new rules to CLAUDE.md, add the actionable check to the appropriate checklist and put supporting details in a reference section — do not duplicate the rule in both places
- When editing CLAUDE.md, check whether any existing reference section restates a checklist item — if so, remove the duplicate and add a `*Rule: see ... Checklist item #N*` pointer instead
- **Content placement** — CLAUDE.md must stay focused on rules and process that Claude enforces every session (checklists, behavioral rules, formatting requirements). Domain-specific coding knowledge, architectural reference, and detailed technical context that Claude only needs when working on specific features should live in separate reference files (e.g. `repository-information/CODING-GUIDELINES.md`, `repository-information/TOKEN-BUDGETS.md`). Replace the extracted content with a one-line pointer: `*See \`repository-information/FILE.md\` — section "X"*`. Claude reads these files on demand when the relevant feature area is being worked on
- **Section separators** — every `##` section in CLAUDE.md must end with a double-ruled banner. When adding a new `##` section, add the following block between the end of its content and the next `##` heading:
  ```
  ---
  > **--- END OF SECTION NAME ---**
  ---
  ```
  Replace `SECTION NAME` with the section's heading in ALL CAPS. The only exception is Developer Branding (the final section), which has no separator after it

### Token Budget Reference
*See `repository-information/TOKEN-BUDGETS.md` — section "Pre-Commit Checklist"*

---
> **--- END OF PRE-COMMIT CHECKLIST ---**
---

## Pre-Push Checklist
**Before every `git push`, verify ALL of the following:**

1. **Branch name** — confirm the branch being pushed is the `claude/*` branch assigned to THIS session. If a different branch name is checked out (e.g. `main`, or a `claude/*` branch from a prior session), STOP — do not push. Switch to the correct branch or ask the user for guidance. **This item is never skipped** — it applies on every repo including the template repo
2. **Remote URL** — run `git remote -v` and verify the origin URL matches the repo this session is working on. If the URL has changed or does not match (e.g. context drifted mid-session to a different repo), STOP — do not push. This catches context drift that occurred after the Session Start Checklist and after Pre-Commit item #0
3. **Commit audit** — run `git log origin/main..HEAD --oneline` and verify that every commit listed was created by THIS session. Look for commit messages, timestamps, or SHAs that do not match work performed in this session. If any commit appears to be inherited from a prior session or a different repo, STOP — do not push. Remove the stale commits (interactive rebase or reset) before proceeding, or ask the user for guidance
4. **No cross-repo content** — run `git diff origin/main..HEAD` and scan for references to a different org/repo than the current one. Specifically, look for hardcoded org names or repo names in URLs, import paths, or configuration that do not match the current repo's `org/repo` identity (from `git remote -v`). References to `ShadowAISolutions/autoupdatehtmltemplate` are expected when `IS_TEMPLATE_REPO` matches the current repo name and in provenance markers — only flag references to a *third* repo that is neither the current repo nor the template origin. If suspicious cross-repo content is found, STOP and ask the user to verify before pushing
5. **Push-once enforcement** — verify that no push has already been made to this `claude/*` branch in this session. If a push already succeeded earlier in the session, STOP — do not push again (see Deployment Flow rules). The only exception is recovery from a failed workflow where the branch still exists on the remote but the merge did not happen

### Abort protocol
If any pre-push check fails, do NOT proceed with `git push`. Instead:
- State which check failed and why
- Do NOT silently fix the issue and push — the failure may indicate context contamination that requires user judgment
- Ask the user how to proceed (discard commits, fix and retry, or abandon the push)

### Token Budget Reference
*See `repository-information/TOKEN-BUDGETS.md` — section "Pre-Push Checklist"*

---
> **--- END OF PRE-PUSH CHECKLIST ---**
---

## Template Variables

These variables are the **single source of truth** for repo-specific values. When a variable value is changed here, Claude Code must propagate the new value to every file in the repo that uses it.

| Variable | Value | Where it appears | Description |
|----------|-------|------------------|-------------|
| `YOUR_ORG_NAME` | ShadowAISolutions | README<br>CITATION.cff<br>STATUS.md<br>ARCHITECTURE.md<br>issue template config | GitHub org or username that owns this repo.<br>Auto-detected from `git remote -v` on forks.<br>Frozen as a placeholder on the template repo so drift checks can detect forks and replace `ShadowAISolutions` with the fork's actual org |
| `YOUR_ORG_LOGO_URL` | https://logoipsum.com/logoipsum-avatar.png | `index.html`<br>template HTML | URL to the org's logo image.<br>Used in HTML pages for branded logos.<br>Replace with your org's actual logo URL after forking |
| `YOUR_REPO_NAME` | SAIS_PFC_Website | README<br>CITATION.cff<br>STATUS.md<br>ARCHITECTURE.md<br>issue template config | Name of this GitHub repository.<br>Auto-detected from `git remote -v` on forks.<br>Frozen as a placeholder on the template repo so drift checks can detect forks and replace `autoupdatehtmltemplate` with the fork's actual name |
| `YOUR_PROJECT_TITLE` | CHANGE THIS PROJECT TITLE TEMPLATE | `<title>` tag in<br>`index.html`<br>and template HTML | Human-readable project title shown in browser tabs.<br>Independent of the repo name — set this to whatever you want users to see as the page title |
| `DEVELOPER_NAME` | ShadowAISolutions | LICENSE<br>README<br>CITATION.cff<br>FUNDING.yml<br>GOVERNANCE.md<br>CONTRIBUTING.md<br>PR template<br>"Developed by:" footers<br>(all files) | Name used for attribution, copyright, and branding throughout the repo.<br>On forks, defaults to the new org name unless explicitly overridden by the user |
| `DEVELOPER_LOGO_URL` | https://www.shadowaisolutions.com/SAIS%20Logo.png | HTML splash screen<br>`LOGO_URL` variable<br>(in `index.html`<br>and template) | URL to the developer's logo shown on the "Code Ready" and "Website Ready" splash screens.<br>Replace with your own logo URL after forking |
| `IS_TEMPLATE_REPO` | No | CLAUDE.md<br>workflow deploy<br>job condition | Controls whether this repo is treated as the template or a fork.<br>Compared against the actual repo name from `git remote -v` — if they match, this is the template repo (drift checks, version bumps, and deployment are all skipped).<br>If `No` or doesn't match, it's a fork.<br>Drift checks set this to `No` as their first step |

### How variables work
- **In code files** (HTML, YAML, Markdown, etc.): use the **resolved value** (e.g. write `MyOrgName`, not `YOUR_ORG_NAME`)
- **In CLAUDE.md instructions**: the placeholder names (`YOUR_ORG_NAME`, `DEVELOPER_NAME`, etc.) may appear in examples and rules — Claude Code resolves them using the table above

---
> **--- END OF TEMPLATE VARIABLES ---**
---

## Version Bumping
*Rule: see Pre-Commit Checklist item #1. Reference details below.*
- The `VERSION` variable is near the top of each `.gs` file (look for `var VERSION = "..."`)
- Format includes a `g` suffix: e.g. `"01.13g"` → `"01.14g"`
- Do NOT bump VERSION if the commit doesn't touch the `.gs` file

### GAS Projects
Each GAS project has a code file and a corresponding embedding page. Register them in the table below as you add them.

| Project | Code File | Embedding Page |
|---------|-----------|----------------|
| *(Project name)* | `googleAppsScripts/<Project Name>/<CodeFile>.gs` | `live-site-pages/<page-name>.html` |

---
> **--- END OF VERSION BUMPING ---**
---

## Build Version (Auto-Refresh for embedding pages)
*Rules: see Pre-Commit Checklist items #2, #3, #4. Reference details below.*
- Look for `<meta name="build-version" content="...">` in the `<head>`
- Format includes a `w` suffix: e.g. `"01.11w"` → `"01.12w"`
- Each embedding page polls `version.txt` every 10 seconds — when the deployed version differs from the loaded version, it auto-reloads

### Auto-Refresh via version.txt Polling
- **All embedding pages must use the `version.txt` polling method** — do NOT poll the page's own HTML
- **Version file naming**: the version file must be named `<page-name>.version.txt`, matching the HTML file it tracks (e.g. `index.html` → `index.version.txt`, `dashboard.html` → `dashboard.version.txt`). The `.version.txt` double extension ensures the version file sorts **after** the `.html` file alphabetically
- Each version file holds only the current build-version string (e.g. `01.08w`)
- The polling logic fetches the version file (~7 bytes) instead of the full HTML page, reducing bandwidth per poll from kilobytes to bytes
- URL resolution: derive the version file URL relative to the current page's directory, using the page's own filename. See the template file (`live-site-templates/AutoUpdateOnlyHtmlTemplate.html`) for the implementation
- **The `if (!pageName)` fallback is critical** — when a page is accessed via a directory URL (e.g. `https://example.github.io/myapp/`), `pageName` resolves to an empty string. Without the fallback to `'index'`, the poll fetches `.version.txt` (wrong file) and triggers an infinite reload loop
- Cache-bust with a query param: `fetch(versionUrl + '?_cb=' + Date.now(), { cache: 'no-store' })`
- Compare the trimmed response text against the page's `<meta name="build-version">` content
- The template in `live-site-templates/AutoUpdateOnlyHtmlTemplate.html` already implements this pattern — use it as a starting point for new projects

### New Embedding Page Setup Checklist
When creating a **new** HTML embedding page, follow every step below:

1. **Copy the template** — start from `live-site-templates/AutoUpdateOnlyHtmlTemplate.html`, which already includes:
   - `<meta name="build-version" content="...">` in the `<head>`
   - Version file polling logic (10-second interval)
   - Version indicator pill (bottom-right corner)
   - Green "Website Ready" splash overlay + sound playback
   - AudioContext handling and screen wake lock
2. **Choose the directory** — create a new subdirectory under `live-site-pages/` named after the project (e.g. `live-site-pages/my-project/`)
3. **Create the version file** — place a `<page-name>.version.txt` file in the **same directory** as the HTML page (e.g. `index.version.txt` for `index.html`), containing only the initial build-version string (e.g. `01.00w`)
4. **Update the polling URL in the template** — ensure the JS version-file URL derivation matches the HTML filename (the template defaults to deriving it from the page's own filename)
5. **Create `sounds/` directory** — copy the `sounds/` folder (containing `Website_Ready_Voice_1.mp3`) into the new page's directory so the splash sound works
6. **Set the initial build-version** — in the HTML `<head>`, set `<meta name="build-version" content="01.00w">` and match it in `<page-name>.version.txt`
7. **Update the page title** — replace `YOUR_PROJECT_TITLE` in `<title>` with the actual project name
8. **Register in GAS Projects table** — if this page embeds a GAS iframe, add a row to the GAS Projects table in the Version Bumping section above
9. **Add developer branding** — ensure `<!-- Developed by: DEVELOPER_NAME -->` is the last line of the HTML file

### Directory Structure (per embedding page)
```
live-site-pages/
├── <page-name>/
│   ├── index.html               # The embedding page (from template)
│   ├── index.version.txt        # Tracks index.html build-version (e.g. "01.00w")
│   └── sounds/
│       └── Website_Ready_Voice_1.mp3
```
For pages that live directly in `live-site-pages/` (not in a subdirectory), the version file and `sounds/` folder sit alongside the HTML file (e.g. `live-site-pages/index.html` + `live-site-pages/index.version.txt`).

---
> **--- END OF BUILD VERSION ---**
---

## Commit Message Naming
*Rule: see Pre-Commit Checklist item #9. Reference details below.*
- Both version types use the `v` prefix — suffix indicates type: `g` = Google Apps Script, `w` = website
- If neither a `.gs` file nor an embedding HTML page was updated: no version prefix needed
- Example: `v01.19g Fix sign-in popup to auto-close after authentication`
- Example: `v01.19g v01.12w Add auth wall with build version bump`

---
> **--- END OF COMMIT MESSAGE NAMING ---**
---

## ARCHITECTURE.md Version Nodes
*Rule: see Pre-Commit Checklist item #6. Reference details below.*

The Mermaid diagram in `repository-information/ARCHITECTURE.md` contains nodes that display version strings. When a build-version is bumped, **all** nodes showing that version must be updated — not just the HTML page node.

### Version-bearing nodes

| Node ID | What it represents | Example text | Tracks |
|---------|--------------------|-------------|--------|
| `INDEX` | `index.html` | `INDEX["index.html\n(build-version: 01.00w)"]` | Current build-version of the embedding page |
| `VERTXT` | `index.version.txt` | `VERTXT["index.version.txt\n(01.00w)"]` | Must always match INDEX — it's the polling file for that page |
| `TPL` | `AutoUpdateOnlyHtmlTemplate.html` | `TPL["...\n(build-version: 01.00w — never bumped)"]` | Frozen at `01.00w` — never changes |

### Why the miss happens
The `.version.txt` file gets bumped by Pre-Commit item #3, but the VERTXT *Mermaid node* is a separate representation of the same version. It's easy to bump the file and forget the diagram node. Always check both INDEX and VERTXT nodes together when bumping a build-version.

### Adding new pages
When a new embedding page is created (see New Embedding Page Setup Checklist), add corresponding nodes to the diagram:
- A page node: `NEWPAGE["page-name.html\n(build-version: XX.XXw)"]`
- A version file node: `NEWVER["page-name.version.txt\n(XX.XXw)"]`
- Both must be updated together on every version bump for that page

---
> **--- END OF ARCHITECTURE.MD VERSION NODES ---**
---

## GAS Code Constraints
*See `repository-information/CODING-GUIDELINES.md` — section "GAS Code Constraints"*

---
> **--- END OF GAS CODE CONSTRAINTS ---**
---

## Race Conditions — Config vs. Data Fetch
*See `repository-information/CODING-GUIDELINES.md` — section "Race Conditions — Config vs. Data Fetch"*

---
> **--- END OF RACE CONDITIONS ---**
---

## API Call Optimization (Scaling Goal)
*See `repository-information/CODING-GUIDELINES.md` — section "API Call Optimization (Scaling Goal)"*

---
> **--- END OF API CALL OPTIMIZATION ---**
---

## UI Dialogs — No Browser Defaults
*See `repository-information/CODING-GUIDELINES.md` — section "UI Dialogs — No Browser Defaults"*

---
> **--- END OF UI DIALOGS ---**
---

## Phantom Edit (Timestamp Alignment)
- When the user asks for a **phantom edit** or **phantom update**, touch every file in the repo with a no-op change so all files share the same commit timestamp on GitHub
- **Skip all version bumps** — do NOT increment `build-version` in HTML pages or `VERSION` in `.gs` files
- For text files: add a trailing newline
- For binary files (e.g. `.mp3`): append a null byte
- **Reset `repository-information/CHANGELOG.md`** — replace all entries with a fresh template (keep the header, version suffix note, and an empty `[Unreleased]` section with `*(No changes yet)*`). This gives the repo a clean history starting point
- **Update `Last updated:` in `README.md`** — set the timestamp to the real current time (run `TZ=America/New_York date '+%Y-%m-%d %I:%M:%S %p EST'`). This is the only substantive edit besides the no-op touches
- Commit message: `Auto Update HTML Template Created` (no version prefix)

---
> **--- END OF PHANTOM EDIT ---**
---

## Merge Conflict Prevention (Auto-Merge Workflow)
The auto-merge workflow merges claude/* branches into main using `git merge --ff-only` with a `-X theirs` fallback. The `-X theirs` strategy auto-resolves content conflicts by preferring the incoming branch.

**Why this matters:** Every claude/* push triggers the workflow. If a prior workflow already merged a different claude branch into main (advancing main beyond this branch's fork point), a fast-forward is no longer possible. The fallback merge can hit content conflicts — especially in `CHANGELOG.md`, which is modified on every commit by the Pre-Commit Checklist. Without `-X theirs`, the merge fails with exit code 1, the auto-merge job fails, and the deploy job is skipped (its condition requires auto-merge success).

**Why `-X theirs` is safe:** The claude branch is always branched from main and contains strictly newer changes. When both sides modify the same lines (e.g. CHANGELOG.md's `[Unreleased]` header timestamp), the claude branch's version is always the one we want. The `-X theirs` strategy resolves exactly this class of conflict — same-line edits where the incoming branch has the latest content.

**What this does NOT cover:** If the conflict is structural (e.g. a file was deleted on main but modified on the branch), `-X theirs` may not produce the desired result. These cases are rare in the claude/* workflow and would need manual intervention.

---
> **--- END OF MERGE CONFLICT PREVENTION ---**
---

## Execution Style
- For clear, straightforward requests: **just do it** — make the changes, commit, and push without asking for plan approval
- Only ask clarifying questions when the request is genuinely ambiguous or has multiple valid interpretations
- Do not use formal plan-mode approval workflows for routine tasks (version bumps, file moves, feature additions, bug fixes, etc.)

---
> **--- END OF EXECUTION STYLE ---**
---

## Pushback & Reasoning
- When you have a well-founded technical or design opinion, **make your case and defend it** — do not fold at the first sign of disagreement. Explain the reasoning clearly, cite concrete consequences, and hold your position until one of two things happens: (a) the user presents a counterargument that genuinely changes the calculus, or (b) the user explicitly overrides the decision (e.g. "do it anyway", "I understand, but I want X")
- A user questioning your recommendation is not the same as overriding it — questions are an invitation to explain further, not to capitulate
- If you are eventually convinced the user is right, say so honestly and explain what changed your mind
- If the user overrides you despite your reasoning, comply without passive-aggression — state the tradeoff once, then execute cleanly

---
> **--- END OF PUSHBACK & REASONING ---**
---

## User-Perspective Reasoning
- When organizing, ordering, or explaining anything in this repo, **always reason from the user's perspective** — how they experience the flow, read the output, or understand the structure. Never reason from internal implementation details (response-turn boundaries, tool-call mechanics, API round-trips) when the user-facing view tells a different story
- The trap: internal mechanics can suggest one ordering/grouping, while the user's actual experience suggests another. When these conflict, the user's experience wins every time
- Before finalizing any structural decision (ordering lists, grouping related items, naming things), ask: "does this match what the user sees and expects?" If the answer requires knowing implementation details to make sense, the structure is wrong
- **Example — bookend ordering:** the Bookend Summary table is ordered by the chronological flow as the user experiences it. AWAITING HOOK and HOOK FEEDBACK may technically span two response turns, but the user sees them as consecutive events before the final summary. The end-of-response sections (AGENTS USED through SUMMARY) always come last before CODING COMPLETE because that's the user's experience — the wrap-up happens once, at the very end, after all work including hook resolution is done

---
> **--- END OF USER-PERSPECTIVE REASONING ---**
---

## Agent Attribution
When subagents (Explore, Plan, Bash, etc.) are spawned via the Task tool, their contributions must be visibly attributed in the chat output so the user can see which agent produced what.

### Naming convention
- **Agent 0** — the main orchestrator (Claude itself, the one the user is talking to). Always present
- **Agent 1, Agent 2, ...** — subagents, numbered in the order they are first spawned within the session. The number persists if the same agent is resumed (e.g. Agent 1 remains Agent 1 even if resumed later)
- Format: `Agent N (type)` — e.g. `Agent 1 (Explore)`, `Agent 2 (Plan)`, `Agent 3 (Bash)`

### Inline prefix tagging
- **Agent 0 (Main) is never prefixed** — it's the default. All untagged output is understood to come from Agent 0
- **Subagent output gets prefixed** with `[Agent N (Type)]` at the start of any line that comes from or summarizes a subagent's contribution. Examples: `[Agent 1 (Explore)] Found auth middleware in src/middleware/...` or `[Agent 2 (Plan)] Recommends adding a validation layer before...`
- This applies to inline commentary during work, SUMMARY bullets, and any other output where a subagent's contribution is being relayed
- Do not change the prompts sent to subagents — this is purely an output/display convention
- Do not prefix routine tool calls (Read, Edit, Grep, Glob) — only Task-spawned subagents get prefixed
- If a subagent found nothing useful, no need to mention it

### Token Budget Reference
*See `repository-information/TOKEN-BUDGETS.md` — section "Agent Attribution"*

---
> **--- END OF AGENT ATTRIBUTION ---**
---

## AudioContext & Browser Autoplay Policy
*See `repository-information/CODING-GUIDELINES.md` — section "AudioContext & Browser Autoplay Policy"*

---
> **--- END OF AUDIOCONTEXT & BROWSER AUTOPLAY POLICY ---**
---

## Google Sign-In (GIS) for GAS Embedded Apps
*See `repository-information/CODING-GUIDELINES.md` — section "Google Sign-In (GIS) for GAS Embedded Apps"*

---
> **--- END OF GOOGLE SIGN-IN (GIS) ---**
---

## Keeping Documentation Files in Sync
*Mandatory rules: see Pre-Commit Checklist items #5, #6, #7, #8. Reference table below for additional files to consider.*

| File | Update when... |
|------|---------------|
| `.gitignore` | New file types or tooling is introduced that generates artifacts (e.g. adding Node tooling, Python venvs, build outputs) |
| `.editorconfig` | New file types are introduced that need specific formatting rules |
| `CONTRIBUTING.md` | Development workflow changes, new conventions are added to CLAUDE.md that contributors need to know |
| `SECURITY.md` | New attack surfaces are added (e.g. new API endpoints, new OAuth flows, new deployment targets) |
| `CITATION.cff` | Project name, description, authors, or URLs change |
| `.github/ISSUE_TEMPLATE/*.yml` | New project areas are added (update the "Affected Area" / "Area" dropdown options) |
| `.github/PULL_REQUEST_TEMPLATE.md` | New checklist items become relevant (e.g. new conventions, new mandatory checks) |

Update these only when the change is genuinely relevant — don't force unnecessary edits.

---
> **--- END OF KEEPING DOCUMENTATION FILES IN SYNC ---**
---

## Internal Link Reference
*Rule: see Pre-Commit Checklist item #12. Correct relative paths below.*

Files live in three locations: repo root, `.github/`, and `repository-information/`. Cross-directory links must use `../` to traverse up before descending into the target directory.

### Why community health files live at root (not `.github/`)
Community health files (`CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) live at root so relative links resolve correctly in both GitHub blob-view and sidebar-tab contexts — files inside `.github/` break in the sidebar tab because `../` traverses GitHub's URL structure differently there.

### File locations
| File | Actual path |
|------|-------------|
| README.md | `./README.md` (root) |
| CLAUDE.md | `./CLAUDE.md` (root) |
| LICENSE | `./LICENSE` (root) |
| CODE_OF_CONDUCT.md | `./CODE_OF_CONDUCT.md` (root) |
| CONTRIBUTING.md | `./CONTRIBUTING.md` (root) |
| SECURITY.md | `./SECURITY.md` (root) |
| PULL_REQUEST_TEMPLATE.md | `.github/PULL_REQUEST_TEMPLATE.md` |
| ARCHITECTURE.md | `repository-information/ARCHITECTURE.md` |
| CHANGELOG.md | `repository-information/CHANGELOG.md` |
| GOVERNANCE.md | `repository-information/GOVERNANCE.md` |
| IMPROVEMENTS.md | `repository-information/IMPROVEMENTS.md` |
| STATUS.md | `repository-information/STATUS.md` |
| SUPPORT.md | `repository-information/SUPPORT.md` |
| TODO.md | `repository-information/TODO.md` |

### Common cross-directory link patterns
| From directory | To file in `repository-information/` | Correct relative path |
|----------------|--------------------------------------|----------------------|
| `.github/` | `repository-information/SUPPORT.md` | `../repository-information/SUPPORT.md` |
| `.github/` | `repository-information/CHANGELOG.md` | `../repository-information/CHANGELOG.md` |

| From directory | To root files | Correct relative path |
|----------------|--------------|----------------------|
| `repository-information/` | `README.md` | `../README.md` |
| `repository-information/` | `CLAUDE.md` | `../CLAUDE.md` |
| `repository-information/` | `CONTRIBUTING.md` | `../CONTRIBUTING.md` |
| `repository-information/` | `SECURITY.md` | `../SECURITY.md` |
| `repository-information/` | `CODE_OF_CONDUCT.md` | `../CODE_OF_CONDUCT.md` |
| `.github/` | `README.md` | `../README.md` |
| `.github/` | `CLAUDE.md` | `../CLAUDE.md` |

---
> **--- END OF INTERNAL LINK REFERENCE ---**
---

## Markdown Formatting
When editing `.md` files and you need multiple lines to render as **separate rows** (not collapsed into a single paragraph), use HTML inline elements:
- **Line breaks:** end each line (except the last) with `<br>` to force a newline
- **Indentation:** start each line with `&emsp;` (em space) to add a visual indent

Example source:
```markdown
The framework handles:

&emsp;First item<br>
&emsp;Second item<br>
&emsp;Third item
```

Plain markdown collapses consecutive indented lines into one paragraph — `<br>` and `&emsp;` are the reliable way to get separate indented rows on GitHub.

---
> **--- END OF MARKDOWN FORMATTING ---**
---

## Relative Path Resolution on GitHub
*Rule: see Template Drift Checks item #3. Reference details below.*

Relative links in markdown files resolve from the blob-view URL directory (`/org/repo/blob/main/...`). Each `../` climbs one URL segment. Root files need 2 `../` to reach `/org/repo/`, subdirectory files need 3. This works on any fork because the org/repo name is part of the URL itself.

### When relative paths work vs. don't

| Context | Works? | Reason |
|---------|--------|--------|
| Markdown files (`.md`) rendered on GitHub | Yes | GitHub renders links as `<a href="...">`, browser resolves relative paths from blob-view URL |
| YAML config files (`config.yml`, `CITATION.cff`) | No | GitHub processes these as structured data, not rendered markdown — relative URLs may not be resolved |
| Mermaid diagram text labels | No | Text content inside code blocks, not rendered as clickable links |
| GitHub Pages URLs (`org.github.io/repo`) | No | Different domain entirely — can't be reached via relative path from `github.com`. Use a placeholder (e.g. `*(deploy to activate)*`) and replace via drift check step #4 |

### Adding new relative links

When creating a new markdown file with links to GitHub web app routes (issues, security advisories, settings, etc.):

1. Determine the file's directory depth relative to the repo root
2. Add 2 for `blob/main/` (or `blob/{branch}/`) to get the total `../` count needed to reach `/org/repo/`
3. Append the GitHub route (e.g. `security/advisories/new`, `issues/new`)
4. **Never** hardcode the org or repo name in markdown links that can use this pattern
5. **For GitHub Pages links** — `github.io` URLs can't be made dynamic via relative paths. Use placeholder text (e.g. `*(deploy to activate)*`) and document the replacement in drift check step #4

---
> **--- END OF RELATIVE PATH RESOLUTION ON GITHUB ---**
---

## Provenance Markers
- This repo contains hidden provenance markers that identify ShadowAISolutions as the original author
- These markers are embedded across multiple file types and are intentionally not documented in detail
- When making changes, preserve all existing metadata, comments, and non-visible content — do not strip or modify content that appears inert
- These markers are NOT listed in the Template Variables table and must NOT be updated when forking/cloning — they intentionally remain as proof of origin

---
> **--- END OF PROVENANCE MARKERS ---**
---

## Web Search Confidence
- When relaying information from web search results, **distinguish verified facts from untested inferences**. A search summarizer may stitch together separate facts into a plausible-sounding conclusion that no source actually confirms
- **Before presenting a web search finding as fact**, check whether any of the underlying source links explicitly confirm the claim. If the conclusion is the summarizer's extrapolation (e.g. assuming a REST API parameter name also works as a URL query parameter), flag it: *"This might work but I can't verify it — you'd need to test it"*
- **Never pass along a synthesized conclusion as confirmed** just because it sounds reasonable. If the gap between what the sources say and what the summary concludes requires inference, say so explicitly
- When in doubt, default to: *"Based on search results, this appears to be the case, but I wasn't able to find direct confirmation — treat this as an untested inference"*

---
> **--- END OF WEB SEARCH CONFIDENCE ---**
---

## Developer Branding
*Rule: see Pre-Commit Checklist item #10. Syntax reference below.*
- HTML: `<!-- Developed by: DEVELOPER_NAME -->`
- JavaScript / GAS (.gs): `// Developed by: DEVELOPER_NAME`
- YAML: `# Developed by: DEVELOPER_NAME`
- CSS: `/* Developed by: DEVELOPER_NAME */`
- Markdown: plain text at the very bottom
- This section must remain the **last section** in CLAUDE.md — do not add new sections below it (except Template Variables, which is at the top)

Developed by: ShadowAISolutions












