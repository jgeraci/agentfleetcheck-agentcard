# Anti-Pattern Card — Annotated

The file `anti-pattern.json` validates against the schema, but is a discoverability disaster. Here's why, line by line.

## `name`: "Ultra Smart Everything Bot v2 PRO MAX"

Marketing puffery. Tells the matcher nothing. Adds embedding distance from any specific task. Use the actual name of the thing, plainly.

## `description`: "The world's smartest AI assistant ..."

This description has **zero** matchable substance. The embedding vector of this string is roughly equidistant from every possible task query. Rankers will surface this card almost never — and when they do, it'll be for the wrong queries.

Better: describe the *tasks the agent handles*, in the words a user would use.

## `tags`: ["ai", "smart", "advanced", ...]

Every one of these is a low-information token. They're so common that any decent ranker has down-weighted them (IDF reduction). Net signal contribution: ~0. Net cost: dilutes the signal from any meaningful tags you might have added.

Better: domain-specific tags. `research`, `medical`, `legal`, `code`, etc.

## `example_queries`: ["do something", "help me", "I need assistance"]

These match *every* query and *no* query well. Be specific. The whole point of `example_queries` is to anchor your agent to the actual phrasings users will type.

Better: 5–30 concrete queries in the wordings users actually use.

## `capabilities`: 7 vague capabilities

Each capability's `description` is a single word ("Code stuff."). The matcher can't distinguish between this agent's "research" capability and any other agent's "research" capability — so the IDF edges are useless.

Worse: the seven capabilities collectively claim 7 different domains. Either the agent does one well and is lying about six, or it spreads thin and does all seven mediocrely. Rankers detect this pattern (low specificity + high breadth) and discount accordingly.

Better:
- Pick 1–3 domains the agent is *actually* good at.
- Give each capability a real description with examples.
- Set `priority_capabilities` to the top 2–3.

## `keywords`: 33 variations of the same handful of concepts

Classic stuffing. Rankers detect this trivially: very high token count + high overlap among tokens + tokens that already appear in description/tags. The IDF weight of every word in this list collapses, and most rankers will discount the agent overall for adversarial signal.

Better: 3–10 *actually-uncommon* tokens that don't appear in your descriptions. Brand names, internal identifiers, scientific terms, file formats.

## Missing fields that would help

- `url`, `protocols`, `auth_scheme` — no one can actually call this agent
- `anti_use_cases` — would help a lot here to narrow down what the agent really does
- `intended_use_cases` — narrative would force the writer to be specific
- `examples` on each capability — would prove the capability is real

## The takeaway

Discovery cards are an honesty / specificity instrument. Cards that try to claim everything end up ranking for nothing. The card asks: *"What are you actually good at, in the exact phrasings someone might use to look for that thing?"* — answer that, and ranking falls out.
