# AGENTS.md

## Code Review Rules

### Identity and collision analysis

- Trace changed mappings, lookup keys, deduplication, caches, exception or allowlist matches, flattened traversals, and persistence identities. Flag a collision only when two distinct valid inputs (such as objects, paths, tenants, templates, containers, revisions, or jobs) can concretely map to the same identity, or when required identity is discarded before a policy or decision.
- For each suspected collision, name the two inputs, cite the producer where their identities diverge and the consumer where they collapse, and give the smallest local reproduction with expected versus actual behavior. Label the finding `unverified` unless an existing test or execution evidence in the PR proves it.
- Treat this as hypothesis generation: do not claim exhaustive coverage, do not infer safety from finding nothing, and omit vague possibilities that lack both a reachable code path and concrete colliding inputs.
