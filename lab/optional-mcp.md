# Optional Lab - 🔌 MCP

> **Workshop note:** the facilitator demos Part A during the DEMO segment. If your stack shows the MCP servers under **Assistant → Settings → Integrations → MCP servers**, run this hands-on; otherwise treat it as a read-along.

MCP (Model Context Protocol) lets the Assistant act on systems *outside* Grafana. Your stack ships with two pre-configured integrations:

| MCP server | What it does |
|:--|:--|
| `Kubernetes [a]` | Read pod/deployment state, delete pods, scale workloads |
| `GitHub Repositories / Issues / Pull Requests [a]` | Read repos, file issues, manage PRs against `field-eng-appenv-mirror` |

**The scenario, continuing from Lab 5:** `productcatalogservice` is leaking postgres connections, crashing, and restarting. Two follow-ups: **Part A** restarts it cleanly via the Kubernetes MCP; **Part B** files the engineering issue via the GitHub MCP. They're independent - do either or both.

---

## Step 1 - Verify the MCPs are connected

**Assistant → Settings → Integrations → MCP servers**: confirm `Kubernetes [a]` and the three GitHub `[a]` servers show healthy with their tool counts. If not, flag a facilitator.

---

## Part A - Remediate with Kubernetes MCP

### A1 - Inspect

New conversation:

```text
Using the Kubernetes MCP, list the productcatalogservice pods in the ecommerce-prod namespace. For each one, show its status, age, and restart count.
```

Expect elevated restart counts and young pod ages - the crash loop from Lab 5. Then drill in:

```text
Get the full details for the productcatalogservice pod with the highest restart count. Include recent events.
```

Look for restart events and OOM/crash exit codes (often `exit 137` here - the leak inflates memory before postgres rejects).

### A2 - Remediate

```text
Delete the productcatalogservice pod with the highest restart count to force a fresh start. Show me the result before I confirm.
```

The Assistant drafts the action and asks for confirmation before executing. Approve, then verify:

```text
List the productcatalogservice pods again and tell me which one is new.
```

The replacement pod should show age in seconds and 0 restarts - and a clean connection pool.

> [!WARNING]
> **That was a real cluster action.** The production guardrails: keep confirmation prompts on (auto-approve only for well-tested workflows), scope the MCP's service account tightly (here: one namespace, fixed verbs), and match allowed verbs to blast radius - deleting a pod is recoverable, deleting a deployment is not.

That's the full loop: the Assistant **read** cluster state, **reasoned** about the worst offender, **wrote** (after your confirmation), and **verified** - Assistant-as-agent, not chatbot.

### A3 - How it's wired (reference)

The Kubernetes MCP server runs as a pod in the cluster, exposed via an Ingress, and registered under **Settings → Integrations → MCP servers** with a URL + `Authorization` header. Its Kubernetes service account RBAC defines what it can touch; the **Tools** tab shows each tool with Read/Write tags and per-tool **Default / Auto-approve / Always ask** approval modes. Registering your own: **Add MCP server** → URL → auth header → save.

---

## Part B - File a GitHub issue

### B1 - Draft, review, submit

```text
Using the GitHub MCP, draft an issue in the field-eng-appenv-mirror repo titled "productcatalogservice leaks postgres connections - causes crash loop". Summarize the symptom (homepage 500s, missing products), the root cause (connections not closed), the signature (sawtooth in pg_stat_activity, "pq: sorry, too many clients already" in logs), and the recommended fix (review connection handling; add a k6 load test to CI). Don't submit yet - show me the draft first.
```

Read the draft critically - edit with the Assistant if needed - then:

```text
Submit the issue.
```

Click through the returned URL to verify it on GitHub.

> [!TIP]
> You never specified the org/repo details - a baked-in "Default GitHub Repo" **Rule** anchors GitHub MCP actions to `field-eng-appenv-mirror`. Use the same pattern in your own setup.

### B2 - Close the loop with your Lab 6 Skill

Add a final step to your `Investigate frontend 500s` Skill:

```text
7. If the root cause is confirmed, draft a GitHub issue in field-eng-appenv-mirror summarizing the symptom, root cause, and recommended fix. Show me the draft before submitting.
```

Now the next "storefront is broken" runs the full investigation AND drafts the ticket - the end-to-end workflow, encoded once.

### B3 - How it's wired (reference)

GitHub hosts the MCP endpoints (`https://api.githubcopilot.com/mcp/x/issues`, `/repos`, `/pull_requests`) - you register each with Streamable HTTP transport and an `Authorization: Bearer <PAT>` header (`repo` scope). The workshop registers three separate integrations, one per toolset, keeping each server's tool count low.

---

## Worth knowing

- **>16 enabled tools degrades quality** - enable only what you use
- **Remote MCP servers only** (no local); "Everybody" scope needs an auth-header token, "Just me" allows OAuth
- **Privacy**: MCP calls flow through the same LLM provider as chat - don't wire up tools handling data you wouldn't send there

---

## ✅ Checklist

- [ ] Verified the Kubernetes + GitHub MCP servers are healthy
- [ ] Listed, inspected, deleted, and re-verified a pod via natural language
- [ ] Drafted, reviewed, and submitted a GitHub issue - then **closed it** (shared repo, keep it tidy: [issues list](https://github.com/grafana/field-eng-appenv-mirror/issues))
- [ ] Added the GitHub follow-up step to your Lab 6 Skill
