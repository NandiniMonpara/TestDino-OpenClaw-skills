# Failure Alerts — Real-Time CI Failure Notifications

Polls TestDino every 15 minutes and sends an alert when new failures appear.
Stays silent when CI is clean or only flaky tests are failing.

## What triggers an alert

- Any test with `status=failed` in the last 15 minutes
- Only sends a message if failures are found — no noise when everything passes

## What the alert includes

- Test name
- Branch name
- Error category (timeout, element_not_found, assertion_failure, network_issues)
- Number of failures found

## Cron Config

Paste this into your `openclaw.json` under the `crons` key:

```json
{
  "crons": [
    {
      "name": "testdino-failure-watch",
      "schedule": "*/15 * * * *",
      "prompt": "Call the TestDino health tool to get my project ID. Then call list_testcase with by_status=failed and by_time_interval=1d. Look only at test cases that appear recent (within the last 15 minutes based on timestamps if available). If there are any failed tests, send me an alert listing: the test names, the branch they are on, and the error category. If there are zero failures, do not send any message — stay completely silent."
    }
  ]
}
```

## Alert Frequency Options

| Schedule | Meaning |
|---|---|
| `*/15 * * * *` | Every 15 minutes (recommended) |
| `*/10 * * * *` | Every 10 minutes |
| `*/30 * * * *` | Every 30 minutes |
| `0 * * * *` | Once per hour |
| `*/5 * * * 1-5` | Every 5 minutes on weekdays only |

## Combining Digest + Alerts

Run both together — the digest gives you the morning overview, the alerts
catch new failures during the day:

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
    },
    {
      "name": "testdino-failure-watch",
      "schedule": "*/15 * * * *",
      "prompt": "Call the TestDino health tool to get my project ID. Then call list_testcase with by_status=failed and by_time_interval=1d. Look only at test cases that appear recent (within the last 15 minutes based on timestamps if available). If there are any failed tests, send me an alert listing: the test names, the branch they are on, and the error category. If there are zero failures, do not send any message — stay completely silent."
    }
  ]
}
```

## Example Alert Output

```
TestDino Alert — New CI Failure

2 failures detected:

• checkout.spec.ts — assertion_failure — branch: main
• payment-flow.spec.ts — timeout_issues — branch: main
```

## Tips

- If you get too many alerts during a known flaky period, temporarily change
  the schedule to `0 * * * *` (hourly) until things stabilize.
- To monitor a specific branch only, add `on branch X` to the prompt.
- To suppress alerts outside working hours, use a schedule like
  `*/15 8-18 * * 1-5` (every 15 min, 8am–6pm weekdays).
