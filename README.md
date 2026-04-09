# Playwright Test Skill

Automated QA testing for GitHub PR preview deployments using [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills and [Playwright MCP](https://github.com/anthropics/mcp-server-playwright).

## What it does

1. Reads test credentials from `.env` (never asks the user)
2. Fetches the latest preview/deployment URL from the PR via GitHub MCP
3. Optionally fetches Jira ticket context to focus testing on acceptance criteria
4. Logs in and runs test scenarios autonomously using Playwright
5. Produces a structured test report with screenshots

## Skills

| Command | Model | Use case |
|---------|-------|----------|
| `/test-pr-fast <PR> [JIRA-123]` | Haiku | Quick smoke tests (3-5 scenarios) |
| `/test-pr <PR> [JIRA-123]` | Sonnet | Standard testing (happy path + edge cases) |
| `/test-pr-thorough <PR> [JIRA-123]` | Opus | Deep testing (full coverage, responsive, accessibility) |

**PR** can be a number, URL, or `owner/repo/pull/number` format. The Jira ticket is optional and used to focus tests on the described requirements.

## Setup

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Node.js (for the Playwright MCP server)
- GitHub MCP server (built-in to Claude Code or configured separately)
- Jira/Atlassian MCP server (optional)

### Install

```bash
git clone https://github.com/BuildHero/playwright-test-skill.git
cd playwright-test-skill

cp .env.example .env
# Edit .env with your test credentials
```

### Configuration

The `.mcp.json` file configures the Playwright MCP server:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-playwright"]
    }
  }
}
```

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `TEST_USER_EMAIL` | Yes | Login email for the test user |
| `TEST_USER_PASSWORD` | Yes | Login password for the test user |
| `BUILDOPS_BASE_URL` | No | Fallback URL if no preview URL is found in the PR |

## Usage

Open Claude Code in this project directory and run a skill:

```
/test-pr 42
/test-pr https://github.com/org/repo/pull/42
/test-pr-fast 42 PROJ-1234
/test-pr-thorough 42 PROJ-1234
```

## How the skills differ

- **Fast** (Haiku) -- Happy-path smoke tests only, 3-5 scenarios, minimal regression checks.
- **Standard** (Sonnet) -- Happy path + edge cases, form validation, console error checks, and basic regression.
- **Thorough** (Opus) -- All of the above plus responsive testing, accessibility checks, network request analysis, and full Jira acceptance criteria coverage.

## Project Structure

```
.
├── CLAUDE.md                           # Claude Code project instructions
├── .claude/
│   ├── settings.json                   # Permission settings
│   └── skills/
│       ├── test-pr/SKILL.md            # Standard test skill
│       ├── test-pr-fast/SKILL.md       # Fast test skill
│       └── test-pr-thorough/SKILL.md   # Thorough test skill
├── .mcp.json                           # Playwright MCP server config
├── .env.example                        # Environment variable template
└── .gitignore
```
