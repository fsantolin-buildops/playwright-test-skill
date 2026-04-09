# Playwright Test Skill

Automated QA testing for GitHub PR preview deployments using Playwright MCP.

## Skills

| Command | Model | Use case |
|---------|-------|----------|
| `/test-pr-fast <PR> [JIRA-123]` | Haiku | Quick smoke tests (3-5 scenarios) |
| `/test-pr <PR> [JIRA-123]` | Sonnet | Standard testing (happy path + edge cases) |
| `/test-pr-thorough <PR> [JIRA-123]` | Opus | Deep testing (full coverage, responsive, accessibility) |

## How it works

1. Reads test credentials from `.env` (never asks the user)
2. Fetches the **last** preview/deployment URL from the PR via GitHub MCP
3. Optionally fetches Jira ticket context via Jira MCP to focus testing
4. Logs in and runs test scenarios autonomously (never pauses to ask)
5. Produces a structured test report

## Setup

```bash
cp .env.example .env
# Edit .env — fill in TEST_USER_EMAIL and TEST_USER_PASSWORD
```

## Requirements

- Claude Code CLI with Playwright MCP (configured in `.mcp.json`)
- GitHub MCP server (via Claude Code built-in or configured MCP)
- Jira/Atlassian MCP server (optional — for Jira ticket context)
- Node.js (for npx to run the Playwright MCP server)
