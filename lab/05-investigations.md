# Lab 5 - 🔎 Investigations

*The diagnostic loop, part 2 of 4: **investigations tell you why.** Your Lab 4 dashboard shows what changed; the investigation explains it.*

## Learning objectives

- Root-cause a failure your facilitator injected - without knowing what broke
- Run a Deep Investigation and watch the multi-agent fan-out across metrics, logs, and traces
- Read the report (Report / Hypotheses / Sources) and refine it through the Workspace conversation
- Defend your root cause with evidence at the reveal

---

## The scenario

> "We just got a page: the storefront is degraded, customers are complaining, and the 'Frontend Success 99.5%' SLO is burning. What broke?"

Your facilitator has enabled one or more failure flags on your stack - the environment has a dozen ways to break, from misbehaving services to strained dependencies - and won't say which. At the end of the lab you'll be asked: **what was broken, what's the root cause, and what's your evidence?**

> [!IMPORTANT]
> **No peeking at the Feature Flags dashboard** - it gives away the answer. Your **Lab 4 dashboard is fair game though** - that's exactly what it's for. And **leave the failure running**: Labs 6 and 7 build on this same incident.

---

## Step 1 - Start a Deep Investigation

Navigate to **Assistant → Investigations** and click **+ New Investigation** - it opens the Workspace with **Investigation** mode pre-selected (the docs also call this Deep Investigation).

First, gather symptoms the way you would on call: refresh your storefront's App URL, and check your **Lab 4 baseline dashboard** - which panels moved? Errors up? Connections climbing toward the limit? Restarts ticking? Fold what you see into the problem description. For example:

```text
The storefront is degraded - users are hitting errors and some content isn't loading. The "Frontend Success 99.5%" SLO is burning. Investigate the storefront services and their dependencies, including any recent changes, and report the most likely root cause with supporting evidence.
```

> [!TIP]
> Symptoms + business impact + scope, no presumed cause (let the agents build the hypotheses), and an explicit ask for evidence. If your dashboard showed something concrete - "database connections are climbing" - include it: observed facts sharpen the investigation without biasing it.

---

## Step 2 - Watch the hypotheses form

The investigation works like a scientist: specialist agents fan out across metrics, logs, traces, and even your Kubernetes and GitHub integrations, then post **hypotheses** (H1, H2, H3...) that appear as chips at the top of the conversation. Watch the live updates as evidence arrives - each hypothesis moves from **Open** to **Suspected**, gets demoted to **Symptom**, or ends up **Blocked** when a check can't be completed. Your custom Rules apply here too (look for the Rules count at the top).

The run takes several minutes and continues in the background - feel free to start Lab 6 and come back (find runs under **Assistant → Investigations**).

---

## Step 3 - Read the report

When the run completes, a structured report lands on the canvas beside the conversation, with three tabs:

- **Report** - the root-cause narrative: incident timeline, failure-propagation chain, blast radius table, evidence gaps, and recommended next steps (with confidence levels)
- **Hypotheses** - every hypothesis considered, its final state, and the evidence for each (with **Disprove** buttons - this is the alternatives view)
- **Sources** - every query and check the report's claims trace back to

Read the Report end-to-end hunting for three things: the **smoking-gun evidence** (a log line, metric pattern, or trace), the **failure pattern** (incidents usually reduce to a few mechanisms: a shared resource saturating, a dependency erroring, a config or code change altering behavior, or a service crash-looping - which is yours?), and a **root cause statement** you could defend to a teammate. A good report connects the user-facing symptom, the failing component, and the underlying mechanism - not just one of them.

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

Notice the suggested follow-ups under the report - including **Create skill** ("document this failure mode for faster future triage"). Hold that thought: it's exactly what Lab 6 does.

---

## Step 5 - Report back (the reveal)

When the room's investigations have landed, your facilitator will ask:

1. **What was broken?** (symptom + failing component)
2. **What's the root cause?** (the mechanism, not just the service name)
3. **What's your evidence?** (the log line / metric / trace that convinced you)
4. **What would you do about it?** (fix + prevention)

Then the flag(s) get revealed. Close but not exact is normal - that's why the evidence and the Hypotheses view matter more than the headline. Afterwards, look at your Lab 4 dashboard one more time: you should now be able to point at the exact panels that were telling the story all along.

---

> [!NOTE]
> Investigations is Cloud-only, works best with metrics + logs + traces connected, and is still LLM output - verify before acting.

---

## ✅ Checklist

- [ ] Launched a Deep Investigation described in terms of symptoms
- [ ] Watched hypotheses form and change state (Open → Suspected / Symptom) as evidence arrived
- [ ] Identified the root cause and its supporting evidence in the report
- [ ] Refined or converted the report via the Workspace conversation
- [ ] Answered the reveal questions: what broke, root cause, evidence, fix
