# webforge

Claude Code skill: a 5-agent website build and hardening crew driven by three rulebooks (web and Supabase security, UX laws and HCI frameworks, pre-launch checklist).

1. **Planner-Orchestrator** builds a requirement matrix and file-ownership plan. Nothing is built until you approve it.
2. **Security Builder**, 3. **UX Builder**, 4. **Launch Builder** run in parallel, one rulebook each.
5. **QC Agent** verifies every requirement in the real code against the approved plan, fixes gaps, and issues a SHIP / NOT-READY verdict.

## Install

```bash
git clone https://github.com/almanalaysay93-gif/webforge ~/.claude/skills/webforge
```

## Use

```
/webforge make this Next.js + Supabase app production ready
```

Run artifacts land in `<project>/.webforge/` (PLAN.md, builder reports, QC_REPORT.md).
