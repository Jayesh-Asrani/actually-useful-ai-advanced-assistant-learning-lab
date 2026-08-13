# AI for Observability in Grafana Cloud — Prompt Smarter, Ship Faster

Hands-on workshop: power-user techniques for **Grafana Assistant** and **Investigations**, run against a live e-commerce environment (metrics, logs, traces — plus a real agentic AI app).

## Before you start

- You'll need the **workshop credentials** provided by your facilitator (stack URL + login). Your environment is already built and live — no account creation or setup required.
- Work through the labs in order. Each one builds on the last.

## Labs

| # | Lab | What you'll do | Time |
|---|---|---|---|
| 0 | [Log In & Get Oriented](lab/00-login-and-orient.md) | Log into your pre-built stack and find your way around a live e-commerce app (plus its AI shopping agents) | ~10 min |
| 1 | [Get Ready — baseline prompts](lab/01-get-ready.md) | Send a vague prompt, then a structured one — and see why "action verb + scope + time range" changes everything | ~10 min |
| 2 | [Infrastructure Memories](lab/02-infrastructure-memories.md) | Run a discovery scan so the Assistant learns your services, metrics, and dependencies — then watch answers get specific | ~10 min |
| 3 | [Better Prompting](lab/03-better-prompting.md) | Master the power-user toolkit: verbs, modes, `@` mentions, iterative refinement, and one-click Quickstarts | ~18 min |
| 4 | [Dashboarding Mode](lab/04-dashboarding-mode.md) | Build a full storefront dashboard without writing a single query — your healthy baseline | ~8 min |
| 5 | [Investigations](lab/05-investigations.md) | Something just broke — and no one will tell you what. Root-cause it with a multi-agent Deep Investigation and defend your evidence at the reveal | ~22 min |
| 6 | [Rules and Skills](lab/06-rules-and-skills.md) | Encode what you just learned as an always-on Rule and a reusable Skill the Assistant finds on its own | ~10 min |
| 7 | [Automations & gcx](lab/07-automations-and-gcx.md) | Run the Assistant on a schedule, then query production and talk to the Assistant from your terminal | ~12 min |
| 8 | [Challenge: Actually Useful Prompts](lab/08-challenge-useful-prompts.md) | Craft your most "actually useful" prompt and share it — best one wins | ~7 min |

## Optional (fast finishers)

- [Watchers & Assistant Search](lab/optional-watchers-and-search.md) — describe what "healthy" looks like and let a Watcher flag drift; find anything in Grafana just by describing it
- [MCP](lab/optional-mcp.md) — restart the broken pod and file the GitHub issue — without leaving the Assistant


## After the workshop

- Try Assistant free on real data at [play.grafana.org](https://play.grafana.org) — no account needed
- Docs: [Grafana Assistant](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/) · [Investigations](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/guides/investigation/)
- What's new: [grafana.ai](https://grafana.ai)
