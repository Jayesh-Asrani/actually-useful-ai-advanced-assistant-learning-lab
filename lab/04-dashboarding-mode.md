# Lab 4 - 📊 Dashboarding Mode

Build a storefront overview dashboard without writing a single query, using the Assistant's **Dashboarding** mode.

**Time: ~8 minutes**

---

## Step 1 - Switch modes

Open a new Assistant conversation and switch the mode selector (bottom-left of the prompt input) to **Dashboarding**.

## Step 2 - Build the dashboard

Send:

```text
Create a dashboard called "Storefront Overview - <your initials>" with:
- A row for the frontend: request rate, error rate, and P95 latency
- A row for productcatalogservice: request rate, error rate, and P95 latency
- A row for infrastructure: postgres connection count and productcatalogservice pod restarts
Use the workshop Prometheus data source, last 3 hours as the default time range.
```

Review what the Assistant proposes, then let it create the dashboard.

## Step 3 - Iterate

Refine it conversationally — a few ideas:

```text
Add a stat panel at the top showing current frontend success rate as a percentage, with thresholds at 99.5% (green) and 99% (red).
```

```text
Change the postgres connections panel to show the max connection limit as a horizontal line.
```

```text
Add a text panel at the top explaining what this dashboard is for and who owns it.
```

## Step 4 - Set your baseline

Open the dashboard and take in the healthy state - request rates steady, errors near zero. Then ask the Assistant:

```text
Look at @Storefront Overview - <your initials>. Summarize the current health of the storefront and tell me which panel you'd watch most closely if something started to go wrong.
```

Keep this dashboard open in a tab. In the next lab, something is going to break - and you'll see it here first.

---

## ✅ Checklist

- [ ] Built a multi-row dashboard entirely through Dashboarding mode
- [ ] Iterated on panels conversationally (thresholds, annotations, layout)
- [ ] Used an `@` mention to have the Assistant analyze your own dashboard
- [ ] Kept the dashboard open as your baseline for Lab 5
