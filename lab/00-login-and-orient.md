# 0 — Log In & Get Oriented

*Time: ~10 min. Your environment is already built and live. You just log in and find your way around.*

## The app you'll be observing

A cloud-native **astronomy e-commerce store** ("ecommerce") running on **Kubernetes**, made of ~20 microservices (`frontend`, `cartservice`, `checkoutservice`, `paymentservice`, `productcatalogservice`, `recommendationservice`, `currencyservice`, and more) in the `ecommerce-prod` namespace. It emits full OpenTelemetry metrics, logs, and traces.

It's also an **agentic AI app**: a `chatservice` with shopping agents (`general_agent`, `product_agent`, `cart_agent`) backed by Claude and GPT models. So today you'll observe a real microservices app *and* a real AI agent app — and use AI to do it.

## 1. Log in

1. Open the workshop Grafana Cloud URL (e.g. `https://<your-stack>.grafana.net`).
2. Sign in with the credentials your facilitator provided.

## 2. Open the Assistant

In the **top bar**, click **Open Grafana Assistant** (the sparkle ✦). A chat panel opens on the right. Say hi — if it answers, you're set. There's also an **Assistant** entry in the left nav.

## 3. Know the map (left nav)

You'll move between these areas today:

- **Drilldown** — queryless exploration of Metrics, Logs, and Traces.
- **Observability** — the heart of the stack:
  - **Application** — service inventory + RED metrics + Service Map for the e-commerce services.
  - **Kubernetes** — clusters, namespaces, workloads, nodes, cost, alerts.
  - **Agent** — *Agent Observability*: usage & cost, conversations, versions, tools, and quality for the shopping agents.
  - **Entity graph** / **RCA workbench** — the knowledge graph (Asserts) and root-cause workbench.
  - **Frontend**, **Database** — frontend and DB observability.
- **Testing & synthetics → Performance** — the **k6** project (`ecommerce`). Load tests here generate traffic *and* inject failures (your facilitator drives these in Module 4).
- **Assistant** and **AI & machine learning** — the AI co-pilot and ML/Sift features.

## 4. Quick sanity check

- **Observability → Application** → confirm you see services in the `ecommerce-prod` namespace.
- **Observability → Kubernetes** → confirm a cluster shows up.
- **Drilldown → Traces** → confirm spans from `frontend` / `checkoutservice`.

If anything's empty, flag a facilitator (a background load test may need starting).

## 5. Investigations access

In Module 4 you'll run a multi-agent investigation, which needs the **Assistant Investigation User** role — pre-assigned on your workshop login. To confirm: open the Assistant and check you can switch to **Investigation** mode.

Head to **[Lab 1 — 🎉 Get Ready →](./01-get-ready.md)**
