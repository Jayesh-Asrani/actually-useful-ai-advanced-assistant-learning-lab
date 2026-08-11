# Lab 6 - 🧠 Challenge: Actually Useful Prompts

You've made it to the last exercise!

By now you have:

- A fully-oriented Grafana Cloud stack with metrics, logs, and traces from the e-commerce storefront
- Infrastructure Memories built for your environment
- Power-prompting techniques: action verbs, modes, `@` mentions, iterative refinement
- A completed Deep Investigation with a root cause
- Your own Rule and Skill encoding the investigation workflow

## The Goal

Time to put it all together and get some **useful** insights out of your environment.

At the end of this workshop, be ready to share your **best 1–2 prompts**, including the responses and a short note on why they are useful, and when you would use this in real life.

> [!TIP]
> Think like someone on-call. Good and useful prompts usually try to:
>   - Find errors that matter
>   - Spot patterns before they become incidents
>   - Suggest what to look at next
>   - Reduce toil by performing actions/tasks

## Prompt Ideas

Here are some prompt examples to get your juices flowing:

- "What are the most common causes of errors across the ecommerce-prod services right now?"
- "Are there any unusual spikes in request patterns or status codes in the last 3 hours?"
- "Which service is closest to becoming a problem, and why?"
- "If I were debugging this system, what should I investigate first?"
- "Summarize what changed in this environment in the last 2 hours as an incident update I can post to Slack."

**Bonus territory — the AI agent app:** your stack also observes a real agentic AI application (`chatservice` with shopping agents on Claude and GPT models) under **Observability → Agent** (Agent Observability). Some of the most interesting prompts live there:

- "How much have the shopping agents spent on tokens today, and which model is the most expensive?"
- "Are any chatservice agent conversations failing their evals? Show me an example."
- "Compare latency between the general_agent and product_agent over the last hour."

> [!IMPORTANT]
> Use everything you've learned: pick a verb, scope it, time-box it, `@`-mention the source if it's ambiguous — and if your first answer is mediocre, refine instead of retyping.

## Share-out

When the facilitator calls time, be ready with:

1. Your best prompt (paste-ready)
2. The response (screenshot or summary)
3. One sentence: when would you use this in real life?
