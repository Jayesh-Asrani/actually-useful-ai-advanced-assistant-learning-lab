# Lab 2 - 🧠 Infrastructure Memories

## Learning objectives

- Understand what Infrastructure Memories are and how they give the Assistant context about your stack
- Review the auto-discovered memories for the workshop environment
- See how memories change the quality of the Assistant's responses

**Time: ~5 minutes**

---

## What are Infrastructure Memories?

Infrastructure Memories are an automatic knowledge-building system. The Assistant scans your connected data sources (Prometheus, Loki, Tempo) and builds structured documentation about your services, infrastructure, and monitoring setup. This gives the Assistant pre-loaded context so it doesn't have to rediscover your environment every conversation.

Each memory captures five categories per service:

1. **Identity and purpose** - service name, function, namespace, cluster, tech stack
2. **Key metrics** - actual metric names and labels (latency, error rate, traffic, saturation)
3. **Deployment topology** - Kubernetes resources, replica counts, scaling configs
4. **Dependencies** - upstream/downstream connections, databases, message queues
5. **Log structure** - available log labels, formats, field names

Memories are stored as searchable chunks and retrieved via semantic search when you ask the Assistant a question. They refresh weekly, with manual refresh available.

---

## Step 1 - Run the discovery scan (or refresh the memories)

> [!IMPORTANT]
> **Memories are org-wide** - they belong to the stack, not to your user. There is one shared set of memories per stack, so what you see depends on whether a scan has already been run on yours.

1. Open the Assistant
2. Click the **three-dot menu** and go to **Settings > Assistant memories**

What you do next depends on what the page shows:

**If you see the "Discover Your Infrastructure" page** (no memories yet): click **Start Discovery Scan**. The Assistant will crawl your connected data sources (Prometheus, Loki, Tempo) and build memories for each discovered service. The scan takes a minute or two - you can stay on the page and watch it progress.

**If memories are already listed** (a scan has run on this stack before): click **Refresh memories** instead. This triggers a fresh scan on demand so the memories reflect the environment's current state - then skip ahead to Step 2 while it runs, since the existing memories are already browsable.

> [!NOTE]
> Memories are generated from the data sources the Assistant has access to. Prometheus is the primary driver - services need metrics in a connected Prometheus data source to be discovered. Loki and Tempo enrich the memories with log structure and trace data.
>
> Because memories are org-wide, on a shared stack you'd coordinate scans with your team rather than everyone triggering their own - on your dedicated workshop stack, it's all yours.

---

## Step 2 - Review the generated memories

Once the scan completes, you should see memories listed for the workshop stack. Browse through a few - notice how each one documents a specific service with its metrics, topology, and dependencies.

Click into a memory to expand it. Each one shows the datasources used, a service dependency graph, and structured sections covering identity, key metrics, deployment topology, dependencies, and log structure.

---

## Step 3 - See memories in action

In Lab 1, you sent a baseline prompt and got a generic response. Now try a similar question and notice the difference:

Open a new conversation and send:

```text
What services are running in the ecommerce-prod namespace and what does each one do?
```

The Assistant should draw on its memories to give you a detailed answer - service names, their functions, what metrics they expose, and how they relate to each other. Compare this to the generic response you got in Lab 1 before you knew memories existed.

---

## Step 4 - Test memory-powered troubleshooting

Now try something more targeted:

```text
What metrics should I look at to check the health of productcatalogservice?
```

Because the Assistant has memories about productcatalogservice - its key metrics, deployment topology, and dependencies - it should recommend specific metric names and labels rather than generic advice. This is the foundation that makes the prompting techniques in Lab 3 more effective.

---

## How memories are managed

- **Org-wide scope**: one shared set of memories per stack - a scan or refresh by any authorized user updates them for everybody
- **Auto-refresh**: memories regenerate weekly to stay current
- **Manual refresh**: click **Refresh memories** in the Assistant memories settings to trigger a scan on demand
- **RBAC**: memories respect data source permissions. Users only see memories from data sources they're authorized to access
- **Scope**: the Assistant MCP User role or higher is required to configure memory generation

See the [Grafana documentation on Infrastructure Memory](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/features/infrastructure-memory/) for full details.

---

## Checklist

- [ ] Ran the discovery scan (or refreshed existing memories) from Assistant > Settings > Assistant memories
- [ ] Reviewed the generated memories for the workshop stack
- [ ] Asked the Assistant about services and got a memory-powered response
- [ ] Asked about specific service health and got concrete metric recommendations
- [ ] Understand that memories are org-wide, auto-refresh weekly, and respect RBAC
