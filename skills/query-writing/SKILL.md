---
name: query-writing
description: Writing SPARQL queries, SHACL-SPARQL constraints and generated queries that run in reasonable time. Use before writing or changing a constraint query, when a shape or a query is slow, when generating queries, or when a speed claim must say which engine it is for.
---

# Writing queries and shapes that run in reasonable time

## Rules and the checklist

Before writing or changing a constraint query, find the rules that apply, with `{{ask}} <what the constraint checks> --kind standard --activity query-writing --tech shacl`, and follow the checklist (section 4.1 of the query rules, RDF-17). Read the record whole when the query answers weakly. A constraint runs once per focus node, so its cost is multiplied by the nodes the shape targets. Most of its speed rules (R1 to R4, R8) are workarounds for how rdflib plans a query, and the record says which rules are portable.

The query rules record why a suite was slow and what fixed it, rule by rule, with a tag on each saying whether it holds for any engine or only for rdflib, pySHACL, the harness or the machine. Its harness rules (H1 to H8) are the ones to apply to a test suite: translate each distinct query text once while pySHACL runs, sort the `PREFIX` block before keying, parse a fixture once and copy it (two parses of one document never compare equal as sets, because blank-node labels differ), distribute by test, match the workers to the cores that do real work, and check the machine (Low Power Mode, memory pressure, other applications) before reading a slow run as a regression. A shape or a fixture that changes for speed is replayed against the old results first (R7).

## The cost of a query

A query that is generated, or a SHACL-SPARQL constraint that is added, states its cost in the terms of the query rules: how many focus nodes or solutions it runs over, and which of the rules it relies on. Say which engine each speed claim is for.

## Writing a SHACL-SPARQL constraint

- **Declare prefixes inside the query**, with `PREFIX` lines at the top of `sh:select`. Never point `sh:prefixes` at a namespace IRI without `sh:declare` triples for every prefix used, since pySHACL silently falls back to the file's `@prefix` lines where other engines fail.
- **A subject with zero matches must still produce a row.** Put the counted pattern in `OPTIONAL`, so "exactly one" does not pass when there are none.
- **Choose the counting form by the number of counts.** One count is a flat query, `OPTIONAL { … }` then `GROUP BY $this HAVING (COUNT(DISTINCT ?x) != 1)`. Two or more, or high cardinality, need one grouped sub-query per count, each anchoring the subject before its own `OPTIONAL`. Several flat `OPTIONAL`s multiply rows, which `COUNT(DISTINCT …)` survives and plain `COUNT` and `SUM` do not.
- **To probe that a test catches a broken shape, break a triple pattern**, never `FILTER (false)`. rdflib drops a constant false filter, so it removes no solutions (TSD-32), and a `NOT EXISTS` that contains one is inverted.
- **Test every cardinality rule at zero**, not only at too many.
- **pySHACL drops a node shape's message reached through `sh:node`.** Put the message on the property shape that uses it.

## Findings about cost

How to write a query so that it runs in reasonable time is a design record, not a finding: the query rules of this domain. A cost finding goes in the findings record as TSD-22 to TSD-33 do, with its engine and version, and the rule that rests on it cites it. When you find that a rule holds on one engine only, say so in both places, and mark it in the table of what is specific to rdflib (section 2.1 of the query rules). A result on one engine is a hypothesis for the others until it is run on them.
