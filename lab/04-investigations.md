# Lab 4 - 🔎 Investigations

## Learning objectives

- Investigate a real failure your facilitator injected into the storefront - without knowing what broke
- Run a Deep Investigation against the storefront
- Watch a multi-agent swarm fan out across metrics, logs, traces, and profiles
- Read the structured investigation report and refine it through the Workspace conversation
- Understand where Investigations fits in the incident response workflow

---

## The scenario

> "We just got a page: the storefront 'Frontend Success 99.5%' SLO is burning fast. The homepage is showing 500 errors, products are missing, and customers are complaining. We deployed something earlier today. What broke, and what do we need to roll back?"

This is the canonical use case for Investigations - a problem that touches multiple services (frontend → product catalog → postgres), needs cross-signal correlation, and a team that doesn't have 30 minutes to do it by hand.

And here's the workshop twist: **your facilitator just broke your environment - and won't tell you how.** Your job is to figure out what broke and why, with evidence. At the end of this lab the facilitator will ask you three questions: *What was broken? What's the root cause? What's your evidence?* Then comes the reveal.

---

## Step 1 - Notice the symptoms (something is wrong)

Your environment ships with fault-injection **feature flags** that break the e-commerce app in realistic ways - connection leaks, failing reads, cart errors, checkout slowdowns, payment timeouts. Your facilitator has just enabled one or more of them (for example `productCatalogStopClosingPostgresConnections` or `productCatalogReadFromPostgres`) on your stack. **Which ones? That's the mystery.**

Start where a real on-call engineer starts - with the symptoms:

1. Open your storefront's App URL and click around: homepage, a product page, the cart. Refresh a few times.
2. Note what you observe: errors? missing content? slow pages? Which user actions are affected?
3. Jot down your symptom description - you'll feed it to the investigation in the next step, and the better you describe the symptom, the better the investigation starts.

> [!IMPORTANT]
> **No peeking.** The Feature Flags dashboard on your stack would tell you the answer in one glance - that's not how production incidents work. Stay out of it until the reveal at the end of this lab.

> [!NOTE]
> **Leave the failure running after this lab.** In Lab 5 you'll encode this investigation into a reusable Skill and run it against the same live incident - the failure you just root-caused is the one your Skill debugs faster next time. Your facilitator handles flag cleanup at the end.

---

## Step 2 - Start a Deep Investigation

In the left sidebar, navigate to **Assistant → Investigations**. You'll land on the workbooks list where past investigations live.

Click **+ New Investigation** to open the **Assistant Investigations** prompt. At the bottom-left of the input, confirm the **Deep Investigation** mode is selected (it should be the default).

You'll be prompted to describe the problem. Describe **what you observed in Step 1** - symptoms only, since you don't know the cause. For example:

```text
The storefront is degraded - users are hitting errors and some content isn't loading. The "Frontend Success 99.5%" SLO is burning. Investigate the storefront services and their dependencies, including any recent changes, and report the most likely root cause with supporting evidence.
```

Swap in your own symptom details (which pages, which actions, what you saw). Submit the investigation.

> [!TIP]
> **Why this prompt works:**
> - States the symptoms you actually observed and the business impact (SLO burning)
> - Scopes the search (storefront services and dependencies) without presuming the cause - let the agents build the hypotheses
> - Explicitly asks about recent changes (a leading cause of incidents)
> - Asks for both a conclusion AND the supporting evidence

---

## Step 3 - Watch the multi-agent fan-out

Investigations kicks off a Lead investigator first, which then spawns specialized sub-agents that work the problem in parallel - a Prometheus Specialist for metrics, a Loki Error Specialist and Loki Specialist for logs, a Tempo Specialist for traces, and an MCP Specialist for things like recent code changes. The Workspace conversation alongside the report shows progress as they work.

As the investigation advances, more specialist agents join the Agent activity timeline and you can watch each one's work bars fill in:

Hover any agent's bar in the Agent activity timeline to see exactly what that agent was checking, and what kind of cause it ended up attributing - **Local Cause** (a symptom in one component), **Systemic Cause** (a contributing factor across the chain), or **Root Cause** itself.

For example, hovering the Prometheus Specialist surfaces a `[Local Cause]` finding for productcatalogservice memory usage:

Other agents land on broader causes - here the Prometheus Specialist's Postgres query error check is flagged as a `[Systemic Cause]`, and the MCP Specialist's check of productcatalogservice v3.73.0 code changes ends up labeled as the `[Root Cause]`:

What the specialists find depends on which failure your facilitator picked. For example, if your incident turns out to be the postgres connection-leak scenario, the tell-tale signals look like:

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

Start in the **Detailed report** view. Read it end-to-end - don't just scan. You're hunting for three things: the **smoking-gun evidence**, the **failure pattern**, and a **root cause statement** you could defend to a teammate. For the connection-leak scenario, as a worked example, those look like:

1. **The smoking-gun log line.** The `pq: sorry, too many clients already` error should show up in the findings - that's postgres telling you it's out of connections.
2. **The sawtooth pattern.** Look for evidence the postgres connection count spikes, the product catalog crashes, connections drop, and the cycle repeats. That pattern is the signature of a connection leak.
3. **The root cause statement.** The expected finding: productcatalogservice is not closing its postgres connections. Connections accumulate until postgres hits its max (100), the service errors out, restarts (which closes connections), and the cycle repeats.

Then switch to **Tree View** to see what else the Assistant considered. Each node shows which agent ran which check and what cause level it found - Local Cause, Systemic Cause, or Root Cause. This is the alternatives view - useful evidence when you're explaining to the team why *this* is the cause and not something else.

Click into **Timeline** to see the chronological sequence the investigation reconstructed - which symptoms appeared first, when the deployment fired, when the connection pool exhausted, and how it cascaded to frontend 500s. Each event has the relevant chart attached.

> [!NOTE]
> **What "good" looks like:** the report should connect the user-facing symptom, the failing component, and the underlying mechanism - not just one of them. If it only finds one piece, that's a partial result you'd need to follow up on. If it correlates all three with a clear hypothesis, the Assistant did the cross-signal correlation that's hard to do manually.

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

## Step 6 - Report back (the reveal)

When the room's investigations have landed, your facilitator will ask each table:

1. **What was broken?** (the user-facing symptom and the failing component)
2. **What's the root cause?** (the mechanism, not just the service name)
3. **What's your evidence?** (the log line, metric pattern, or trace that convinced you)
4. **What would you do about it?** (rollback, restart, fix - and how you'd prevent recurrence)

Then the facilitator reveals which flag(s) were actually flipped. Compare: did your investigation name the right mechanism? If your root cause was close but not exact, that's normal - and it's exactly why the report's Tree View and evidence links matter more than the headline.

---

## ✅ Checklist

- [ ] Observed the storefront symptoms firsthand (without peeking at the Feature Flags dashboard)
- [ ] Launched a Deep Investigation described in terms of the symptoms
- [ ] Watched the multi-agent fan-out complete and hovered an agent to see its cause-level attribution
- [ ] Read the Detailed report, scanned Tree View and Timeline, and identified the root cause
- [ ] Refined the report via the Workspace conversation (added a hint or asked the Assistant to convert findings into another artifact)
- [ ] Reported what broke, the root cause, and your evidence at the reveal
