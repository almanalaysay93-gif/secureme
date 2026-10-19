# SecureMe

SecureMe is a Claude Code skill for turning a website or web app into a production-ready, launch-checked build. It combines security, API resilience, data/performance, observability/recovery, technical SEO, UX/accessibility, and content/forms/compliance work.

The workflow has nine roles. A Planner-Orchestrator first inspects the project and writes an atomic requirement matrix; nothing is built until the user approves it. Seven specialist builders then work in owned file areas: Security, API & Resilience, Data & Performance, Observability & Recovery, Technical SEO, UX & Accessibility, and Content/Forms/Compliance. A Quality Gate Agent applies handoffs, runs project and runtime checks, verifies every requirement against the actual code, fixes in-scope gaps, and issues a SHIP, SHIP-WITH-MANUAL-ITEMS, or NOT-READY verdict.

## Install

```bash
git clone https://github.com/almanalaysay93-gif/secureme ~/.claude/skills/secureme
```

## Use

```text
/secureme make this Next.js + Supabase app production ready
```

The canonical launch checklist is `references/launch-checklist.md`. Run artifacts are written to the target project's `.secureme/` directory: `PLAN.md`, specialist reports and handoffs, and `QC_REPORT.md`. No agent commits, pushes, deploys, or writes real secrets.