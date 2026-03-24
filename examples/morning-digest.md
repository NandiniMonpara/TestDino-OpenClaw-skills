# Morning Digest — Daily CI Health Summary

Sends a test health digest every weekday at 9am to your configured OpenClaw
notification channel (Telegram, Slack, Discord, etc.).

## What it covers

- Total CI runs in the last 24 hours
- New failures vs the day before
- Flaky test count and trend
- Top failing tests by name and error category
- Any tests that recovered (passed after previously failing)

## Cron Config

Paste this into your `openclaw.json` under the `crons` key:

```json
{
  "crons": [
    {
      "name": "testdino-morning-digest",
      "schedule": "0 9 * * 1-5",
      "prompt": "Call the TestDino health tool to get my project ID. Then call list_testruns with by_time_interval=1d to get all runs from the last 24 hours. For the most recent run, call get_run_details to get full stats. Then call list_testcase with by_status=failed and by_time_interval=1d to get failed tests, and list_testcase with by_status=flaky and by_time_interval=1d to get flaky tests. Format as a morning digest with these sections: (1) Run summary — how many runs, overall pass rate; (2) Failures — list failed test names and error categories; (3) Flaky tests — count and names; (4) Recoveries — any tests that passed today after failing yesterday. End with a one-line health status. Keep it short."
    }
  ]
}
```

## Schedule Options

Change the schedule to match your timezone or preference:

| Schedule | Meaning |
|---|---|
| `0 9 * * 1-5` | 9am weekdays |
| `0 8 * * 1-5` | 8am weekdays |
| `0 9 * * *` | 9am every day including weekends |
| `0 6 * * 1-5` | 6am weekdays (early alert) |

## Full openclaw.json Example

```json
{
  "env": {
    "TESTDINO_PAT": "your-personal-access-token"
  },
  "crons": [
    {
      "name": "testdino-morning-digest",
      "schedule": "0 9 * * 1-5",
      "prompt": "Call the TestDino health tool to get my project ID. Then call list_testruns with by_time_interval=1d to get all runs from the last 24 hours. For the most recent run, call get_run_details to get full stats. Then call list_testcase with by_status=failed and by_time_interval=1d to get failed tests, and list_testcase with by_status=flaky and by_time_interval=1d to get flaky tests. Format as a morning digest with these sections: (1) Run summary — how many runs, overall pass rate; (2) Failures — list failed test names and error categories; (3) Flaky tests — count and names; (4) Recoveries — any tests that passed today after failing yesterday. End with a one-line health status. Keep it short."
    }
  ]
}
```

## Example Output

```
TestDino Morning Digest — Monday

Runs: 4 runs in the last 24 hours. Pass rate: 91%.

Failures (3):
• checkout.spec.ts — assertion_failure — main branch
• payment-flow.spec.ts — timeout_issues — main branch
• user-auth.spec.ts — element_not_found — develop branch

Flaky (2):
• sidebar-nav.spec.ts
• modal-close.spec.ts

Recoveries (1):
• login-redirect.spec.ts — passing again after 2 days

Overall: 3 real failures need attention. 2 known flaky tests still noisy.
```
