# Lab 4 - 🔎 Investigations

## Learning objectives

- Root-cause a failure your facilitator injected - without knowing what broke
- Run a Deep Investigation and watch the multi-agent fan-out across metrics, logs, and traces
- Read the report (Detailed / Tree View / Timeline) and refine it through the Workspace conversation
- Defend your root cause with evidence at the reveal

---

## The scenario

> "We just got a page: the storefront is degraded, customers are complaining, and the 'Frontend Success 99.5%' SLO is burning. What broke?"

Your facilitator has enabled one or more failure flags on your stack - connection leaks, failing reads, cart errors, checkout slowdowns - and won't say which. At the end of the lab you'll be asked: **what was broken, what's the root cause, and what's your evidence?**

> [!IMPORTANT]
> **No peeking at the Feature Flags dashboard** - it gives away the answer. And **leave the failure running**: Lab 5's Skill investigates this same incident.

---

## Step 1 - Start a Deep Investigation

Navigate to **Assistant → Investigations** and click **+ New Investigation** (Deep Investigation mode is the default).

Describe the problem in terms of symptoms - glance at your storefront's App URL first if you want specifics. For example:

```text
The storefront is degraded - users are hitting errors and some content isn't loading. The "Frontend Success 99.5%" SLO is burning. Investigate the storefront services and their dependencies, including any recent changes, and report the most likely root cause with supporting evidence.
```

> [!TIP]
> Symptoms + business impact + scope, no presumed cause (let the agents build the hypotheses), and an explicit ask for evidence.

---

## Step 2 - Watch the multi-agent fan-out

A Lead investigator spawns specialists that work in parallel - Prometheus (metrics), Loki (logs), Tempo (traces), MCP (recent changes). Watch the **Agent activity timeline** fill in, and hover any agent's bar to see what it checked and the cause level it attributed: **Local Cause**, **Systemic Cause**, or **Root Cause**.

The run takes a few minutes and continues in the background - feel free to start Lab 5 and come back (find runs under **Assistant → Investigations**).

---

## Step 3 - Read the report

When the workbook flips to **Completed**, the header shows the Root Cause headline. Three views:

- **Detailed report** - findings, key observations, next steps
- **Tree View** - the hypothesis tree: what else was considered, and why this cause won
- **Timeline** - the reconstructed event sequence, charts attached

Read the Detailed report end-to-end hunting for three things: the **smoking-gun evidence** (a log line, metric pattern, or trace), the **failure pattern** (e.g. a repeating spike-crash-restart sawtooth), and a **root cause statement** you could defend to a teammate. A good report connects the user-facing symptom, the failing component, and the underlying mechanism - not just one of them.

---

## Step 4 - Refine the report

The report is a draft, not a verdict. In the Workspace conversation alongside it, add context and click **Regenerate report**:

```text
Add a "Prior incidents" section noting if this failure pattern has occurred before, and recommend a k6 load test to prevent regression in CI.
```

Or convert findings into another artifact:

```text
Turn the root cause and remediation into a Slack message I can paste into #incident-storefront.
```

---

## Step 5 - Report back (the reveal)

When the room's investigations have landed, your facilitator will ask:

1. **What was broken?** (symptom + failing component)
2. **What's the root cause?** (the mechanism, not just the service name)
3. **What's your evidence?** (the log line / metric / trace that convinced you)
4. **What would you do about it?** (fix + prevention)

Then the flag(s) get revealed. Close but not exact is normal - that's why the evidence and Tree View matter more than the headline.

---

> [!NOTE]
> Investigations is Cloud-only, works best with metrics + logs + traces connected, and is still LLM output - verify before acting.

---

## ✅ Checklist

- [ ] Launched a Deep Investigation described in terms of symptoms
- [ ] Watched the fan-out and hovered an agent for its cause-level attribution
- [ ] Identified the root cause and its supporting evidence in the report
- [ ] Refined or converted the report via the Workspace conversation
- [ ] Answered the reveal questions: what broke, root cause, evidence, fix
