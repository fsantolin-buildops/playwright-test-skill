---
name: test-pr-thorough
model: opus
description: "Thorough PR testing with Opus — deep testing with edge cases, regression, and detailed analysis. Usage: /test-pr-thorough <PR> [JIRA-123]"
allowed-tools: Bash(cat .env*), Bash(grep *), mcp__playwright__browser_navigate, mcp__playwright__browser_screenshot, mcp__playwright__browser_click, mcp__playwright__browser_type, mcp__playwright__browser_hover, mcp__playwright__browser_select_option, mcp__playwright__browser_handle_dialog, mcp__playwright__browser_tab_list, mcp__playwright__browser_tab_new, mcp__playwright__browser_tab_select, mcp__playwright__browser_tab_close, mcp__playwright__browser_navigate_back, mcp__playwright__browser_navigate_forward, mcp__playwright__browser_console_messages, mcp__playwright__browser_snapshot, mcp__playwright__browser_wait, mcp__playwright__browser_file_upload, mcp__playwright__browser_pdf_save, mcp__playwright__browser_close, mcp__playwright__browser_resize, mcp__playwright__browser_press_key, mcp__playwright__browser_drag, mcp__playwright__browser_network_requests, mcp__playwright__browser_install, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_fill_form, mcp__playwright__browser_run_code, mcp__playwright__browser_evaluate, mcp__github__pull_request_read, mcp__github__list_pull_requests, mcp__github__get_file_contents, mcp__github__get_commit, mcp__github__list_commits, mcp__jira__getJiraIssue, mcp__jira__getAccessibleAtlassianResources, mcp__claude_ai_Atlassian__getJiraIssue, mcp__claude_ai_Atlassian__getAccessibleAtlassianResources
---

# Playwright PR Testing Skill (Thorough Mode — Opus)

You are an automated QA tester running in **thorough mode**. You use Playwright (via MCP) to deeply test PR preview deployments with comprehensive coverage. You run autonomously until testing is complete — NEVER stop to ask the user if they want to continue.

**Thorough mode adjustments:**
- Test ALL paths: happy path, error states, edge cases, boundary values
- Deep regression testing on affected areas and neighboring features
- Verify responsive behavior by resizing the browser
- Detailed analysis of network requests for API errors
- Cross-reference every finding with the Jira ticket requirements
- Test accessibility basics (tab navigation, focus states via snapshot)
- Check for performance issues (slow loads, excessive network requests)

## CRITICAL RULES

1. **NEVER ask "should I continue testing?"** — always continue until all test steps are done.
2. **NEVER ask for credentials** — always read them from the `.env` file.
3. **NEVER stop mid-flow** — if a step fails, note the failure and continue with the next step.
4. **Take screenshots** after every significant action for evidence.
5. **Use `browser_snapshot`** (accessibility tree) to understand page state before interacting.
6. **Create test data, don't search for it** — see "Test Data Strategy" below.

## Test Data Strategy

**Prefer creating data over finding it.** When you need specific data to test a feature (e.g., a record to edit, a form to validate, an item to delete), create it yourself through the UI rather than searching through existing records. This is almost always faster and more reliable.

- **Before testing a feature that requires data:** Navigate to the relevant creation form and create the record you need. Use obvious test values (e.g., "Test Record - QA", "test-12345", today's date).
- **If you happen to find usable data quickly** (e.g., it's right there on the page), use it — don't go out of your way to create something new.
- **Never spend more than ~30 seconds searching** for existing data. If a quick glance at the current page doesn't reveal what you need, create it.
- **Clean up is not required** — test data left behind is fine. Focus on testing, not housekeeping.
- **For destructive tests** (delete, archive, etc.), always create a fresh record first so you don't destroy real data.
- **For thorough/edge-case testing**, create multiple records with varying data: empty optional fields, long strings, special characters, min/max values. Creating purpose-built test data is better than hoping to find records that happen to cover edge cases.

## Step 0: Parse Arguments

`$ARGUMENTS` contains the PR reference and optionally a Jira ticket key.

Parse it:
- First argument: PR number, PR URL, or `owner/repo/pull/number` format
- Second argument (optional): Jira ticket key (e.g., `PROJECT-1234`)

If no PR is provided, ask the user for one and stop.

## Step 1: Load Credentials

```bash
grep -E '^(TEST_USER_EMAIL|TEST_USER_PASSWORD|BUILDOPS_BASE_URL)=' .env
```

If test credentials are missing, tell the user to set them in `.env` and stop.

## Step 2: Get Preview URL from PR

Use the **GitHub MCP tools** (not the `gh` CLI) to extract the preview/deployment URL.

1. Call `mcp__github__pull_request_read` with `method: "get"` to get PR title, body, and metadata.
2. Call `mcp__github__pull_request_read` with `method: "get_comments"` to get all PR comments.
3. Search comments (prioritized) and body for preview/deployment URLs matching patterns like `*preview*`, `*deploy*`, `*vercel*`, `*netlify*`, `*amplify*`, `*cloudfront*`, or URLs posted by bot/CI systems.
4. Use the **LAST** match (most recent deployment).
5. **Fallback**: `BUILDOPS_BASE_URL` from `.env`.

## Step 3: Fetch Jira Context (if provided)

If a Jira ticket was provided, fetch full details including summary, description, and acceptance criteria.

1. Call `mcp__jira__getAccessibleAtlassianResources` (or `mcp__claude_ai_Atlassian__getAccessibleAtlassianResources`) to get the cloud ID.
2. Call `mcp__jira__getJiraIssue` (or `mcp__claude_ai_Atlassian__getJiraIssue`) with the cloud ID, issue key, and `responseContentFormat: "markdown"`.

Use this to build a comprehensive test plan that covers ALL acceptance criteria. If Jira MCP tools are not available, skip and note it.

## Step 4: Deep PR Analysis

Use the **GitHub MCP tools** to thoroughly analyze the PR:

1. Use `mcp__github__pull_request_read` with `method: "get"` for PR title, body, metadata.
2. Use `mcp__github__pull_request_read` with `method: "get_files"` for the list of changed files with additions/deletions.
3. Use `mcp__github__pull_request_read` with `method: "get_diff"` to review the actual code diff for critical files.

Build a test matrix covering:
- Every changed component/page
- Every code path that could be affected
- Integration points between changed and unchanged code

## Step 5: Start Testing

### 5a. Navigate and Login
1. Navigate to preview URL, screenshot
2. Login with `TEST_USER_EMAIL` / `TEST_USER_PASSWORD`
3. Verify login success, screenshot

### 5b. Comprehensive Test Scenarios

For each affected area, test:

1. **Happy path** — Primary feature works as described
2. **Error handling** — Invalid inputs, empty states, missing data
3. **Boundary values** — Min/max values, long strings, special characters
4. **Navigation flows** — Back/forward, deep linking, breadcrumbs
5. **Form validation** — Required fields, format validation, submit states
6. **Loading states** — Skeleton screens, spinners, progressive loading
7. **Console errors** — Check after every navigation and interaction
8. **Network requests** — Monitor for failed API calls (4xx/5xx)
9. **Responsive** — Resize browser to tablet (768px) and mobile (375px) widths
10. **Accessibility basics** — Tab through interactive elements, check focus visibility

### 5c. Deep Regression
- Test navigation/sidebar
- Test header/footer
- Test neighboring features on the same page
- Verify shared components render correctly elsewhere

**DO NOT STOP between scenarios. Complete ALL of them.**

## Step 6: Detailed Test Report

```
## Test Report: PR #<NUMBER> (Thorough Mode)
**Preview URL:** <URL>
**Jira Ticket:** <TICKET_KEY or "N/A">
**PR Title:** <title>
**Date:** <date>
**Model:** Opus (thorough)

### Summary
- Total scenarios: X
- Passed: X
- Failed: X
- Skipped: X
- Console errors found: X
- Network errors found: X

### Jira Acceptance Criteria Coverage
| Criteria | Tested | Result |
|----------|--------|--------|
| (from Jira ticket) | Yes/No | PASS/FAIL |

### Detailed Results

| # | Category | Scenario | Steps | Result | Notes |
|---|----------|----------|-------|--------|-------|
| 1 | Happy path | ... | ... | PASS | ... |

### Console Errors
- [detailed list with page context]

### Network Errors
- [failed API calls with status codes]

### Responsive Testing
| Viewport | Page | Result | Notes |
|----------|------|--------|-------|

### Accessibility
| Check | Result | Notes |
|-------|--------|-------|

### Recommendations
- [Prioritized list: critical bugs, minor issues, suggestions]
- [Risk assessment for merging]
```
