---
name: probing
description: How triple stores really behave, and how to record what we find. Use before relying on what a store does with SPARQL (an update's order, scope, datatypes, escapes, atomicity, cost), when running generated SPARQL on rdflib, Oxigraph, Jena or another engine, when writing a probe or an engine test, and whenever a store does something unexpected.
---

# Probing triple stores

The record of how each store behaves is {{discovery_record}}. It is a source of truth for the project. Every claim about a store elsewhere, in a sketch, a decision record, a Validation Pack or a code comment, cites a finding there (`TSD-nn`), or adds one.

## Before relying on a store

1. Find every finding that touches what you are doing, with `{{ask}} <the behaviour> --kind discovery`, and read each with `read TSD-nn`, which lists first the entries and terms the finding bears on. When the query answers weakly, read the findings at a glance (§5 of the record) instead.
2. Check the versions. A finding holds for the versions it names. If yours differ, treat it as a hypothesis and re-run its reproduction.
3. If the record is silent, do not reason from memory about what SPARQL or a store does. Probe it. Two rules of an early typed intermediate representation were written from memory and were wrong (findings §7.2).
4. Validate on the strictest engine a consumer may use, not only on rdflib, which accepts IRIs and updates that others refuse (TSD-35, TSD-37).

## Probing a store

- **Run the smallest query that shows the behaviour,** on every store you have, and record each result.
- **Read the source where you can,** at the version you ran. Name the file and function. Source tells you the mechanism, and an observation tells you it really happens. Aim for both.
- **Fix what the engine is free to vary.** On rdflib, the order of solutions from a non-lazy join follows `PYTHONHASHSEED` (TSD-01), so a probe whose answer depends on order fixes the seed or sweeps it, in child processes. A probe that runs once in the test process measures luck.
- **Trace as lightly as you can.** Read one variable of a solution, not all of it. A heavy trace once gave a result we did not reproduce (findings §7.1).
- **Say how parameters reached the store,** as initial bindings or as substituted text (findings §4.4). Some findings depend on it.
- **When a probe contradicts the source, re-read the source** before recording anything.

## Recording a finding

Add an entry to the record in the shape of its §6.0, and a row to its §5 table.

| Part | Holds |
|---|---|
| identifier | the next `TSD-nn`, never reused |
| evidence | Source, Observed, Documented or Reasoned, as §1 defines, and the run record (date, store and version, Python, kind of machine, never the machine or person) |
| context | what you were building or testing, and why it mattered |
| how we found it | the steps in order, with what each showed |
| what the store does | the smallest SPARQL, and the result on each store |
| why | the mechanism, citing the source |
| what it means for the project | the decisions, debt, tests and documents it touches |
| reproduce it | a query to run, or a short plan for a script when the setup has layers |

Then:

- Turn the probe into a test, in the suite of the tool it concerns, opting in to every engine the suite has, so a change in a store fails a test. A probe of cost is not a test. It goes to a benchmark, which reports per engine and version and fails only on a wrong result, never on time.
- Point the documents that rely on the behaviour at the finding, and correct any that it contradicts, with a dated note.
- A finding that turns out wrong is corrected in place, dated, and its story goes in the record's §7.
- A finding that answers an open question in full goes in that row's Answered by column, and the row stays, with the question's own words in its Question cell. A finding that answers in part is named in the row's next step, and the question stays open.
- A departure from the specification worth reporting upstream is written up in {{upstream_reports}}, with its tracker search and a reproduction complete enough to file from that file alone. Filing is {{maintainer}}'s decision.

## Tests that run SPARQL on an engine

A test that executes generated SPARQL opts in to the engines the suite runs it on. The suite's own task runs it on the default engine, and a conformance task runs it on every engine we have, with the tests marked `conformance`. What the engines do differently is recorded in the findings, and the sections above say how to probe and record a new difference. Points from it matter for any such test.

- **Conformance is not part of `check`.** A test whose subject is one engine's behaviour, such as a sweep over hash seeds on rdflib or a probe of a TSD entry, is marked `conformance` and runs in the suite's `check:<suite>-conformance` task, with every engine we have, on demand and before a merge. A new engine joins that task and adds nothing to `check`.
- On rdflib, the order of solutions from a non-lazy join follows `PYTHONHASHSEED` (TSD-01). A test whose result depends on that order passes or fails by process. Fix the seed or sweep it in child processes.
- rdflib drops a result row whose projected variables are all unbound (TSD-10). Project a variable you know is bound.
- The cost of a query on rdflib depends on its form, and on Oxigraph it largely does not (TSD-22 to TSD-33). A cost measured on one engine is a hypothesis for the other.

**Prove conformance before trusting a new default.** A suite that has only ever run on one engine is not known to agree with another, however routine switching its default sounds. Giving one tool an engine abstraction so its own conformance task could try Oxigraph found a real divergence on one of its 126 tests, a `GROUP_CONCAT` over a group with an unbound row that is unbound on Oxigraph and skips the row on rdflib (TSD-41). Once the whole suite, its reasoner tests included, passed on both, Oxigraph became its default and the conformance task now runs rdflib. Run the whole suite on the candidate default first, with every test passing, before flipping it: where it does not, keep the suite's current engine as the default, add the conformance task anyway so the divergence stays visible, and record what you found as a store-discovery finding, not something the suite quietly works around.

## Measuring cost

A measurement of cost belongs in a benchmark under `{{benchmark_root}}`, which fails on a wrong result and never on time.
