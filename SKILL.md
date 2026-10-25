---
name: secureme
description: "Production website build, hardening, and launch-readiness crew driven by security, resilience, performance, observability, SEO, UX/accessibility, and content/compliance checklists. Agent 1 plans and orchestrates (user must approve before code changes), agents 2-8 implement disjoint specialist work in parallel, agent 9 verifies every requirement against the approved plan and fixes gaps, and an independent bundled Cloudflare audit feeds planning and the final release gate. Use for /secureme, production hardening, pre-launch audits, or making a website/app launch-ready."
metadata:
  slash_command: /secureme
  triggers: /secureme, secureme, secure website hardening, production-ready hardening, launch checklist, website pre-launch audit
---

# SecureMe

SecureMe turns the launch checklist and independent Cloudflare audit findings into an approved, parallel build plan and a verified release gate. Cloudflare audits the target application before planning and after all implementation/QC changes; SecureMe builders remediate approved findings.

Read [Cloudflare integration](references/cloudflare-audit.md) and the bundled [audit skill](vendor/cloudflare-security-audit/SKILL.md). The lead runs its complete six-phase workflow at each audit stage, following its companion guides and independent verification rules. This is an integrated stage, not an optional suggestion to invoke another skill.

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
| — | Independent Cloudflare audit | CFA | Lead-coordinated reconnaissance, hunting, fresh verification, validated findings and coverage; read-only application access. |
| 9 | Quality Gate Agent | QC | Apply handoffs, run project checks, verify every matrix row against real code, fix in-scope gaps, detect conflicts/scope creep, and issue the release verdict. |

`<skill-dir>` is the directory containing this file. Pass absolute paths to subagents; they start cold.

## Non-negotiable operating rules

- Nothing is built until the user explicitly approves `.secureme/PLAN.md`.
- One owner per requirement and one owner per file. Overlaps are represented as `DUP -> <canonical ID>`.
- QC checks code and runtime evidence, not builder reports.
- Audit stages never overlap source mutations. Auditors do not fix application code; builders cannot independently verify their own fixes.
- A final audit must match the final source, pass both bundled validators, and satisfy the release rules in `references/cloudflare-audit.md`. Missing or incomplete audit evidence never becomes a clean security pass.
- Builders only implement `BUILD` rows assigned to them. `VERIFY`, `MANUAL`, `N-A`, and `DUP` rows are not silently expanded.
- Dashboard-only work—billing caps, uptime alerts, Search Console verification, analytics accounts, backup settings, credential rotation, and outreach—is `MANUAL`.
- Never print, copy, rotate, commit, push, deploy, or move real secrets. No history rewriting.

## Run artifacts

SecureMe orchestration artifacts live in `<project>/.secureme/`. Cloudflare raw audit artifacts live in separate external run directories by default; see `references/cloudflare-audit.md` for the explicit opt-in exception:

```
.secureme/
  PLAN.md
  AUDIT.md
  reports/{security,api,data,observability,seo,ux,content}.md
  handoffs/{security,api,data,observability,seo,ux,content}.md
  QC_REPORT.md
```

Add `.secureme/` to `.gitignore` when appropriate. Do not add generated reports to the application bundle.

## Phase 0: Intake

1. Target the current working directory unless the user names another path.
2. Use the user's brief verbatim. If an app exists without a brief, use: `harden and make launch-ready as-is`. If there is no app and no brief, ask what the site should do.
3. If an approved plan already exists, ask whether to resume or re-plan.
4. Treat the supplied `references/launch-checklist.md` as the canonical checklist for the run.
5. Read `references/cloudflare-audit.md`; check audit prerequisites and budgets, select external output paths, and initialize `.secureme/AUDIT.md`. Explain the integrated audit stages and bundled skill to the user.

## Phase 0.5: Independent baseline audit

For an existing application, the lead runs the full bundled Cloudflare audit now, with source writes paused. Follow `references/cloudflare-audit.md`; record artifacts, validator results, source identity, fingerprint mappings, and blockers in `.secureme/AUDIT.md`. With no application source, defer only this baseline and retain the mandatory final audit. Planning may proceed with explicit audit capability gaps; never pretend a blocked audit completed.

## Phase 1: Plan (Agent 1)

Spawn one general-purpose agent with:

```
You are Agent 1, the SecureMe Planner-Orchestrator.
Read: <skill-dir>/agents/planner.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Project root: <absolute project path>
User brief: <brief, verbatim>
Audit adapter: <skill-dir>/references/cloudflare-audit.md
Audit input: <project>/.secureme/AUDIT.md and its linked baseline findings, coverage ledger, metadata, and reports (or explicit baseline deferral/blocker)
Write: <project>/.secureme/PLAN.md
Return: at most 30 lines with counts by owner/action, dependencies, manual actions, risks, and open questions.
```

The planner must inspect the actual stack, existing implementation, auth/database/payments/uploads, package manager, routes, and deployment assumptions. It must use atomic IDs, objective done-when checks, and explicit ownership. Include Cloudflare findings with stable CFA IDs, fingerprints, one remediation owner, deduplicated plan links, and exact validation blockers. Preserve unresolved findings through approval and QC.

## Phase 2: Approval gate

Show the planner summary, link to `.secureme/PLAN.md` and `.secureme/AUDIT.md`, audit findings/coverage gaps, dependencies, manual actions, and open questions. Ask **Approve**, **Revise**, or **Cancel**. Silence or questions are not approval. On revision, send feedback to Agent 1 or respawn it with the existing plan. On approval, set `Status: APPROVED <YYYY-MM-DD>` in the plan. The approved plan is the contract.

## Phase 3: Build waves (Agents 2-8)

Install only approved dependencies once before builders start. Follow the waves in the plan; for an existing app, default to all seven builders in parallel. For greenfield work, default to SEO/CONTENT or the planned scaffold owner first, then run the remaining builders after the scaffold exists. Each builder receives:

```
You are Agent <2-8>, the SecureMe <domain> Builder.
Read: <skill-dir>/agents/builder.md and <skill-dir>/agents/<domain>.md
Rulebook: <skill-dir>/references/<relevant rulebook>.md
Approved plan: <project>/.secureme/PLAN.md
Mapped audit evidence: <project>/.secureme/AUDIT.md and approved finding rows for this owner
Owner code: <SEC|API|DATA|OBS|SEO|UX|CONTENT>
Project root: <absolute project path>
Report: <project>/.secureme/reports/<domain>.md
Handoffs: <project>/.secureme/handoffs/<domain>.md
Return: DONE/PARTIAL/BLOCKED/SKIPPED counts and blockers in at most 12 lines.
```

If a builder needs another owner's file, it writes a precise handoff instead of editing it. Ask the user about decision-blocking items only after the current wave completes.

## Phase 4: Quality gate (Agent 9)

Spawn QC after each build wave, and always after the final wave. Interim QC is provisional. After QC finishes handoffs and all in-plan fixes, the lead runs the independent final audit described below, then resumes QC in read-only verdict mode:

```
You are Agent 9, the SecureMe Quality Gate Agent.
Read: <skill-dir>/agents/qc.md
Rulebooks: <skill-dir>/references/security.md, <skill-dir>/references/ux-laws.md, <skill-dir>/references/launch-checklist.md
Approved plan: <project>/.secureme/PLAN.md
Builder reports: <project>/.secureme/reports/
Handoffs: <project>/.secureme/handoffs/
Audit adapter: <skill-dir>/references/cloudflare-audit.md
Audit state and linked evidence: <project>/.secureme/AUDIT.md
Mode: <prepare/fix | final read-only verdict>
Project root: <absolute project path>
Write: <project>/.secureme/QC_REPORT.md
Return: SHIP / SHIP-WITH-MANUAL-ITEMS / NOT-READY, status counts, and every FAIL/FLAG in at most 25 lines.
```

QC may fix only gaps inside the approved plan. It gets at most two mutation passes; after that report unresolved items honestly. Final read-only verdict calculation does not authorize a third fix pass.

## Phase 4.5: Independent final audit and release verdict

Pause builders and QC source writes. The lead runs a new full Cloudflare audit of the final application, using prior findings/ledgers and fresh independent verifiers as `references/cloudflare-audit.md` requires. Validate both findings and coverage, update `AUDIT.md`, and pass the resulting current-source evidence to QC for its final read-only verdict.

If QC still has a permitted fix pass, it may fix an in-plan finding and the lead must repeat the final audit afterward. New scope requires an approved plan amendment. Any source change makes previous final evidence stale. At the pass/budget limit, missing capabilities, or incomplete validation, report the exact gaps and apply the adapter release rules; never issue `SHIP` from stale or missing evidence.

## Phase 5: Report and memory

Tell the user the verdict, counts for PASS/FIXED/MANUAL/N-A/FAIL/FLAG/UNVERIFIED, and the manual actions copied from QC_REPORT.md. Link to `.secureme/PLAN.md`, `.secureme/AUDIT.md`, `.secureme/QC_REPORT.md`, and the final external audit report. Include audit completion, profile/scope, confirmed/unresolved counts, independent closure evidence, and remaining coverage/validation gaps; do not claim complete security from one run. Append a dated brief, verdict, and open-items entry to `PROJECT_MEMORY.md`.
