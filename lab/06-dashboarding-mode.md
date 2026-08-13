# Lab 6 - 📊 Dashboarding Mode

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

## Step 4 - Stress-test it

Open the dashboard and check: do the panels show the incident from Lab 4? Ask the Assistant:

```text
Look at @Storefront Overview - <your initials>. Which panel best captures the incident from earlier, and what would you add to catch it faster next time?
```

---

## ✅ Checklist

- [ ] Built a multi-row dashboard entirely through Dashboarding mode
- [ ] Iterated on panels conversationally (thresholds, annotations, layout)
- [ ] Used an `@` mention to have the Assistant analyze your own dashboard
