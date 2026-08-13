# Lab 7 - ⚙️ Automations & gcx

The Assistant isn't only a chat panel. In this lab you'll run it two more ways: **on a schedule** with Automations, and **from your terminal** with gcx, the Grafana CLI.

*The diagnostic loop, part 4 of 4: **keep watch, everywhere.** The incident from Lab 5 is still running - a well-built automation should catch it without being asked, and the CLI should confirm it without opening a browser.*

**Time: ~12 minutes** · You'll need your laptop's terminal for Part 2 (macOS/Linux, or Windows with the pre-built binary). Tip: kick off the gcx install (Part 2, Step 1) in the background before starting Part 1.

---

## Part 1 - Automations (~5 min)

Automations run a saved prompt for you — on a schedule or on demand — with each run landing in its own Assistant conversation. One caution on a workshop stack: a *scheduled* automation keeps consuming Assistant tokens after the session ends. So we'll do this with guardrails: create one with **no schedule** (manual-only runs), trigger it once, and delete it when you're done.

### Step 1 - Create a manual-only Automation

1. Navigate to **Assistant → Settings → Automations** and click **+ New automation**
2. **Basics:** name it `Storefront summary - <your initials>`, visibility **Just me**
3. **Schedule:** leave it **empty** — this makes the automation manual-only
4. **Prompt:**

```text
Summarize the top 5 errors and slowest endpoints across the ecommerce-prod services in the last hour. Flag anything outside normal range and end with one recommended next action.
```

5. Save

### Step 2 - Run it and inspect the result

Trigger a **manual run** from the automation's page. Each run creates a dedicated Assistant conversation — open it when the run completes and read the result end-to-end.

This is the part that makes Automations more than a cron job: every run is a full conversation you can inspect after the fact, follow up in, and compare against previous runs.

Here's the test: the Lab 5 incident is still live. Did your automation's summary surface it — same service, same error signature your investigation found? A scheduled version of this prompt would have flagged the incident before anyone opened a dashboard.

### Step 3 - Think in schedules

You won't schedule this one today, but note what you *would* schedule back home:

- **Daily morning summary** (`0 9 * * 1-5`): errors + slowest endpoints in the last 24h
- **Weekly capacity check** (`0 9 * * 1`): per-pod CPU/memory/restarts over 7 days, flag >80% CPU

Rule of thumb: default to longer intervals (the minimum is 15 minutes) — runs count against the monthly Assistant token budget, and team-wide schedules accumulate fast.

### Step 4 - Clean up

Delete your automation (or at minimum confirm it has no schedule and is disabled). Shared stack budgets thank you.

---

## Part 2 - gcx: the Assistant in your terminal (~7 min)

[gcx](https://github.com/grafana/gcx) is the CLI for Grafana — structured access to dashboards, alerts, metrics, logs, traces, and the Assistant itself, from your terminal or your AI coding agent.

### Step 1 - Install

macOS/Linux quick install:

```sh
curl -fsSL https://raw.githubusercontent.com/grafana/gcx/main/scripts/install.sh | sh
```

(or `brew install grafana/grafana/gcx`; Windows: grab the [latest release binary](https://github.com/grafana/gcx/releases/latest))

Verify: `gcx --version`

### Step 2 - Log into your workshop stack

```sh
gcx login workshop --server https://<your-stack>.grafana.net
```

Select **OAuth** at the prompt — it opens a browser; sign in with your workshop credentials. Press Enter to skip the cloud token selection. Then verify:

```sh
gcx config check
```

### Step 3 - Query production from the terminal

```sh
# what dashboards exist?
gcx dashboards list

# error logs from the storefront namespace - spot the signature your Lab 5 investigation found
gcx logs query '{namespace="ecommerce-prod"} |= "error"' --since 1h

# request rate as a terminal graph 📈 - swap in the service your investigation blamed
gcx metrics query 'sum(rate(traces_spanmetrics_calls_total{service_name="frontend"}[5m]))' --since 1h -o graph
```

(If a query returns nothing, run `gcx metrics labels` or `gcx logs labels` to discover what's actually there — that's the intended agentic workflow: explore, then query.)

### Step 4 - Talk to the Assistant from the CLI

This is where the loop closes with Lab 5:

```sh
# your Deep Investigation from Lab 5, in the terminal
gcx assistant investigations list

# ask the Assistant a question without opening Grafana
gcx assistant prompt "List the top 3 services by error rate in the last hour in ecommerce-prod"
```

### Step 5 - See the agent angle

gcx ships a bundle of agent skills for tools like Claude Code — alert investigation, dashboard creation, SLO management, and more:

```sh
gcx agent skills list
```

This is the punchline: the same access you just used by hand is what gives an AI coding agent eyes on production — investigate an alert, draft the fix, and instrument the service without leaving the editor.

---

## ✅ Checklist

- [ ] Created a **manual-only** Automation, ran it once, and read the run's conversation
- [ ] Deleted (or disabled) the Automation
- [ ] Installed gcx and logged into your workshop stack
- [ ] Ran at least one metrics or logs query from the terminal
- [ ] Listed your Lab 5 investigation via `gcx assistant investigations list`
