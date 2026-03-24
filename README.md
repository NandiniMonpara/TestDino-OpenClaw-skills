# TestDino for OpenClaw

Stop checking CI dashboards. TestDino brings your Playwright test intelligence
into OpenClaw so failures, flaky tests, and CI trends come to you via Slack or chat.

Ask your assistant things like:
- "What broke in CI today?"
- "Why is the payment test failing?"
- "Is the login test flaky or a real bug?"
- "Is it safe to merge the release branch?"
- "Give me a weekly CI health summary"

---

## Requirements

- [OpenClaw](https://openclaw.dev) installed and running
- [TestDino account](https://app.testdino.com) with a Personal Access Token
- Node.js 18+
- [mcporter](https://www.npmjs.com/package/mcporter) (CLI tool for running MCP servers)

---

## Setup

### Step 1: Get your TestDino PAT

1. Log in to [app.testdino.com](https://app.testdino.com)
2. Go to **Settings → Personal Access Tokens**
3. Generate a new token and copy it

---

### Step 2: Install mcporter and testdino-mcp globally

```bash
npm install -g mcporter
npm install -g testdino-mcp
```

---

### Step 3: Configure mcporter

Create or edit `~/.mcporter/mcporter.json`:

```json
{
  "mcpServers": {
    "testdino": {
      "command": "testdino-mcp",
      "env": {
        "TESTDINO_PAT": "your-personal-access-token"
      }
    }
  },
  "imports": []
}
```

**Verify mcporter works before continuing:**

```bash
mcporter call testdino.health
```

You should see your account name and list of projects. If this fails, fix it before moving on — if mcporter doesn't work, the bot won't work either.

---

### Step 4: Configure openclaw.json

Open `~/.openclaw/openclaw.json` and add the `env` block:

```json
{
  "env": {
    "TESTDINO_PAT": "your-personal-access-token"
  }
}
```

Also add the skill entry under `skills`:

```json
{
  "skills": {
    "entries": {
      "testdino": {
        "apiKey": "your-personal-access-token"
      }
    }
  }
}
```

---

### Step 5: Install the skill

**Option A — manually (works now):**

Copy `SKILL.md` from this repo into your OpenClaw workspace skills folder:

```
~/.openclaw/workspace/skills/testdino/SKILL.md
```

Create the folder if it doesn't exist.

**Option B — via ClawHub (coming soon):**

```bash
clawhub install testdino
```

---

### Step 6: Add TestDino to your TOOLS.md

Open `~/.openclaw/workspace/TOOLS.md` and add this section at the bottom:

```markdown
## TestDino — Playwright CI Intelligence

For TestDino queries, always use the `exec` tool with `mcporter`. Always run `health` first to get the projectId, then substitute it for `X`.

| User asks | Commands to run |
|---|---|
| Check connection | `exec: mcporter call testdino.health` |
| What broke today | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_testcase projectId=X by_status=failed by_time_interval=1d` |
| Recent test runs | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_testruns projectId=X limit=10` |
| Why is [test] failing | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.debug_testcase projectId=X testcase_name="name"` |
| Flaky tests | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_testcase projectId=X by_status=flaky by_time_interval=weekly` |
| Safe to merge branch Y? | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_testcase projectId=X by_branch=Y by_status=failed` |
| Weekly CI summary | `exec: mcporter call testdino.health` → `exec: mcporter call testdino.list_testruns projectId=X by_time_interval=weekly` → `exec: mcporter call testdino.get_run_details projectId=X counter=N` |
| Timeout failures | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_testcase projectId=X by_error_category=timeout_issues` |
| Manual test cases | `exec: mcporter call testdino.health` then `exec: mcporter call testdino.list_manual_test_cases projectId=X` |

Never use invented commands like `testdino check-connection` or `testdino.get_ci_status`. Only use the commands above.
```

> This step is required. Without it, the bot won't know how to call TestDino tools on a fresh session.

---

### Step 7: Restart OpenClaw and verify

```bash
openclaw gateway --force
```

Then ask your bot in Slack:

```
Check my TestDino connection
```

You should see your account name and available projects.

---

## How to Use

Once set up, just ask in plain English. Examples:

**Check connection**
```
Check my TestDino connection
```

**CI Failures**
```
What broke in CI today?
Show me failures on the develop branch
List all timeout failures from this week
Show failed tests in the last 3 days
```

**Flaky Test Analysis**
```
Is the checkout test flaky?
Why is "Verify user login" failing?
Show me all flaky tests this week
```

**Merge Safety**
```
Is it safe to merge the auth-refactor branch?
Any new failures on the feature/cart branch?
```

**CI Summaries**
```
Give me a weekly CI summary
How did CI look yesterday?
Show test run stats for the last month
```

**Manual Test Cases**
```
List all manual test cases
Show me critical priority test cases
```

---

## Automated Alerts and Digests (Cron)

### Morning CI Digest — daily at 9am weekdays

Ask OpenClaw to set up a cron:

```
Set up a daily morning digest for my TestDino CI at 9am on weekdays
```

Or paste this config directly into `openclaw.json`:

```json
{
  "crons": [
    {
      "name": "testdino-morning-digest",
      "schedule": "0 9 * * 1-5",
      "prompt": "Run: mcporter call testdino.health — get the projectId. Then run: mcporter call testdino.list_testruns projectId=X by_time_interval=1d. Send a morning digest: total runs, new failures, flaky test count, top failing tests. Keep it short and scannable."
    }
  ]
}
```

### Failure Alerts — every 15 minutes

```json
{
  "crons": [
    {
      "name": "testdino-failure-watch",
      "schedule": "*/15 * * * *",
      "prompt": "Run: mcporter call testdino.health — get the projectId. Then run: mcporter call testdino.list_testcase projectId=X by_status=failed by_time_interval=1d. If there are failures, send an alert with test names, branches, and error categories. If no failures, stay quiet."
    }
  ]
}
```

See the [`examples/`](./examples/) folder for more cron templates.

---

## Available Tools

| Tool | What it does |
|---|---|
| `health` | Validate PAT, get project IDs |
| `list_testruns` | Browse test runs with filters |
| `get_run_details` | Full details for a specific run |
| `list_testcase` | Filter test cases by status, branch, browser, error type |
| `get_testcase_details` | Error messages, stack traces, logs, artifacts |
| `debug_testcase` | Historical failure patterns + flaky detection |
| `list_manual_test_cases` | Search manual test cases |
| `get_manual_test_case` | Details for a specific manual test case |
| `create_manual_test_case` | Create a new manual test case |
| `update_manual_test_case` | Update an existing manual test case |
| `list_manual_test_suites` | List suite hierarchy |
| `create_manual_test_suite` | Create a new test suite |

---

## Troubleshooting

**"Error validating PAT" in the bot**
- Run `mcporter call testdino.health` in your terminal first. If that fails, the PAT in `~/.mcporter/mcporter.json` is wrong.
- If terminal works but bot fails, make sure `testdino-mcp` is installed globally (`npm install -g testdino-mcp`).

**Bot gives generic responses instead of TestDino data**
- Make sure you completed Step 6 (adding TestDino to `TOOLS.md`) — this is the most commonly missed step.
- Restart the gateway: `openclaw gateway --force`
- Make sure `SKILL.md` is in `~/.openclaw/workspace/skills/testdino/SKILL.md`

**Config invalid / unrecognized key errors on gateway start**
- Do not add an `mcp` top-level key to `openclaw.json` — OpenClaw does not support it
- Run `openclaw doctor --fix` to clean up any invalid keys

---

## About TestDino

TestDino is a Playwright test intelligence platform. It tracks CI runs, classifies
failures by error category, detects flaky tests through historical pattern analysis,
and gives you detailed debugging data across every test execution.

- Website: [testdino.com](https://testdino.com)
- Docs: [docs.testdino.com](https://docs.testdino.com)
- Support: support@testdino.com
