# Lab 3 - 💬 Better Prompting

## Learning objectives

- Apply action verbs and explicit scope to get higher-quality Assistant responses
- Pick the right Assistant mode for the task at hand
- Use `@` mentions to pin the Assistant to specific data sources, dashboards, and panels
- Iteratively refine a debugging conversation
- Save a reusable prompt as a Quickstart and run it from a button
- Give feedback that actually gets routed to Grafana engineers

---

## The scenario

> "I keep getting mediocre answers from the Assistant. My prompts feel like a guessing game. What am I doing wrong?"

The Assistant's quality is largely controlled by what you put in. This lab walks through the techniques that move the needle most.

---

## Step 1 - Action verbs and explicit scope

Open the Assistant and start a **new conversation**.

Try this poor prompt first and observe the response:

```text
errors
```

Now compare with each of these prompts. Send them one at a time in **separate new conversations** so prior context doesn't leak:

```text
List the top 5 services by error rate in the last hour.
```

```text
Show error rates for the productcatalogservice over the last hour and highlight any spikes.
```

```text
Compare frontend error rates this hour vs the same hour yesterday.
```

Notice how each verb shapes the output:

- **List** produces a ranked enumeration
- **Show** generates a visualization
- **Compare** triggers a side-by-side analysis

The Assistant is much better at responding to a specific verb + specific scope than a noun phrase alone.

---

## Step 2 - Pick the right mode

The Assistant has multiple **modes** you can pick from the dropdown at the bottom-left of the prompt input. Each mode loads a different system prompt and toolset, so picking the right one is the single biggest lever on response quality for non-default tasks. Click the mode selector and you'll see the full list:

| Mode | When to use it | Doc |
|:--|:--|:--|
| **Assistant** (default) | Broad questions and general tasks across Grafana - the right default for anything not covered by the specialist modes below | (no dedicated page - implied baseline across the [Get started](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/get-started/) workflows) |
| **Deep Investigation** | Incidents that span multiple services, require more than one signal type, or need a structured report. Swarms specialist agents (Prometheus, Loki, Tempo, Pyroscope) in the background. Requires the Investigations entitlement + `Assistant Investigation User` role | [Run investigations](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/investigation/) |
| **Dashboarding** | Find, understand, create, or edit dashboards. Loads the dashboard-specific tools and prompts | [Manage dashboards](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/dashboarding/) |
| **Learn** | Guided coach mode - suggests personalized observability lessons based on your role, the Grafana products you use, and the services in your environment | [Learn mode](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/learn-mode/) |
| **k6 Script Authoring** | Create, analyze, or convert k6 performance test scripts. Can discover endpoints from your observability data | [k6 Script Authoring mode](https://grafana.com/docs/grafana-cloud/testing/k6/author-run/k6-script-authoring-mode/) |
| **Knowledge Graph** (Preview) | Troubleshoot entity relationships, diagnose connectivity issues, manage custom rules. Requires the knowledge graph to be configured | [Knowledge Graph mode](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/knowledge-graph/) |

Try switching modes for the same prompt and notice the difference:

```text
Show me what's happening with the productcatalogservice.
```

Send it once in **Assistant** mode (default) and once in **Deep Investigation** mode (you'll explore this more in Lab 4). The Assistant-mode response is a quick chat-style summary; Deep Investigation kicks off a background multi-agent run that produces a structured report.

> [!TIP]
> **Naming mismatch worth knowing:** the docs call it "Investigation" mode, but the UI labels it "Deep Investigation." Same feature. The "Deep" prefix is UI-only and distinguishes it from the lighter inline investigation behavior in default Assistant mode.

> [!NOTE]
> **There's no single "modes" reference page in the docs.** Each mode lives on its own guide page (linked above), and the default Assistant mode doesn't have a dedicated page at all. The list above is the consolidated reference until Grafana publishes one.

---

## Step 3 - Pin context with `@` mentions

`@` mentions tell the Assistant exactly which data source, dashboard, or panel to use. This removes ambiguity when you have multiple data sources.

In a new conversation, type `@` in the prompt input. You should see a dropdown of available references - data sources first, then dashboards.

Try this prompt, selecting your Prometheus data source when the `@` dropdown appears:

```text
Using @<your-prometheus-datasource>, show me the request rate for the productcatalogservice in the last hour.
```

Then try this one, selecting a dashboard if any are available (look for a Postgres or storefront dashboard):

```text
Look at @<a-dashboard>. Which panel has the most concerning trend right now?
```

`@` mentions are especially useful when:

- The same metric name exists in multiple data sources
- You want the Assistant to read a specific dashboard before answering
- You're debugging and want to anchor the conversation to one panel

---

## Step 4 - Iterative refinement

The biggest mistake users make is treating each prompt as a one-shot. Real investigations are conversations.

In a new conversation, walk through this sequence. **Send each prompt as a separate message in the same conversation** - building on the previous response:

```text
What's the error rate for the productcatalogservice over the last hour?
```

```text
Which endpoint or operation is producing those errors?
```

```text
Show me example log lines for the worst endpoint.
```

```text
What do those errors have in common - is there a pattern in the messages, timing, or upstream dependency?
```

Each follow-up narrows the investigation. The Assistant uses the prior context to keep getting more specific without you re-stating the setup. This pattern - **broad question → identify the worst offender → drill into examples → look for patterns** - is the bread and butter of agent-assisted debugging.

> [!TIP]
> **When to start a new conversation:** if you're switching topics (e.g., from latency to authentication), start a new chat. Context from a prior topic can confuse the model and produce stale answers.

---

## Step 5 - Save a Quickstart prompt

If you find yourself sending the same prompt repeatedly, save it as a Quickstart. Quickstart prompts become one-click buttons in the Assistant.

1. Navigate to **Assistant → Settings** in the left sidebar
2. Find the **Quickstart prompts** section
3. Click **Add quickstart prompt**

4. Title it `Storefront health check`
5. Body:

```text
For the e-commerce storefront over the last hour, summarize: frontend request rate and error rate, productcatalogservice request rate and error rate, checkout success rate, and any noticeable spikes or drops. Highlight anything outside normal range and call out any service that looks unhealthy.
```

6. Scope: **Just me** for testing (later, an admin can promote it to **Everybody** for the team)
7. Save

Close the settings panel and open a new conversation - your Quickstart should now appear as a one-click button. Click it and you should get the full storefront health summary without typing anything.

If you don't see it, confirm the **Enabled** toggle is on for the Quickstart under **Assistant Settings → Quickstart prompts**.

> [!NOTE]
> **Why this matters:** Quickstarts encode your team's "how to start a check" knowledge into the Assistant's UI. New team members get expert-level prompts for free. Promote successful personal Quickstarts to team-wide as they prove themselves.

> [!TIP]
> Want the same prompt to run **on a schedule** instead of on demand? That's an **Automation** - covered hands-on in [Lab 7 - Automations & gcx](./07-automations-and-gcx.md).

---

## Step 6 - Give feedback that engineers actually see

Every Assistant response has thumbs-up and thumbs-down buttons. Scroll back to any response from this lab and locate the feedback controls - you don't need to submit anything right now. The point is to know they're there.

In practice: clicking thumbs-down and adding a specific note (e.g., "Picked the wrong data source - I expected Loki not Prometheus") routes that feedback to the team improving the Assistant's prompt handling. Thumbs-up with notes about what worked is equally valuable.

> [!NOTE]
> Quality feedback with concrete notes is one of the highest-leverage things you can do. The Assistant team uses these to refine system prompts, tool selection, and tuning. Even one well-written thumbs-down can change how the Assistant responds to a class of queries.

---

## ✅ Checklist

- [ ] Sent prompts with different action verbs and compared the output styles
- [ ] Opened the mode selector and can name when you'd reach for each mode
- [ ] Used `@` to reference a data source and a dashboard
- [ ] Walked through an iterative investigation in one conversation
- [ ] Saved a Quickstart prompt and ran it from the button
- [ ] Located the thumbs-up / thumbs-down feedback controls on an Assistant response
