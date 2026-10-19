---
name: webforge
description: "Production website build, hardening, and launch-readiness crew driven by security, resilience, performance, observability, SEO, UX/accessibility, and content/compliance checklists. Agent 1 plans and orchestrates (user must approve before code changes), agents 2-8 implement disjoint specialist work in parallel, and agent 9 verifies every requirement against the approved plan and fixes gaps. Use for /webforge, production hardening, pre-launch audits, or making a website/app launch-ready."
metadata:
  slash_command: /webforge
  triggers: /webforge, webforge, web forge, production-ready hardening, launch checklist, website pre-launch audit
---

# WebForge

WebForge turns the launch checklist into an approved, parallel build plan and a verified release gate.

## Agent roster

| # | Agent | Owner code | Scope |
|---|---|---|---|
| 1 | Planner-Orchestrator | PLAN | Inspect the project, build the requirement matrix, assign files, define waves, and write no application code. |
| 2 | Security Builder | SEC | Secrets, HTTPS, rate limiting, uploads, bot protection, destructive-action confirmations, auth/database security, CSP, and payment idempotency. |
| 3 | API & Resilience Builder | API | Spending/concurrency controls, timeouts, retries, caching, duplicate-submission protection, and resilient external-service boundaries. |
| 4 | Data & Performance Builder | DATA | Query shape, indexes, pagination, compression, asset delivery, load/concurrency test setup, and Core Web Vitals implementation support. |
| 5 | Observability & Recovery Builder | OBS | Error tracking, uptime integration points, error boundaries, empty/error states, backups, restore drills, and operational runbooks. |
| 6 | Technical SEO Builder | SEO | Sitemap, robots, canonicals, metadata, social cards, schema, headings, image SEO, URL hygiene, broken links, analytics/Search Console handoffs, and favicon package. |
| 7 | UX & Accessibility Builder | UX | UX laws, responsive behavior, dark mode, navigation, focus/hover states, loading states, skip links, contrast, CTA hierarchy, and mobile interaction quality. |
| 8 | Content, Forms & Compliance Builder | CONTENT | Search, FAQ, newsletters, validation, password visibility, contact access, 404, legal-page scaffolding, cookie consent, print styles, UTM, clipboard, and update timestamps. |
| 9 | Quality Gate Agent | QC | Apply handoffs, run project checks, verify every matrix row against real code, fix in-scope gaps, detect conflicts/scope creep, and issue the release verdict. |

`<skill-dir>` is the directory containing this file. Pass absolute paths to subagents; they start cold.

## Non-negotiable operating rules

- Nothing is built until the user explicitly approves `.webforge/PLAN.md`.
- One owner per requirement and one owner per file. Overlaps are represented as `DUP -> <canonical ID>`.
- QC checks code and runtime evidence, not builder reports.
- Builders only implement `BUILD` rows assigned to them. `VERIFY`, `MANUAL`, `N-A`, and `DUP` rows are not silently expanded.
- Dashboard-only work—billing caps, uptime alerts, Search Console verification, analytics accounts, backup settings, credential rotation, and outreach—is `MANUAL`.
- Never print, copy, rotate, commit, push, deploy, or move real secrets. No history rewriting.

## Run artifacts

All artifacts live in `<project>/.webforge/`:

```
.webforge/
  PLAN.md
  reports/{security,api,data,observability,seo,ux,content}.md
  handoffs/{security,api,data,observability,seo,ux,content}.md
  QC_REPORT.md
```

Add `.webforge/` to `.gitignore` when appropriate. Do not add generated reports to the application bundle.

## Phase 0: Intake

1. Target the current working directory unless the user names another path.
2. Use the user's brief verbatim. If an app exists without a brief, use: `harden and make launch-ready as-is`. If there is no app and no brief, ask what the site should do.
3. If an approved plan already exists, ask whether to resume or re-plan.
4. Treat the supplied `references/launch-checklist.md` as the canonical checklist for the run.

## Phase 1: Plan (Agent 1)

Spawn one general-purpose agent with:

```
You are Agent 1, the WebForge Planner-Orchestrator.
Read: <skill-dir>/agents/planner.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Project root: <absolute project path>
User brief: <brief, verbatim>
Write: <project>/.webforge/PLAN.md
Return: at most 30 lines with counts by owner/action, dependencies, manual actions, risks, and open questions.
```

The planner must inspect the actual stack, existing implementation, auth/database/payments/uploads, package manager, routes, and deployment assumptions. It must use atomic IDs, objective done-when checks, and explicit ownership.

## Phase 2: Approval gate

Show the planner summary, link to `.webforge/PLAN.md`, dependencies, manual actions, and open questions. Ask **Approve**, **Revise**, or **Cancel**. Silence or questions are not approval. On revision, send feedback to Agent 1 or respawn it with the existing plan. On approval, set `Status: APPROVED <YYYY-MM-DD>` in the plan. The approved plan is the contract.

## Phase 3: Build waves (Agents 2-8)

Install only approved dependencies once before builders start. Follow the waves in the plan; for an existing app, default to all seven builders in parallel. For greenfield work, default to SEO/CONTENT or the planned scaffold owner first, then run the remaining builders after the scaffold exists. Each builder receives:

```
You are Agent <2-8>, the WebForge <domain> Builder.
Read: <skill-dir>/agents/builder.md and <skill-dir>/agents/<domain>.md
Rulebook: <skill-dir>/references/<relevant rulebook>.md
Approved plan: <project>/.webforge/PLAN.md
Owner code: <SEC|API|DATA|OBS|SEO|UX|CONTENT>
Project root: <absolute project path>
Report: <project>/.webforge/reports/<domain>.md
Handoffs: <project>/.webforge/handoffs/<domain>.md
Return: DONE/PARTIAL/BLOCKED/SKIPPED counts and blockers in at most 12 lines.
```

If a builder needs another owner's file, it writes a precise handoff instead of editing it. Ask the user about decision-blocking items only after the current wave completes.

## Phase 4: Quality gate (Agent 9)

Spawn QC after each build wave, and always after the final wave:

```
You are Agent 9, the WebForge Quality Gate Agent.
Read: <skill-dir>/agents/qc.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Approved plan: <project>/.webforge/PLAN.md
Builder reports: <project>/.webforge/reports/
Handoffs: <project>/.webforge/handoffs/
Project root: <absolute project path>
Write: <project>/.webforge/QC_REPORT.md
Return: SHIP / SHIP-WITH-MANUAL-ITEMS / NOT-READY, status counts, and every FAIL/FLAG in at most 25 lines.
```

QC may fix only gaps inside the approved plan. It gets at most two passes; after that report unresolved items honestly.

## Phase 5: Report and memory

Tell the user the verdict, counts for PASS/FIXED/MANUAL/N-A/FAIL/FLAG/UNVERIFIED, and the manual actions copied from QC_REPORT.md. Link to `.webforge/PLAN.md` and `.webforge/QC_REPORT.md`. Append a dated brief, verdict, and open-items entry to `PROJECT_MEMORY.md`.
