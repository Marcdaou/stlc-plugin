---
name: stlc
description: Orchestrates the AI-native STLC loop one approved stage at a time - Plan (highest-priority To Do Jira ticket in the sprint -> intent.md), Design (real Figma screens + spec.md), Build (code + unit tests in a separate git worktree), Deploy (Vercel PREVIEW for the user to test) and PR (pull request for the engineering team). Use whenever the user types /stlc, says "run the STLC", "pick up the next ticket", "next Jira story", "take a ticket from Jira to PR", or wants to move a sprint ticket through plan/design/build/deploy/PR, even if they only name one stage.
argument-hint: "[sprint name, default 'Sprint 0'] [ticket key to resume]"
---

# /stlc orchestrator

You run five stages. Each one is done by its own specialized subagent from this plugin. **You** own the human approval gates: subagents can't ask the user anything, so after each stage you relay its report, show its artifacts and links, and **wait for explicit approval** before you launch the next one. If the user asks for changes, relaunch the same agent with their feedback (in revise mode) and ask again.

| # | Stage  | Agent (`subagent_type`) | Main MCPs / tools | Output |
|---|--------|-------------------------|-------------------|--------|
| 1 | Plan   | `stlc:plan-agent`   | Atlassian (Jira) | `intent.md` |
| 2 | Design | `stlc:design-agent` | Figma, Atlassian | Figma file + `spec.md` |
| 3 | Build  | `stlc:build-agent`  | git worktree, npm, Vitest, Atlassian | code + tests, `plan.md`, `test-report.md`, local commit |
| 4 | Deploy | `stlc:deploy-agent` | git push, Vercel, browser, Atlassian | preview URL |
| 5 | PR     | `stlc:pr-agent`     | `gh`, Atlassian | PR URL |

## Shared conventions (pass these to every agent)
- **Work folder:** `docs/stlc/<TICKET-KEY>/` holds `intent.md`, `spec.md`, `plan.md`, `test-report.md` and `status.md` (stage, links, approvals with timestamps). Until Build it lives in the main checkout. Build moves it into the worktree so it gets committed with the feature.
- **Worktree:** `../worktrees/<key-lower>` on branch `feat/<key-lower>-<slug>`.
- **Sprint:** taken from the argument, defaulting to `Sprint 0`.
- **Project rules:** each agent reads the repo's `CLAUDE.md` (and any style guide it points to) and follows it, including logging rules and UI rules.
- **Never:** deploy to production, merge the PR, or push to `main`.

## Running a stage
Brief the agent with: the stage, the repo root path, the ticket key (once known), the work-folder path, the worktree path (from Build on), any user feedback and the mode (`run` or `revise`). Ask it to end with a **stage report**: what it did, the artifacts with paths/links, the external writes it made (Jira/Figma/Vercel/GitHub, with links), and open questions.

After each agent returns:
1. Verify the main claims yourself: the files exist, the links are present and the build/test numbers are in the test report. Don't relay unverified success.
2. Update `status.md`.
3. Show the user a short summary, the links and the gate question:
   - Plan → "Approve intent?"
   - Design → "Approve design?" (include the Figma screenshots/links)
   - Build → "Approve build for preview deploy?" (include the diff summary and test results)
   - Deploy → "Test the preview at <url>. Approve, or report bugs?" Bugs → relaunch build-agent in revise mode (fix + regression test), then deploy-agent again.
   - PR → report the PR URL. Done.

## Resuming
If the user gives a ticket key, or `docs/stlc/*/status.md` exists in the main checkout or `../worktrees/*`, resume at the first stage that hasn't been approved.

## Failure handling
If an agent reports a missing or unauthorized MCP (Atlassian, Figma, Vercel) or a logged-out `gh`, stop and tell the user exactly which one. Don't swap in another tool silently. If there's no To Do ticket in the sprint, say so and stop.
