# stlc: AI-native STLC plugin for Claude Code

`/stlc` takes the top Jira ticket all the way to a pull request, with **your approval between every stage**:

1. **Plan** (`plan-agent`): picks the highest-priority *To Do* ticket in the sprint (default `Sprint 0`) via the Jira MCP and writes `intent.md`.
2. **Design** (`design-agent`): draws real screens in a new Figma file on your own seat and writes `spec.md` (requirements + unit-test plan).
3. **Build** (`build-agent`): implements the spec in a separate **git worktree**, writes **unit tests** (Vitest) for every requirement and gets build + tests green.
4. **Deploy** (`deploy-agent`): pushes the branch and gets a **Vercel preview** (never production) for you to test.
5. **PR** (`pr-agent`): opens a GitHub PR for your engineering team and moves the ticket to In Review.

All stage docs live in `docs/stlc/<TICKET>/` and are committed with the feature.

## Install
```
/plugin marketplace add Marcdaou/stlc-plugin
/plugin install stlc@stlc-marketplace
```

## Requirements
- **Atlassian MCP** (Jira), authenticated
- **Figma MCP** (official Figma plugin) with a seat that can create design files
- **Vercel MCP** (`https://mcp.vercel.com`) and a linked project (`.vercel/project.json`)
- `git`, `gh` (logged in), Node/npm
- Optional: Playwright or Claude-in-Chrome MCP for preview smoke tests

## Usage
```
/stlc                 # Sprint 0, next ticket
/stlc "Sprint 1"      # another sprint
/stlc ORB-12          # resume a ticket
```
