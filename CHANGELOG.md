# Changelog

All notable changes to the Agent Discovery Card specification will be documented here.

The spec follows [Semantic Versioning](https://semver.org/). Breaking changes will bump the major version. The `card_version` field on a card indicates which spec version it targets.

## [0.1.0] — 2026-06-12

Initial draft of the Agent Discovery Card specification.

### Added
- Required fields: `agent_id`, `name`, `description`, `version`, `capabilities`
- Recommended reach fields: `url`, `protocols`, `auth_scheme`, `provider`, `tags`
- Discovery fields: `example_queries`, `intended_use_cases`, `anti_use_cases`, `priority_capabilities`, `keywords`
- `Capability` definition with `name`, `description`, `tags`, `examples`, `input_schema`, `output_schema`
- Dependency declaration via `depends_on`
- Vocabulary reference via `vocabulary`
- Optional operational fields: `latency_class`, `cost_class`, `availability`, `contact`
- Provenance fields: `source`, `card_version`
- Reserved field names for future use
- JSON Schema (`schema/agent-discovery-card.schema.json`)
- Best-practices guide
- Examples: minimal, research-summarizer (well-tuned), anti-pattern (annotated)

### Notes
- The well-known location is `https://{authority}/.well-known/agents.json`
- The spec is compatible with — but independent of — Google A2A AgentCard and MCP server descriptors
- No reference ranker or validator CLI ships with this version; both are planned for 0.2.0
