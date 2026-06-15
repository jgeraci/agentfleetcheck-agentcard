# TopoRank AgentCard Specification

**Version:** 0.1.0 (2026-06-12)
**Status:** Draft. Breaking changes possible before 1.0.

This document specifies the format of a TopoRank AgentCard (also called an Agent Discovery Card) — a JSON document an agent (or its provider) publishes to advertise its capabilities to discovery rankers, orchestrators, and other agents.

The spec is vendor-neutral and Apache 2.0 licensed. Reference consumers include TopoRank, but conformance to this spec does not require using any particular ranker.

The key words "MUST," "MUST NOT," "REQUIRED," "SHALL," "SHALL NOT," "SHOULD," "SHOULD NOT," "RECOMMENDED," "MAY," and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

---

## 1. Location

An agent SHOULD publish its card at:

```
https://{authority}/.well-known/agents.json
```

following the [Well-Known URI](https://datatracker.ietf.org/doc/html/rfc8615) convention. If the agent is reachable at a sub-path, the card MAY be served from the root of that sub-path.

The card MUST be served with content type `application/json`.

## 2. Required Fields

Every card MUST contain the following top-level fields:

### `agent_id` (string)

A globally unique identifier. RECOMMENDED format: reverse-DNS or URL-path-like (`com.example.scheduler`, `com.example.research-finder`). MUST NOT contain whitespace.

### `name` (string)

Human-readable display name. SHOULD be ≤ 64 characters.

### `description` (string)

Natural-language description of what the agent does. SHOULD be ≥ 40 and ≤ 400 characters. This is the primary input to embedding-based matchers — write it for a *human* who needs to decide if your agent is the right one for a task they're describing in natural language.

### `version` (string)

[Semantic version](https://semver.org/) of the agent itself (NOT the card schema). Cards from different versions of the same `agent_id` MAY have different capabilities; the version lets rankers handle this.

### `capabilities` (array of `Capability`)

Non-empty list of things the agent can do. See § 5.

## 3. Recommended Fields (Reach & Identity)

These are not strictly required but heavily improve discoverability and routability.

### `url` (string)

A URL where the agent is reachable. The protocol-specific format is governed by `protocols` below.

### `protocols` (array of string)

Which transport protocols this agent implements. Allowed values: `a2a`, `mcp`, `http`, `openapi`, `grpc`. May include multiple; an orchestrator picks whichever it supports.

### `auth_scheme` (string)

Authentication scheme. Examples: `none`, `bearer`, `oauth2`, `x402`, `mtls`.

### `provider` (string)

The organization or individual operating the agent. Useful for trust / preference signals.

### `tags` (array of string)

Coarse domain buckets at the agent level. Free-form but SHOULD draw from a controlled vocabulary where one exists (see § 7). Examples: `research`, `coding`, `vision`, `scheduling`.

## 4. Discovery Fields (Recommended for Rank Quality)

These are what set discovery cards apart from transport metadata. They give the ranker the strongest signal about *when* this agent should be surfaced.

### `example_queries` (array of string)

Natural-language queries the agent expects to be a good match for. The single strongest input to discovery quality. 5–30 examples is a good range. Mix specific phrasings, paraphrases, and intent-level statements.

```json
"example_queries": [
  "summarize this PDF",
  "what's in this document",
  "extract the key points from a research paper",
  "I need a TL;DR of this 40-page report"
]
```

### `intended_use_cases` (array of string)

Narrative descriptions of what the agent is *meant* for. Longer-form than `example_queries`, written for someone deciding between several agents.

### `anti_use_cases` (array of string)

What the agent should NOT be matched for. Rankers MAY use these as negative-relevance signals. Examples: "writing code in any language other than Python," "real-time conversation."

### `priority_capabilities` (array of string)

Subset of `capabilities[*].name` that the agent considers most representative. Rankers MAY weight these higher when computing relevance.

### `keywords` (array of string)

Additional lexical tokens to aid keyword and IDF-based matchers. Use sparingly — over-stuffing is detectable and discounted.

## 5. Capability

Each item in `capabilities` is an object:

```json
{
  "name": "summarize_pdf",
  "description": "Take a PDF (URL or attachment) and return a concise summary at the requested length.",
  "tags": ["document", "summarization"],
  "examples": [
    {
      "input": "Summarize https://example.com/paper.pdf in 200 words.",
      "output": "This paper introduces a method for ..."
    }
  ],
  "input_schema": { "type": "object", "properties": { ... } },
  "output_schema": { "type": "object", "properties": { ... } }
}
```

Required: `name` (string, lowercase-snake_case RECOMMENDED), `description` (string).
Optional: `tags`, `examples`, `input_schema`, `output_schema`.

`examples` SHOULD be present for at least the priority capabilities. They are the strongest signal that a capability does what its name claims.

`input_schema` and `output_schema` SHOULD be JSON Schema if present.

## 6. Dependencies

### `depends_on` (array of string)

`agent_id`s of other agents this agent calls in its normal operation. Graph-aware rankers use this for cluster-coherent ranking — when a task requires both a primary agent and one of its declared dependencies, ranking them together is preferred.

A depends-on relationship SHOULD reflect a real runtime dependency, not affinity / advertising.

## 7. Common Vocabularies

A controlled vocabulary for `tags`, capability `name`s, and capability `tags` is OPTIONAL for the agent but RECOMMENDED — agents that use canonical terms get sharper edges in graph-based rankers because they share IDF-weighted edges with other agents using the same terms.

This spec does NOT mandate a vocabulary. A community-maintained vocabulary is being developed at [TBD URL] and MAY be referenced via the `vocabulary` field:

```json
"vocabulary": "https://schema.example.com/agents/v1"
```

## 8. Optional Operational Fields

### `latency_class` (string)

Order-of-magnitude latency expectation: `realtime` (< 100ms), `interactive` (< 2s), `batch` (> 2s).

### `cost_class` (string)

Order-of-magnitude cost expectation: `free`, `cheap` (< $0.01/call), `moderate`, `expensive`.

### `availability` (object)

```json
"availability": {
  "regions": ["us-east-1", "eu-west-1"],
  "uptime_sla": 0.99
}
```

### `contact` (string)

Email or URL for the operator. Useful for ranker operators to reach out when the agent appears in many high-stakes rankings.

## 9. Provenance

### `source` (string)

A URL identifying where the card was retrieved from, if not from the well-known location. Useful when cards are aggregated by a registry.

### `card_version` (string)

[Semantic version](https://semver.org/) of the card schema this card targets — i.e., the version of THIS spec. Example: `"0.1.0"`. Allows backward-compatible evolution.

## 10. Reserved Fields

Implementations MUST ignore unknown top-level fields. Future versions of this spec may add fields; agents declaring `card_version` allow consumers to skip unsupported fields cleanly.

The following field names are reserved for future use and SHOULD NOT be used: `score`, `ranking`, `authority`, `internal`, `outcome`, `centrality`. These belong to the ranker's output, not the card's content.

## 11. Validation

A card MUST validate against the JSON Schema in `schema/agent-discovery-card.schema.json`. A reference CLI for validation will be provided at [TBD] under the spec repository.

## 12. Examples

See the `examples/` directory in the spec repository for annotated good and bad cards covering:

- A minimal compliant card
- A research/PDF-summarization agent (well-tuned)
- A multi-skill orchestrator
- A common-mistake card (anti-pattern) with annotations

---

*Version 0.1.0 — feedback welcome via GitHub Issues.*
