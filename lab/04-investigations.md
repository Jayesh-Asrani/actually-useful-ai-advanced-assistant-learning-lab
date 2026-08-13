# Lab 4 - 🔎 Investigations

## Learning objectives

- Inject a real failure into the storefront using the environment's feature flags
- Run a Deep Investigation against the storefront
- Watch a multi-agent swarm fan out across metrics, logs, traces, and profiles
- Read the structured investigation report and refine it through the Workspace conversation
- Understand where Investigations fits in the incident response workflow

---

## The scenario

> "We just got a page: the storefront 'Frontend Success 99.5%' SLO is burning fast. The homepage is showing 500 errors, products are missing, and customers are complaining. We deployed something earlier today. What broke, and what do we need to roll back?"

This is the canonical use case for Investigations - a problem that touches multiple services (frontend → product catalog → postgres), needs cross-signal correlation, and a team that doesn't have 30 minutes to do it by hand.

And in this workshop, **you** get to cause the incident. Your environment ships with fault-injection feature flags - you'll flip one, watch your storefront break, and then let the Assistant find what you did.

---

## Step 1 - Break your storefront (inject a failure)

Your stack has a **Feature Flags** dashboard that controls failure scenarios in the e-commerce app. Open it:

```text
https://<your-stack>.grafana.net/d/appenv-feature-flags/feature-flags?from=now-3h&to=now&timezone=browser&refresh=30s
```

Scroll to the **Flag Details** panel - each flag has a description and **Enable / Disable** actions. Turn on the flag for this lab's scenario:

- **`productCatalogStopClosingPostgresConnections`** - the product catalog stops closing its postgres connections. Connections pile up until postgres hits its limit, the service crashes and restarts, and the storefront starts throwing 500s. This is the incident the rest of this lab investigates.

Want more chaos (or a different scenario)? Also try:

- **`productCatalogReadFromPostgres`** - forces catalog reads through postgres, amplifying the connection-leak blast radius
- Browse the rest of the flag list - cart failures, checkout slowdowns, payment timeouts, image slow-loads - each one produces a different investigation

Confirm the flip in the **Flag State History** panel (your flag should show **On**), then open your storefront's App URL and refresh the homepage a few times - within a few minutes you should start seeing errors and missing products.

> [!TIP]
> **Flip the flag, then keep reading.** The failure needs a few minutes of telemetry to become findable. By the time you've read Step 2 and written your investigation prompt, there will be plenty of evidence to discover.

> [!NOTE]
> **Leave the flag on after this lab.** In Lab 5 you'll encode this exact investigation into a reusable Skill and run it against the same live incident - that's the payoff: the failure you created here is the one your Skill debugs faster next time. Your facilitator handles flag cleanup at the end.

---

## Step 2 - Start a Deep Investigation

In the left sidebar, navigate to **Assistant → Investigations**. You'll land on the workbooks list where past investigations live.

Click **+ New Investigation** to open the **Assistant Investigations** prompt. At the bottom-left of the input, confirm the **Deep Investigation** mode is selected (it should be the default).

You'll be prompted to describe the problem. Use this:

```text
The storefront homepage is showing 500 errors and products aren't displaying. The "Frontend Success 99.5%" SLO is burning. Investigate the full request chain - frontend, productcatalogservice, postgres - including recent deployments. Report the most likely root cause with supporting evidence.
```

Submit the investigation.

> [!TIP]
> **Why this prompt works:**
> - States the symptom (500 errors, missing products) and the business impact (SLO burning)
> - Names the suspected dependency chain (frontend → productcatalog → postgres)
> - Explicitly asks about recent deployments (a leading cause of incidents)
> - Asks for both a conclusion AND the supporting evidence

---

## Step 3 - Watch the multi-agent fan-out

Investigations kicks off a Lead investigator first, which then spawns specialized sub-agents that work the problem in parallel - a Prometheus Specialist for metrics, a Loki Error Specialist and Loki Specialist for logs, a Tempo Specialist for traces, and an MCP Specialist for things like recent code changes. The Workspace conversation alongside the report shows progress as they work.

As the investigation advances, more specialist agents join the Agent activity timeline and you can watch each one's work bars fill in:

Hover any agent's bar in the Agent activity timeline to see exactly what that agent was checking, and what kind of cause it ended up attributing - **Local Cause** (a symptom in one component), **Systemic Cause** (a contributing factor across the chain), or **Root Cause** itself.

For example, hovering the Prometheus Specialist surfaces a `[Local Cause]` finding for productcatalogservice memory usage:

Other agents land on broader causes - here the Prometheus Specialist's Postgres query error check is flagged as a `[Systemic Cause]`, and the MCP Specialist's check of productcatalogservice v3.73.0 code changes ends up labeled as the `[Root Cause]`:

For this scenario, the most useful signals to watch for:

- **Logs**: `pq: sorry, too many clients already` (postgres connection exhaustion) and `invalid memory address or nil pointer dereference` (service crash)
- **Metrics**: postgres connection count over time, productcatalogservice request rate dropping, frontend error rate climbing
- **Traces**: 500 responses from productcatalogservice and failed downstream calls from frontend
- **Deployments**: any recent deployment annotations on productcatalogservice in the last few hours

This usually takes a few minutes. The investigation runs in the background, so you can navigate elsewhere in Grafana and return - find your in-progress and completed runs under **Assistant → Investigations**.

---

## Step 4 - Read the report

When the investigation completes, the page header shows the Assistant's Root Cause headline and the workbook status flips to **Completed**.

You get three views you can switch between:

- **Detailed report** - the synthesized findings, key observations, and recommended next steps
- **Tree View** - the hypothesis tree showing how each specialist's findings combined into a single root cause
- **Timeline** - the chronological event sequence the investigation reconstructed, with the relevant charts attached to each event

Start in the **Detailed report** view. Read it end-to-end - don't just scan. For this scenario, look for these three things explicitly:

1. **The smoking-gun log line.** The `pq: sorry, too many clients already` error should show up in the findings - that's postgres telling you it's out of connections.
2. **The sawtooth pattern.** Look for evidence the postgres connection count spikes, the product catalog crashes, connections drop, and the cycle repeats. That pattern is the signature of a connection leak.
3. **The root cause statement.** The expected finding: productcatalogservice is not closing its postgres connections. Connections accumulate until postgres hits its max (100), the service errors out, restarts (which closes connections), and the cycle repeats.

Then switch to **Tree View** to see what else the Assistant considered. Each node shows which agent ran which check and what cause level it found - Local Cause, Systemic Cause, or Root Cause. This is the alternatives view - useful evidence when you're explaining to the team why *this* is the cause and not something else.

Click into **Timeline** to see the chronological sequence the investigation reconstructed - which symptoms appeared first, when the deployment fired, when the connection pool exhausted, and how it cascaded to frontend 500s. Each event has the relevant chart attached.

> [!NOTE]
> **What "good" looks like for this investigation:** the report should connect three things explicitly - the frontend 500 errors, the product catalog restarts, and the postgres connection exhaustion. If it only finds one piece, that's a partial result you'd need to follow up on. If it correlates all three with a clear hypothesis (connection leak), the Assistant did the cross-signal correlation that's hard to do manually.

---

## Step 5 - Refine the report

The report isn't directly editable - but the investigation is still a collaborative artifact, not a one-shot AI output. You can refine it through the **Workspace conversation** alongside the report.

Try adding a hint or correction. In the conversation panel, send something like:

```text
The report missed that this same connection-leak pattern caused an incident two weeks ago - the rollback was the right call. Add a "Prior incidents" section referencing the recurrence and recommend a k6 load test against postgres connection limits to prevent regression in CI.
```

Then click **Regenerate report**. The Assistant uses your additional context to produce an updated version.

You can also ask the Assistant to convert findings into another artifact:

```text
Turn the root cause and remediation into a Slack message I can paste into #incident-storefront.
```

```text
Draft the rollback steps as a backlog item I can paste into our issue tracker.
```

> [!NOTE]
> **Why this matters:** the LLM output is a strong starting point but it's not always exactly right. Treating the report as a draft you tighten through conversation - rather than a verdict - is how teams use Investigations in real incidents.

---

> [!NOTE]
> **MCP works in Investigations too.** The Workspace conversation can call MCP servers like any regular Assistant chat, so you can ask it to file a GitHub issue, post to Slack, etc. directly from the workbook. We're not exercising that flow in this lab. See the [Grafana Assistant docs on MCP actions in Skills](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/skills/#take-action-with-mcp) for the supported environments (Slack, web Assistant, investigation mode, and backend agents).

---

---

## ✅ Checklist

- [ ] Enabled a failure flag on the Feature Flags dashboard and confirmed the storefront broke
- [ ] Launched a Deep Investigation against the storefront
- [ ] Watched the multi-agent fan-out complete and hovered an agent to see its cause-level attribution
- [ ] Read the Detailed report, scanned Tree View and Timeline, and identified the root cause
- [ ] Refined the report via the Workspace conversation (added a hint or asked the Assistant to convert findings into another artifact)
