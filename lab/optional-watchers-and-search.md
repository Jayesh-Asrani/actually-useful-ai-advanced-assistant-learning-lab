# Optional Lab - 👁️ Watchers & Assistant Search

Two AI Week releases, both about getting answers *before* you go looking: **Watcher Agents** (agentic alerting, public preview) and **Assistant Search** (natural-language search across Grafana, public preview). Your facilitator demoed both — this lab lets you try them in your own stack.

**Time: ~10 minutes**

---

## Part 1 - Assistant Search

Assistant Search is intent-based, not keyword-based: describe what you're after instead of remembering exact names or folders.

### Step 1 - Open Search

Open the search entry point (top bar) and describe artifacts instead of naming them. Try these:

```text
the dashboard that shows feature flags
```

```text
the investigation about product catalog errors from today
```

```text
alerts related to the frontend service
```

### Step 2 - Search from inside a conversation

The Assistant uses the same search to pull artifacts into a conversation mid-flow. In a chat, try:

```text
Find my investigation from earlier and summarize its root cause in two sentences.
```

> [!TIP]
> This is the "stop hunting through menus" feature. The more artifacts your stack accumulates (dashboards, investigations, alerts, skills), the more valuable intent-based search becomes.

---

## Part 2 - Watcher Agents

Watchers move alerting from static thresholds to agentic monitoring: you describe what "right" looks like, and the Watcher keeps checking whether reality drifts from it. Watchers currently work with Prometheus and Loki data in Grafana Cloud.

### Step 1 - Create a Watcher

Navigate to **Assistant → Watchers** in the left nav and click **+ New watcher**. The flow is **Calibrate → Watch → Escalate**: you describe what to watch, the agent drafts and validates its own checks against your real data, then scans on a schedule and escalates (it can even open an investigation) when something looks off. There's also a dedicated **Watcher Calibration** mode in the Assistant's mode selector.

Describe healthy storefront behavior:

```text
The productcatalogservice should serve requests without elevated error rates, and the frontend homepage should respond successfully. Postgres connections should stay well below the maximum and not show a repeating spike-and-drop pattern. Flag anything that drifts from this.
```

### Step 2 - Watch it evaluate

The Watcher evaluates on its schedule. If the workshop's fault scenario is still active on your stack (or your facilitator re-enables it), you should see the Watcher flag the drift — without you having written a single alert rule, threshold, or PromQL expression.

### Step 3 - Close the loop

Feed a firing Watcher straight into the Assistant or an Investigation to begin remediation:

```text
My storefront Watcher just fired. Investigate what it caught and tell me if it's the same connection-leak pattern from earlier.
```

> [!NOTE]
> **Watchers vs classic alerting:** classic alert rules are precise but require you to enumerate failure conditions up front. Watchers cover the cases where you can't easily hand-write the rule — "it should look normal" — at the cost of LLM evaluation on a schedule. Most teams will run both.

---

## ✅ Checklist

- [ ] Found at least two artifacts via natural-language Search
- [ ] Pulled a found artifact into an Assistant conversation
- [ ] Created a Watcher from a plain-language description of healthy behavior
- [ ] (If the fault scenario was active) Saw the Watcher flag the drift and handed it to the Assistant
