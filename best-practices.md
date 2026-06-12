# Best Practices for Discoverable Agent Cards

This document explains *how* to write the fields in your Agent Discovery Card so your agent surfaces for the right queries.

The spec is descriptive. This document is prescriptive — opinions, not requirements.

---

## TL;DR

1. Write the **description** for a *human* searching for a tool. Not a marketing tagline.
2. Provide **5–30 example queries** in the wordings your users actually type.
3. List **anti-use cases** — saying what you're NOT for is often more discoverability lift than another use case.
4. Each capability gets **at least one input/output example**.
5. Use **canonical capability names** if a vocabulary exists in your domain.
6. Don't keyword-stuff. It's detectable and rankers discount it.

---

## The single highest-leverage field

`example_queries`.

Most discovery rankers under the hood use embedding similarity between the query and your agent's description. If your description says *"document summarization tool"* and the user types *"give me the TL;DR of this report"*, embedding similarity will probably get it right — but only just. If you provide example queries that include phrasings like that, you give the ranker concrete anchors and your match probability shoots up.

### How many

5–30 is a good range. Past 30 you're padding; if you have 100 truly distinct example queries, you probably have several agents bundled into one.

### Cover paraphrase, intent, and edge cases

```json
"example_queries": [
  "summarize this PDF",                    // canonical
  "what's the TL;DR of this paper",        // paraphrase
  "I need to understand a long doc fast",  // intent
  "key points from a 60-page report",      // edge case
  "summarize https://arxiv.org/abs/...",   // with concrete-ish input
  "boil this down for me"                  // colloquial
]
```

### Mirror your users' language

If your users are developers, include developer phrasing ("ingest this PDF and return JSON of the headings"). If your users are researchers, include researcher phrasing ("extract the methodology from this study"). The matcher will surface you in *whichever* style appears in the live query.

## The second highest-leverage field

`anti_use_cases`.

Counterintuitive but very effective. If your agent is a **Python** REPL, declare that you're NOT for JavaScript or Ruby. The reason is that embedding similarity puts "Python REPL" close to "Ruby REPL" in the matcher's vector space — so without a negative signal, you'll get surfaced for Ruby queries and either fail or annoy a user.

```json
"anti_use_cases": [
  "writing code in languages other than Python",
  "running long-lived processes",
  "executing untrusted user-supplied code"
]
```

Rankers that support negative signals (TopoRank does) will penalize agents whose anti-uses overlap with the live query. Even rankers that don't support negatives use these for surfacing context to the calling agent.

## The `description` field

Three patterns to avoid:

❌ **Marketing copy.** "The world's smartest research assistant, powered by next-gen AI."
- Tells the matcher nothing concrete. Embedding distance from any specific task is high.

❌ **Capability laundry list.** "PDF reading, summarization, translation, sentiment analysis, code review, image generation."
- All capabilities at once dilutes each one's signal. Better: declare each as a `Capability` with its own description.

❌ **Implementation details.** "FastAPI service with vLLM backend running Llama-3-70B at 8-bit quantization."
- Tells the matcher about your stack, not what you do.

✅ **Natural description of what tasks you handle, in the language users use.**

> "Reads PDFs, plain-text documents, and web pages. Returns concise summaries at lengths you specify (e.g., 'in 200 words'). Best for academic papers, long reports, and technical documentation."

## Capability descriptions

Each capability is its own matchable entity. The capability's `name` is the **anchor** for the graph (rankers join your agent to others through shared capability names). The capability's `description` is what the embedding matcher reads.

```json
{
  "name": "summarize_document",
  "description": "Take a document (PDF URL or plain text) and return a summary at the specified length.",
  "examples": [
    {
      "input": "summarize https://arxiv.org/abs/2024.12345 in 100 words",
      "output": "This paper introduces ..."
    }
  ]
}
```

Tip: pick `name` values that other agents in your space are likely to use. If `summarize_document` is already common, use it — not `doc_compactor`. Sharing names creates IDF-weighted edges in graph-based rankers, which lifts both of you for relevant queries.

## Capability examples

The `examples` array on a capability is the single strongest signal that the capability does what its name claims. Provide at least one example for every priority capability.

```json
"examples": [
  {
    "input": "summarize https://example.com/paper.pdf in 200 words",
    "output": "The paper presents a method for ..."
  }
]
```

Use *real* phrasings. The matcher cares about the lexical/semantic content; it doesn't care if the output is shortened or paraphrased for the card.

## Priority capabilities

If you have 12 capabilities but 3 of them are 80% of your value, list those 3 in `priority_capabilities`. Rankers will weight them higher when judging your relevance — and you'll be surfaced for what you're actually good at, not what you can technically do.

```json
"priority_capabilities": ["summarize_document", "extract_citations", "compare_papers"]
```

## Anti-pattern: keyword stuffing

Don't do this:

❌
```json
"keywords": [
  "summarization", "summarize", "summary", "TLDR", "tldr", "tl;dr",
  "abstract", "abstracting", "brief", "briefing", "condense", "shorten",
  "extract", "extracting", "extraction", "boil", "compact", "compactor",
  "research", "paper", "papers", "document", "documents", "doc", "docs"
]
```

The `keywords` field is for tokens that wouldn't naturally appear in your `description` or `example_queries`. Domain abbreviations, project names, internal jargon. Not for variations of your core terms — the matcher handles those already, and stuffing makes the IDF weight of every keyword lower (because they're now low-information signals).

✅
```json
"keywords": [
  "scholar", "PubMed", "arXiv", "DOI", "BibTeX"
]
```

## Dependencies

`depends_on` is for *runtime* dependencies, not preferences. If your "literature review" agent always calls your "summarize_document" agent, declare it:

```json
"depends_on": ["jgeraci.summarize_document"]
```

A graph-aware ranker will then treat queries that need both as natural matches for your agent ahead of others that don't have the full pipeline.

Don't declare dependencies as advertising. Rankers that observe the dependency don't fire MAY detect this and discount you.

## Versioning

The `version` field is the agent's version. If your card changes in a way that affects what queries it should match (you added a new capability), bump the version. Rankers can re-rank when versions change.

The `card_version` field is the spec version this card targets (currently `0.1.0`). Don't change this for your own purposes — set it to the spec version you wrote the card against.

## How to test

The reference CLI (TBD) will let you:

```bash
agent-discovery-card validate ./agents.json
agent-discovery-card score ./agents.json --against "summarize this PDF"
```

Until that ships, validate against the JSON Schema in `schema/agent-discovery-card.schema.json` using any standard JSON Schema validator (ajv, jsonschema, etc.).

---

*Last updated 2026-06-12. Feedback welcome via GitHub Issues.*
