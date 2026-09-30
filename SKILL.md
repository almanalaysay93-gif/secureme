---
name: webforge
description: "5-agent website build and hardening crew driven by three rulebooks: web and Supabase security, UX laws and HCI frameworks, and the pre-launch checklist. Agent 1 plans and orchestrates (user must approve the plan before anything is built), agents 2-4 build in parallel (security, UX, launch), agent 5 is final QC that verifies every requirement against the approved plan and fixes gaps. Use whenever the user types /webforge, or asks to build, harden, audit-and-fix, or make launch-ready a website or web app against security rules, UX laws, or a pre-launch checklist, e.g. 'make this site production ready', 'apply the security plan and launch checklist', 'build my landing page with the 3 rulebooks', 'run the 5 agents with plan approval and QC'. Prefer /buildme for pure visual-design craft; prefer this skill when compliance with the security, UX-law, and launch checklists is the goal."
metadata:
  slash_command: /webforge
  triggers: /webforge, webforge, web forge, security + ux + launch checklist build, production-ready hardening
---

# WebForge

Five agents turn three rulebooks into a verified build:

| # | Agent | Rulebook | Job |
|---|-------|----------|-----|
| 1 | Planner-Orchestrator | all 3 | Inspect project, build requirement matrix, assign owners and files, define waves. Writes no code. |
| 2 | Security Builder | `references/security.md` | Implement security items it owns. |
| 3 | UX Builder | `references/ux-laws.md` | Implement UX-law items it owns. |
| 4 | Launch Builder | `references/launch-checklist.md` | Implement launch-checklist items it owns. |
| 5 | QC Agent | all 3 + approved plan | Verify every ID in real code, apply handoffs, fix gaps, final report. |

`<skill-dir>` below = the base directory shown when this skill loaded. Pass absolute paths to subagents; they start cold and know nothing about this conversation.

## Why the flow looks like this

- **Subagents cannot talk to the user.** Their output only reaches you. So the approval gate lives in the main thread: agent 1 drafts, you present, the user decides. You then dispatch agents 2-5 exactly as the approved plan says. You are agent 1's hands for dispatch, not a second planner.
- **One owner per requirement, one owner per file.** The three rulebooks overlap (rate limiting, secrets, upload limits, UX laws appear in two docs each). Without a single owner, two builders implement the same thing twice or overwrite each other's edits when running in parallel.
- **QC checks code, not reports.** Builder reports say what they intended. QC re-verifies each requirement with its "done-when" check against the actual files.

## Workspace

All run artifacts go in `<project>/.webforge/`:

```
.webforge/
  PLAN.md              agent 1 output, status DRAFT -> APPROVED
  reports/security.md  agent 2 report
  reports/ux.md        agent 3 report
  reports/launch.md    agent 4 report
  handoffs/<domain>.md edits a builder needed in a file it does not own
  QC_REPORT.md         agent 5 final verdict
```

Add `.webforge/` to `.gitignore` if the project is a git repo and it is not listed.

## Phase 0: Intake

1. Target = current working directory unless the user names another path.
2. Need a brief: what is being built or hardened. If the user's message has none and the target has no app code, ask one question: what should the site be and do. If an app exists and no brief is given, default brief = "harden and make launch-ready as-is".
3. If `.webforge/PLAN.md` already exists with status APPROVED, ask whether to resume (skip to the first unfinished phase) or re-plan.

## Phase 1: Plan (Agent 1)

Spawn one `general-purpose` agent:

```
You are Agent 1, the WebForge Planner-Orchestrator.
Read and follow: <skill-dir>/agents/planner.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Project root: <abs project path>
User brief: <brief, verbatim>
Write the plan to: <project>/.webforge/PLAN.md
Return: a summary of at most 25 lines (counts per owner and action, dependencies to add, manual user actions, open questions).
```

Keep the agent's id so revisions can go back to it with context intact.

## Phase 2: Approval gate (hard stop)

Nothing gets built until the user explicitly approves. This is the user's explicit requirement for the skill, so do not treat silence, "looks interesting", or a question as approval.

1. Show the user the planner's summary plus a link to `.webforge/PLAN.md`. Surface open questions and the dependency list (new packages are supply-chain decisions, the user signs off on each).
2. Ask with AskUserQuestion: **Approve** / **Revise** (user says what) / **Cancel**.
3. Revise: send the feedback to agent 1 with SendMessage (fall back to a fresh planner spawn with the feedback plus the existing PLAN.md if the agent is gone). Return to step 1. Loop as many times as needed.
4. Approve: set `Status: APPROVED <YYYY-MM-DD>` in PLAN.md. From here the plan is the contract; builders and QC treat it as the scope.

## Phase 3: Build (Agents 2-4)

1. Install the approved dependency list once, before spawning builders (parallel agents editing `package.json` race each other). Use the project's package manager (lockfile tells you which).
2. Run the waves in PLAN.md. Default for an existing app: one wave, all three builders spawned in the same message so they run in parallel. Greenfield: wave 1 = Launch Builder alone (scaffolds the app), wave 2 = Security + UX in parallel.
3. Each builder is a `general-purpose` agent:

```
You are Agent <2|3|4>, the WebForge <Security|UX|Launch> Builder.
Read and follow: <skill-dir>/agents/builder.md (your domain section: <SECURITY|UX|LAUNCH>)
Your rulebook: <skill-dir>/references/<security|ux-laws|launch-checklist>.md
Approved plan: <project>/.webforge/PLAN.md
Your owner code in the plan: <SEC|UX|LCH>
Project root: <abs project path>
Report to: <project>/.webforge/reports/<security|ux|launch>.md
Handoffs to: <project>/.webforge/handoffs/<security|ux|launch>.md
Return: counts of DONE / PARTIAL / BLOCKED / SKIPPED(N-A) and any blocker in at most 10 lines.
```

4. If a builder returns BLOCKED items that need a user decision, collect them and ask the user after all builders in the wave finish, not mid-wave.

## Phase 4: QC (Agent 5)

Spawn after every build wave finishes:

```
You are Agent 5, the WebForge QC Agent and final quality gate.
Read and follow: <skill-dir>/agents/qc.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Approved plan: <project>/.webforge/PLAN.md
Builder reports: <project>/.webforge/reports/
Handoffs: <project>/.webforge/handoffs/
Project root: <abs project path>
Write: <project>/.webforge/QC_REPORT.md
Return: verdict (SHIP / SHIP-WITH-MANUAL-ITEMS / NOT-READY), counts per status, and every FAIL or FLAG in at most 20 lines.
```

If QC returns NOT-READY with fixable items it could not finish (ran out of room, needed a decision), resolve the decision with the user, then send QC back via SendMessage for one more pass. Two QC passes max; after that, report what remains.

## Phase 5: Report

Tell the user, briefly:
- Verdict and counts (PASS / FIXED-BY-QC / MANUAL / N-A / FAIL).
- Manual actions they must do themselves (dashboards, credential rotation, Search Console, uptime monitor, spending caps), copied from QC_REPORT.md.
- Links to `.webforge/QC_REPORT.md` and `.webforge/PLAN.md`.

Then append a short dated entry (date, brief, verdict, open items) to `PROJECT_MEMORY.md` at the project root, creating it if missing, so other agents see the run.

## Guardrails for every agent

These come from the rulebooks and from the fact that agents act on the user's real project:
- Never print, commit, or move real secrets. Leaked credentials go in the report as "rotate in dashboard", never rotated or pasted by an agent.
- No git history rewriting (`git-filter-repo`, force push), no commits, no pushes, no deploys unless the user asks. Scanning history with `gitleaks` is fine.
- Dashboard-only settings (Supabase Auth redirect URLs, leaked-password protection, billing caps) are MANUAL items for the user, not something to automate through a browser.
- Stay inside the approved plan. A needed change outside it becomes a FLAG for the user, not a silent expansion.
