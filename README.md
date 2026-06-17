# AgentFleetCheck AgentCard

An open, vendor-neutral spec for how AI agents describe themselves so they can be **found** — by orchestrators, by other agents, by humans.

It is to agent discovery what [`robots.txt`](https://www.robotstxt.org/) was to the early web: a tiny, conventional file at a well-known location that lets crawlers and rankers do their job better.

Developed and maintained by CertivaAI (brand of [112358 Spark Corp](https://github.com/jgeraci)). Apache 2.0. Use it without us — that's the point.

## What is it

An Agent Discovery Card is a JSON file an agent (or its provider) publishes at:

```
https://your-agent.example.com/.well-known/agents.json
```

It describes the agent's name, capabilities, how to reach it, example queries it expects to handle, and what it's *not* meant for. Discovery and certification engines (such as AgentFleetCheck), registries, marketplaces, and orchestrators ingest these cards to decide which agent to surface for a given task.

You don't need to use any particular ranker. The spec is independent of any ranker implementation.

## Why does this exist

Agents are proliferating faster than registries can hand-curate them. Today most discovery layers fall back to "tag match" or "keyword cosine over the description string" — brittle when one agent says "translate to French" and another says "convert English text to a different natural language."

The spec solves this by letting agents declare:

- **Example queries** they want to be matched for (the strongest discoverability signal)
- **Anti-uses** — what they should NOT be ranked for
- **Priority capabilities** — which of their skills are most representative
- **Dependencies** on other agents (for graph-aware rankers)
- **Conformance to common protocols** (A2A, MCP, etc.)

A well-written card costs an hour and pays back every time an orchestrator considers your agent.

## Compatibility with A2A and MCP

The card is **compatible** with [Google A2A AgentCard](https://google.github.io/A2A/) and MCP server descriptors. The required fields are a subset of A2A; the optional fields are additive. An agent can publish a single `/.well-known/agents.json` that satisfies all three.

We don't compete with A2A or MCP — those are *transport* metadata. This spec is *discovery* metadata. They sit in the same file.

## Files

| File | Purpose |
|---|---|
| `SPEC.md` | The normative spec, human-readable |
| `schema/agent-discovery-card.schema.json` | JSON Schema, machine-readable |
| `examples/` | Annotated example cards (good + bad) |
| `best-practices.md` | How to write fields that rank well |
| `CHANGELOG.md` | Version history |

## Status

Version 0.1.0 — pre-1.0, breaking changes possible. Feedback welcome via GitHub Issues.

## License

Apache 2.0. The spec is free to use, implement, embed in registries, or extend. The reference implementations of compliant rankers (such as AgentFleetCheck) are not part of this repository and have their own licenses.

## Who's behind this

Designed and maintained by Joe Geraci ([@jgeraci](https://github.com/jgeraci)) at CertivaAI (112358 Spark Corp). The AgentFleetCheck certification engine is one consumer of these cards, but the spec is deliberately independent of any single ranker — including ours — so adoption isn't tied to one vendor's roadmap.

Contributions, criticisms, and competing implementations welcome.
