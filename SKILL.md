---
name: testdino
description: Connect OpenClaw to TestDino for real-time Playwright CI intelligence. Ask about test failures, flaky tests, run history, and CI health in plain English.
homepage: https://github.com/testdino-hq/TestDino-OpenClaw-skills
metadata: {"openclaw": {"primaryEnv": "TESTDINO_PAT", "requires": {"bins": ["mcporter", "testdino-mcp"], "env": ["TESTDINO_PAT"]}, "install": [{"type": "node", "pkg": "mcporter", "bin": "mcporter"}, {"type": "node", "pkg": "testdino-mcp", "bin": "testdino-mcp"}]}}
---

# TestDino — Playwright CI Intelligence

Use the `exec` tool to call TestDino. Always use `mcporter`:

```
mcporter call testdino.<tool> [params]
```

**Important:** Always call `health` first to get the `projectId`. Every other tool requires it.

---

## Commands (copy these exactly)

**Check connection / health check:**
```
exec: mcporter call testdino.health
```

**What broke in CI today? / Show failures:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testcase projectId=X by_status=failed by_time_interval=1d
```

**Show recent test runs:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testruns projectId=X limit=10
```

**Why is [test name] failing? / Debug [test]:**

If the user did not provide a test name, ask: "Which test case name would you like to debug?" and wait for their answer before running anything.

Once you have the real test name:
```
exec: mcporter call testdino.health
exec: mcporter call testdino.debug_testcase projectId=X testcase_name="EXACT TEST NAME FROM USER"
```

**Show flaky tests:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testcase projectId=X by_status=flaky by_time_interval=weekly
```

**Is it safe to merge branch Y?:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testcase projectId=X by_branch=Y by_status=failed
```

**Weekly CI summary:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testruns projectId=X by_time_interval=weekly
exec: mcporter call testdino.get_run_details projectId=X counter=N
```

**Show timeout failures:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_testcase projectId=X by_error_category=timeout_issues by_time_interval=1d
```

**List manual test cases:**
```
exec: mcporter call testdino.health
exec: mcporter call testdino.list_manual_test_cases projectId=X
```

---

## Missing parameters

- No projectId → run health first to get it
- No test name → ask: "Which test case name?"
- No time range → ask: "Today (1d), 3 days (3d), weekly, or monthly?"

---

## Response format

- Failures: bullet list, lead with count ("3 failures found")
- Merge safety: lead with yes/no
- Debug: 2-3 sentences on dominant pattern, note if flaky
