<!-- SPDX-License-Identifier: MIT -->

# Triple-store discovery

**What this is.** A record of how each triple store we ran actually behaves: its features, its limits, and how it reads the standards it implements, SPARQL 1.1 Query and Update above all. Each entry says what we were doing, how we found out, what the store does, why, and how to see it again.

**Status:** a living record, begun 2026-10-13 from work on generated SPARQL Update operations. Read it before relying on how a store behaves, and add to it when you find something.

**Stores covered:** rdflib 7.6.0 and Oxigraph 0.5.11 (through `pyoxigraph`), Oxigraph 0.5.12 for the findings a test suite probes, of which TSD-01 and TSD-17 changed (R-H), and Oxigraph 0.5.13 for TSD-40, TSD-41 and TSD-43. Apache Jena ARQ 5.1.0 is covered for TSD-15, TSD-16, TSD-25 to TSD-27, TSD-34, TSD-42, TSD-43, TSD-50 to TSD-54, TSD-56, TSD-57, TSD-60 and TSD-61, run in process through a small Java test harness on an in-memory transactional dataset (`DatasetFactory.createTxnMem()`). Jena TDB2 and Fuseki appear in TSD-42, TSD-44, TSD-62, TSD-65 and TSD-66. Each finding names the versions it was seen on. A finding holds for those versions only, until it is checked again. **Tools above a store covered:** pySHACL 0.40.1, which validates shapes on rdflib (TSD-23, TSD-24, TSD-31). TSD-22 to TSD-33 are about the cost of running queries and shapes, and the query rules say which of them are properties of rdflib or pySHACL and which hold anywhere.

## Contents

1. [How to read and write this record](#1-how-to-read-and-write-this-record)
2. [Why we started looking](#2-why-we-started-looking)
3. [How we reach each store](#3-how-we-reach-each-store)
4. [A short primer on how SPARQL evaluates an update](#4-a-short-primer-on-how-sparql-evaluates-an-update)
5. [The findings at a glance](#5-the-findings-at-a-glance)
6. [The findings](#6-the-findings)
7. [Corrections, and what they taught us](#7-corrections-and-what-they-taught-us)
8. [Open questions](#8-open-questions)

## 1. How to read and write this record

Every finding has an identifier, `TSD-nn`, that other documents cite. It is never reused. A finding that turns out wrong is corrected in place, with a dated note, and the story goes in §7. An open question has an identifier `TSD-Qn`, and §8 lists them.

Each finding states its evidence, as one or more of these.

| Evidence | Means | Example |
|---|---|---|
| **Source** | we read the store's code, at a named version and path | rdflib's `plugins/sparql/evaluate.py`, `evalJoin` |
| **Observed** | we ran a probe and saw the result, and a script or a test shows it | a script that runs an update under twelve hash seeds |
| **Documented** | the store's own documentation says so | a page of the store's manual |
| **Reasoned** | it follows from the specification or from other findings, and is not yet observed | marked as a hypothesis until it is |

A finding is only as strong as its evidence. Source tells us the mechanism, Observed tells us it really happens on this version, and the two together are what we aim for. A finding built only on Documented or Reasoned evidence says so in its first line.

**A run record** goes with every Observed finding: the date, the store and its version, the Python version, and the kind of machine, never the machine or person. Twenty-one run records appear below.

| Run | Machine | Python | Stores |
|---|---|---|---|
| **R-A** | a Linux x86_64 container, 15 GB, 2026-10-12 | 3.14.8 | rdflib 7.6.0, pyoxigraph 0.5.11 |
| **R-B** | a macOS arm64 workstation, 2026-10-10 to 2026-10-13 | 3.14.7 | rdflib 7.6.0, pyoxigraph 0.5.11 |
| **R-C** | a Linux x86_64 container, 15 GB, 2026-10-13 | 3.14.8 | rdflib 7.6.0, pyoxigraph 0.5.11 |
| **R-D** | a Linux x86_64 container, 4 logical cores, 15 GB, used for the test-suite speed work on query and shape cost, 2026-10-16 to 2026-10-17 | 3.14.8 | rdflib 7.6.0, pyoxigraph 0.5.11, pySHACL 0.40.1 |
| **R-E** | a macOS arm64 laptop with four performance and four efficiency cores and 8 GB, used to repeat R-D's runs, 2026-10-16 to 2026-10-17 | 3.14.7 | rdflib 7.6.0, pyoxigraph 0.5.11, pySHACL 0.40.1 |
| **R-F** | a macOS arm64 laptop, used for a read over a chain of versions, 2026-10-20 | 3.14.7 | rdflib 7.6.0, pyoxigraph 0.5.11 |
| **R-G** | a macOS arm64 laptop, used to prepare TSD-34's upstream report, 2026-10-20 | 3.14.7 | rdflib 7.6.0, rdflib `main` at `62bf240` (7.7.0a0), rdflib `next` at `0fdac39` (8.0.0a0) |
| **R-H** | a Linux x86_64 container, 4 logical cores, 2026-10-20, where a fresh bootstrap installed pyoxigraph 0.5.12 | 3.14.8 | rdflib 7.6.0, pyoxigraph 0.5.11 and 0.5.12 |
| **R-I** | a macOS arm64 laptop, used for posting-cost and graph-partition benchmarks, 2026-10-24 and 2026-10-27 | 3.14.7 | pyoxigraph 0.5.13 |
| **R-J** | a macOS arm64 laptop, used to isolate TSD-41, 2026-10-27 | 3.14.7 | rdflib 7.6.0, pyoxigraph 0.5.13, and 0.5.11 for the minimal probes |
| **R-K** | a macOS arm64 laptop, used to compare pyoxigraph releases, 2026-10-27 | 3.14.7 | pyoxigraph 0.5.11, 0.5.12 and 0.5.13 |
| **R-L** | a macOS arm64 laptop, used for the aggregate matrix of TSD-43, 2026-10-27 | 3.14.7, and OpenJDK 25.0.2 for Jena | rdflib 7.6.0, pyoxigraph 0.5.13, Apache Jena ARQ 5.1.0 |
| **R-M** | a Linux x86_64 container, used for TSD-42, 2026-10-28, each pyoxigraph version in its own scratch virtual environment | 3.14.10 | pyoxigraph 0.5.11 and 0.5.13 |
| **R-N** | a Linux x86_64 container, 15 GB, used for TSD-42's Jena and RDF4J runs and for TSD-48, 2026-10-28, Java in a forked JVM with a 10 GB heap | Java 25.0.4.1 | Jena 5.1.0 (an in-memory dataset and an in-memory TDB2 store), RDF4J 6.1.0 `MemoryStore` |
| **R-S** | a macOS arm64 laptop with 8 logical cores, used to run store probes for TSD-42, 2026-10-27 | 3.14.7 | rdflib 7.6.0, pyoxigraph 0.5.11, 0.5.12 and 0.5.13 |
| **R-T** | a macOS arm64 laptop, used to run the same conformance tests on three engines, 2026-10-27 | 3.14.7, and OpenJDK 25.0.2 for Jena | rdflib 7.6.0, pyoxigraph 0.5.13, Apache Jena ARQ 5.1.0 in memory |
| **R-U** | the same laptop, used to run the standalone Java reproduction of TSD-50's `CLEAR GRAPH` departure, 2026-10-27 | OpenJDK 25.0.2 | Apache Jena ARQ 5.1.0 (`DatasetFactory.createTxnMem()`) |
| **R-V** | a macOS arm64 laptop, used to run generated queries on Jena and to probe a read set, 2026-10-27 | 3.14.7, and OpenJDK 25.0.2 for Jena | rdflib 7.6.0, pyoxigraph 0.5.13, Apache Jena ARQ 5.1.0 |
| **R-W** | the same laptop, used to run the store probes and the query-cost experiments with Jena, 2026-10-27. Jena's JVM ran with its full compiler and the serial collector | 3.14.7, and OpenJDK 25.0.2 for Jena | rdflib 7.6.0, pyoxigraph 0.5.13, Apache Jena ARQ 5.1.0 |
| **R-X** | the same laptop, used to run the conformance probes and standalone Java reproductions against Jena, 2026-10-27 | 3.14.7, and OpenJDK 25.0.2 for Jena | rdflib 7.6.0, pyoxigraph 0.5.13, Apache Jena ARQ 5.1.0 |
| **R-Y** | the same laptop, used to read the W3C texts of SPARQL 1.1 Query and the SPARQL 1.2 Query working draft and to search the engines' issue trackers through the GitHub search API, 2026-10-27 | not applicable | not applicable |

**How to add a finding.** Say what you were testing, how you found it, the smallest SPARQL that shows it, the result on each store you have, the reason if you can find it in the source, and how to reproduce it. If reproducing it takes more than one query, write a short plan for a script.

## 2. Why we started looking

We generate SPARQL 1.1 Update operations from a description of how an application stores its data in an RDF store. The description says which graph an aggregate lives in, how two writers are kept from overwriting each other, and what receipt each write leaves. The generated operations are guarded writes. For example, a compare-and-set replace deletes an aggregate's old state and writes its new state only if the version row still says the sequence the caller read.

Until we set out to prove things about those operations, nothing executed them. Proving things about a query means knowing what the query does. Three questions turned that into store discovery.

1. **Does the generated SPARQL do what the description intends?** We rebuilt the generator's output as a typed intermediate representation, and every rule of its checker is a claim about SPARQL. Two of those claims were wrong when first written (§7), and only running the queries showed it.
2. **What does a write cost?** A composite aggregate's replace was suspected, then shown, to cost time or memory quadratic in the aggregate's size. The cost depends on how the store applies an update, which is not fixed by the standard.
3. **What may a generator assume about a store?** A generator can choose the right form of an operation for a store only if it knows what the store does, including how the store behaves under concurrent writers. The probes in this record are the first entries of that knowledge.

```mermaid
flowchart LR
  P["description of the storage<br/>(configuration)"] --> G["generate"]
  G --> Q["SPARQL Update text"]
  Q --> R["rdflib 7.6.0"]
  Q --> O["Oxigraph 0.5.11"]
  R --> D["what each store did"]
  O --> D
  D --> T["this record"]
  T --> CR["the generator's choices<br/>per store"]
```

## 3. How we reach each store

The first two stores are reached in different ways, and the difference matters to several findings.

```mermaid
flowchart TB
  E["test fixture<br/>one test, each engine"] -->|"rdflib"| RD["rdflib Dataset<br/>in process"]
  E -->|"oxigraph"| OX["facade with an<br/>rdflib-shaped API"]
  OX --> AD["adapter around<br/>pyoxigraph.Store, in memory"]
  RD -.->|"parameters as<br/>initBindings"| RD
  AD -.->|"parameters substituted<br/>into the text"| AD
```

- **rdflib** runs in the test process. An update is `Dataset.update(text, initBindings={...})`, so a request-time parameter such as `$root` is an *initial binding*, a value the engine starts from, and never text.
- **Oxigraph** runs through an adapter around `pyoxigraph.Store`. The adapter writes each parameter into the query text as an encoded term, as a caller with no initial-binding API would. A thin facade offers the part of rdflib's `Dataset` API the tests use.
- **A test can opt in to each engine** and run on each one, and a test whose subject is one engine's behaviour is marked as a conformance test.
- **A benchmark** of the composite replace runs each size in a child process under a memory cap, on both engines.
- **Jena ARQ** is run in process through a small Java harness. The harness runs each request in one write transaction.

So "the store" in a finding means the store as reached this way. A finding that depends on initial bindings, for example, says which mechanism it used.

## 4. A short primer on how SPARQL evaluates an update

Most findings below are about where a store departs from, or interprets, a part of SPARQL 1.1 that few people need to think about. This section sets out those parts once.

### 4.1 `DELETE … INSERT … WHERE`, by the specification

SPARQL 1.1 Update (§3.1.3) defines `DELETE { D } INSERT { I } WHERE { P }` in three steps. Evaluate `P` once, against the dataset as it was before the operation, giving a sequence of solutions. Fill `D` and `I` with each solution, skipping any triple with an unbound variable. Remove every filled deletion, then add every filled insertion. The result does not depend on the order of the solutions.

```mermaid
flowchart LR
  D0["dataset before"] --> W["WHERE, evaluated once"]
  W --> S["solutions"]
  S --> FD["fill DELETE<br/>for each solution"]
  S --> FI["fill INSERT<br/>for each solution"]
  FD --> UD["all deletions"]
  FI --> UI["all insertions"]
  UD --> A["remove all deletions,<br/>then add all insertions"]
  UI --> A
  A --> D1["dataset after"]
```

We call a store that does exactly this **snapshot**. A store may instead take the solutions one at a time and, for each, remove its deletions and add its insertions. Then a triple one solution inserts can be removed by a later solution, and the order of solutions decides the result. TSD-01 is about exactly this.

### 4.2 Groups are evaluated bottom-up

A `WHERE` clause is a group, and it may hold inner groups: `{ … }`, `OPTIONAL { … }`, a `UNION` branch, a `GRAPH g { … }`. The specification translates the query into an algebra (§18.2) in which each inner group is evaluated on its own and then joined with what came before it. So an inner group cannot *see* a variable bound outside it, although its result is *joined* with it. `FILTER NOT EXISTS { … }` is the exception. It is evaluated once per outer solution, with that solution's values substituted.

```mermaid
flowchart TB
  O["outer group<br/>BIND(1 AS ?a)"] --> J["join, on shared variables"]
  N["inner group, evaluated alone<br/>BIND(?a + 1 AS ?b)<br/>?a is unknown here"] --> J
  J --> R["?a = 1, ?b unbound"]
  O2["outer solution ?a = 1"] --> NE["NOT EXISTS, per outer solution<br/>FILTER(?a = 1) sees ?a"]
```

This decides two things that matter here: what a `BIND` inside an inner group computes (TSD-03), and how much a `GRAPH ?g` inside an inner group reads (TSD-04).

### 4.3 A `FILTER` applies to its whole group

A `FILTER` is not a step in a sequence. It applies to every solution of the group it is written in, after all the group's patterns have matched, wherever in the group it is written. A filter that tests whether a variable is bound, written at the top of a group whose patterns also bind that variable, therefore sees it bound by the data. TSD-13 shows what this did to a guard.

### 4.4 Request-time parameters: initial bindings or text

A generated operation leaves its request-time values as variables, written `$root`, `$epoch` and so on. A caller fills them in one of two ways.

| Way | How | Example |
|---|---|---|
| initial bindings | the engine starts evaluation with the variable bound | rdflib's `initBindings`, a prepared query's parameters |
| text substitution | the caller writes the value into the query text before sending it | the SPARQL 1.1 Protocol, which has no binding parameters, and our Oxigraph adapter |

The two are not interchangeable. Some functions accept only a variable (`BOUND`), so substituted text can stop a query parsing (TSD-11).

## 5. The findings at a glance

`✔` means the store does what SPARQL 1.1 says, `✘` that it departs from it, and a word where the specification leaves the choice to the store or where it is a matter of capacity.

| # | Finding | Specification | rdflib 7.6.0 | Oxigraph 0.5.11 (and 0.5.12 where it differs) | Evidence |
|---|---|---|---|---|---|
| [TSD-01](#tsd-01-the-order-in-which-a-deleteinsertwhere-is-applied) | applying a `DELETE`/`INSERT` | all deletions, then all insertions | ✘ one solution at a time, in hash order | ✘ one solution at a time, in solution order. ✔ from 0.5.12 | Source, Observed, Documented |
| [TSD-02](#tsd-02-what-filling-a-template-once-per-solution-costs) | filling a template once per solution | not specified | quadratic in time | quadratic in time and memory | Source, Observed |
| [TSD-03](#tsd-03-what-a-bind-inside-an-inner-group-can-see) | a `BIND` in an inner group | sees only its group | ✔ | ✔ | Observed |
| [TSD-04](#tsd-04-a-graph-variable-bound-outside-an-inner-group) | `GRAPH ?g` inside an inner group, `?g` bound outside | joined afterwards | passes the binding in, reads one graph | passes a constant `?g` in on 0.5.11, reads one graph. **A computed `?g` is not passed in, and a constant one is not from 0.5.13** (TSD-42). Jena ARQ: reads one graph (TSD-42) | Observed |
| [TSD-05](#tsd-05-a-template-quad-whose-graph-variable-is-unbound) | template quad, graph variable unbound | skipped | ✘ written to a new blank-node graph | ✔ skipped | Source (rdflib), Observed |
| [TSD-06](#tsd-06-a-request-whose-second-operation-fails-and-graph-management) | a request whose second operation fails | should be atomic | keeps the first operation | rolls the first back | Observed |
| [TSD-07](#tsd-07-numeric-datatypes-are-not-kept-by-oxigraph) | `"1"^^xsd:long` stored and read | the term is kept | ✔ kept | ✘ becomes `xsd:integer` | Observed |
| [TSD-08](#tsd-08-a-zero-length-path-from-a-term-that-is-in-no-triple) | zero-length path from a term in no triple | our reading: matches the term | matches it | matches nothing | Observed |
| [TSD-09](#tsd-09-unicode-escapes-and-an-escaped-backslash) | `\u` escapes and an escaped backslash | processed before parsing (§19.2) | whole-text pre-pass | inside string tokens | Source (rdflib), Observed |
| [TSD-10](#tsd-10-a-result-row-whose-projected-variables-are-all-unbound) | a result row with every projected variable unbound | kept | ✘ dropped | ✔ kept | Observed |
| [TSD-11](#tsd-11-bound-and-text-substitution) | `BOUND(<iri>)` after text substitution | a syntax error | syntax error | syntax error | Observed |
| [TSD-12](#tsd-12-initial-bindings-are-visible-inside-inner-groups-on-rdflib) | initial bindings inside an inner group | not specified | visible | not tested, the adapter substitutes text | Observed |
| [TSD-13](#tsd-13-a-filter-at-the-top-of-a-group-sees-what-the-groups-patterns-bound) | a top-level `FILTER(BOUND(?x))` with `?x` in the group's patterns | sees `?x` bound by the data | ✔ | ✔ | Observed |
| [TSD-14](#tsd-14-graph-management-operations-on-rdflib) | `CREATE GRAPH`, `DROP GRAPH` of an absent graph | defined | `CREATE` not implemented, `DROP` succeeds | `DROP` fails | Observed |
| [TSD-15](#tsd-15-a-sub-select-and-the-variables-around-it) | a sub-select and the variables around it | evaluated on its own, only its projections leave it | ✘ an outer binding leaks in | ✔. Jena ARQ ✔ (TSD-56) | Observed |
| [TSD-16](#tsd-16-min-and-max-over-iris) | `MIN` and `MAX` over IRIs | the lowest or highest IRI | ✘ a plain literal of the IRI text | ✔ an IRI. Jena ARQ ✔ an IRI (TSD-56) | Observed |
| [TSD-17](#tsd-17-a-chain-of-additions-and-subtractions) | `a - b + c` and `a / b / c` | grouped from the left | ✔ | ✘ grouped from the right. ✔ from 0.5.12 | Observed, Documented |
| [TSD-18](#tsd-18-countdistinct-over-xsdlong-and-xsdinteger-terms) | `COUNT(DISTINCT)` over `xsd:long` and `xsd:integer` | two terms, so two | counts them apart | counts them together, since it stores both as `xsd:integer` | Observed |
| [TSD-19](#tsd-19-the-order-of-group_concat) | the order of `GROUP_CONCAT` | not specified | the order the data was stored in, whatever the hash seed (this query shape) | the order of the solutions | Observed |
| [TSD-20](#tsd-20-a-filter-inside-not-exists-and-a-parameter) | a `FILTER` inside `NOT EXISTS` and a parameter or a sub-select's projection | sees them by substitution | ✔ by initial binding and by text | ✔ by text | Observed |
| [TSD-21](#tsd-21-an-empty-values-block-in-an-update) | an empty `VALUES` block in an update `WHERE` | no solution | ✘ raises `AttributeError`, before changing anything | ✔ no solution, nothing written | Observed |
| [TSD-22](#tsd-22-rdflib-parses-and-translates-a-querys-text-on-every-call) | parsing a query's text | not specified | ✘ costs 20 ms for a 20-line query, 0.5 ms when prepared | 0.17 ms, parse and run | Source, Observed |
| [TSD-23](#tsd-23-pyshacl-runs-a-sparql-constraint-once-per-focus-node) | how often a SPARQL-based constraint is run | defined per focus node | pySHACL: once per focus node | pySHACL runs on rdflib only | Source, Observed |
| [TSD-24](#tsd-24-the-prefix-block-pyshacl-writes-changes-order-between-validations) | the order of the `PREFIX` lines pySHACL writes | not specified | varies between validations | not applicable | Source, Observed |
| [TSD-25](#tsd-25-rdflib-fixes-the-order-of-a-groups-patterns-before-it-knows-the-initial-bindings) | the order of a group's patterns | not specified | ✘ fixed at translation, ignores initial bindings | ✔ no sensitivity seen. Jena ARQ: none (TSD-57) | Source, Observed |
| [TSD-26](#tsd-26-a-chain-of-three-or-more-groups-is-joined-by-evaluating-the-later-ones-whole) | joining a chain of groups | not specified | evaluates the later groups whole | not tested. Jena ARQ: no growth with size (TSD-57) | Source, Observed |
| [TSD-27](#tsd-27-a-filter-is-applied-after-its-groups-patterns-and-rdflib-does-not-move-it-earlier) | when a `FILTER` runs | after the group's patterns (scope) | ✘ after all of them | 1.3 to 1.5 times between forms. Jena ARQ: 0.9 to 1.1 times (TSD-57) | Source, Observed |
| [TSD-28](#tsd-28-presence-and-absence-tests-the-forms-differ-little) | forms of a presence or absence test | equal | 1.0 to 1.4 times | 1.1 to 1.2 times | Observed |
| [TSD-29](#tsd-29-two-parses-of-one-document-never-give-equal-triple-sets) | two parses of one document | blank nodes are fresh | fresh labels | fresh labels | Observed |
| [TSD-30](#tsd-30-a-querys-undeclared-prefixes-resolve-from-the-data-graphs-bindings) | a prefix the query does not declare | an error | resolved from the graph's bindings | not tested | Source, Observed |
| [TSD-31](#tsd-31-pyshacl-copies-the-data-graph-on-every-validation-unless-told-not-to) | copying the data graph for a validation | not specified | pySHACL copies it, 0.09 s for 5,000 to 14,000 triples | not applicable | Source, Observed |
| [TSD-32](#tsd-32-a-constant-filter-whose-value-is-false-is-dropped-by-rdflib) | `FILTER (false)` | removes every solution | ✘ removes none | ✔ | Source, Observed |
| [TSD-33](#tsd-33-on-oxigraph-count-over-a-statically-false-filter-returns-no-row) | `COUNT(*)` over a filter that is statically false | one row, 0 | ✔ except TSD-32 | ✘ no row | Observed |
| [TSD-34](#tsd-34-rdflib-evaluates-a-filter-not-exists-in-a-graph-block-against-the-wrong-graph-for-some-solutions) | `FILTER NOT EXISTS` without its own `GRAPH`, inside a `GRAPH` block that holds a sub-select and a `UNION` | tests the block's graph | ✘ sometimes tests the default graph, by hash seed | ✔ | Source, Observed |
| [TSD-35](#tsd-35-a-literal-as-the-graph-of-a-graph-block-is-refused-by-oxigraph-and-parsed-by-rdflib) | a literal as the graph of a `GRAPH` block | a syntax error | ✘ parsed, the block matches nothing | ✔ a syntax error | Observed |
| [TSD-36](#tsd-36-a-bound-object-does-not-give-the-seek-a-bound-subject-gives) | the same row reached by a bound object or by a bound subject | not specified | not tested | the bound object scans the predicate's matches, the bound subject seeks | Observed |
| [TSD-40](#tsd-40-a-zero-or-more-property-paths-cost-changed-between-oxigraph-0511-and-0513) | a `*` property path from a bound object | not specified | not tested | 2.4 times faster on 0.5.13 than on 0.5.11, where enumeration is unchanged | Observed |
| [TSD-41](#tsd-41-an-aggregate-over-an-expression-that-errors-on-one-row-of-a-group-is-unbound-on-oxigraph-and-skips-the-row-on-rdflib) | `GROUP_CONCAT`, `SUM` and `AVG` over an expression that errors on one row of the group. `COUNT` over such an expression | an error for `GROUP_CONCAT`, `SUM` and `AVG`. `COUNT` removes the error | ✘ skips the row. ✘ `COUNT` counts it | ✔ unbound. ✘ `COUNT` unbound | Source, Observed |
| [TSD-42](#tsd-42-oxigraph-reads-every-graph-for-a-graph-g-whose-g-is-bound-outside-a-computed-name-on-every-version-and-a-constant-one-inside-an-optional-from-0513) | `GRAPH ?g` with `?g` bound outside, by a constant or computed by an expression, in an `OPTIONAL` or the same group | joined afterwards | reads one graph (R-S, R-M) | a computed `?g`: ✘ reads every graph on 0.5.11 to 0.5.13. A constant `?g` in an `OPTIONAL`: passes it in, ✘ reads every graph from 0.5.13. Jena ARQ and RDF4J read one graph (R-W, R-N) | Observed |
| [TSD-43](#tsd-43-what-an-aggregate-does-with-a-row-that-has-no-value-or-errors-for-every-aggregate-and-what-each-remedy-does) | each aggregate over a group with an unbound row, a row of the wrong type and no row, and the four remedies | `COUNT` removes an error row. `SUM`, `AVG` (unless the count is 0) and `GROUP_CONCAT` are an error. `MIN` is unbound. `MAX` is `2` or unbound, the text allowing both | ✘ skips the row for `SUM`, `MIN`, `GROUP_CONCAT`, counts it for `COUNT(expr)`, raises for `SUM` over an IRI, returns a row for a grouped query over no rows. 54 of 91 cells conform | ✔ except `COUNT(expr)` and `AVG` over an all-unbound group. 84 of 91 conform. Jena ARQ 5.1.0: 84 of 91 | Source, Observed, Documented |
| [TSD-45](#tsd-45-oxigraph-passes-a-constant-graph-variable-into-an-optional-before-0513-and-a-computed-one-never) | recorded with TSD-42 | see TSD-42 | see TSD-42 | see TSD-42 | Observed |
| [TSD-48](#tsd-48-rdf4j-counts-0-for-count-over-a-pattern-with-no-variable-that-matches) | `COUNT(*)` over a pattern with no variable that matches | 1, from one empty solution | not tested | not tested. **RDF4J 6.1.0 gives 0** | Observed |
| [TSD-50](#tsd-50-on-jena-arq-in-memory-a-failed-request-is-rolled-back-and-clear-graph-of-an-absent-graph-fails-where-drop-graph-does-not) | a request whose second operation fails. `CLEAR GRAPH` and `DROP GRAPH` of a graph that is not there | should be atomic. A store that does not record empty graphs always returns success for both (§3.2.1, §3.2.2) | not tested | not tested. Jena ARQ in memory: rolled back. ✘ `CLEAR GRAPH` fails and `DROP GRAPH` succeeds | Observed |
| [TSD-51](#tsd-51-jena-arq-matches-a-zero-length-path-from-a-term-in-no-triple-with-the-term-itself) | zero-length path from a term in no triple | our reading: matches the term | as TSD-08 | as TSD-08. Jena ARQ: matches it, as rdflib does | Observed |
| [TSD-52](#tsd-52-jena-arq-keeps-xsdlong-counts-distinct-terms-groups-arithmetic-from-the-left-and-divides-integers-into-a-decimal) | `"1"^^xsd:long` stored and read, `COUNT(DISTINCT)` over `xsd:long` and `xsd:integer`, a chain of `-` and `+`, a division of integers | the term is kept, distinct is by term, from the left, `xsd:decimal` (§17.3) | as TSD-07, TSD-18 and TSD-17 | as TSD-07, TSD-18 and TSD-17. Jena ARQ: ✔ all four, so `"8 / 4 / 2"` is `"1.0"^^xsd:decimal` | Observed |
| [TSD-53](#tsd-53-jena-arq-applies-a-deleteinsert-in-the-specifications-order-skips-an-unbound-template-graph-and-tests-the-right-graph-in-tsd-34s-form) | probes P1 (order of an update), P4 (unbound graph variable in a template) and TSD-34's `FILTER NOT EXISTS` | as TSD-01, TSD-05 and TSD-34 | as TSD-01, TSD-05 and TSD-34 | as TSD-01, TSD-05 and TSD-34. Jena ARQ: ✔ all three | Observed |
| [TSD-60](#tsd-60-an-aggregate-over-if-of-an-unbound-variable-raises-on-rdflib-and-is-unbound-on-oxigraph-and-jena) | `SUM(IF(?d = "x", 1, 0))` and `MAX(IF(...))` over a row whose `?d` is unbound | an error, so the aggregate is unbound | ✘ `AttributeError` from `SUM` when the result is read, and no row from `MAX` | ✔ unbound. Jena ARQ 5.1.0 ✔ unbound | Source, Observed |
| [TSD-61](#tsd-61-rdflib-orders-a-string-against-a-number-where-the-specification-gives-a-type-error) | `"abc" >= 5`, `"abc" < 5` | a type error, so unbound | ✘ a boolean | ✔ unbound. Jena ARQ 5.1.0 ✔ unbound | Observed |
| [TSD-54](#tsd-54-a-blank-node-label-loaded-twice-is-one-node-on-rdflib-and-jena-with-labels-kept-and-two-on-oxigraph) | a blank node label in a document loaded twice into one store | a label is scoped to its document, so two nodes (RDF 1.1 N-Quads) | one node (N-Quads) | two nodes. Jena ARQ: two by default, one when the parser is given `LabelToNode.createUseLabelEncoded()` | Observed |
| [TSD-55](#tsd-55-rdflibs-n3-writes-a-multi-line-literal-in-triple-quotes-which-n-quads-refuses) | `Literal.n3()` of a literal holding a newline, used as an N-Quads term | N-Quads has no triple-quoted form | triple quotes (Notation3). `serialize(format="nt")` escapes | refuses the line. Jena ARQ refuses it | Observed |
| [TSD-56](#tsd-56-jena-arq-evaluates-a-sub-select-on-its-own-and-orders-iris-as-iris) | TSD-15 and TSD-16 on Jena | as TSD-15 and TSD-16 | as TSD-15 and TSD-16 | Jena ARQ: ✔ both, as Oxigraph | Observed |
| [TSD-57](#tsd-57-jena-arq-shows-none-of-the-growth-that-tsd-25-to-tsd-27-record-on-rdflib) | the cost forms of TSD-25 to TSD-27 and TSD-22 on Jena | not specified | as TSD-25 to TSD-27 | Jena ARQ: no growth with size, a constant 2 to 3 times in E2 only | Observed |
| [TSD-62](#tsd-62-fuseki-on-tdb2-applies-concurrent-updates-one-at-a-time-and-a-read-does-not-wait-for-them) | concurrent SPARQL updates to one TDB2 dataset on Fuseki | not specified | not applicable | not applicable | Observed (Fuseki 6.2.0, TDB2) |
| [TSD-63](#tsd-63-rdflib-rewrites-an-xsddatetimes-z-as-0000-when-it-parses) | the lexical form of an `xsd:dateTime` ending in `Z` | kept, a literal is its lexical form | ✘ rewritten as `+00:00` on parse | not tested | Observed |
| [TSD-44](#tsd-44-a-small-update-on-fuseki-tdb2-costs-its-commit-so-batching-updates-is-the-lever-for-write-throughput) | many small updates to one TDB2 dataset, one request each or batched | not specified | not applicable | not applicable | Observed (Fuseki 6.2.0, TDB2) |
| [TSD-65](#tsd-65-a-deleteinsert-fills-every-template-quad-once-per-solution-on-every-engine-measured-whether-or-not-the-quad-is-then-skipped-and-only-jena-applies-a-quad-with-no-variable-once-per-operation) | what a generated replace and four other forms cost at M = N up to 10,000, on six engines, for both families, with the empty cases and the zero-length path | each template filled once per solution | quadratic for every form that keeps the payload in the template | the same, and Jena applies a quad with no variable once | Source (Jena, Oxigraph) and Observed (six engines) |
| [TSD-66](#tsd-66-jena-tdb2-and-fuseki-over-it-apply-an-update-as-arq-in-memory-does-roll-back-a-request-whose-second-operation-fails-and-apply-a-request-the-client-abandoned) | TSD-53's order and skip, TSD-06's failing request, a blank node in a template, and an abandoned request, on TDB2 and Fuseki | all deletions then all insertions, the quad skipped, a request should be atomic, a blank node minted per solution | not applicable | not applicable | Source and Observed (Jena 5.1.0, TDB2 on disk, Fuseki main over HTTP) |
| [TSD-67](#tsd-67-rdf4j-610s-memorystore-applies-a-deleteinsert-one-solution-at-a-time-writes-a-template-quad-whose-graph-is-unbound-and-rolls-back-a-request-whose-second-operation-fails) | TSD-01, TSD-05 and TSD-06 on RDF4J, and a generated composite replace | as those, and the replace should leave only the payload | not applicable | not applicable | Observed (RDF4J 6.1.0 MemoryStore): ✘ per solution, ✘ the quad written, rolls back. ✘ the composite replace leaves the old members and the old head (addendum) |
| [TSD-71](#tsd-71-an-optional-join-and-a-union-that-stands-for-it-cost-differently-on-each-engine) | an `OPTIONAL` join against a `UNION` standing for it, in a summed read | not specified | `OPTIONAL` 1.2 times the plain sum, `UNION` 6.0 | `OPTIONAL` 4.1, `UNION` 1.7 (0.5.13) | Observed |
| [TSD-72](#tsd-72-a-running-balance-clamped-at-a-ceiling-is-not-one-sparql-query-and-the-ordered-read-that-replaces-it) | a running balance clamped at a ceiling | not expressible in SPARQL 1.1 | an ordered read folded by the caller, 1.2 to 1.6 times the sum | the same, 1.9 to 2.0 times (0.5.13). Jena from Java, 1.7 to 2.6 | Reasoned, Observed |

## 6. The findings

### 6.0 The shape of an entry

Each entry has the same parts, so a reader can find what they need.

- **Context.** What we were building or testing when it came up, and why it mattered then.
- **How we found it.** The steps, in order, and the evidence each gave.
- **What the store does.** The smallest SPARQL that shows it, and the result on each store.
- **Why.** The mechanism, from the source where we read it.

---

### TSD-01. The order in which a `DELETE`/`INSERT`/`WHERE` is applied

**Evidence:** Source and Observed, on both stores. Runs R-A and R-B. **Corrected 2026-10-13**, see §7.1. **Oxigraph 0.5.12 (2026-10-20, R-H):** Documented, Source and Observed. It applies the specification's order (step 7).

**Context.** A composite aggregate's replace deletes every triple of the aggregate and inserts the new state. A cheaper form of it would put the payload in `VALUES` rows beside the sweep's rows, so each payload triple is filled once. That form is right only if the store removes every deletion before it adds any insertion, because a payload often repeats a triple the aggregate already holds, an unchanged name for example. So we needed to know how each store applies an update.

**How we found it.**

1. *Source, rdflib.* We read `rdflib/plugins/sparql/update.py`, `evalModify`, in 7.6.0. After evaluating the `WHERE` it loops `for c in list(res):` and, inside the loop, does `dg -= _fillTemplate(u.delete.triples, c)` and then `dg += _fillTemplate(u.insert.triples, c)` (and the same for each graph's quads). So it removes and adds per solution.
2. *A first probe.* The probe P1 stores one triple, then runs an update whose two solutions, written as `VALUES` rows, respectively insert and delete that triple. Under the specification the triple survives in either row order. In run R-A rdflib kept it in both orders, and we concluded that rdflib is snapshot. That conclusion was wrong.
3. *The contradiction.* In run R-B the same probe lost the triple on rdflib. A trace of the order of solutions, and then the same probe run with `PYTHONHASHSEED` fixed to each of 0 to 11, showed the result changing with the seed: lost in both orders for 6 seeds, kept in both for 4, and dependent on the row order for 2. The probe's test, which asserted "kept", failed in 2 of 3 runs with no seed set.
4. *Source again.* rdflib translates the probe's `WHERE { VALUES … }` to `Join(BGP{}, ToMultiSet(Values))`, and the join is not marked lazy. `evaluate.py`, `evalJoin`, then does `b = set(evalPart(ctx, join.p2))`. A Python `set` of solutions iterates in an order that follows the hash of each solution, and string hashes change with `PYTHONHASHSEED` per process. So the order of solutions, and with it the result of the update, can change from one process to the next.
5. *Source, Oxigraph.* In Oxigraph 0.5.11, `lib/oxigraph/src/sparql/update.rs`, `eval_delete_insert` does `prepared.execute(...).collect::<Result<Vec<_>, _>>()?` to gather every computed `DeleteInsertQuad`, then `for mutation in mutations` applies each `Delete` or `Insert` in turn. The reads happen before any write. The order inside the `Vec` is the order of solutions, deletions then insertions for each, which the probe confirms.
6. *Observed, Oxigraph.* P1 on Oxigraph loses the triple when the inserting row comes first and keeps it when the deleting row comes first, every time.
7. *Oxigraph 0.5.12, 2026-10-20.* A fresh bootstrap installed pyoxigraph 0.5.12, which the adapter's version range (`>=0.5.11,<0.6`) allowed, and the P1 tests failed on Oxigraph. P1 kept the triple in both row orders. The release notes of 0.5.12 say "SPARQL Update: apply all deletions before insertions", and in `lib/oxigraph/src/sparql/update.rs` at the tag `0.5.12`, `eval_delete_insert` still collects every mutation into a `Vec`, then loops over it twice, removing each `Delete` in the first loop and adding each `Insert` in the second. With 0.5.11 reinstalled, P1 behaved as in step 6 again.

**What the store does.**

```sparql
# stored beforehand: GRAPH <urn:g> { <urn:s> <urn:p> <urn:o> }
DELETE { GRAPH <urn:g> { ?d <urn:p> <urn:o> } }
INSERT { GRAPH <urn:g> { ?i <urn:p> <urn:o> } }
WHERE  { VALUES (?i ?d) { (<urn:s> UNDEF) (UNDEF <urn:s>) } }
```

| Store | Row order as written | Is `<urn:s> <urn:p> <urn:o>` still there? |
|---|---|---|
| specification | either | yes |
| rdflib 7.6.0 | either | depends on the process's hash seed |
| Oxigraph 0.5.11 | inserting row first | no |
| Oxigraph 0.5.11 | deleting row first | yes |
| Oxigraph 0.5.12 | either | yes |

```mermaid
sequenceDiagram
  participant W as WHERE
  participant R as rdflib evalModify
  participant G as graph
  W->>R: solutions, from a set, in hash order
  loop each solution in turn
    R->>G: remove this solution's deletions
    R->>G: add this solution's insertions
  end
  Note over R,G: insert row then delete row, the triple is lost
  Note over R,G: delete row then insert row, the triple survives
```

```mermaid
sequenceDiagram
  participant W as WHERE
  participant X as Oxigraph eval_delete_insert
  participant G as store
  W->>X: solutions, in evaluation order
  X->>X: collect every Delete and Insert into one Vec
  loop each mutation in the Vec
    X->>G: apply it
  end
  Note over X,G: the Vec keeps solution order, so the last word on a triple wins
```

To reproduce the rdflib result, run the update twelve times per row order, each time with a different `PYTHONHASHSEED`, and expect a mix of outcomes. A store that applied the specification's order would keep the triple every time.

**Why.** rdflib 7.6.0 and Oxigraph 0.5.11 do not keep the specification's two phases. rdflib interleaves per solution, and its solution order for a non-lazy join comes from a Python `set`. Oxigraph separates reading from writing, which protects the `WHERE` from the update's own changes, and 0.5.11 then applied the collected mutations in solution order rather than all deletions first. 0.5.12 applies all the deletions and then all the insertions, which is the specification's order.

Two consequences follow for a generated replace. A form that puts the whole payload in the `INSERT` template stays right on both, because every solution adds every payload triple again and the last solution restores anything an earlier one removed, and that redundancy is what TSD-02 measures. A form that carries the payload in `VALUES` rows is unsafe on rdflib 7.6.0 and Oxigraph 0.5.11, and safe on Oxigraph 0.5.12. A test of order-dependent behaviour on rdflib must fix the hash seed or sweep it.

---

### TSD-02. What filling a template once per solution costs

**Evidence:** Source and Observed, on both stores. Run R-A for the benchmark, R-B for the first measurement.

**Context.** A composite replace sweeps the root and every member of an aggregate in one `OPTIONAL` pattern, which gives one solution per stored triple, M of them. Its `INSERT` template holds the N payload triples, and none of them uses a sweep variable. So, by the definition in §4.1, the payload is filled M times before the union removes the copies. Whether a store does that literally was the question.

```sparql
DELETE { GRAPH <data> { ?s ?p ?o } … }
INSERT { GRAPH <data> { …N payload triples… } … }
WHERE  {
  … the version-row guard, one solution …
  OPTIONAL { GRAPH <data> { $root <ownedPath> ?s . ?s ?p ?o } }   # M solutions
}
```

```mermaid
flowchart TB
  G["guards<br/>1 solution"] --> J["join"]
  SW["sweep<br/>M stored triples"] --> J
  J --> SOL["M solutions"]
  SOL --> DEL["DELETE fills: M"]
  SOL --> INS["INSERT fills: M x N<br/>the whole payload each time"]
```

**How we found it.**

1. *Source, rdflib.* `evalModify`'s per-solution loop (TSD-01) calls `_fillTemplate` on the whole `INSERT` template for every solution, and `Graph.__iadd__` adds each filled triple.
2. *Observed, rdflib, R-B.* A scratch script reduced the replace to its essentials and timed it at M = N: 0.007 s at 10, 0.077 s at 100, 5.6 s at 1000. Ten times the size took about seventy times as long.
3. *Source, Oxigraph.* `eval_delete_insert` collects every filled quad into a `Vec` before applying any (TSD-01), so the M × N filled quads exist in memory at once.
4. *Observed, both, R-A.* A benchmark ran M = N from 10 to 10,000, each size in a child process capped at 8 GB.

**What the store does.**

| M = N | rdflib 7.6.0 | Oxigraph 0.5.11 |
|---|---|---|
| 10 | 0.03 s | 0.0005 s |
| 100 | 0.17 s | 0.0056 s |
| 1,000 | 9.2 s | 0.59 s |
| 3,000 | 83 s | 14.4 s |
| 10,000 | 983 s | did not finish |
| fitted growth k, 100 and up | 1.87 | 2.26 |

At 10,000 Oxigraph tried to allocate about 7 GB and failed under the cap, and an uncapped run was killed at 14 GB on a 15 GB machine. Separately, a run at 2,000 used 1.5 GB and one at 4,000 used 5.8 GB, so memory grows as the square of the size, and 10,000 would need about 36 GB. Every finished run left exactly the payload, the bystander aggregate untouched and the version row at the next sequence.

**Why.** Both stores fill the template once per solution, as the specification's definition reads. rdflib pays in time, adding the same triples again and again. Oxigraph is fast per quad but holds every filled quad at once.

A two-operation form of the replace is safe only where a failing request rolls back (TSD-06), which Oxigraph does and rdflib does not. So no single linear form fits both stores. A benchmark of this kind should fail on a wrong result and never on time or memory, and a run that does not finish is recorded with its exit status.

---

### TSD-03. What a `BIND` inside an inner group can see

**Evidence:** Observed on both stores, R-B. rdflib is also covered by tests.

**Context.** A static check tracks, for each point of a `WHERE`, which variables are bound in every solution, so it can refuse a template variable that might not be bound. Its first version let an inner group see the enclosing group's variables. A review then "fixed" the `GRAPH` block the same way. Both were wrong, and only running queries showed it (§7.2).

**How we found it.** We ran the same six queries on each store, each binding `?a` outside and computing `?b` from it inside.

**What the store does.**

| Query | rdflib | Oxigraph | Specification |
|---|---|---|---|
| `BIND(1 AS ?a) { BIND(?a + 1 AS ?b) }` | `?b` unbound | `?b` unbound | unbound |
| `BIND(1 AS ?a) GRAPH <g> { ?s ?p ?o BIND(?a + 1 AS ?b) }` | unbound | unbound | unbound |
| `BIND(1 AS ?a) OPTIONAL { BIND(?a + 1 AS ?b) }` | unbound | unbound | unbound |
| `BIND(1 AS ?a) { BIND(?a + 1 AS ?b) } UNION { BIND(5 AS ?b) }` | unbound and 5 | not run | unbound and 5 |
| `BIND(1 AS ?a) BIND(?a + 1 AS ?b)` | 2 | 2 | 2 |
| `BIND(1 AS ?a) FILTER NOT EXISTS { FILTER(?a = 1) }` | no row, so `?a` is seen | no row | no row |

Each row is one `SELECT ?a ?b WHERE { … }` over any dataset with one triple in `<g>`. Select `?a` as well as `?b`, or rdflib drops the row (TSD-10).

**Why.** The algebra of §4.2. Both stores follow it. So a nested group, an `OPTIONAL`, a `UNION` branch and a `GRAPH` block start with nothing in scope, and only `NOT EXISTS` starts from the enclosing scope.

---

### TSD-04. A graph variable bound outside an inner group

**Evidence:** Observed on both stores, R-A and R-B.

**Context.** A named-graph replace sweeps the root's graph with `OPTIONAL { GRAPH ?g { ?s ?p ?o } }`, where `?g` is bound by a `BIND` at the top of the `WHERE`. By §4.2 the inner group is evaluated on its own, so `GRAPH ?g` there ranges over every graph until the join narrows it. The result is right either way. The question was the cost, and whether a static check should refuse the form.

**How we found it.**

1. *Reasoned.* rdflib's translation of the template and of the form the check already accepted, `BIND(… AS ?g) GRAPH ?g { … }`, differ only in `LeftJoin` against `Join`. Both are `Extend[?g]` beside an independent `Graph[?g]`, so neither is cheaper by the specification.

   ```mermaid
   flowchart TB
     subgraph accepted["same group"]
       J1["Join"] --> E1["Extend ?g"]
       J1 --> G1["Graph ?g"]
     end
     subgraph template["inside OPTIONAL"]
       J2["LeftJoin"] --> E2["Extend ?g"]
       J2 --> G2["Graph ?g"]
     end
   ```

2. *Observed, rdflib, R-B.* On 50 graphs of 200 triples, both forms returned the 200 triples of the bound graph in about 10 ms, against 237 ms for `GRAPH ?g { ?s ?p ?o }` with nothing bound.
3. *Observed, both, R-A.* Probe P3 timed the `OPTIONAL` form over 100 graphs of 3,000 triples against 1 graph. The ratio was 1.1 on both stores.

To repeat the timing, load 100 graphs of 3,000 triples, then time `SELECT (COUNT(*) AS ?n) WHERE { BIND(<urn:g:0> AS ?g) OPTIONAL { GRAPH ?g { ?s ?p ?o } } }` against the same with one graph loaded, taking the best of three. A ratio near 1 means the binding is passed in.

**What the store does.** Both pass the outer binding of `?g` into the graph pattern, so they read one graph. Oxigraph stopped doing so for a left join in 0.5.13 (TSD-42). A store that followed the algebra literally would read every graph and then discard most of what it read.

**Why.** Not read for either store. The timings show both pass the left side's bindings into the right side, as an engine does when it evaluates a join lazily (rdflib has `evalLazyJoin` for that, and TSD-Q1 asks when it is used).

**Correction, 2026-10-28.** The Oxigraph half of this finding holds on 0.5.11 and 0.5.12 for a `?g` bound to a constant, which is the form the probe used. It does not hold on 0.5.13, and it never held for a `?g` computed by an expression, which is the form a generator emits (TSD-42). The result is unchanged and only the cost differs. The rdflib half was re-run for both forms and holds.

---

### TSD-05. A template quad whose graph variable is unbound

**Evidence:** Source (rdflib) and Observed, both stores, R-A and R-B.

**Context.** We tested what an unguarded write does when the caller leaves out `$root`. The graph name `?g` is computed from `$root`, so it is unbound.

**How we found it.** A test expected nothing to be written and found a new graph. A direct probe confirmed it, and probe P4 now checks both stores.

**What the store does.**

```sparql
INSERT { GRAPH ?g { <urn:s> <urn:p> <urn:o> } } WHERE { }
```

| Store | Result |
|---|---|
| specification | skipped, since the quad has an unbound variable |
| rdflib 7.6.0 | the triple is written into a new graph named by a blank node |
| Oxigraph 0.5.11 | skipped, nothing written |

Run the update on an empty `Dataset` and list `ds.graphs()` with at least one triple to see the rdflib result.

**Why.** rdflib's `evalModify` does `cg = ctx.dataset.get_context(c.get(g))` for each graph of the template (Source). With `?g` unbound, `c.get(g)` is `None`, and the observed result is a graph with a fresh blank-node name, so `get_context(None)` must create one. We did not read `get_context` itself. With parameter guards on, a missing `$root` is refused before this point. Without them, on rdflib, it leaves an orphan graph that no audit names.

**Addendum, 2026-10-31.** What this costs a form that relies on the skip. A replace whose `INSERT` blocks name their graphs by variables bound in one branch of a `UNION`, so that the sweep's solutions leave them unbound (form F2′ of TSD-65), wrote the whole `INSERT` once per sweep solution on rdflib 7.6.0. At M = 100 stored triples the quads outside the replace's own graphs went from 100 to 21,300, which is 100 solutions times the template's 213 quads, each into a fresh blank-node graph, under every one of twelve hash seeds. A form that depends on the skip therefore needs it probed, P4, beside the order, P1.

---

### TSD-06. A request whose second operation fails, and graph management

**Evidence:** Observed on both stores, R-A and R-B.

**Context.** A two-operation form of a replace is safe only if a request whose second operation fails leaves no trace of its first. SPARQL 1.1 Update says a request *should* be atomic. Probe P2 asks each store.

**How we found it.** P2 sends `INSERT DATA { … } ; <a failing operation>` and reads back. Finding a failing operation took two tries on rdflib (TSD-14).

**What the store does.**

| Store | Failing second operation | The request | The first insert |
|---|---|---|---|
| rdflib 7.6.0 | `LOAD <urn:no-such>` | fails | kept |
| Oxigraph 0.5.11 | `DROP GRAPH <urn:absent>` | fails with `RuntimeError` | rolled back |

```mermaid
sequenceDiagram
  participant C as caller
  participant S as store
  C->>S: INSERT DATA t, then a failing operation, in one request
  alt rdflib
    S-->>C: error
    Note over S: t is stored
  else Oxigraph
    S-->>C: error
    Note over S: t is not stored
  end
```

On each store, run the request and then check whether `<urn:s> <urn:p> <urn:o>` is in `<urn:g>`.

**Why.** We did not read either store's transaction code. Oxigraph applies an update inside a transaction (TSD-01's source shows `self.transaction`), which is consistent with the rollback. This is only the failure half of request atomicity. Its isolation half, what a concurrent writer sees between the two operations, needs concurrent writers.

**Other stores, from their documentation (Documented, not probed, gathered 2026-10-14).** SPARQL 1.1 Update says a request *should* be treated atomically, a recommendation the working group chose not to make a requirement ([working draft of October 2010](https://www.w3.org/TR/2010/WD-sparql11-update-20101014/), [a comment on the list](https://lists.w3.org/Archives/Public/public-rdf-dawg-comments/2011May/0012.html)). So it is each store's choice.

| Store | What its documentation says | Reading |
|---|---|---|
| Apache Jena, TDB2 | TDB2 is transactional only, with serializable transactions that cover SPARQL Update ([TDB transactions](https://jena.apache.org/documentation/tdb/tdb_transactions.html), [Jena transactions](https://jena.apache.org/documentation/txn)) | atomic per request, by inference. The documentation does not name the request as the transaction's scope |
| Amazon Neptune | a community answer says the operations of one request run as one transaction and roll back together ([Repost](https://www.repost.aws/questions/QUE09h4w48RFOW6I8RrId6SQ/are-multiple-sparl-update-queries-in-same-post-message-one-transaction)). The official page states strict isolation without that detail ([transaction semantics](https://docs.aws.amazon.com/neptune/latest/userguide/transactions.html)) | atomic per request, unconfirmed |
| Virtuoso | SPARQL updates "are generally not meant to be transactional", and the recommended mode, `log_enable(2)`, commits every operation as it is done. One mode autocommits per triple inserted or deleted ([tip](https://docs.openlinksw.com/virtuoso/virtuosotipsandtrickssparulupdatestrl), [`log_enable`](https://docs.openlinksw.com/virtuoso/fn_log_enable)) | **not atomic per request** in its recommended mode, and possibly not atomic within one operation (TSD-Q13) |

So atomicity is a property of the store and, on Virtuoso, of its configuration. A generated operation that needs two operations to commit together cannot assume it, and it must be known per deployment, not per product.

---

### TSD-07. Numeric datatypes are not kept by Oxigraph

**Evidence:** Observed on both stores, R-A and R-B.

**Context.** Sequences, epochs and positions are pinned to `xsd:long`, because `"1"^^xsd:long` and `"1"^^xsd:integer` are different RDF terms and a pattern that matches by term will not match one with the other. A parameter guard tested `DATATYPE($epoch) = xsd:long` for that reason. On Oxigraph every guarded operation wrote nothing, and probe P5 found why.

**What the store does.**

```sparql
INSERT DATA { GRAPH <urn:g> { <urn:s> <urn:p> "1"^^<http://www.w3.org/2001/XMLSchema#long> } }
SELECT (DATATYPE(?o) AS ?d) WHERE { GRAPH <urn:g> { ?s ?p ?o } }
```

| Store | Stored object's datatype | `DATATYPE(?o)` |
|---|---|---|
| rdflib 7.6.0 | `xsd:long` | `xsd:long` |
| Oxigraph 0.5.11 | `xsd:integer` | `xsd:integer` |

A guard `DATATYPE($p) = xsd:long` could therefore never pass on Oxigraph. The guard `DATATYPE($p) = DATATYPE("0"^^xsd:long)` compares the parameter with what the store makes of a constant. P5 asserts that a constant, a substituted parameter, a `VALUES` row and a stored value report one datatype on each store, and that a simple literal stays `xsd:string` in every form. Because every number on Oxigraph is an `xsd:integer`, requests and stored rows match each other there. What stays is that a pinning to `xsd:long` cannot be checked on Oxigraph.

**Why.** Not read. The observation is that Oxigraph stores an integer-derived literal as `xsd:integer`, and reports it so in every expression.

---

### TSD-08. A zero-length path from a term that is in no triple

**Evidence:** Observed on both stores, R-A and R-B. The specification's reading is ours and is not yet checked against the text.

**Context.** A sweep starts with `$root <ownedPath> ?s`, and the owned path can take zero steps, so `?s` includes the root itself. A test asked what happens when the root is in no triple of the data graph.

**What the store does.**

```sparql
# the graph <urn:g> holds only <urn:a> <urn:p> <urn:b>
SELECT ?x WHERE { GRAPH <urn:g> { <urn:lonely> <urn:p>* ?x } }
```

| Store | `?x` |
|---|---|
| rdflib 7.6.0 | `<urn:lonely>` |
| Oxigraph 0.5.11 | no rows |

A sweep that also needs `?s ?p ?o` to match is unaffected, since a term in no triple has none.

---

### TSD-09. Unicode escapes and an escaped backslash

**Evidence:** Observed on both stores, R-B. Found by a property test.

**Context.** A literal encoder escaped a backslash as `\\`. A value holding a backslash followed by `u0041` therefore became the text `"\\u0041"`. SPARQL 1.1 §19.2 says `\u` escapes are processed before the rest of the grammar.

**What the store does.**

| Query text | rdflib 7.6.0 | Oxigraph 0.5.11 |
|---|---|---|
| `"\\u0041"` | syntax error | the five characters `A` |
| `"A"` | `A` | `A` |

```mermaid
flowchart LR
  T["query text<br/>backslash backslash u0041"] --> RP["rdflib: a pre-pass over<br/>the whole text replaces<br/>backslash u0041 first"]
  RP --> RE["leaves a lone backslash<br/>before A: syntax error"]
  T --> OP["Oxigraph: escapes read<br/>inside the string token"]
  OP --> OE["escaped backslash, then u0041:<br/>the value keeps them"]
```

To see it, run `INSERT DATA { GRAPH <urn:e> { <urn:s> <urn:p> "\\u0041" } }` on each store, then read the value back. Code that writes SPARQL text must assume the stricter rdflib reading.

---

### TSD-10. A result row whose projected variables are all unbound

**Evidence:** Observed on both stores, R-A and R-B. Found while making a test of TSD-03 pass.

**What the store does.** `SELECT ?b WHERE { BIND(1 AS ?a) { BIND(?a + 1 AS ?b) } }` has one solution, with `?b` unbound. Oxigraph returns one row with `?b` empty. rdflib returns no rows. A test that counts rows on rdflib must therefore project at least one variable it knows is bound.

---

### TSD-11. `BOUND` and text substitution

**Evidence:** Observed on both stores, R-B.

**Context.** The first parameter guard was `FILTER(BOUND($root) && …)`. A caller that substitutes text writes `FILTER(BOUND(<urn:x>) && …)`.

**What the store does.** Both refuse it as a syntax error (rdflib `ParseException`, Oxigraph `SyntaxError`), as the grammar requires, since `BOUND` takes a variable. A type test such as `isIRI(<urn:x>)` parses, and `isIRI(?root)` with `?root` unbound is an error, which fails the filter. A guard that survives text substitution is therefore a type test per parameter, never `BOUND`.

---

### TSD-12. Initial bindings are visible inside inner groups on rdflib

**Evidence:** Observed on rdflib, R-B. Oxigraph not tested, since our adapter substitutes text.

**What the store does.** With `initBindings={"root": <urn:a>}`, `{ FILTER(isIRI(?root)) }` in its own inner group passes on rdflib, so the initial binding is seen inside the group. An engine that joined initial bindings in only at the end would make the guard fail every time, which would fail closed. A guard in its own group therefore works for an initial-binding caller on rdflib and for a text-substituting caller anywhere. Another engine's initial-binding API must be probed before it is relied on.

---

### TSD-13. A filter at the top of a group sees what the group's patterns bound

**Evidence:** Observed on both stores, R-B.

**Context.** The first guard was a `FILTER` at the top of the `WHERE`, whose patterns also matched `$root`.

**What the store does.**

```sparql
# <urn:a> and <urn:b> each have <urn:seq> 0 in <urn:m>, and ?root is not bound by the caller
INSERT { GRAPH <urn:m> { ?root <urn:w> true } }
WHERE  { FILTER(BOUND(?root)) GRAPH <urn:m> { ?root <urn:seq> ?s } }
```

| Form | rdflib | Oxigraph |
|---|---|---|
| `FILTER` at the top of the group | writes both `<urn:a>` and `<urn:b>` | writes both |
| `{ FILTER(…) }` as its own group | writes nothing | writes nothing |

```mermaid
flowchart TB
  subgraph top["FILTER at the top of the group"]
    P1["patterns bind ?root<br/>from the data"] --> F1["FILTER applied after,<br/>?root is bound"]
    F1 --> W1["every aggregate written"]
  end
  subgraph own["FILTER in its own group"]
    F2["inner group alone:<br/>?root unbound, FILTER fails"] --> J2["join with nothing"]
    J2 --> W2["nothing written"]
  end
```

A guard written as its own group runs once per request, where the top-level form ran once per solution and cost about 60% on a 20,000-triple sweep on rdflib.

---

### TSD-14. Graph management operations on rdflib

**Evidence:** Observed, R-A.

**What the store does.** On rdflib 7.6.0, `CREATE GRAPH` is not implemented, and `DROP GRAPH` of a graph that does not exist succeeds. On Oxigraph 0.5.11, `DROP GRAPH` of an absent graph fails, as the specification says it should without `SILENT`. A probe or a test that needs an operation to fail at run time on rdflib can use `LOAD` of a source that does not exist.

---

### TSD-15. A sub-select and the variables around it

**Evidence:** Observed on both stores, R-C. Answers TSD-Q9.

**Context.** A merge and a quarantine operation join a sub-select, `{ SELECT ?claim ... GROUP BY ?claim HAVING ... }`, to the patterns around it. A static check treats a sub-select as evaluated on its own, so it starts with nothing in scope and only its projections leave it, and it refuses an inner variable that is also used outside without being projected.

**What the store does.**

```sparql
# <urn:g2> holds three subjects, <urn:s1> to <urn:s3>
SELECT ?c WHERE { VALUES ?s { <urn:s1> } { SELECT (COUNT(*) AS ?c) WHERE { GRAPH <urn:g2> { ?s <urn:p> ?o } } } }
SELECT ?c WHERE { GRAPH <urn:g1> { { SELECT (COUNT(*) AS ?c) WHERE { ?s ?p ?o } } } }   # <urn:g1> holds two triples
```

| Query | Specification | rdflib | Oxigraph |
|---|---|---|---|
| the count with `?s` bound outside the sub-select | 3, since `?s` inside is another variable | **1**, the outer binding is let in | 3 |
| the count inside `GRAPH <urn:g1>` | 2, the sub-select reads the active graph | 2 | 2 |

**Jena ARQ 5.1.0 (R-X).** 3 and 2, as Oxigraph (TSD-56).

A sub-select that reuses an outer variable's name without projecting it therefore gives different results on the two stores. A sub-select inside a `GRAPH` block reads that graph on both.

---

### TSD-16. `MIN` and `MAX` over IRIs

**Evidence:** Observed on both stores, R-C. Answers TSD-Q6.

**Context.** A merge picks a claim's canonical owner with `MIN(?owner)` over the owners' IRIs, then compares it with each owner and writes a link from every other owner to it.

**What the store does.**

```sparql
INSERT DATA { GRAPH <urn:g> { <urn:c> <urn:p> <urn:o:2> , <urn:o:10> , <urn:o:1> } }
SELECT (MIN(?o) AS ?l) (MAX(?o) AS ?h) WHERE { GRAPH <urn:g> { <urn:c> <urn:p> ?o } }
```

| Store | `MIN` | `MAX` |
|---|---|---|
| rdflib 7.6.0 | the literal `"urn:o:1"` | the literal `"urn:o:2"` |
| Oxigraph 0.5.11 | the IRI `<urn:o:1>` | the IRI `<urn:o:2>` |

Both choose by the IRI's text, so `urn:o:10` sorts before `urn:o:2`. `IRI(MIN(STR(?o)))` gives the IRI on both. Jena ARQ 5.1.0 gives the IRIs `<urn:o:1>` and `<urn:o:2>`, as Oxigraph does (R-X, TSD-56).

A first port of the merge equalled its template, so on rdflib it wrote a plain literal as the link's object, and because an IRI is never equal to a literal, `?other != ?canonical` also held for the canonical owner, which got a link to itself. On Oxigraph it wrote IRIs and no self-link. Projecting `IRI(MIN(STR(?owner)))` gives an IRI on both, though the `MIN` itself is still different on the two stores.

---

### TSD-17. A chain of additions and subtractions

**Evidence:** Observed on Oxigraph, R-C, directly through `pyoxigraph`, and on rdflib. **Oxigraph 0.5.12 (2026-10-20, R-H):** Observed and Documented. It groups from the left. The release notes of 0.5.12 say "group operators of equal precedence from left to right".

**Context.** A gap scan compares `COUNT(DISTINCT ?seq)` with `?S - ?L + 1`. On Oxigraph every audit row for a complete stream was reported as a gap.

**What the store does.**

```sparql
SELECT (10 - 3 + 2 AS ?x) WHERE {}      SELECT (10 - 3 - 2 AS ?x) WHERE {}      SELECT (8 / 4 / 2 AS ?x) WHERE {}
```

| Query | Specification (from the left) | rdflib | Oxigraph 0.5.11 | Oxigraph 0.5.12 |
|---|---|---|---|---|
| `10 - 3 + 2` | 9 | 9 | **5**, as `10 - (3 + 2)` | 9 |
| `10 - 3 - 2` | 5 | 5 | **9**, as `10 - (3 - 2)` | 5 |
| `8 / 4 / 2` | 1 | 1 | **4**, as `8 / (4 / 2)` | 1 |
| `(10 - 3) + 2` | 9 | 9 | 9 | 9 |

The gap scan's template, written as a chain, counted wrongly on Oxigraph 0.5.11. Writing the parentheses of a nested arithmetic operand, as `(?S - ?L) + 1`, is correct on either store and on either version.

---

### TSD-18. `COUNT(DISTINCT)` over `xsd:long` and `xsd:integer` terms

**Evidence:** Observed on both stores, R-C. Answers TSD-Q10 for the counts.

**What the store does.** Three triples with the objects `"1"^^xsd:long`, `"1"^^xsd:integer` and `"2"^^xsd:long`, counted with `COUNT(DISTINCT ?o)`.

| Count | rdflib | Oxigraph |
|---|---|---|
| over the stored objects | 3 | 2, since it stores both `1`s as `xsd:integer` (TSD-07) |
| over a `VALUES` row of `"1"^^xsd:long` and `"1"^^xsd:integer` | 2 | 1 |

A gap scan that counts distinct `seq` terms therefore sees a term written with another integer type as one more on rdflib and none on Oxigraph, so a duplicate `seq` written that way is reported on rdflib only. Comparisons with `>=` and `<=` work across numeric types on both.

---

### TSD-19. The order of `GROUP_CONCAT`

**Evidence:** Observed on rdflib, R-C. Answers TSD-Q7 for the query shape of a duplicate audit.

**What the store does.** Six owners stored in a fixed order and concatenated by a duplicate audit's query, run in child processes under `PYTHONHASHSEED` 0 to 5 (12 seeds in the exploration), gave the same string every time, in the order the owners were stored. Unlike the update of TSD-01, this query does not go through a hash-ordered join. We take the order to be the store's scan order and no guarantee, and a different query shape or store could differ.

---

### TSD-20. A `FILTER` inside `NOT EXISTS` and a parameter

**Evidence:** Observed on both stores, R-C. Answers TSD-Q8.

**What the store does.** `FILTER NOT EXISTS { GRAPH <g> { ?claim <claimedBy> ?other . FILTER(?other != $owner) } }`, after a sub-select that projects `?claim`, keeps a claim held by the owner or by nobody and drops one held by someone else. The result is the same when `$owner` is put in the text and when it is an initial binding (rdflib).

### TSD-21. An empty `VALUES` block in an update

**Evidence:** Observed on both stores, rdflib 7.6.0 and Oxigraph 0.5.x.

**What the store does.** `VALUES (?a ?b) { }` as the first element of an update's `WHERE`, with a `DELETE`/`INSERT` template after it, gives no solution on Oxigraph, so nothing is written. rdflib raises an `AttributeError` while evaluating the block. It does so before it changes anything, so the store is left as it was. An append with an empty rows slot therefore writes nothing on either store, and on rdflib it raises.

---

### TSD-22. rdflib parses and translates a query's text on every call

**Evidence:** Source (rdflib 7.6.0) and Observed, R-D and R-E. A property of rdflib. Oxigraph was timed from text only.

**Context.** A test suite that validates shapes with pySHACL on rdflib took 427 s. One validation of a fixture took 13 s, and we wanted to know where.

**How we found it.**

1. `cProfile` of that one test. Under the profiler, which inflates figures two to three times, the pySHACL call took 34 s of 36 s, and 26.6 s of it was `pyparsing`, the library rdflib's SPARQL grammar is written in.
2. The profile's callers table. 937 calls of `Processor.query`, each calling `parseQuery`, and `translateQuery` another 3.9 s.
3. A probe that counted the texts: 937 calls, 30 distinct texts.
4. A prototype that translated each distinct text once. The test went from 13.2 s to 1.8 s with the same result, and the whole suite from 427 s to 128 s.

**What the store does.** The same twenty-line constraint query, run for one focus node (experiment E5).

| Way of running it | Milliseconds per run |
|---|---|
| rdflib, text each time | 20.4 |
| rdflib, prepared once with `prepareQuery` | 0.48 |
| Oxigraph, text each time (parse and run) | 0.17 |

**Why.** `rdflib/plugins/sparql/processor.py`, `SPARQLProcessor.query` (line 144). For a `str` it evaluates `translateQuery(parseQuery(strOrQuery), base, initNs)`, and for an already translated query it skips both. The grammar is `pyparsing`, in Python. We did not read Oxigraph's parser. Its figure is consistent with a compiled one.

Parsing and translating is most of what a SHACL-SPARQL validation costs on rdflib (TSD-23 says why it happens so often). This is not a property of SPARQL or of SHACL, and a different engine or validator may not have the cost.

### TSD-23. pySHACL runs a SPARQL constraint once per focus node

**Evidence:** Source (pySHACL 0.40.1) and Observed, R-D. A property of pySHACL.

**Context.** TSD-22 asked why a validation issued 937 queries.

**What the tool does.** One constraint, 40 focus nodes, a shapes graph with a single `sh:sparql` constraint: 40 calls of `Graph.query`, all with one text. We counted them by wrapping `Graph.query`.

**Why.** `pyshacl/constraints/sparql/sparql_based_constraints.py`, `_evaluate_sparql_constraint` (line 113), loops over the focus nodes. For each it calls `sparql_query_helper.py`, `pre_bind_variables` (line 280), which runs `check_invalid_sparql` (a set of regular-expression checks of the text, about 0.4 ms a call in the profile of TSD-22) and returns `$this` as an initial binding, and then `_validate_sparql_query` (line 169) calls `Graph.query(text, initBindings=...)`. The SHACL specification defines a SPARQL-based constraint by its result for one focus node, with `$this` pre-bound, and says nothing about batching. Running once per node is pySHACL's choice.

The cost of a node shape is therefore the number of its focus nodes times the cost of one run, and a shape that targets a common class has one focus node per instance of that class in the data. A rule that targeted `skos:Concept` ran 15,713 times over 21 validations. Whether Jena SHACL or another validator also runs once per node is not tested (TSD-Q14).

### TSD-24. The PREFIX block pySHACL writes changes order between validations

**Evidence:** Source (pySHACL 0.40.1) and Observed, R-D. A property of pySHACL.

**Context.** After TSD-22's cache went in, one group of tests still made 512 parses, and 511 of the texts were distinct.

**What the tool does.** pySHACL writes the constraint's `PREFIX` declarations in front of its `sh:select` text. Twelve validations of one shapes document, parsed afresh each time, wrote the same five prefixes in two different orders (nine times and three times). A text is therefore not a stable cache key.

**Why.** `pyshacl/helper/sparql_query_helper.py`, `__init__` (line 57), starts a dict with `rdf`, `rdfs` and `owl`, and adds each `sh:declare` it finds by iterating the shapes graph. `apply_prefixes` (line 200) writes the dict in that order. We did not trace why the order changes. The likely cause is that the declarations are blank nodes, whose labels differ between parses, and that the graph yields triples in the order of its indexes (Reasoned).

### TSD-25. rdflib fixes the order of a group's patterns before it knows the initial bindings

**Evidence:** Source (rdflib 7.6.0) and Observed on both stores, R-D. A property of rdflib's evaluator. Oxigraph showed no sensitivity.

**Context.** Two shape queries cost far more than the work they did. A reference-code uniqueness query took 2.0 s over 405 focus nodes, and a documentation rule spent 0.7 ms of its 1.4 ms on a single type test.

**What the store does.** A uniqueness test on nodes of one scheme, in two forms with the same solutions (experiment E2). Milliseconds per focus node, with `$this` as the focus.

```sparql
# naive: join every other node of the scheme
SELECT … WHERE { $this ex:inScheme ex:S ; ex:code ?c .
                 ?o ex:inScheme ex:S ; ex:code ?c . FILTER (?o != $this) }
# improved: look the other node up by the value, then test its scheme
SELECT … WHERE { $this ex:inScheme ex:S ; ex:code ?c .
                 FILTER EXISTS { ?o ex:code ?c . FILTER (?o != $this) FILTER EXISTS { ?o ex:inScheme ex:S } } }
```

| Nodes | rdflib naive | rdflib improved | Oxigraph naive | Oxigraph improved |
|---|---|---|---|---|
| 200 | 3.5 | 0.21 | 0.058 | 0.063 |
| 400 | 7.2 | 0.21 | 0.052 | 0.067 |
| 800 | 13.8 | 0.22 | 0.058 | 0.065 |

Doubling the nodes doubles the naive form's cost per focus node on rdflib, so its total is quadratic. Neither the improved form nor either form on Oxigraph grows. A second case (E1) types a variable bound by an earlier pattern. Its cost on rdflib grows with the number of nodes of that type, and `FILTER EXISTS` removes it.

| Schemes (400 focus nodes) | rdflib pattern | rdflib `FILTER EXISTS` | Oxigraph pattern | Oxigraph `FILTER EXISTS` |
|---|---|---|---|---|
| 16 | 0.23 | 0.21 | 0.061 | 0.062 |
| 64 | 0.61 | 0.14 | 0.038 | 0.039 |
| 256 | 2.26 | 0.14 | 0.038 | 0.039 |

**Why.** `rdflib/plugins/sparql/algebra.py`, `simplify` (line 779) calls `reorderTriples` (line 122) on each basic graph pattern when the query is translated. It sorts the patterns by `_knownTerms` (line 104), the number of variable terms not yet bound by the patterns before it. At translation `$this` is an ordinary variable, because initial bindings arrive only when the query runs, so `?s a ex:Scheme` (one unknown term) sorts before `$this ex:inScheme ?s` (two). Then `evalBGP` (`evaluate.py`, line 77) runs the patterns in that order as nested loops, and the first one scans every node of the type. `FILTER EXISTS` is evaluated for each solution of its group with that solution's values substituted (`operators.py`, `Builtin_EXISTS`, line 580), so inside it `?s` is bound and the same pattern is a lookup. The two forms mean the same by the specification, which defines `EXISTS` by substitution (SPARQL 1.1 Query §8.1, and the evaluation of `EXISTS` in §18).

SPARQL does not fix the order in which an engine matches the patterns of a group. Oxigraph, which showed no difference, orders them itself. We did not read its planner. That its naive forms are already fast is Observed. The rewrite is harmless elsewhere, since it means the same, but on an optimising engine it bought nothing we measured.

### TSD-26. A chain of three or more groups is joined by evaluating the later ones whole

**Evidence:** Source (rdflib 7.6.0) and Observed, R-D. A property of rdflib's evaluator. Not measured on Oxigraph.

**Context.** A parent-code query joined `?parent skos:notation ?parentCode` after a `UNION` group, and cost as much as the uniqueness join of TSD-25 although it never named another node of the scheme.

**What the store does.** A two-part group passes the left side's bindings into the right side. A group of three or more parts does not. Counting `Graph.triples` lookups over 300 `ex:has` edges:

| Query | Lookups | Scans with an unbound subject |
|---|---|---|
| `ex:root ex:has ?a . ?a ex:name ?n . ?n ex:kind ?k` (one pattern) | 601 | 0 |
| `ex:root ex:has ?a . { ?a ex:name ?n . ?n ex:kind ?k }` (two parts) | 601 | 0 |
| `ex:root ex:has ?a . { ?a ex:name ?n } { ?n ex:kind ?k }` (three parts) | 302 | 1 |

**Why.** `algebra.py`, `analyse` (line 792) marks a `Join` lazy only when none of its operands is itself a join. `evaluate.py`, `evalJoin` (line 156) then either pushes the left side's bindings into the right (`evalLazyJoin`, line 141) or evaluates the right side whole into a set and hash-joins (`_join`). A chain of three parts is left-deep, so its outer join has a join for an operand and is not lazy. Evaluating an operand whole is cheaper than one lookup per left row when the left side is large. It is expensive when the left side is one row, as it is for every constraint query, because `$this` is pre-bound: the operand's whole extent (here, every notation of the scheme) is built for each focus node.

What this does not explain. Moving the identity lookup of a pair-enumerating query, which we label W8, ahead of its pair enumeration took that query from 67 ms to 32 ms, and by this mechanism the join stays non-lazy either way. We did not isolate why it helped (TSD-Q15). TSD-Q1 asks about the order of a lazy join's solutions, which this finding does not answer.

### TSD-27. A `FILTER` is applied after its group's patterns, and rdflib does not move it earlier

**Evidence:** Source (rdflib 7.6.0) and Observed on both stores, R-D. The scope is the specification's (§4.3). The cost is the evaluator's.

**Context.** A slot-overlap query put three cheap guards (every variant has a condition, every condition is an interval condition, no variant has two) in the group that then enumerates candidate points. It took 266 ms per slot.

**What the store does.** Guards that reject 90% of the focus nodes, in the group of an enumeration or in a group of their own (experiment E3). Milliseconds per focus node, 200 focus nodes.

| Values per node | rdflib same group | rdflib own group | Oxigraph same group | Oxigraph own group |
|---|---|---|---|---|
| 10 | 9.6 | 0.51 | 0.090 | 0.069 |
| 20 | 38.4 | 1.4 | 0.103 | 0.072 |
| 30 | 81.9 | 3.5 | 0.127 | 0.099 |

On two real slot queries the gain was 266 ms to 132 ms for overlap and 119 ms to 82 ms for coverage, smaller than the synthetic gain, which rejects 90% of the nodes. The gain follows the share of focus nodes the guards reject, so it is large only when they reject most.

**Why.** A `FILTER` applies to the whole group it is written in (§4.3). rdflib wraps the whole group in one `Filter` node (`algebra.py`, `translateGroupGraphPattern`, around line 343), and `evalFilter` (`evaluate.py`, line 208) tests each solution of `evalPart(ctx, part.p)`, so every solution is built before any guard runs. In the synthetic case a guard in its own group is the left operand of a lazy join (TSD-26), so a node it rejects never reaches the right operand. We did not establish that every real query meets that condition, and the real gain is smaller. Oxigraph orders filters itself, and its two forms differ by 1.3 to 1.5 times.

The sub-group is exactly equivalent when the guards read only variables the sub-group binds, so the change is safe to make and to check by replaying recorded validations.

### TSD-28. Presence and absence tests: the forms differ little

**Evidence:** Observed on both stores, R-D. Not explained from source.

**Context.** A documentation rule counted English notes with `OPTIONAL`, `GROUP BY` and `HAVING (COUNT(DISTINCT ?n) < 1)`. We asked whether another form was cheaper.

**What the store does.** The three forms, over 600 focus nodes with a note on five in six, in milliseconds per focus node. Only rdflib was timed, since Oxigraph rejects the `GROUP BY` form when the adapter substitutes `$this`.

| Form | rdflib |
|---|---|
| `OPTIONAL`, `GROUP BY`, `HAVING (COUNT(DISTINCT ?n) < 1)` | 0.178 |
| `OPTIONAL`, `FILTER (!BOUND(?n))` | 0.130 |
| `FILTER NOT EXISTS` | 0.126 |

Experiment E4 compares the last two on both stores: 1.0 to 1.2 times on rdflib and 1.1 to 1.2 times on Oxigraph, which is noise. On a real fixture the first form cost 1.19 ms and `FILTER NOT EXISTS` 0.85 ms (29% less). Turning two pure existence patterns of another query into `FILTER EXISTS` saved 12%.

### TSD-29. Two parses of one document never give equal triple sets

**Evidence:** Observed, R-D. Not a store property. A parser mints blank nodes (the RDF 1.1 Turtle and N-Triples specifications scope blank-node labels to a document), so this holds for any parser.

**Context.** A test fixture of seven Turtle files was parsed afresh for each of twenty tests. Two tests that validated the same unchanged fixture never shared a validation, although the validation cache keys on a graph's content.

**What happens.** The fixture has 768 blank-node terms. Two parses compared as sets differ, and they are isomorphic. A copy of one parse (`Graph() + graph`) keeps the labels, so two copies compare equal as sets. A cache keyed on triple content cannot hit across parses. Building a fixture once and giving each test a copy cost 0.14 s against 0.28 s to parse.

### TSD-30. A query's undeclared prefixes resolve from the data graph's bindings

**Evidence:** Source (rdflib 7.6.0) and Observed, R-D. A property of rdflib.

**Context.** The first version of the translation cache of TSD-22 keyed on all of a graph's namespace bindings, and missed: 1,258 parses for 94 distinct texts in one test file.

**What the store does.** `g.query("SELECT ?s WHERE { ?s ex:p ?o }")` on a graph with `ex` bound gives its rows, and without the binding raises `Unknown namespace prefix : ex`.

**Why.** `rdflib/graph.py`, `Graph.query` (line 1716): `initNs = initNs or dict(self.namespaces())`. So a translation depends on the text and on the graph's bindings for the prefixes the text uses and does not declare. A prefix the text declares itself wins over the graph's. A cache key therefore holds the text, the base and the bindings of the undeclared prefixes the text uses, and replaying a recorded validation needs the data graph's bindings restored. Whether Oxigraph has a fallback was not tested.

### TSD-31. pySHACL copies the data graph on every validation unless told not to

**Evidence:** Source (pySHACL 0.40.1) and Observed, R-D. A property of pySHACL.

**What the tool does.** `validate(..., inplace=False)` (`pyshacl/entrypoints.py`, line 40) clones the data graph (`rdfutil/load.py`, `clone_graph`). In a profile of 24 validations of a 5,000 to 14,000 triple graph, the clones took 2.2 s, about 0.09 s each. That is a fixed cost per validation that grows with the size of the data, about 9% of a validation in that profile. `inplace=True` would remove it, and pySHACL then may change the graph (inference, rules). Whether that is safe with graphs shared between tests is TSD-Q16.

### TSD-32. A constant `FILTER` whose value is false is dropped by rdflib

**Evidence:** Source (rdflib 7.6.0) and Observed on both stores, R-D. A defect in rdflib. Oxigraph is correct.

**Context.** A benchmark's test of its own pass criterion used `FILTER (false)` for a deliberately wrong query, and the check did not fail.

**What the store does.** Rows returned by `SELECT ?s WHERE { ?s ex:p ?o FILTER (<constant>) }` over two triples.

| Constant | rdflib | Oxigraph | Correct |
|---|---|---|---|
| `false`, `0`, `""`, `0.0` | 2 | 0 | 0 |
| `true`, `1`, `"x"` | 2 | 2 | 2 |
| `!true`, `1 = 2`, `true && false`, `false || false` | 0 | 0 | 0 |

The same drop occurs inside a `UNION` branch and a sub-group, and it inverts `EXISTS` and `NOT EXISTS`: `FILTER NOT EXISTS { ?s ex:p ?x FILTER (false) }` returns 0 rows on rdflib where 2 are correct.

**Why.** `rdflib/plugins/sparql/algebra.py`, `collectAndRemoveFilters` (line 278) returns `and_(*filters)`, which for one filter is that filter itself (`operators.py`, `and_`, line 975). `translateGroupGraphPattern` then tests `if filters:` and builds the `Filter` node only when that is true. A lone constant whose Python truth value is false (`Literal(False)`, `Literal(0)`, `Literal("")`) fails the test, and the algebra has no `Filter` node. A compound expression is an object that is always true, which is why `true && false` works.

Because rdflib ignores a lone constant false filter, a probe that must fail on any engine should break a triple pattern instead.

### TSD-33. On Oxigraph, `COUNT(*)` over a statically false filter returns no row

**Evidence:** Observed, R-D. Not explained from source. A probable defect in Oxigraph 0.5.11.

**What the store does.** `SELECT (COUNT(*) AS ?n) WHERE { ?s ex:p ?o FILTER (<constant>) }`.

| Filter | rdflib | Oxigraph | Correct |
|---|---|---|---|
| `FILTER (1 = 2)`, a pattern that matches nothing, `FILTER (?o = ex:zz)` | one row, `n = 0` | one row, `n = 0` | one row, `n = 0` |
| `FILTER (false)` | one row, `n = 2` (TSD-32) | no row | one row, `n = 0` |
| `FILTER (true && false)` | one row, `n = 0` | no row | one row, `n = 0` |

An aggregate query without `GROUP BY` always has exactly one solution (SPARQL 1.1 Query §11, Aggregates), so no row is wrong. It happens when the engine can decide the filter is false before it runs, and not when it has to evaluate it. A caller that reads the first row of such a `COUNT` should expect none. The mechanism is open (TSD-Q17).

### TSD-34. rdflib evaluates a `FILTER NOT EXISTS` in a `GRAPH` block against the wrong graph for some solutions

**Evidence:** Source (rdflib 7.6.0) and Observed on both stores, R-F and R-G. A defect in rdflib. Oxigraph is correct. We first put this down to initial bindings, which this finding corrects. Reported upstream as [RDFLib/rdflib#3585](https://github.com/RDFLib/rdflib/issues/3585). The Why was corrected 2026-10-20 (§7.5).

**Context.** A read of the members of a version over a long chain is slow on rdflib. A form with the nearest-checkpoint test moved into a sub-select took 0.15 s where the original took about 100 s, but its result kept members that a later version had dropped. We took the cause to be rdflib ignoring `initBindings` inside the sub-select, and recorded that as the reason the form was not used.

**How we found it.**

1. *Initial bindings in a sub-select.* We ran `SELECT ?m WHERE { GRAPH <g> { { SELECT ?m WHERE { $v <p> ?m } } } }` with `v` as an initial binding on rdflib and as text on Oxigraph, and variants with the variable projected, unprojected, under `LIMIT`, under an aggregate, and inside `FILTER NOT EXISTS { SELECT ... }`. Every variant applied the value on both stores. The first cause was wrong.
2. *The members query.* We rebuilt the sub-select form on a six version chain. rdflib kept a dropped member and Oxigraph did not, with the checkpoint test in the sub-select. Moving the drop test into a sub-select instead gave the right answer on both. Replacing the sub-select by a pattern that returns the same rows also gave the right answer.
3. *Bisecting.* Cutting the query down, the wrong answer needed all of a sub-select, a `UNION` and a `FILTER NOT EXISTS` with no `GRAPH` of its own, in one `GRAPH` block. Dropping any one gave the right answer.
4. *The seed.* The same query gave a different answer in different runs. Sweeping `PYTHONHASHSEED` over 32 child processes gave the wrong answer for 13 seeds and the right one for 19, with a constant in place of the parameter, and 15 and 17 with the parameter. So a parameter plays no part.
5. *The graph.* We traced the `NOT EXISTS` body. For the wrong solution its context's graph was the `Dataset` and not the named graph, so the body, which has no `GRAPH`, searched the default graph and found nothing.
6. *The line.* Replacing the two lines of `evalGraph` that reset the graph with `pass` gave the right answer for all 8 seeds we tried, where the stock code gave the wrong answer for 6 of them. **Corrected 2026-10-20 (R-G):** it gives the right answer on all 32 seeds, but it is not a fix. It breaks a `FILTER EXISTS` outside the block on every seed (Why).

**What the store does.** Data in `<g>` is a chain `v3 prev v2 prev v1` where `v1` has members `a` and `b`, `v2` drops `a` and adds `c`. The members of `v3` are `b` and `c`.

```sparql
SELECT ?m WHERE { GRAPH <g> {
  { SELECT ?cp WHERE { ?cp <cp> ?t } }
  <v3> <prev>* ?w . ?w <prev>* ?cp .
  { ?w <has> ?m } UNION { ?w <adds> ?m }
  FILTER NOT EXISTS { ?x <drops> ?m ; <prev> ?z . ?z <prev>* ?w . <v3> <prev>* ?x }
} }
```

| Form | rdflib, over 32 seeds | Oxigraph | Correct |
|---|---|---|---|
| as written, constant `<v3>` | `b c` for 19 seeds, `a b c` for 13 | `b c` | `b c` |
| as written, `$v` as an initial binding (text on Oxigraph) | `b c` for 17, `a b c` for 15 | `b c` | `b c` |
| the sub-select replaced by `?cp <cp> ?t .` | `b c` for all 32 | `b c` | `b c` |
| the sub-select kept, no `UNION` (`?w <has> ?m`) | `b` for every seed we tried | `b` | `b` |

**Jena ARQ 5.1.0 (R-X, 2026-10-27).** The query as written, with a constant `<v3>`, gives `b c` and the control with the sub-select replaced gives `b c`, as TSD-53 recorded. Jena evaluates each in one run with no hash seed, so there is no sweep to make. The defect is rdflib's alone among the three engines. Whether Fuseki over TDB2 shows it is TSD-Q19's remainder.

**Why.** *Corrected 2026-10-20 (R-G, §7.5). The first text had `evalGraph` reset the graph after it yields a solution, and took step 6's change for the fix.* `rdflib/plugins/sparql/evaluate.py`, `evalGraph` (line 220), evaluates the block's pattern in a context whose graph is the named graph. Before it yields each solution it sets `x.ctx.graph = prev_graph` (lines 242 and 251), the enclosing graph. `Builtin_EXISTS` (`operators.py`, line 580) is the only code that reads a solution's own context. It thaws that context and evaluates the body in it. The reset makes an `EXISTS` outside the block read the enclosing graph, and for the solution being yielded it does.

A solution's `ctx` is the context object it was made in, and solutions made without a new binding scope share one, so the reset also reaches solutions the block has not yet produced. `evalFilter` (line 208) evaluates a filter inside the block on each solution as the generator produces it. Once the first solution has left the block, the shared context's graph is the dataset, which here reads the default graph (`default_union` is false), so the next `NOT EXISTS` body finds nothing and keeps the member.

Which solution comes first decides how many are affected. A sub-select joined to a `UNION` is a non-lazy join (`evalJoin`, line 156), whose right side is built in a Python `set`, so the order follows the hash seed (TSD-01). `Dataset` is created with `default_union` false. A body that has a `GRAPH` of its own does not depend on the context's graph, and putting one in the body gave the right answer for all 32 seeds. We did not isolate why the sub-select matters and the `UNION` alone does not (TSD-Q19). A plain `FILTER EXISTS` is affected too. `GRAPH <g1> { { SELECT ?o WHERE { ?a <p> ?o } LIMIT 1 } { ?s <p> ?o } UNION { ?s <r> ?o } FILTER EXISTS { ?s <q> ?x } }` gave 15 different wrong answers over 32 seeds and the right one for none, both adding and dropping rows.

**The fix (R-G).** Removing the reset (step 6) is not one. It gives the right answer inside the block, and the wrong one on every seed for an `EXISTS` outside it, after `GRAPH <iri>`, after `GRAPH ?g` and after a nested `GRAPH`. Yielding `FrozenBindings(ctx, x.items())`, a new solution on `evalGraph`'s own context, whose graph is already the enclosing one, mutates no shared context and was right in every case on every seed. With that change, rdflib's own SPARQL, W3C, dataset and conjunctive-graph tests at `main` `62bf240` give the same result as without it, with the same W3C outcomes test by test. None of them covers the defect. It reproduces on 7.6.0, on `main` (7.7.0a0, `62bf240`) and on `next` (8.0.0a0, `0fdac39`), and [RDFLib/rdflib#3585](https://github.com/RDFLib/rdflib/issues/3585) reports it with a standalone harness of these cases and proposes the change.

**What this means for a workaround.**

- Initial bindings inside a sub-select work on both stores. The form with the sub-select is still not usable on rdflib.
- **Added 2026-10-20.** The sub-select form with a `GRAPH` inside each `NOT EXISTS` body is right. It read every version of 24 random histories (checkpoint every 2, 3 and 4 versions, stray adds) as a reference model gives, on rdflib under 16 hash seeds in child processes and on Oxigraph. The read of version 100 of a set of 500 took 1.28 s on rdflib, where the form without a sub-select took 98.6 s, and 0.005 s on Oxigraph, where it took 1.7 s.
- The read without a sub-select showed no wrong answer in any history we built, on 192 runs over 16 seeds. Its two `NOT EXISTS` bodies depend on solution variables, so it is the one form where the defect could change an outcome.
- A fast form on rdflib should put a `GRAPH` inside each `FILTER [NOT] EXISTS` body, or avoid a sub-select beside a `UNION` in one `GRAPH` block, and be tested on a sweep of hash seeds against a reference model.
- A test of a `FILTER [NOT] EXISTS` inside `GRAPH` on rdflib must sweep seeds in child processes. One run in the test process measures luck.

**Which queries can show the defect.** A scan of the rendered text of the update operations we generate found many with a `FILTER NOT EXISTS` with no `GRAPH` of its own inside a `GRAPH` block, and none with a `FILTER EXISTS`. For most of them the body mentions only `$root`, one value in every solution of the block, and the solutions of a row block do share one context on rdflib. We counted the contexts of the yielded solutions with a row of three heads and three current versions. One replace yielded three solutions in one context, and an append yielded three in each of three. The `OPTIONAL` that reads the head does it, so a sub-select is not needed for the sharing (the sub-select's own part stays open, TSD-Q19).

A write guard of that kind still cannot be wrong, for a reason that does not depend on how many solutions share a context. `evalGraph` resets the context's graph only when a solution leaves the block (wording corrected 2026-10-20, §7.5), and a body that mentions only `$root` gives the same answer for every solution. If the first is dropped nothing is yielded and nothing is reset. If the first is kept, all are kept, and the wrong graph keeps what the right graph keeps. The defect can only keep a solution wrongly after another was kept rightly, which needs a body that depends on a solution variable. Our test on 28 such operations used a tombstoned row, three heads and three current versions, 16 seeds in child processes and Oxigraph once, with a live row as the control. The same sweep still passes when the `GRAPH` is taken out of a template, so only a text scan guards against that.

The read with bodies that depend on solution variables passed a sweep of 192 random histories (checkpoint every 2, 3 and 4 versions, 16 seeds) and a nine version history with stray adds on the text that had no `GRAPH`. In that read the solutions did not share a context. We put the `GRAPH` in anyway, so that no rule has an exception and a later rdflib cannot expose the defect.

### TSD-35. A literal as the graph of a `GRAPH` block is refused by Oxigraph and parsed by rdflib

**Evidence:** Observed on both stores, 2026-10-21.

**What the store does.** A caller that binds a graph parameter to a literal, for example `GRAPH "g" { ... }` after substitution, gives a text the SPARQL 1.1 grammar refuses, since a `GRAPH` block takes a variable or an IRI (`VarOrIri`). Oxigraph refuses it as a syntax error when it parses the update. rdflib parses it, the block matches no graph, and the update writes nothing. A guarded write that is given a literal for a graph writes nothing on both stores, by different routes, so the outcome a test asserts is the same. A test that expects the request to be accepted and do nothing holds on rdflib only.

### TSD-36. A bound object does not give the seek a bound subject gives

**Evidence:** Observed on Oxigraph 0.5.13, run R-I, 2026-10-24 and repeated 2026-10-27, probing a posting-cost benchmark. Not tested on rdflib, so it is a hypothesis there.

**Context.** The benchmark's checkpointed reads find the latest checkpoint of an account at or before a read point, then fold the postings after it. Partitioning the postings into a graph per period made the fold flat in the history, after which this lookup was the only term still growing. It rose from 0.04 ms at 2 closes an account to 1.06 ms at 200, so it grew with the number of closes and so with time.

**How we found it.** Three forms of the same lookup were timed against the same store, at 1,000, 20,000 and 100,000 events, which is 2, 40 and 200 closes for each account. All three returned the same balance. The median of 40 runs is reported.

| Form | What is bound | 2 closes | 40 closes | 200 closes |
|---|---|---|---|---|
| A | the account, as an object, with `FILTER(?pos <= p)`, `ORDER BY DESC(?pos)` and `LIMIT 1` | 0.040 ms | 0.224 ms | 1.059 ms |
| B | the account and the exact position, both as objects | 0.032 ms | 0.114 ms | 0.485 ms |
| C | the checkpoint itself, as the subject | 0.020 ms | 0.020 ms | 0.023 ms |

**What the store does.** Form B binds every term that identifies the row and still grows linearly in the account's checkpoints, at about half of form A's cost. Removing the sort and the filter therefore accounts for half of the cost and the scan for the other half. Only form C, which names the subject, is flat.

```sparql
# B, both objects bound, still scans the account's checkpoints
SELECT ?b WHERE { ?c ex:account <urn:acct:a0> ; ex:position 100000 ; ex:balance ?b }
# C, the subject bound, seeks
SELECT ?b WHERE { <urn:checkpoint:100000:a0> ex:balance ?b }
```

**Why.** The planner picks one pattern as the entry point and joins the rest to it. With `?c` unbound it enters on a predicate-object match, here `ex:account <urn:acct:a0>`, which yields every checkpoint of that account, and tests each against the remaining patterns. Binding the position does not make the planner enter there, so the selectivity of the second pattern is not used. We read the behaviour from the timings rather than from the planner, so the mechanism is inferred and the measurement is not.

### TSD-37. Oxigraph refuses an IRI with two fragment separators and rdflib accepts it

**Evidence:** Observed on both stores, 2026-10-24, loading a corpus of 270 ontology documents into Oxigraph for an index. 269 of 270 documents loaded. The exception was one Turtle file of test data.

**What the store does.** That file's subjects concatenate two absolute IRIs, each with its own `#`, for example `http://example.org/ns#http://example.org/examples/Contract#acceptingParty` (the shape of the IRIs, with the host names replaced). A second `#` is not a legal IRI code point, so Oxigraph refuses the document outright with a syntax error naming the line and column, and loads none of it. rdflib parses the same document and returns 190 triples, with the malformed text as the subject IRI. Loading the file with `pyoxigraph.Store().load(path=..., format=RdfFormat.TURTLE)` raises `SyntaxError`, and `rdflib.Graph().parse(..., format="turtle")` returns the 190 triples.

### TSD-38. Oxigraph's read-only open costs a fraction of a read-write open, and a query over the whole corpus is sub-millisecond

**Evidence:** Measured 2026-10-24 on macOS (Darwin 27, Apple silicon), pyoxigraph 0.5.13, for an index of ontology documents. The store held the corpus in a named graph per document plus a synthetic index graph of 16,800 quads, 72,832 quads and 23.7 MB on disk in total, built in 1.17s.

**What the store does.** `Store(path)` opens read-write in 141 ms cold and about 32 ms warm, because RocksDB recovers its write-ahead log and may compact. `Store.read_only(path)` opens in 23.8 ms, repeatably, doing neither. Queries themselves are then far cheaper than either open: a property-path traversal two hops out took 0.1 ms, a two-tag conjunctive lookup 0.3 ms, a `CONTAINS` filter over every `rdfs:label` in every graph 0.6 ms, and a `GROUP BY` histogram over 16,800 quads 0.9 ms. A whole process, counting interpreter start, importing pyoxigraph, opening read-only and running a query, costs 50 ms end to end. To repeat it, build a store with the corpus, then time `pyoxigraph.Store.read_only(path)` against `pyoxigraph.Store(path)` and the queries above. The open is a fixed cost and the queries are not, so the result may change at ten times the quads.

### TSD-39. One Oxigraph bulk load per document fragments the store past a default open-file limit

**Evidence:** Observed 2026-10-25 on macOS, pyoxigraph 0.5.13, building an index of ontology documents. A build failed with `Too many open files` on an SST file numbered above 2,300. Reproduced at `ulimit -n 256`.

**What the store does.** Each `Store.bulk_load` call writes SST files of its own and ingests them, and `Store.optimize()` does not merge them back. Loading 270 small documents with one `bulk_load` each left 1,016 SST files after compaction. A query, or a `Store.read_only` open, has to open them, so it exceeds macOS's default soft limit of 256 open files. Parsing each document with `pyoxigraph.parse` and writing all of them in one `bulk_extend` leaves 9 files, takes the same 2.1 seconds, and gives an identical graph: 56,032 quads and 5,173 blank nodes, none shared across documents, with `rename_blank_nodes=True` keeping each document's blank nodes its own as a per-document load did. The build passed where the limit was 1,048,576 and failed in a terminal where it was 256, so a tool that opens many files must be tested at the default limit and not only in an agent's shell. To reproduce it, run `ulimit -n 256`, bulk load each of the documents separately into one store, then query it.

### TSD-40. A zero-or-more property path's cost changed between Oxigraph 0.5.11 and 0.5.13

**Evidence:** Observed on Oxigraph 0.5.11 and 0.5.13, run R-I, 2026-10-27, measuring a posting-cost benchmark's subtree fold. Not tested on rdflib.

**Context.** The benchmark's committed figures were taken on 0.5.13 on 2026-10-24. Repeating the run on 2026-10-27, in a checkout whose environment had 0.5.11, read one root's subtree in 148.93 ms where the record said 43.89 ms, and another's in 127.52 ms where the record said 0.95 ms. The figures looked irreproducible, and the engine version was the only difference.

**How we found it.** Three forms of the same fold over one root's subtree, 56 accounts and 10,063 postings, on one store, each returning the identical sum and count. The two forms that do not use a path are the controls.

| Form | 0.5.11 | 0.5.13 |
|---|---|---|
| `?acc ex:partOf* <root>`, the subtree reached by a path | 178.88 ms | **73.74 ms** |
| the 56 accounts enumerated in a `VALUES` block | 73.92 ms | 73.34 ms |
| `ex:partOf/ex:partOf` and `ex:partOf` in a `UNION`, the depth written out | 147.27 ms | 147.40 ms |

**What the store does.** The path form is 2.4 times faster on 0.5.13, where neither control moves. On 0.5.13 the path costs what enumerating the same accounts costs, so the path is no longer the expensive part. Writing the depth out as a `UNION` of fixed-length paths is twice the cost of either on both versions, so it is not an optimization.

**Why.** We did not read Oxigraph's source for this. The controls hold the machine, the data and the query shape fixed, so the change is in how the engine evaluates `*`, and the mechanism is inferred rather than observed. A cost measured for a property path is therefore a cost for one patch version and not for the engine, and enumerating the accounts is the stable choice, since it measured the same on both versions.

### TSD-41. An aggregate over an expression that errors on one row of a group is unbound on Oxigraph and skips the row on rdflib

**Evidence:** Source and Observed, run R-J, on rdflib 7.6.0 and Oxigraph 0.5.11 and 0.5.13, 2026-10-27. The specification text is SPARQL 1.1 Query §18.5.1, read from the W3C Recommendation. Oxigraph's source is `lib/spareval/src/eval.rs` at v0.5.13, read on GitHub. Answers TSD-Q20. Corrected 2026-10-27, see §7.6, since the first version of this entry blamed the `UNION` and the nested sub-selects. Retitled 2026-10-27 from "A `GROUP_CONCAT` over a `UNION` of nested aggregate sub-selects drops one group's contribution on Oxigraph", since the minimal reproducer needs no `UNION` and no sub-select, so the aggregate over an expression that errors is the better characterisation.

**Context.** Generated SPARQL had run on rdflib alone, with no engine abstraction. Giving it one, so that a conformance task could run it on Oxigraph too, surfaced one disagreement in a suite of 126 tests, not a defect in the new wrapper: the wrapper's round-trip of every other binding in every other test matched rdflib exactly. The test checked a query that builds a string with `GROUP_CONCAT` over the rows of a record, one row per condition, and it failed every run on Oxigraph.

**How we found it.** In order.

1. Running the whole suite on Oxigraph gave one failure, every time. Running the one generated query, for a profile with two conditions over four records, on both engines gave the same `decision` for every record and a different `closures` for one. That record relies on the closure through one condition alone, so one of its two condition rows carries a closure and the other none. rdflib named it. Oxigraph left `closures` unbound.
2. Each condition's branch alone, in the outer aggregate, agreed on both engines. So did both branches' rows with the outer aggregate removed. The rows were identical, so the disagreement was in the outer `GROUP_CONCAT`, not in the sub-selects, the `UNION` or the idiom that produces them.
3. Dropping `DISTINCT`, then `STR`, still disagreed. Wrapping the argument in `COALESCE(STR(?relied), "")` agreed.
4. The same disagreement without a `UNION`, a sub-select or a `GROUP BY`, in one pattern: `VALUES ?x { <urn:a> UNDEF }`. The same for `SUM`, `AVG`, `MIN`, `MAX`, and for `COUNT` over an expression, but not for `COUNT(?x)`, `COUNT(*)` or `SAMPLE`.
5. Reading each engine's aggregate code (below).

**What the store does.** The smallest query, over a solution with `?x` bound and one without.

```sparql
SELECT (GROUP_CONCAT(STR(?x)) AS ?g) WHERE { VALUES ?x { <urn:a> UNDEF } }
```

| Aggregate over `<urn:a>` and an unbound `?x` | Specification (§18.5.1) | rdflib 7.6.0 | Oxigraph 0.5.11 and 0.5.13 |
|---|---|---|---|
| `GROUP_CONCAT(STR(?x))`, with or without `DISTINCT`, and `GROUP_CONCAT(?x)` | an error, so `?g` is unbound | ✘ `"urn:a"`, the row is skipped | ✔ unbound |
| `SUM(?x)`, `AVG(?x)` over `1` and an unbound | an error | ✘ `1` | ✔ unbound |
| `MIN(?x)`, `MAX(?x)` over `1` and an unbound | `MIN` unbound. `MAX` unbound or `1`, since §15.1 orders no value lowest and the text does not say whether an error element is no value (TSD-43) | `1` | unbound |
| `COUNT(STR(?x))` | `1`, since `Count` removes errors | ✘ `2`, it counts the error | ✘ unbound |
| `COUNT(?x)`, `COUNT(*)`, `SAMPLE(?x)` | `1`, `2`, any bound value | ✔ | ✔ |
| `GROUP_CONCAT(COALESCE(STR(?x), ""))` | `"urn:a "` or `" urn:a"`, by the order of the group | ✔ | ✔ |
| `GROUP_CONCAT(STR(?x))` over an empty group | `""` | ✔ | ✔ |

In the generated query the group is a record's rows, one per condition. A condition that did not rely on a closure leaves `?closure` unbound, and a record that relied through one condition has one row with a closure and one without. Oxigraph turns that mix into an unbound `?closures`. A record whose rows all rely, or none does, agrees on both engines. The generated query's `UNION`, nesting and the idiom that leaves a variable unbound only produce the unbound row. `COUNT(DISTINCT ?x)` and `SAMPLE` are safe on both. A `GROUP_CONCAT(DISTINCT COALESCE(STR(?x), ""))` also agrees, but puts the empty element at either end by the group's order, which differs between engines, and needs a trim. A string built instead from one coalesced `SUM` flag per distinct member, in IRI order, with no `GROUP_CONCAT`, returned the same strings on both engines.

**Why.** The specification defines an aggregate over `Flatten(M)`, a multiset of lists in which `ListEval` keeps an error, and says "`ListEval((unbound), μ) = (error)`" (§18.5.1). `Count` removes the error elements before counting. `Sum` and `GroupConcat` do not. `GroupConcat(S, sep) = CONCAT(S0, sep, ...)` over a sequence with an error element is an error, and an aggregate whose value is an error leaves its variable unbound in the row. So the specification gives unbound for `GROUP_CONCAT` and `SUM`, and Oxigraph follows it. Its `AccumulatorWrapper::Expression` and `DistinctExpression` (`eval.rs`, `accumulate`) set the accumulator to `None` the first time the argument's evaluator returns `None`, which is an error or an unbound value, and `finish` then returns no value. That is the poisoning the specification asks for in `GROUP_CONCAT`, `SUM` and `AVG`. It is also applied to `COUNT` over an expression that is not a plain variable (`CountAccumulator` is built inside the same wrapper, while `COUNT(?x)` takes the `CountInternal` path that skips a missing value), where the specification says to remove the error. That one is a departure from the specification on Oxigraph's side.

rdflib's `GroupConcat.update` (`rdflib/plugins/sparql/aggregates.py`) catches the unbound value as an `NotBoundError` value and returns without appending, with the comment "skip UNDEF". `Sum`, `Average`, `Minimum` and `Maximum` skip it the same way, and `Counter` counts it when the expression is not a bare variable. A different error, such as division by zero, is appended as its message (`GROUP_CONCAT(STR(?x/0))` gives `"divide by 0"`). So rdflib's `GROUP_CONCAT` and `SUM` are the lenient departure, and its `COUNT(STR(?x))` over-counts. Corrected 2026-10-27 by TSD-43: `MIN` unbound is what the specification gives on either reading, so rdflib's skip departs and Oxigraph conforms. `MAX` is the open one, and Jena ARQ answers as Oxigraph does.

The first version of this entry guessed that the order of `GROUP_CONCAT` (TSD-19) or the idiom that leaves a variable unbound made the engines take different paths through the `UNION`. That was wrong. Neither the order nor the `UNION` matters, and the rows the two engines feed to the aggregate are identical.

A query written to run on rdflib may rely on its leniency without knowing. A possible upstream report, not filed: Oxigraph's `COUNT(expr)` over an erroring expression returns an unbound value where §18.5.1.2 says to count the rest. Jena ARQ 5.1.0 counts the rest (TSD-43). TSD-43 works through every aggregate and four remedies, and shows that the coalesce this entry first recommended changes the meaning of `SUM` and `AVG`.

---

### TSD-42. Oxigraph reads every graph for a `GRAPH ?g` whose `?g` is bound outside, a computed name on every version and a constant one inside an `OPTIONAL` from 0.5.13

**Evidence:** Observed. The constant forms on Oxigraph 0.5.11, 0.5.12 and 0.5.13, run R-K, 2026-10-27, restated with a store-probe benchmark on those versions and on rdflib 7.6.0, run R-S, and on Apache Jena ARQ 5.1.0, run R-W, the same day. The computed forms and the generated replace on Oxigraph 0.5.11 to 0.5.13 and rdflib 7.6.0, run R-M, and on Jena 5.1.0 and RDF4J 6.1.0, run R-N, 2026-10-28. Found twice, on 2026-10-27 and 2026-10-28, and first recorded as two findings, of which TSD-45 now points here. The mechanism is not read from source (TSD-Q24).

**Context.** Raising the pyoxigraph lock to 0.5.13, the floor TSD-40 set, made probe P3 fail. P3 is TSD-04's measure, `OPTIONAL { GRAPH ?g { ?s ?p ?o } }` with `?g` bound outside, timed over 100 graphs of 3,000 triples against 1 graph, and its ratio rose from about 1 to between 68 and 98 against a bound of 8. The first question was whether the failure was 0.5.13's or the probe's form. The second, once the first was answered, was whether a generator's updates pay for it.

**How we found it.**

1. *Observed, R-K, R-S and R-W.* Four forms with a constant `?g`, each counting the 3,000 triples of one graph, timed over 100 graphs against 1, best of five, on each version.

| Form | 0.5.11 | 0.5.12 | 0.5.13 |
|---|---|---|---|
| `BIND(<g> AS ?g) OPTIONAL { GRAPH ?g { ?s ?p ?o } }` | 1.0 | 1.0 | **73 to 78** |
| `VALUES ?g { <g> } OPTIONAL { GRAPH ?g { ?s ?p ?o } }` | 1.0 | 1.0 | **73 to 78** |
| `BIND(<g> AS ?g) GRAPH ?g { ?s ?p ?o }`, a join | 1.0 | 1.0 | 1.0 |
| `GRAPH <g> { ?s ?p ?o }`, a constant | 1.0 | 1.0 | 1.0 |

The store-probe benchmark reproduces the table in run R-S (macOS arm64, best of five, 100 graphs of 3,000 triples against 1). Its ratios are a hypothesis about one machine, and the rdflib column is new evidence. rdflib 7.6.0 reads the one graph under every form, with ratios between 1.4 and 2.5 that come from a larger dataset and not from a scan, since the constant form, which cannot scan, has the highest.

| Form | rdflib 7.6.0 | 0.5.11 | 0.5.12 | 0.5.13 |
|---|---|---|---|---|
| bind-optional | 1.4 to 1.7 | 1.0 | 1.0 | **79** |
| values-optional | 1.4 | 1.0 | 1.0 | **82** |
| join | 1.8 | 1.0 | 1.0 | 1.0 |
| constant | 2.4 to 2.5 | 1.0 | 1.0 | 1.0 |

Jena ARQ 5.1.0 in run R-W (one run, 100 graphs of 3,000 triples against 1, best of five, the JVM with its full compiler). Each cell is the ratio, with the one-graph and the 100-graph time in milliseconds. The times include the pipe to the JVM and the JSON answer, so they say nothing about Jena's speed against the in-process engines.

| Form | Jena ARQ 5.1.0 |
|---|---|
| bind-optional | 1.0 (7.7 to 7.3 ms) |
| values-optional | 1.0 (6.7 to 6.4 ms) |
| join | 1.0 (5.4 to 5.2 ms) |
| constant | 0.8 (4.6 to 3.7 ms) |

*Observed, R-M, the probe's own ratio.* Three runs of P3's own ratio in a scratch environment per version. 0.5.11 gave 1.0, 1.8 and 1.0, 0.5.12 gave about 1, and 0.5.13 gave 98.5, 81.8 and 84.7. The engine version alone moved the ratio.

2. *Observed, R-M.* The probe's query and its neighbours, `COUNT(*)` over 100 graphs of 3,000 triples against 1 graph, the best of three, the ratio of the two. The count is 3,000 in every cell, so the result is right. `computed` is `IRI(CONCAT("urn:g:", ENCODE_FOR_URI(STR("0")), ""))`, the shape of a generated graph name.

   | Form | 0.5.11 | 0.5.12 | 0.5.13 |
   |---|---|---|---|
   | constant `BIND(<g0> AS ?g) OPTIONAL { GRAPH ?g { … } }` (the probe) | 1.0 | 1.1 | **75** |
   | constant `VALUES ?g { <g0> } OPTIONAL { GRAPH ?g { … } }` | 1.0 | 1.0 | **45** |
   | constant `BIND`, one group, no `OPTIONAL` | 0.9 | 1.0 | 1.0 |
   | constant `VALUES`, one group, no `OPTIONAL` | 1.0 | 1.0 | 1.0 |
   | **computed** `BIND(… AS ?g) OPTIONAL { GRAPH ?g { … } }` (what a generator emits) | **51** | **65** | **67** |
   | **computed** `BIND`, one group, no `OPTIONAL` | **99** | **91** | **85** |
   | `GRAPH <g0> { … }`, a constant | 1.3 | 1.0 | 1.1 |
   | `GRAPH ?g { … }`, nothing bound (the scan, for scale) | 80 | 109 | 82 |

   rdflib 7.6.0 gives 1.0 to 1.3 for both the constant and the computed `OPTIONAL` form.
3. *Observed, R-M.* A generated named-graph replace, guards off, run on a store holding the root's graph and ballast aggregates of 100 triples each, the best of three replaces, with the graph name computed in the query (`query`, the default) and supplied by the caller (`caller`).

   | Quads in other graphs | 0.5.11 query / caller (ms) | 0.5.12 query / caller (ms) | 0.5.13 query / caller (ms) | rdflib query / caller (ms) |
   |---|---|---|---|---|
   | 0 | 1.0 / 0.9 | 0.9 / 0.8 | 1.4 / 1.0 | 32 / 27 |
   | 10,000 | 6.6 / 0.8 | 7.9 / 1.4 | 6.0 / 0.9 | 30 / 24 |
   | 100,000 | 67 / 1.0 | 64 / 1.5 | 90 / 1.4 | 48 / 26 |
   | 1,000,000 | 700 / 0.9 | 687 / 1.0 | 707 / 0.9 | not run |
   | 3,000,000 | 2,176 / 1.1 | 2,029 / 1.5 | 1,861 / 1.5 | not run |

4. *Observed, R-N.* The same forms and the same generated replace, rendered by the Python benchmark and run unchanged on Jena's in-memory dataset, Jena's in-memory TDB2 store and RDF4J's `MemoryStore`. Every form read one graph, and the unbound scan read all of them.

   | Form (ratio of 100 graphs to 1) | Jena in-memory | Jena TDB2 | RDF4J |
   |---|---|---|---|
   | constant `BIND`, `OPTIONAL` | 1.0 | 1.5 | 1.6 |
   | constant `VALUES`, `OPTIONAL` | 1.2 | 1.0 | 1.0 |
   | **computed `BIND`, `OPTIONAL`** (the generated form) | **1.8** | **0.9** | **1.3** |
   | computed `BIND`, one group | 0.5 | 1.1 | 1.2 |
   | constant graph in the text | 1.6 | 1.1 | 1.0 |
   | nothing bound (the scan, for scale) | 16.5 | 84.6 | 48.3 |

   The generated replace, `query` against `caller`, took 3 to 8 ms on all three stores at 0, 10,000, 100,000 and 1,000,000 quads in other graphs (for example 3.8 and 3.6 ms on Jena in-memory, 4.1 and 4.8 ms on Jena TDB2, and 3.3 and 2.9 ms on RDF4J, at 1,000,000). These figures are from the run with a 10 GB heap. With a 4 GB heap for each measurement, in its own JVM, only Jena's in-memory TDB2 store at 1,000,000 quads did not finish, and every other row agreed. The first attempt measured every store in one JVM at 4 GB and lost every store after the first to run out of memory, since a JVM does not get that heap back, so the benchmark now runs each measurement in a child JVM with its own cap.

**What the store does.**

- **Oxigraph never passes a graph name computed by an expression into a `GRAPH ?g`,** in an `OPTIONAL` or in the same group, on any of the three versions. It reads every graph and joins afterwards. About 0.7 ms per 1,000 quads in other graphs, and the generated replace pays it.
- **A constant `?g` is passed in on 0.5.11 and 0.5.12 by an `OPTIONAL` as by the same group,** and on 0.5.13 only by the same group. 0.5.13 stopped passing a constant bound by `BIND` or `VALUES` into the right side of an `OPTIONAL`. That is the change TSD-04's probe caught. It does not touch the generated replace, which was already scanning.
- **Jena (both stores) and RDF4J pass a computed name in,** and the generated replace is flat in the size of the dataset on all three (step 4). Of the stores measured, Oxigraph is the only one that does not.
- **A constant graph name in the query text is read as one graph on all three Oxigraph versions,** which is what the caller-named form renders once the name is substituted.
- **rdflib passes the binding in for both,** and the generated replace is flat within the spread of its runs, 25 to 50 ms from no ballast to 100,000 quads, for `query` and `caller` alike. The benchmark's three runs of it order the two namings differently, so the 30 to 48 ms between 0 and 100,000 quads in step 3's table is noise. Jena ARQ in run R-W reads one graph under the four constant forms too, so the constant form's departure is one release's.
- **On Oxigraph, every named-graph replace and tombstone under the default `query` naming costs time in the size of the whole dataset,** whatever the version: 6 ms at 10,000 quads in other graphs, 65 to 90 ms at 100,000, 0.7 s at 1,000,000. The caller-named form is flat at about 1 ms at every size. A form that names its data graph by a bound constant, and a scan whose `VALUES` sits in the same group, are not affected. The other generated forms were not run.
- **The cost of the computed form is a property of the store, and four engines have been measured.** Oxigraph scans, and rdflib, Jena and RDF4J do not, so the computed form is portable in cost across the stores measured except Oxigraph, and not across all stores. The caller-named form was no slower than the computed form on any of them, within the spread of a run (step 3 and step 4), so it bounds the cost from below. Jena on disk, Fuseki over HTTP, RDF4J's `NativeStore`, GraphDB, Virtuoso and Neptune were not measured (TSD-Q25).
- The benchmark on 0.5.13 on 2026-10-28 gave 51 ms (`query`) and 0.6 ms (`caller`) at 100,000 quads, and 717 ms and 0.4 ms at 1,000,000.
- **The constant form's departure is reported** as [oxigraph/oxigraph#2008](https://github.com/oxigraph/oxigraph/issues/2008). The computed form, which no Oxigraph version passes in, is not yet in that report (TSD-Q24). A release that restores the constant form shows as a ratio near 1 in both benchmarks (TSD-Q22). A cost is a hypothesis across patch versions of one engine (TSD-40).

**Why.** Not read. A constant `BIND` is most likely folded into the pattern by the optimizer, which a function call cannot be, and 0.5.13 changed how an `OPTIONAL`'s right side is optimized. The Oxigraph changelog that we fetched lists no 0.5.12 or 0.5.13 entry, and the Python binding does not expose a query plan, so this is a hypothesis. It may be the same change that made TSD-40's path faster.

TSD-04's Oxigraph half was true of a constant `?g` only, and the probe tested that form. A generator that computes `?g` from `$root` never met it on Oxigraph, so the probe measured a form the generator does not emit. The result of such a replace is right, and only its cost differs.

### TSD-43. What an aggregate does with a row that has no value or errors, for every aggregate, and what each remedy does

**Evidence:** Source, Observed and Documented, run R-L, on rdflib 7.6.0, Oxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27. Jena's column was re-run as an executed suite, run R-X, and the specification text of `Min` and `Max` was read again in the SPARQL 1.1 Recommendation and the SPARQL 1.2 working draft, run R-Y. The specification text is SPARQL 1.1 Query §15.1 and §18.5.1, read from the W3C Recommendation. rdflib's source is `rdflib/plugins/sparql/aggregates.py` and `evaluate.py` at 7.6.0, read locally. Oxigraph's accumulators are as TSD-41 read them at v0.5.13, and Jena's source was not read, so its column is Observed only. Extends TSD-41, which probed five aggregates in one row state on two engines. Narrows TSD-Q21.

**Context.** TSD-41 led to a rule that says to coalesce the argument of an aggregate. That was challenged as not worked through for every aggregate and scenario, since a coalesce changes what the group means. This entry sets out every aggregate against every row state, says what the specification prescribes in each cell, and probes each remedy on each engine, so a rule can be derived from the evidence.

**How we found it.** In order.

1. Read §18.5.1 and §15.1 again for each set function. The result is the specification table below.
2. Wrote one data state per row state and one query per aggregate. A group is the subjects of `?s a <urn:T>` with `OPTIONAL { ?s <urn:p> ?x }`, which is how a generated query leaves a variable unbound. Every query also projects a constant, because rdflib drops a result row whose projected variables are all unbound (TSD-10), which would otherwise make an unbound aggregate look like no row.
3. Ran the 13 aggregates over the seven states on rdflib and Oxigraph through a Python wrapper, and on Jena ARQ 5.1.0 through a single-file Java program. 91 cells per engine.
4. Added the remedies, 252 more queries over the same states (36 forms), and nine queries over three groups where one group has no value.
5. Read the aggregate classes of rdflib for each result that differs from the specification.

The run is pinned for rdflib, Oxigraph and Jena in a test suite. Jena's column was first recorded from a one-off runner and is now executed by the suite through the Java harness, and all 352 cells came out as recorded. A conformance run of the suite (R-X) passed 55 tests, with the Jena package's own, and every cell is as recorded for each of the three engines, so Jena's column above is executed and not only recorded.

**What the store does.** The states. `all bound` is two rows with `?x` 1 and 2. `one unbound` adds a third row with no `?x`. `one IRI` and `one blank node` add a third row whose `?x` is `<urn:a>` or a blank node. `all unbound` is two rows with no `?x`. `empty` is no row and no `GROUP BY`. `empty, grouped` is no row with `GROUP BY ?s`. The aggregate is over `?x`, or over `STR(?x)` where the row says `expr`, which errors for an unbound variable and for a blank node and not for an IRI. `GROUP_CONCAT` uses the separator `" "` and its elements are sorted before they are compared, since their order is the engine's. `unbound` is a row whose aggregate has no value, `none` is no row. `a member` is any value of the group, which is all `SAMPLE` promises.

What SPARQL 1.1 prescribes, by subsection.

| Aggregate | all bound | one unbound | one IRI | one blank node | all unbound | empty | empty, grouped |
|---|---|---|---|---|---|---|---|
| `COUNT(?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(expr)` | `2` | `2` | `3` | `2` | `0` | `0` | `none` |
| `COUNT(*)` | `2` | `3` | `3` | `3` | `2` | `0` | `none` |
| `COUNT(DISTINCT ?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(DISTINCT expr)` | `2` | `2` | `3` | `2` | `0` | `0` | `none` |
| `SUM` | `3` | `unbound` | `unbound` | `unbound` | `unbound` | `0` | `none` |
| `AVG` | `1.5 dec` | `unbound` | `unbound` | `unbound` | `0` | `0` | `none` |
| `MIN` | `1` | `unbound` | `<urn:a>` | `_:b` | `unbound` | `unbound` | `none` |
| `MAX` | `2` | `2 or unbound` | `2` | `2` | `unbound` | `unbound` | `none` |
| `SAMPLE` | `a member` | `a member or unbound` | `a member` | `a member` | `unbound` | `unbound` | `none` |
| `GROUP_CONCAT(expr)` | `1 2` | `unbound` | `1 2 urn:a` | `unbound` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(?x)` | `unbound or 1 2` | `unbound` | `unbound or 1 2 urn:a` | `unbound` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(DISTINCT expr)` | `1 2` | `unbound` | `1 2 urn:a` | `unbound` | `unbound` | `""` | `none` |

Reading of the table. `COUNT` removes error elements before it counts (§18.5.1.2), so a row whose argument is unbound or errors is not counted, and `COUNT(*)` counts rows and never looks at an argument. `Sum` is an error when any element is (§18.5.1.3, `op:numeric-add` of an error). `Avg` is `0` when `Count` is 0 and `Sum / Count` otherwise (§18.5.1.4), so over an all-unbound group the first clause gives `0` and over a group with one unbound row `Sum` gives an error. `Min` and `Max` order the flattened values, and the Recommendation orders "no value assigned to the variable or expression in this solution" lowest, then blank nodes, IRIs and literals (§15.1). Read that way a no-value element sorts first, so `Min` is that element and the aggregate is unbound, and `Max` skips it and gives `2`. Read as the other aggregates read an error, `Max` is unbound too. The text does not say which, so the `Max` cell allows both. Read again (R-Y), §18.5.1.5 and §18.5.1.6 define `Min` and `Max` as the first element of `Flatten(M)` ordered by `ORDER BY ASC` and `DESC`, `Min({})` and `Max({})` are errors, and §15.1 puts "no value assigned to the variable or expression in this solution" lowest. §18.5.1.2 says "remove error elements" for `Count`, which implies that `Flatten(M)` can hold error elements, and nothing says what `ORDER BY` does with one that is an error and not a missing value. The SPARQL 1.2 working draft (§18.6.1.6, fetched 2026-10-27) words `Max` through a list ordered by `ORDER BY DESC` in the same way and adds no rule for an error element. So the text supports `2` only if an error element counts as "no value", and supports unbound if an error element is an error as for `Sum`. The cell stays open and the specification does not decide it. A probe pins what the engines do, `2` on rdflib and unbound on Oxigraph and Jena. `Min({})` and `Max({})` are errors (§18.5.1.5, §18.5.1.6), so unbound. `Sample` is any element of `Flatten(M)` (§18.5.1.8), which may include the error, so the `one unbound` cell allows both, and `Sample({})` is an error. `GroupConcat` is `CONCAT` over the sequence (§18.5.1.7) and `CONCAT` accepts string literals (§17.4.3.12), so an error element is an error and read strictly a bare `?x` of numbers or an IRI is a type error too, which engines treat differently, so those cells allow both. An empty group without `GROUP BY` is the one group of `Group((1), Ω)` (§18.2.4.1), so `COUNT` is 0 and `SUM` is 0. With `GROUP BY` there is no group, so no row. `STR` of a blank node is an error (§17.4.2.5 defines it for literals and IRIs only).

What each engine returned. A cross marks a departure from the table above.

**rdflib 7.6.0**

| Aggregate | all bound | one unbound | one IRI | one blank node | all unbound | empty | empty, grouped |
|---|---|---|---|---|---|---|---|
| `COUNT(?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `a row ✘` |
| `COUNT(expr)` | `2` | `3 ✘` | `3` | `3 ✘` | `2 ✘` | `0` | `a row ✘` |
| `COUNT(*)` | `2` | `3` | `3` | `3` | `2` | `0` | `a row ✘` |
| `COUNT(DISTINCT ?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `a row ✘` |
| `COUNT(DISTINCT expr)` | `2` | `3 ✘` | `3` | `3 ✘` | `2 ✘` | `0` | `a row ✘` |
| `SUM` | `3` | `3 ✘` | `raises ✘` | `raises ✘` | `0 ✘` | `0` | `a row ✘` |
| `AVG` | `1.5 dec` | `1.5 dec ✘` | `1.5 dec ✘` | `1.5 dec ✘` | `0` | `0` | `a row ✘` |
| `MIN` | `1` | `1 ✘` | `urn:a ✘` | `_:b` | `unbound` | `unbound` | `a row ✘` |
| `MAX` | `2` | `2` | `2` | `2` | `unbound` | `unbound` | `a row ✘` |
| `SAMPLE` | `a member` | `a member` | `a member` | `a member` | `unbound` | `unbound` | `a row ✘` |
| `GROUP_CONCAT(expr)` | `1 2` | `1 2 ✘` | `1 2 urn:a` | `1 2 _:b ✘` | ` ✘` | `""` | `a row ✘` |
| `GROUP_CONCAT(?x)` | `1 2` | `1 2 ✘` | `1 2 urn:a` | `1 2 _:b ✘` | ` ✘` | `""` | `a row ✘` |
| `GROUP_CONCAT(DISTINCT expr)` | `1 2` | `1 2 ✘` | `1 2 urn:a` | `1 2 _:b ✘` | ` ✘` | `""` | `a row ✘` |

54 of 91 cells conform.

**Oxigraph 0.5.13**

| Aggregate | all bound | one unbound | one IRI | one blank node | all unbound | empty | empty, grouped |
|---|---|---|---|---|---|---|---|
| `COUNT(?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(expr)` | `2` | `unbound ✘` | `3` | `unbound ✘` | `unbound ✘` | `0` | `none` |
| `COUNT(*)` | `2` | `3` | `3` | `3` | `2` | `0` | `none` |
| `COUNT(DISTINCT ?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(DISTINCT expr)` | `2` | `unbound ✘` | `3` | `unbound ✘` | `unbound ✘` | `0` | `none` |
| `SUM` | `3` | `unbound` | `unbound` | `unbound` | `unbound` | `0` | `none` |
| `AVG` | `1.5 dec` | `unbound` | `unbound` | `unbound` | `unbound ✘` | `0` | `none` |
| `MIN` | `1` | `unbound` | `<urn:a>` | `_:b` | `unbound` | `unbound` | `none` |
| `MAX` | `2` | `unbound` | `2` | `2` | `unbound` | `unbound` | `none` |
| `SAMPLE` | `a member` | `a member` | `a member` | `a member` | `unbound` | `unbound` | `none` |
| `GROUP_CONCAT(expr)` | `1 2` | `unbound` | `1 2 urn:a` | `unbound` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(?x)` | `unbound` | `unbound` | `unbound` | `unbound` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(DISTINCT expr)` | `1 2` | `unbound` | `1 2 urn:a` | `unbound` | `unbound` | `""` | `none` |

84 of 91 cells conform.

**Jena ARQ 5.1.0**

| Aggregate | all bound | one unbound | one IRI | one blank node | all unbound | empty | empty, grouped |
|---|---|---|---|---|---|---|---|
| `COUNT(?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(expr)` | `2` | `2` | `3` | `3 ✘` | `0` | `0` | `none` |
| `COUNT(*)` | `2` | `3` | `3` | `3` | `2` | `0` | `none` |
| `COUNT(DISTINCT ?x)` | `2` | `2` | `3` | `3` | `0` | `0` | `none` |
| `COUNT(DISTINCT expr)` | `2` | `2` | `3` | `3 ✘` | `0` | `0` | `none` |
| `SUM` | `3` | `unbound` | `unbound` | `unbound` | `unbound` | `0` | `none` |
| `AVG` | `1.5 dec` | `unbound` | `unbound` | `unbound` | `unbound ✘` | `0` | `none` |
| `MIN` | `1` | `unbound` | `<urn:a>` | `_:b` | `unbound` | `unbound` | `none` |
| `MAX` | `2` | `unbound` | `2` | `2` | `unbound` | `unbound` | `none` |
| `SAMPLE` | `a member` | `a member` | `a member` | `a member` | `unbound` | `unbound` | `none` |
| `GROUP_CONCAT(expr)` | `1 2` | `unbound` | `1 2 urn:a` | `1 2 _:b ✘` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(?x)` | `1 2` | `unbound` | `1 2 urn:a` | `1 2 _:b ✘` | `unbound` | `""` | `none` |
| `GROUP_CONCAT(DISTINCT expr)` | `1 2` | `unbound` | `1 2 urn:a` | `1 2 _:b ✘` | `unbound` | `unbound ✘` | `none` |

84 of 91 cells conform.

One departure is about a function and not an aggregate. rdflib and Jena return the label for `STR` of a blank node, where the specification has an error, which is the cause of the crosses in the `one blank node` column of the `expr` rows for them. That column tests `STR`.

Oxigraph's `COUNT(expr)` is reported upstream as [oxigraph/oxigraph#2009](https://github.com/oxigraph/oxigraph/issues/2009), fixed by the merged [oxigraph/oxigraph#2025](https://github.com/oxigraph/oxigraph/pull/2025), commit 96e22c9, after 0.5.13 and not in a release by 2026-11-12. A source build of that commit, in memory, gives 1 for `COUNT(STR(?x))`, `COUNT(DISTINCT STR(?x))` and `COUNT(LCASE(STR(?x)))` over one bound and one unbound row, where 0.5.13 gives unbound. The report as filed expected 1 for `COUNT(?x + 0)` over `<urn:a>` and an unbound row, which is wrong, since `<urn:a> + 0` is a type error too and the count is 0, as the fixed build gives. Over numeric rows the fixed build gives 1. The other departures below are not filed. They are Oxigraph's `AVG` over an all-unbound group, Jena's `AVG` over the same, Jena's `STR` of a blank node, rdflib's `STR` of a blank node, rdflib skipping or counting error rows, rdflib's `SUM` over an IRI raising, and rdflib's grouped query over no rows. Jena's `GROUP_CONCAT(DISTINCT)` over an empty group is already reported upstream as apache/jena#4240, with a fix proposed in #4243, and rdflib's grouped query over no rows has an open pull request, RDFLib/rdflib#3498.

Summary of the aggregate departures.

| Engine | Departure |
|---|---|
| rdflib | skips an unbound or erroring row for `SUM`, `AVG`, `MIN`, `GROUP_CONCAT`, where the specification gives an error. Counts it for `COUNT(expr)`. Over an all-unbound group gives `0` for `SUM` and `""` for `GROUP_CONCAT`. `SUM` over an IRI or a blank node raises (the query fails with an `AttributeError`), `AVG` over them skips them. Returns one row for a grouped query over no rows. `MIN` over IRIs returns a plain literal (TSD-16). Gives `Max` `2` over a group with an unbound row, which the ordering reading allows |
| Oxigraph | `COUNT(expr)` and `COUNT(DISTINCT expr)` are unbound when any row errors, where §18.5.1.2 removes it. `AVG` over an all-unbound group is unbound where §18.5.1.4's first clause gives `0`. Bare `GROUP_CONCAT(?x)` of numbers is unbound, which is the strict reading. Unbound `Max` over a group with an unbound row is one of the two readings. Otherwise conforms |
| Jena ARQ | `COUNT(expr)` removes the row as §18.5.1.2 asks. `AVG` over an all-unbound group is unbound as Oxigraph's. `GROUP_CONCAT(DISTINCT expr)` over an empty group is unbound where the specification gives `""`. `STR` of a blank node gives its label. Unbound `Max` as Oxigraph. Otherwise conforms |

Remedies. Each form is probed on every state. A slash separates the results of rdflib, Oxigraph and Jena where they differ, and one value means they agree. Only the states that mean something for a remedy are discussed in the text.

| Remedy | Result on the four aggregates that matter |
|---|---|
| (a) `FILTER(BOUND(?x))` in the group | Gives the same answer on all three engines in the states `all bound`, `one unbound`, `all unbound` and `empty` for every aggregate except bare `GROUP_CONCAT(?x)` of numbers (unbound on Oxigraph, the strict reading) and `GROUP_CONCAT(DISTINCT ...)` over an empty group (unbound on Jena). It does not remove a row that is bound and of the wrong type, so `SUM` over an IRI is still an error (unbound on Oxigraph and Jena, a raise on rdflib). A `FILTER(isNumeric(?x))` does remove it, and `SUM`, `AVG`, `MIN` and `MAX` then agree on every engine in every state. The filter removes the row from `COUNT(*)` as well (3 becomes 2). It also removes a subject whose rows all go, so under `GROUP BY ?s` a group with no value is absent from the result and not a group with `SUM` 0 |
| (b) coalesce to the neutral element | `SUM(COALESCE(?x, 0))` agrees on all three engines wherever the values are numbers and equals the filtered sum, so it cannot tell a missing weight from a zero. `AVG(COALESCE(?x, 0))` gives `1` over 1, 2 and a missing row where the filter gives `1.5`, so it changes the denominator. `MAX(COALESCE(?x, 0))` is right only where 0 is below every value. `MIN` has no neutral element. `GROUP_CONCAT(COALESCE(STR(?x), ''))` gives ` 1 2`, an empty element and a stray separator, on every engine (an all-unbound group gives a single space), `SUM(COALESCE(?x, 0))` over an IRI is still an error. `COUNT(COALESCE(STR(?x), ''))` counts the missing row, so it agrees on all three engines and is `COUNT(*)` |
| (c) detect with `COUNT(*) - COUNT(?x)` | The number of rows with no value agrees on all three engines in every state (0, 1, 0, 0, 2, 0). `COUNT(*) - COUNT(STR(?x))` does not, since `COUNT(expr)` differs (TSD-41, above). `IF(COUNT(?x) = COUNT(*), SUM(COALESCE(?x, 0)), 1/0)` is the sum when no row is missing and unbound otherwise, on all three engines, and `0` for an empty group. It propagates the unknown where a coalesce would hide it. It does not catch a bound value of the wrong type |
| (d) a sub-select per row | Computing `?v` with `BIND(?x + 0 AS ?v)` or `BIND(STR(?x) AS ?v)` in a sub-select and aggregating `?v` outside is the bare aggregate again, since `?v` is unbound on the row. Oxigraph and Jena are unbound where the group has a missing row, and rdflib skips it. A `FILTER(BOUND(?v))` outside the sub-select is remedy (a) |

Three groups at once, `GROUP BY ?s`, one with two values, one with none and one with one value. Bare `SUM` is 3, then `0` on rdflib and unbound on Oxigraph and Jena, then 5. The filtered `SUM` is 3 and 5 with no row for the group with no value, on all three engines. `SUM(COALESCE(?x, 0))` is 3, 0, 5 on all three. The detecting `IF` is 3, unbound, 5 on all three. `AVG(COALESCE(?x, 0))` is 1.5, 0, 5 on all three.

**Why.** Specification and source.

- Oxigraph follows §18.5.1 as TSD-41 describes, through `AccumulatorWrapper` setting its accumulator to `None` on the first argument that evaluates to nothing. That gives unbound for `SUM`, `AVG`, `MIN`, `MAX`, `GROUP_CONCAT` and the departures above for `COUNT(expr)`. Its `AVG` over an all-unbound group takes the same path, so it does not reach §18.5.1.4's first clause.
- rdflib's `Sum.update` catches `NotBoundError` and passes ("skip UNDEF") and does not catch `SPARQLTypeError`, which is what an IRI raises from `numeric()`. The error surfaces as an `AttributeError` from the result object, because the evaluator's generator fails inside `SPARQLResult`. `Average.update` catches both and skips. `Extremum.update` catches both and skips ("skip UNDEF or BNode => SPARQLTypeError"), and `set_value` wraps the value in `Literal`, which is why `MIN` over an IRI is a plain literal (TSD-16). `Counter.update` skips a `NotBoundError` and counts any other error, so `COUNT(STR(?x))` counts the row. `evalAggregateJoin` ends with "there were no matches" and yields one empty row whether or not there is a `GROUP BY`, so a grouped query over no rows returns a row.
- Jena's `GROUP_CONCAT(DISTINCT)` over an empty group, and `STR` of a blank node, are observed and not explained.

No single rule fits every aggregate. A row with no value can mean "not a member of the group" (filter), "contributes the neutral element" (coalesce), or "the answer is unknown" (detect and propagate), and each aggregate has the choices the table shows. A coalesce is correct only where the aggregate has a neutral element and the meaning is that absence contributes nothing. `FILTER(BOUND(?x))` removes a group as well as a row. `COUNT(?x)` over a bare variable and `COUNT(*)` agree on all three engines and are the safe pair for detection, while `COUNT(expr)` is not portable, since each engine differs from the others (rdflib counts the row, Oxigraph is unbound, Jena removes the row, which §18.5.1.2 asks for). A grouped query over no rows returns a row on rdflib and none elsewhere. `SUM` over a term that is not a number raises on rdflib and is unbound elsewhere.

### TSD-50. On Jena ARQ in memory a failed request is rolled back, and CLEAR GRAPH of an absent graph fails where DROP GRAPH does not

**Evidence:** Observed on Apache Jena ARQ 5.1.0, in memory, run R-T for the request and run R-U for the departure, 2026-10-27. The specification text is SPARQL 1.1 Update §3.2.1 and §3.2.2, read from the W3C Recommendation. Jena's source was not read. A first answer to TSD-Q13 for Jena, for the in-memory dataset only.

**Context.** TSD-06 asked each store whether a request whose second operation fails keeps its first, because a two-operation replace needs the answer. TSD-Q13 asks it of Jena TDB2. Joining Jena to a conformance run made probe P2 ask it of Jena ARQ. **This is an in-memory dataset, `DatasetFactory.createTxnMem()`, and says nothing yet about TDB2.**

**How we found it.** P2 sends `INSERT DATA { GRAPH <urn:g> { <urn:s> <urn:p> <urn:o> } } ; <a failing operation>` through a Java harness that runs each request in one write transaction, and reads back. The failing operations that Oxigraph and rdflib answer to were tried in turn, and the departure was then isolated in a Java program on `jena-arq` alone, drafted for upstream and not filed.

**What the store does.**

| Second operation | The request | The first insert |
|---|---|---|
| `LOAD <urn:no-such>` | fails | rolled back |
| `CLEAR GRAPH <urn:absent>` | fails, "No such graph" | rolled back |
| `ADD`, `MOVE` or `COPY` from `<urn:absent>` | fails, "No such graph" | rolled back |
| `INSERT ... WHERE { SERVICE <http://127.0.0.1:1/sparql> { ... } }` | fails | rolled back |
| `DROP GRAPH <urn:absent>` | succeeds | kept |
| `CREATE GRAPH <urn:new>` | succeeds, and the graph is not there afterwards | kept |
| `CLEAR SILENT GRAPH <urn:absent>`, `CLEAR DEFAULT`, `CLEAR ALL` on an empty dataset | succeed | not applicable |

The dataset does not record an empty graph. `CREATE GRAPH <g> ; CLEAR GRAPH <g>` fails, and so does `CLEAR GRAPH <g>` after the last triple of `<g>` was deleted. `DROP GRAPH <g>` succeeds in both cases. `CLEAR SILENT GRAPH` is accepted by all three engines.

**Why.** Not read in Jena's source. Each request is one write transaction and an exception aborts it, which is consistent with the rollback and is the same as what Oxigraph does (TSD-06). That `CLEAR` checks for the graph and `DROP` does not is observed only.

**Against the specification.** §3.2.1 and §3.2.2 say a store that records empty graphs SHOULD fail by default on a graph that does not exist, and that "stores that do not record empty graphs will always return success". Jena's dataset does not record them, so `CLEAR GRAPH` should succeed, as `DROP GRAPH` does. Departure, drafted for upstream and not filed. `ADD`, `MOVE` and `COPY` MAY fail on an absent source (§3.2.3 to §3.2.5), so they conform. A two-operation replace is therefore atomic on Jena in memory, as on Oxigraph and unlike rdflib. Whether TDB2 does the same is TSD-Q13.

---

### TSD-51. Jena ARQ matches a zero-length path from a term in no triple with the term itself

**Evidence:** Observed on Apache Jena ARQ 5.1.0 in memory, run R-T, 2026-10-27. Extends TSD-08 and narrows TSD-Q3.

**What the store does.**

```sparql
# the graph <urn:h> holds only <urn:a> <urn:p> <urn:b>
SELECT ?x WHERE { GRAPH <urn:h> { <urn:lonely> <urn:p>* ?x } }
SELECT ?x WHERE { GRAPH <urn:h> { <urn:lonely> <urn:p>+ ?x } }
```

| Store | `*` | `+` |
|---|---|---|
| rdflib 7.6.0 | `<urn:lonely>` | not tested |
| Oxigraph (TSD-08) | no rows | not tested |
| Jena ARQ 5.1.0 | `<urn:lonely>` | no rows |

Two of three engines match the term, which supports TSD-08's reading and leaves Oxigraph as the outlier, though TSD-Q3 is still unanswered from the specification's text.

**Addendum, 2026-11-01.** A sweep's path alone, with its root in no triple, yields the root on Jena TDB2, Fuseki over TDB2 and RDF4J 6.1.0 too, and still nothing on Oxigraph 0.5.13 (TSD-65). Five engines of six match the term. The sweep is still unaffected, since with `?s ?p ?o` it yields nothing on all six.

---

### TSD-52. Jena ARQ keeps xsd:long, counts distinct terms, groups arithmetic from the left and divides integers into a decimal

**Evidence:** Observed on Apache Jena ARQ 5.1.0 in memory, run R-T, 2026-10-27. Extends TSD-07, TSD-17 and TSD-18.

**What the store does.** The probes of those entries, on Jena.

| Probe | Jena ARQ | Oxigraph | rdflib |
|---|---|---|---|
| `"1"^^xsd:long` stored, `DATATYPE` of a constant, a substituted parameter, a `VALUES` row and the stored value | `xsd:long` in all four, and the stored value equals the constant | `xsd:integer` (TSD-07) | `xsd:long` |
| `COUNT(DISTINCT ?o)` over `1^^long`, `1^^integer`, `2^^long` | 3 | 2 (TSD-18) | 3 |
| the same over a `VALUES` row of `1^^long` and `1^^integer` | 2 | 1 | 2 |
| `10 - 3 + 2`, `10 - 3 - 2` | `9`, `5` as `xsd:integer` | `9`, `5` from 0.5.12 | `9`, `5` |
| `8 / 4 / 2` | `"1.0"^^xsd:decimal` | `1` | `1` |

The last row is the specification's. A division of two integers is an `xsd:decimal` (§17.3), and Jena writes the lexical form `1.0`. A test that reads the result with `int(...)` of the term fails on `1.0`, so it must read the number with `int(float(...))` for every engine. The three values `[9, 5, 1]` are as strict as before. Jena departs from the specification in none of these. Oxigraph's canonicalisation is the departure, recorded in TSD-07.

---

### TSD-53. Jena ARQ applies a DELETE/INSERT in the specification's order, skips an unbound template graph and tests the right graph in TSD-34's form

**Evidence:** Observed on Apache Jena ARQ 5.1.0 in memory, run R-T, 2026-10-27. Extends TSD-01, TSD-05 and TSD-34.

**What the store does.**

| Probe | Specification | Jena ARQ 5.1.0 |
|---|---|---|
| P1, `DELETE { ?d } INSERT { ?i } WHERE { VALUES ... }` where one solution deletes the triple another inserts | every deletion, then every insertion | the triple survives with the inserting row first and with the deleting row first, as on Oxigraph from 0.5.12 |
| P4, `INSERT { GRAPH ?g { ... } } WHERE {}` with `?g` unbound | the quad is skipped | nothing is written, as on Oxigraph |
| TSD-34, `FILTER NOT EXISTS` with no `GRAPH` of its own inside a `GRAPH` block that holds a sub-select and a `UNION` | tests the block's graph | `b` and `c` with a constant and with a parameter, with and without the sub-select, as on Oxigraph |

Jena evaluates each in one run, with no hash seed. The rdflib columns that vary with `PYTHONHASHSEED` have no counterpart.

---

### TSD-60. An aggregate over IF of an unbound variable raises on rdflib and is unbound on Oxigraph and Jena

**Evidence:** Source and Observed, run R-V, on rdflib 7.6.0, pyoxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27. rdflib's source is `rdflib/plugins/sparql/aggregates.py` at 7.6.0, read locally. The specification text is SPARQL 1.1 Query §18.5.1. Extends TSD-43 to an argument that is a conditional.

**Context.** A read set aggregates `IF(?decision = "Permitted", 1, 0)` and two similar expressions over a record's per-value rows. It relies on `?decision` never being unbound. An audit predicted that an unbound `?decision` would make the aggregate an error on Oxigraph and Jena and a skipped row on rdflib. We probed it on all three.

**How we found it.** A probe runs the three aggregate expressions of the generated query over `VALUES ?c { 7 "abc" }` with `BIND(IF(<test on ?c>, "Permitted", "Denied") AS ?decision)`, where the test is an error for `"abc"`.

| Aggregate, with `?decision` unbound for one of two rows | rdflib 7.6.0 | Oxigraph 0.5.13 | Jena ARQ 5.1.0 |
|---|---|---|---|
| `SUM(IF(?decision = "Permitted", 1, 0))`, the error being `?c + 1 > 5` | ✘ `AttributeError` when the result is read | unbound | unbound |
| the second flag, with `&& COALESCE(STR(?diagnostic), "") = ""` | ✘ `AttributeError` | unbound | unbound |
| `MAX(IF(?decision = "Permitted", COALESCE(STR(?diagnostic), "z"), ""))` | ✘ no row | unbound | unbound |
| `SUM(IF(COALESCE(?decision, "") = "Permitted", 1, 0))` | `1` | `1` | `1` |

**What the store does.** Oxigraph and Jena follow §18.5.1, an error in the argument makes `SUM` and `MAX` an error, and the projected value is unbound. rdflib's `Sum.update` receives the `SPARQLTypeError` object that the evaluated `IF` returns, treats it as a literal and fails at `value.datatype` (`aggregates.py`, line 133). Because the query is evaluated lazily, the error surfaces on the first read of the result and not from `query()`. The audit's own error, `"abc" >= 5`, does not reach this on rdflib, since rdflib answers it (TSD-61).

Oxigraph and Jena agree in every cell, so the dependence on `?decision` is a property of the query and not of one engine. On rdflib a malformed value would crash the query where the other two engines return an unbound count. `COALESCE(?decision, "")` gives `1` on all three engines for the arithmetic error, which makes the row count as none of the outcomes. A standalone failing test is written up and not filed upstream.

---

### TSD-61. rdflib orders a string against a number where the specification gives a type error

**Evidence:** Observed, run R-V, on rdflib 7.6.0, pyoxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27. The specification text is SPARQL 1.1 Query §17.3, whose operator mapping defines `<`, `>`, `<=` and `>=` for pairs of one kind. rdflib's code was not read.

**What the store does.** `BIND(("abc" >= 5) AS ?r)` is `true` on rdflib, `"abc" < 5` is `false` and `5 < "abc"` is `true`. Oxigraph and Jena leave `?r` unbound. `"abc" = 5` is `false` on all three, as the specification says. A string against a string orders the same on all three. A test such as a containment check `?candLower >= 5` therefore decides on rdflib where a candidate value is a string, and drops the row on Oxigraph and Jena. This is the divergence an audit saw and attributed to the aggregate. It is the comparison. A query that orders a value of unchecked type is guarded with `isNumeric`. A standalone failing test is written up and not filed upstream.

---

### TSD-54. A blank node label loaded twice is one node on rdflib and Jena with labels kept, and two on Oxigraph

**Evidence:** Observed, run R-X, on rdflib 7.6.0, pyoxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27. Found when a Java harness loaded data in chunks. Extends TSD-29.

**Context.** The harness writes a batch of quads as N-Quads and loads it, so two calls that name `_:b` should meet at one node, as an Oxigraph caller who builds a node with `BlankNode("b")` expects. Jena's parser gave two nodes.

**What the engine does.** The same line, `_:b <urn:p> "x" .`, loaded twice into one store, then the distinct subjects counted.

| Engine | Nodes after two loads |
|---|---|
| rdflib 7.6.0, `Dataset.parse(format="nquads")` twice | 1 |
| pyoxigraph 0.5.13, `Store.load` twice | 2 |
| Jena ARQ 5.1.0, `RDFParser` with its default label handling, twice (a Java program) | 2 |
| Jena ARQ 5.1.0, `RDFParser` with `labelToNode(LabelToNode.createUseLabelEncoded())` | 1 |

A test that adds quads in several calls and refers to a blank node by label gets one node on rdflib and two on Oxigraph, so it must not rely on either. Passing `createUseLabelEncoded()` for every load makes a label mean the same node across loads, as in rdflib, and that is a choice and not Jena's default. TSD-29 holds on every engine for a parse that is meant to be independent.

---

### TSD-55. rdflib's n3 writes a multi-line literal in triple quotes, which N-Quads refuses

**Evidence:** Observed, run R-X, on rdflib 7.6.0, pyoxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27. Found when a Java harness wrote N-Quads from rdflib terms.

**What happens.** `Literal("a\nb").n3()` is `"""a<newline>b"""`, the Notation3 and Turtle long form. An N-Quads line built from it is refused by pyoxigraph (`SyntaxError`) and by Jena (a parse error). rdflib's `Graph.serialize(format="nt")` writes the same literal as `"a\nb"` and is correct. `n3()` is a Notation3 writer and not an N-Quads one, so nothing here departs from a specification. Code that builds N-Quads from rdflib terms must escape a literal itself or serialise with the `nt` writer.

---

### TSD-56. Jena ARQ evaluates a sub-select on its own and orders IRIs as IRIs

**Evidence:** Observed, run R-X, on Apache Jena ARQ 5.1.0, 2026-10-27. Extends TSD-15 and TSD-16, and answers their part of TSD-Q12 for in-process ARQ.

**What the engine does.** The queries of TSD-15 and TSD-16, with the same data.

| Query | Specification | rdflib | Oxigraph | Jena ARQ |
|---|---|---|---|---|
| the count with `?s` bound outside the sub-select | 3 | 1 | 3 | 3 |
| the count inside `GRAPH <urn:g1>` | 2 | 2 | 2 | 2 |
| `MIN` and `MAX` over `<urn:o:2>`, `<urn:o:10>`, `<urn:o:1>` | the lowest and highest IRI | the literals `"urn:o:1"` and `"urn:o:2"` | `<urn:o:1>` and `<urn:o:2>` | `<urn:o:1>` and `<urn:o:2>` |
| `IRI(MIN(STR(?o)))` | an IRI | `<urn:o:1>` | `<urn:o:1>` | `<urn:o:1>` |

TSD-17 and TSD-18 were run on Jena (TSD-52), and probes repeat them on the three engines. Jena agrees with Oxigraph and with the specification where rdflib departs. Whether Fuseki differs from in-process ARQ is TSD-Q12's remainder.

---

### TSD-57. Jena ARQ shows none of the growth that TSD-25 to TSD-27 record on rdflib

**Evidence:** Observed with a query-cost benchmark, run R-W, on rdflib 7.6.0, pyoxigraph 0.5.13 and Apache Jena ARQ 5.1.0, 2026-10-27, twice, with the same shape both times. Answers TSD-Q18 for in-process, in-memory ARQ.

**Context.** TSD-25 to TSD-27 measure how the order of a group's patterns and the place of a `FILTER` change the cost of a SHACL-SPARQL constraint on rdflib. The rules built on them assume only rdflib is sensitive. The benchmark's engine list now has Jena, which runs each query through a harness with the focus node substituted into the text, as for Oxigraph. Every call crosses a pipe to the JVM, which costs about 0.15 ms and sets Jena's floor. The JVM runs with its full compiler.

**What the engine does.** Milliseconds per focus node, the naive form over the improved form, at the smallest and largest size (the second run, the first was within 10%).

| Experiment | rdflib | Oxigraph | Jena ARQ |
|---|---|---|---|
| E1, typing a bound variable (16 and 256 schemes) | 1.6x and 14.9x | 1.0x and 1.0x | 1.8x and 1.1x |
| E2, uniqueness by a join (200 and 800 nodes) | 15.7x and 61.1x | 1.0x and 1.0x | 3.4x and 2.7x |
| E3, guards beside an enumeration (10 and 30 values) | 16.2x and 20.8x | 1.1x and 1.3x | 0.9x and 1.1x |
| E4, absence as `OPTIONAL` and `!BOUND` (600 and 1200 nodes) | 1.2x and 1.0x | 1.0x and 1.0x | 1.6x and 1.2x |
| E5, from text on every run, in milliseconds | 12.7 (0.28 prepared) | 0.105 | 0.526 |

The growth exponents of the time per focus node with size are about 1 on rdflib for the naive forms of E1 and E2 and 2 for E3, and between -0.5 and 0.06 on Jena for both forms. The first size on Jena is inflated by the JVM warming up (E1 at 16 schemes costs 0.68 ms against 0.17 at 256).

In-process ARQ does not show TSD-25 to TSD-27's sensitivity. The one difference is E2, where Jena's naive form is 2 to 3 times the improved one at every size, a constant factor and not growth. Jena's absolute times are dominated by the pipe, so the ratios are the evidence and not the milliseconds. The measure is for ARQ over an in-memory dataset. Whether Fuseki over TDB2 differs is TSD-Q18's remainder. A cost is a hypothesis across engines.

---

### TSD-45. Oxigraph passes a constant graph variable into an `OPTIONAL` before 0.5.13, and a computed one never

This finding is recorded with TSD-42, which is the earlier record of the same behaviour. It was found again on 2026-10-28 from the failing probe, and its account of the computed form, which Oxigraph never passes in, and its measurements of the generated replace on Oxigraph, rdflib, Jena and RDF4J (runs R-M and R-N) are carried into TSD-42. The identifier stays, since a store finding is never removed.

### TSD-48. RDF4J counts 0 for `COUNT(*)` over a pattern with no variable that matches

**Evidence:** Observed on RDF4J 6.1.0 `MemoryStore`, run R-N, 2026-10-28. Not tested on other RDF4J versions or stores, nor on rdflib, Oxigraph or Jena. The mechanism is not read from source.

**Context.** Building the Jena and RDF4J side of a graph-naming benchmark (TSD-42), a check of the form `SELECT (COUNT(*) AS ?n) WHERE { GRAPH <g> { <s> <p> "v2" } }` was meant to confirm that a replace had written one triple. It passed on Jena and failed on RDF4J, although a dump of the store showed the triple present.

**How we found it.**

1. *Observed.* The replace was run on RDF4J and the whole store listed. The triple was there.
2. *Observed.* Four counts over a store holding `GRAPH <urn:g> { <urn:s> <urn:p> "v2" }`.

   | Query | RDF4J 6.1.0 |
   |---|---|
   | `COUNT(*)` over `GRAPH <urn:g> { <urn:s> <urn:p> "v2" }` | **0** |
   | `COUNT(*)` over `GRAPH <urn:g> { <urn:s> <urn:p> ?o FILTER(?o = "v2") }` | 1 |
   | `COUNT(*)` over `GRAPH <urn:g> { <urn:s> <urn:p> "absent" }` | 0 |
   | `COUNT(*)` over `<urn:s> <urn:p> "v2"`, no `GRAPH` | **0** |

**What the store does.** SPARQL 1.1 evaluates a basic graph pattern with no variable to one empty solution when it matches and to none when it does not, so `COUNT(*)` is 1 and 0. RDF4J gives 0 in both cases. A pattern with one variable gives the right count. A check that counts over a pattern with no variable therefore cannot tell a match from a miss on RDF4J. Binding a variable and filtering on it, or using `ASK`, avoids it.

**Why.** Not read. The matching case may be evaluated as a boolean test and the count taken over no projected variable.

### TSD-62. Fuseki on TDB2 applies concurrent updates one at a time, and a read does not wait for them

**Evidence:** Observed on Apache Jena Fuseki 6.2.0 with a TDB2 dataset on macOS, 2026-10-28, choosing how many store writers an application runs. Not read from Jena's source, though Jena documents TDB2's transactions as multiple readers and a single writer.

**Context.** The application routes every write through one writer per partition of workspaces, and the number of writers is a setting. Whether more writers write faster depends on whether the store runs their updates in parallel.

**How we found it.** Eight `INSERT DATA` requests of 20,000 triples each, into eight graphs of one dataset, sent one after another and then all at once from eight threads, twice, with the graphs dropped between runs. Then one read (`ASK`) sent while a 200,000-triple insert ran.

| Run | Sequential | Concurrent | Ratio |
|---|---|---|---|
| 1 | 3.09 s | 2.91 s | 0.94 |
| 2 | 2.66 s | 2.82 s | 1.06 |

The read answered in 8 ms and 3 ms while the insert ran.

**What the store does.** Concurrent updates to one dataset take as long as the same updates in sequence, so they are applied one at a time. A read is not blocked by a write in progress, which is what a reader working on the last committed state gives.

### TSD-63. rdflib rewrites an `xsd:dateTime`'s `Z` as `+00:00` when it parses

**Evidence:** Observed on rdflib 7.6.0, 2026-10-28, when a writer compared a job graph a worker sent back with the one the service wrote. Not tested on Oxigraph.

**Context.** A service writes a job node with `"2026-10-28T08:37:17Z"^^xsd:dateTime`. A worker reads the job graph with rdflib and sends it back with its status. The service's writer refused the update as a change to the service's statements, which it was not.

**How we found it.** Comparing the statements of the job node in the store and in the worker's message, term by term. Every statement was the same except the time, which came back as `"2026-10-28T08:37:17+00:00"`.

**What the store does.** rdflib normalises the lexical form of an `xsd:dateTime` when it parses one, so a graph read and written by rdflib holds a different term with the same value. A comparison by term, such as Jena's isomorphism, sees a change. To see it, run `Graph().parse(data='<urn:a> <urn:p> "2026-10-28T08:37:17Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .', format="turtle")` and serialise it. A check that a graph round-tripped through rdflib is unchanged must compare literals by value (Jena's `Node.sameValueAs`) and not by term, and an identity, a hash or a canonical form taken over such a literal must fix its lexical form first.

### TSD-44. A small update on Fuseki TDB2 costs its commit, so batching updates is the lever for write throughput

**Evidence:** Observed on Apache Jena Fuseki 6.2.0 with a TDB2 dataset on macOS (APFS), 2026-10-28, following TSD-62. Not read from Jena's source. Not measured on Linux or Windows, whose file systems sync differently.

**Context.** TSD-62 showed that TDB2 applies one update at a time. Whether the cost of each is the work or the commit decides whether batching helps.

**How we found it.** 200 updates shaped like a save of one workspace, each dropping one graph and inserting 50 triples into it, sent four ways, twice, with the graphs dropped between runs.

| Form | Run 1 | Run 2 | Per save |
|---|---|---|---|
| one request each, in sequence | 13.25 s | 13.75 s | 66 to 69 ms |
| one request each, from 8 threads | 12.85 s | 12.72 s | 64 ms |
| 10 requests of 20 operations | 0.80 s | 0.84 s | 4 ms |
| 1 request of 200 operations | 0.18 s | 0.18 s | 0.9 ms |

**What the store does.** Nearly all of a small update's cost is fixed per request, which Fuseki runs as one write transaction and TDB2 commits to disk, about 65 ms here. The operations inside a request cost under a millisecond each. On TDB2, write throughput is therefore about 15 transactions a second, however many writers send them, and merging the updates waiting into one request multiplies that by the batch size. A batch shares one transaction, so it commits or fails as one.

**Why.** A Jena dataset's transactions are multiple readers and a single writer, which is why concurrent updates queue (TSD-62), and that is Jena's transaction model, not TDB2's alone, since Jena's in-memory dataset has it too. The fixed cost is TDB2's: a commit makes its journal durable before it answers. We infer this from the measurements, and did not read the source or measure the in-memory dataset.

**Addendum, 2026-10-31.** An application's own updates, logged by Fuseki 6.2.0 on the same kind of machine, took 52 ms to 104 ms each on TDB2 whatever their size up to 20,000 characters, and 1 ms to 9 ms each on Fuseki's in-memory dataset, which confirms the commit as the fixed cost. A save of 1,000 triples took 121 ms on TDB2. Still not measured on Linux, where a sync costs less, so the floor here is a hypothesis for a server.

### TSD-64. A cast lookup key joins a stored integer by value on both stores, and Oxigraph joins `xsd:long` with `xsd:integer` too

**Evidence:** Observed on Oxigraph 0.5.13 and rdflib 7.6.0, 2026-10-29, with the probe below. Not read from either engine's source.

**Context.** A lookup's default comparison casts the source value with `STRDT` to the reference data's datatype and joins the result as a term, so an index can serve it. Whether the join is by term or by value decides what an author must name as that datatype.

**What the store does.** Two reference triples, `<urn:a> <urn:n> "7"^^xsd:integer` and `<urn:b> <urn:n> "7"^^xsd:long`, and a pattern `?s <urn:n> ?c` after `BIND (STRDT(text, datatype) AS ?c)`. For example `SELECT ?s WHERE { BIND (STRDT("007", xsd:integer) AS ?c) ?s <urn:n> ?c }`, with each cast, on each store.

| Cast | Oxigraph matches | rdflib matches |
|---|---|---|
| `"7"` to `xsd:integer` | a, b | a |
| `"007"` to `xsd:integer` | a, b | a |
| `"7"` to `xsd:long` | a, b | b |

**Why.** Not read. Oxigraph stores every integer-derived literal as an `xsd:integer` value (TSD-07), so a cast to either type joins both. rdflib joins two literals of one numeric datatype by value, so `"007"` finds 7, and keeps the derived types apart. A cast lookup must therefore name the reference data's own datatype. Oxigraph forgives naming a sibling integer type and rdflib does not, and leading zeros never stop a numeric cast lookup from matching on either store.

### TSD-65. A `DELETE`/`INSERT` fills every template quad once per solution on every engine measured, whether or not the quad is then skipped, and only Jena applies a quad with no variable once per operation

**Evidence:** Observed on Apache Jena 5.1.0 in memory, TDB2 5.1.0 on disk, Fuseki main 5.1.0 over HTTP on TDB2, RDF4J 6.1.0 MemoryStore, Oxigraph 0.5.13 and rdflib 7.6.0, 2026-10-31, on an Apple arm64 laptop. Source for Jena: `org.apache.jena.sparql.modify.UpdateEngineWorker.visit(UpdateModify)` and `TemplateLib.calcQuads` at the tag `jena-5.1.0`. Source for Oxigraph: TSD-01. Extends TSD-02, which measured the composite replace on rdflib and Oxigraph. This measures the named-graph replace on six engines and separates the payload's cost from the patch log's.

**Context.** A generated replace sweeps the aggregate's graph in its `WHERE`, one solution per stored triple (M), and its `INSERT` template holds the payload (N), the patch log's delta (D, in two blocks whose graphs are computed by `BIND`) and the row, claim and receipt (k, about 12). We asked which of four forms is linear and where. The forms are these.

- **F0.** The replace as generated, with the whole payload and the delta in the `INSERT` template.
- **F2′.** The payload and every other `INSERT` block under a graph variable bound in one branch of a `UNION`, so that the sweep's solutions leave them unbound and the specification skips them.
- **F1′.** Two operations, the guarded delete and then the guarded insert.
- **F2.** One operation, the payload and the delta carried as `VALUES` rows in `UNION` branches, so the template holds three single quads and k.

Each form replaced M stored triples by a payload of N = M that kept half of them and changed half, with the real delta in the slots, three times, best time kept, and the result checked as sets after each. Two families were measured, a named-graph replace and a composite replace whose sweep follows an owned property path.

**How we found it.**

1. *Source, Jena.* `visit(UpdateModify)` collects every binding into a `DataBag`, runs `execDelete` over all of them and then `execInsert` over all of them. Each splits its template into quads with no variable, applied once when the `WHERE` has a solution, and quads with a variable, instantiated once per binding by `calcQuads`, which substitutes the binding into every template quad and drops a quad that is not concrete.
2. *Observed.* The four forms above.

**What the store does.** Milliseconds, best of three, at M = N.

| Form | M | Jena in memory | Jena TDB2 on disk | Fuseki over HTTP | Oxigraph 0.5.13 | rdflib 7.6.0 | RDF4J 6.1.0 |
|---|---|---|---|---|---|---|---|
| F0, real delta | 1,000 | 2,313 | 1,821 | 2,205 | 527 | 10,785 | 328 |
| F0, real delta | 3,000 | 27,986 | 17,380 | 17,172 | 6,466 | 95,133 | 2,397 |
| F0, empty delta | 1,000 | 62 | 128 | 127 | 253 | 5,722 | 124 |
| F0, empty delta | 3,000 | 155 | 209 | 239 | 2,647 | 49,314 | 1,084 |
| F2′, payload under a variable graph | 3,000 | 878 | 1,007 | 1,039 | 1,296 | not run, per solution | not run, per solution |
| F2′ | 10,000 | 8,829 | 9,113 | 10,962 | 14,235 | | |
| F1′, two operations | 3,000 | 168 | 184 | 180 | 12 | 16,794 | 142 |
| F1′ | 10,000 | 278 | 385 | 401 | 44 | 186,344 | 1,368 |
| F2, `VALUES` rows | 3,000 | 425 | 438 | 504 | 78 | not run, per solution | not run, per solution |
| F2 | 10,000 | 907 | 1,245 | 1,178 | 271 | | |

F0 at 10,000 was not run, since TSD-02 recorded it not finishing on Oxigraph in 8 GB. Every run that ran left exactly the payload, the delta in the add and del graphs, the row at the next sequence and nothing else changed. The second run of the JVM table moved two cells by more than a factor of two, Jena in memory F0 with an empty delta at 3,000 (155 to 338 ms) and F2′ at 1,000 (159 to 408 ms), so the small Jena cells are a hypothesis about one machine, and the growth from 1,000 to 3,000 and from 3,000 to 10,000 is what the table shows.

*The composite family, 2026-11-01.* A composite replace over an owned path, with a bystander aggregate in the same data graph, M = 2 + 2K triples, half the members kept. The same three forms, derived by the same split. Every form left the bystander untouched at every size. The RDF4J column is TSD-67's addendum.

| Form | M | Jena in memory | Jena TDB2 on disk | Fuseki over HTTP | Oxigraph 0.5.13 | rdflib 7.6.0 | RDF4J 6.1.0 |
|---|---|---|---|---|---|---|---|
| F0, real delta | 1,000 | 1,974 | 1,863 | 1,824 | 821 | 10,219 | wrong |
| F0, real delta | 3,000 | 24,824 | 16,705 | 15,828 | 7,872 | 89,737 | wrong |
| F0, empty delta | 3,000 | 165 | 231 | 266 | 3,450 | 47,560 | wrong |
| F1′ | 3,000 | 187 | 251 | 264 | 16 | 16,976 | wrong |
| F1′ | 10,000 | 469 | 664 | 654 | 54 | not finished in 900 s | wrong |
| F2 | 3,000 | 346 | 430 | 460 | 81 | not run, per solution | not run, per solution |
| F2 | 10,000 | 1,021 | 1,361 | 1,408 | 271 | | |
| the sweep alone, `SELECT`, with the root's members | 3,000 | 18 | 12 | 19 | 2 | 34 | 38 |
| the sweep alone | 10,000 | 41 | 22 | 35 | 6 | 102 | 92 |

The sweep is never the cost. At 10,000 it is under 1% of F0 at 3,000 on every engine.

*The empty cases, 2026-11-01.* Five cases per form per family, each on a fresh store, checked as sets: E1 from an empty aggregate to 50 triples, E2 from 50 to empty, E3 from empty to empty, E4 an identical re-save of 50, E5 the composite's create with its root in no triple. F0, F1′ and F2 each left the same result wherever the form runs, on all six engines for F0 and on the four snapshot engines for the candidates: E1 and E5 create, E2 leaves the graph empty, E3 and E4 move the row and write a receipt and two empty delta graphs. The one exception is RDF4J on the composite family's F0 in E2 and E4, the failure of TSD-67's addendum, and E1, E3 and E5 are right there. After E2 the three Jena engines no longer list the data graph, since the dataset does not record an empty graph (TSD-50), and Oxigraph and rdflib still do.

*The zero-length path, 2026-11-01.* With the root in no triple, the composite sweep yields 0 solutions on all six engines. The path alone, without `?s ?p ?o`, yields the root on Jena in memory, TDB2, Fuseki, rdflib and RDF4J, and nothing on Oxigraph 0.5.13 (TSD-08, TSD-51).

**Why.**

- *F0 is quadratic on every engine, and on Jena it is the delta that costs.* On rdflib, Oxigraph and RDF4J the whole `INSERT` template is filled once per solution, as TSD-02 found, so F0 is M × (N + D + k). On Jena the payload is ground once the parameters are substituted, so it is applied once, and F0 with an empty delta is linear (155 ms at 3,000, a growth of 2.5 for 3.3 times the size). The two delta blocks name their graphs by `?addGraph` and `?delGraph`, computed by `BIND`, so they are template quads and are instantiated once per binding: M × D, which at D = N is 28 s at 3,000.
- *F2′ is quadratic on every engine.* The specification's definition is that each template is filled once per solution. A quad whose graph is unbound is skipped, as TSD-53 and TSD-66 say, but it is substituted and tested first. F2′ makes every `INSERT` quad a template quad, so every engine does M × (N + D + k) substitutions and discards them, at about 45 ns each on Jena and 70 ns on Oxigraph (8.8 s and 14.2 s at 10,000, which is 200 million). That is 20 to 70 times cheaper than F0's insertions and still quadratic. A form that keeps the payload in a template filled by M solutions cannot be linear on an engine that follows the definition, whatever names its graphs.
- *F1′ and F2 are linear, because neither fills the payload per sweep solution.* F1′'s first operation has M solutions and one template quad, and its second has one solution. F2 carries the payload in the solutions, so its template is 3 + k quads filled M + N + 2D + 1 times. F2 costs three to six times F1′ at 10,000, which is those k quads per solution and the parse of three `VALUES` blocks, and both grow linearly from 3,000 to 10,000 on every engine.

The cause of the quadratic cost is therefore the same on every engine and named: the payload, or on Jena the patch log's delta, sits in a template that M solutions fill. The two linear forms are F1′, which needs the request to be one transaction (TSD-66 for Jena, TSD-06 for Oxigraph's failure half), and F2, which needs every deletion applied before any insertion (P1, TSD-53, TSD-66, TSD-01 from Oxigraph 0.5.12). Neither is safe on rdflib or RDF4J. On Jena alone, F0 with the delta's graphs ground would be linear, but that is Jena's shortcut and not the specification, so it is not a form.

---

### TSD-66. Jena TDB2 and Fuseki over it apply an update as ARQ in memory does, roll back a request whose second operation fails, and apply a request the client abandoned

**Evidence:** Observed on Apache Jena 5.1.0, the in-memory dataset, TDB2 on disk in a temporary directory, and Fuseki main 5.1.0 in process over that TDB2 store driven over HTTP, 2026-10-31. Source for the first claim: `org.apache.jena.tdb2.modify.UpdateEngineTDB` at the tag `jena-5.1.0` extends `UpdateEngineMain` and overrides nothing but its constructor and its factory's `accept`, so TDB2 runs ARQ's `UpdateEngineWorker`, and Fuseki serves the dataset's engine. Answers the Jena half of TSD-Q13.

**Context.** TSD-50 and TSD-53 recorded ARQ in memory, and an application's store may be Fuseki over TDB2. A linear replace that is one operation needs the order (P1) and the skip (P4) there, and one that is two operations needs the request to be one transaction (P2).

**What the store does.** Each probe gave the same answer on all three.

| Probe | In memory | TDB2 on disk | Fuseki over HTTP |
|---|---|---|---|
| P1, `DELETE { ?d } INSERT { ?i } WHERE { VALUES ... }`, both row orders | the triple survives in both | the same | the same |
| P4, `INSERT { GRAPH ?g { ... } } WHERE { }` with `?g` unbound | nothing written | nothing written | nothing written |
| P2, `INSERT DATA { ... } ; LOAD <urn:no-such>` | the request fails and the insert is rolled back | the same | HTTP 500, and the insert is rolled back |
| P6, a blank node in an `INSERT` template over 3 solutions | 3 distinct blank nodes, 6 triples | the same | the same |
| P7, a replace at M = 3,000 sent over HTTP and abandoned by the client after 52 ms | | | at 2 s the graph is the old state, and at 22 s the new state |

The Jena engines ran P2 inside `Txn.executeWrite`, as a store layer runs every request.

**Why.** The engine is the same class on all three (source). P2 and P7 are the transaction, a request is one write transaction that an exception aborts and that a client going away does not, which TDB2's documentation says and the source was not read for. P6 is `TemplateLib.calcQuads`, which maps the template's blank nodes afresh for each binding, as the specification says.

What TSD-53 found for ARQ in memory therefore holds for Fuseki over TDB2, by source and by observation. A client that loses its connection during an update learns nothing from the failure, since the update is applied all the same, 22 seconds later at this size. A payload with a blank node is written once per solution of the operation that carries it, so a replace whose `WHERE` has one solution per stored triple would leave M copies of each blank node's structure.

---

### TSD-67. RDF4J 6.1.0's MemoryStore applies a `DELETE`/`INSERT` one solution at a time, writes a template quad whose graph is unbound, and rolls back a request whose second operation fails

**Evidence:** Observed on RDF4J 6.1.0, `MemoryStore` through `SailRepository`, 2026-10-31, with no transaction opened by the caller, so under the store's own default. Its source was not read: `SailUpdateExecutor` could not be fetched at that tag.

**What the store does.**

| Probe | RDF4J 6.1.0 MemoryStore |
|---|---|
| P1, both row orders | the triple is lost with the inserting row first and kept with the deleting row first, as Oxigraph 0.5.11 did (TSD-01) |
| P4, `?g` unbound in a template | one quad written, as rdflib does (TSD-05) |
| P2, `INSERT DATA { ... } ; LOAD <urn:no-such>` | the request fails and the insert is rolled back |

The generated named-graph replace F0 is right on RDF4J, since it relies on neither the order nor the skip, and it is quadratic there as on every per-solution store (TSD-65). A one-operation linear form is not safe on RDF4J. A two-operation form has the failure half of request atomicity, and its isolation half is not probed.

**Addendum, 2026-11-01. The generated composite replace F0 is wrong on RDF4J 6.1.0.** Every composite F0 of a non-empty aggregate, at M = 100 to 3,000 and in the edge cases E2 and E4 (TSD-65), leaves the old members in the data graph beside the new, the old head beside the new, and the add and del graphs empty, while the new payload, the new row and the claim are written. F1′ on the same store deletes nothing in its first operation and is otherwise right, and the named-graph family, whose sweep has no property path, is right in every row. The same `WHERE` run as a `SELECT *` binds every variable in every solution, the four sweep rows and `?addGraph`, `?delGraph`, `?prevRev`, `?txnKey` and `?now` among them. The update behaves as if every variable were unbound in every solution: the template quads with a variable in a triple are skipped, and the two blocks under `GRAPH ?addGraph` and `GRAPH ?delGraph` and the constant triples of the block under the log-graph variable are written into the default graph, which is where P4's quad goes. A diagnostic at M = 4 isolated the trigger. Removing the `OPTIONAL` around the sweep changes nothing, and running the update inside `begin()` and `commit()` changes nothing. Replacing the owned path by a plain `?s ?p ?o`, or by its first alternative alone, `$root ex:hasMilestone ?s`, gives the right result in every graph. So the trigger is a property path with `?` or `*` in an update's `WHERE`, on RDF4J 6.1.0, and `+` was not tested. The cause inside RDF4J is open (TSD-Q36). No generated composite operation, whose sweep is such a path, is right on RDF4J today, and the named-graph family is right because its sweep has none.

---

### TSD-68. Oxigraph refuses a prologue after the `;` of an update request

**Evidence:** Observed on Oxigraph 0.5.13 and rdflib 7.6.0, 2026-11-01, with the probe below, while building a bulk load. Jena ARQ in memory accepted the request in the same run. Not read from either engine's source.

**Context.** A bulk load and a write that records its state digest join several generated operations, each rendered with its own `PREFIX` lines, into one update request.

**What the store does.** SPARQL 1.1 Update's grammar lets each operation of a request carry a prologue (`Update ::= Prologue ( Update1 ( ';' Update )? )?`).

| Request | Oxigraph | rdflib | Jena ARQ |
|---|---|---|---|
| `PREFIX a: <urn:a#> INSERT DATA {...} ; PREFIX b: <urn:b#> INSERT DATA {...}` | refused, "expected one of CREATE, DELETE, INSERT" at the second `PREFIX` | applies both | applies both |
| the same with both prefixes before the first operation | applies both | applies both | applies both |

To see it, run `pyoxigraph.Store().update("PREFIX a: <urn:a#>\nINSERT DATA { GRAPH <urn:g> { a:x a:y a:z } } ;\nPREFIX b: <urn:b#>\nINSERT DATA { GRAPH <urn:g> { b:x b:y b:z } }")`.

### TSD-69. Jena keeps no empty named graph from a TriG body

**Evidence:** Observed on Jena 5 (a shaded writer) on 2026-11-01, when a browser suite saved a part by a patch that only added triples. Not read from Jena's source.

**Context.** A `PATCH` of a part carries two named graphs in TriG, the triples added and the triples removed, and an empty side was written as `<urn:example:patch:del> { }`.

**What the store does.** Parsed into a Jena `Dataset`, an empty graph block leaves no graph behind, so `containsNamedModel` answers false for it. A dataset has no way to hold a named graph that holds no triple, and Fuseki's datasets behave the same. `RDFParser.fromString("<urn:g> { }", Lang.TRIG).toDataset().containsNamedModel("urn:g")` answers false.

### TSD-70. No store compares an `owl:rational` literal by value

**Evidence:** Observed on rdflib 7.6.0, Oxigraph 0.5.13 and Jena ARQ 5.1.0 in memory, 2026-11-08, with the probe below, while choosing how a ledger records an exact proportion. Not read from any engine's source.

**Context.** OWL 2 defines `owl:rational`, whose lexical form is a fraction such as `"1/3"`, and a record that keeps an exact share would state it so, if a store could compare it.

**What the store does.** All three parse the literal, keep its lexical form and datatype, and write it back unchanged. None treats it as a number.

| Probe, over `"1/3"`, `"2/6"` and `"1/2"`, each `^^owl:rational` | rdflib | Oxigraph | Jena ARQ |
|---|---|---|---|
| parse, then write and read again | lexical form and datatype kept | lexical form and datatype kept | lexical form and datatype kept |
| `FILTER (?o = "1/3"^^owl:rational)` | matches `"1/3"` only | matches `"1/3"` only | matches `"1/3"` only |
| `FILTER (?o < "1/2"^^owl:rational)` | matches nothing | matches nothing | matches nothing |
| rdflib `Literal.eq` between `"1/3"` and `"2/6"` | raises `TypeError` | | |

So `"1/3"` and `"2/6"` are different terms and never equal values, and an order comparison is an error that drops the solution. A shape or query that must compare or sum a proportion cannot use the literal, and two `xsd:integer` values, a numerator and a denominator, compare by cross-multiplication on every store.

To see it, load the three triples `<urn:a> <urn:p> "1/3"^^<http://www.w3.org/2002/07/owl#rational>` (and `"2/6"`, `"1/2"`) into each store and run the two filters.

### TSD-71. An `OPTIONAL` join and a `UNION` that stands for it cost differently on each engine

**Evidence:** Observed on rdflib 7.6.0, Oxigraph 0.5.13 and Jena ARQ 5.1.0 in memory, 2026-11-11, Python 3.14.7 on an arm64 laptop, by a benchmark of a ledger's balance reads. No query plan was read, so the mechanism is not known (TSD-Q37).

**Context.** An account's balance is a sum over its postings in a set of named graphs, and a credit counts only if its kind is one the account accepts. The kind is an optional property of a posting, absent on a debit. A read joins to it either by `OPTIONAL` or by a `UNION` of the rows that need no kind and the rows that have one.

**How we found it.** The first full run used `OPTIONAL` on every engine. The kind-filtered read cost 4 times the plain sum on Oxigraph and about 1.2 times on the others. A run with the `UNION` form on every engine reversed it on rdflib. Each engine then read with its cheaper form, and both forced runs were kept for comparison.

**What the store does.** Inside `{ VALUES ?g { … } GRAPH ?g { ?p :position ?pos ; :direction ?d ; :amount ?a . FILTER(?pos <= N) JOIN } }`, summed with `SUM`, where JOIN is one of the two forms. Factor of the 27 reads against the same sum without the join, 20,000 postings, 30% credits:

| JOIN | rdflib | Oxigraph | Jena ARQ |
|---|---|---|---|
| `OPTIONAL { ?p :kind ?k }` | 1.2 | 4.1 | 1.4 |
| `{ ?p :direction :Debit } UNION { ?p :kind ?k }` | 6.0 | 1.7 | 1.6 |

A single read of one subtree on Oxigraph, outside the committed run, took about 24 ms for the sum, 199 ms with `OPTIONAL` and 33 ms with `UNION`.

**The two forms are not equivalent.** The `UNION` above drops a posting that is neither a debit nor has a kind, a credit with no kind, where `OPTIONAL` keeps it with `?k` unbound. The figures were measured on data in which every credit has a kind, so they compare like with like there, and a form that keeps such a credit, for example a third branch `{ ?p :direction :Credit . FILTER NOT EXISTS { ?p :kind ?z } }`, is being measured.

**What it means for a project.** The cheaper form of an optional join is a property of the engine, so a generated read that must run on several engines either chooses its form per engine or accepts a factor of 4 to 6 on one of them. Check that a `UNION` written for an `OPTIONAL` keeps every row the `OPTIONAL` keeps.

**Addendum, 2026-11-11, the corrected form and a second instance.** With a third branch keeping a credit that has no kind, `{ ?p :direction :Credit . FILTER NOT EXISTS { ?p :kind ?z } }`, the `UNION` form costs 0.2 to 0.4 of the plain read more than before, and each engine's cheaper form is unchanged. Factor against the plain read, 5,000 postings, 30% credits: rdflib `OPTIONAL` 1.7 and `UNION` 6.6, Oxigraph 4.1 and 2.1, Jena ARQ in memory called from Java 1.3 and 2.1, Jena TDB2 called from Java 1.3 and 2.5. The same benchmark found a second instance on Oxigraph: an ordered read with `OPTIONAL { ?p :applied ?ap }` gained almost nothing from a checkpoint that cut every other read on every engine to 0.02 to 0.05 of its cost, and a minimal probe confirmed the cause on 0.5.13: `VALUES ?g { <g> } GRAPH ?g { ?s :p ?o OPTIONAL { ?s :q ?x } }` cost 100 times as much over a store of 100 graphs as over one, though it reads one graph and returns the same rows, where the same query with no `OPTIONAL`, with a constant graph or with a `UNION` in its place did not grow. **Corrected 2026-11-11**, the same day: this is not Oxigraph's issue 2008, as first written. The same probe on 0.5.11 and 0.5.12 gives the same ratio of about 100, where issue 2008's shape regressed in 0.5.13 alone. Read from the 0.5.13 source, the optimizer rewrites the left join to a LATERAL, and the fitness check that would let the outer join with `VALUES ?g` pass `?g` in tests the rewritten join's right side against its own empty left side, finds a conflict that is not there, and falls back to a hash join that reads every graph. Moving `VALUES ?g` inside the block, or substituting `?g` before planning, removed the growth on all three versions, as that reading predicts. The upstream report, not yet filed, is LATTICE's `oxigraph-optional-inside-graph-reads-every-graph`. The rdflib case is still unexplained (TSD-Q37).

**Addendum, 2026-11-12, the fix built and checked.** The one-line change the reading suggests, checking the LATERAL's right side against the types its left side gives instead of the outer input types, was built on top of Oxigraph v0.5.13 with the in-memory store (rustc 1.95.0, release). The probe's ratio fell from about 100 to 1.0, with every result count unchanged across the eight forms the upstream report tabulates. No regression suite was run on it, since the optimizer crate has no unit tests of its own and the W3C suite needs the RocksDB build. The merged [oxigraph/oxigraph#2025](https://github.com/oxigraph/oxigraph/pull/2025), which fixes TSD-43's `COUNT(expr)`, does not touch the optimizer and leaves this at about 100. Neither change affects issue 2008's shape, about 77 on every build. This answers TSD-Q37 for Oxigraph's `OPTIONAL`, and rdflib's `UNION` stays open.

**Reproduce it.** `mise run check:ledger-credit-kinds` in LATTICE, with `LEDGER_KINDS_FORM=optional` or `union` to force one form on every engine.

### TSD-72. A running balance clamped at a ceiling is not one SPARQL query, and the ordered read that replaces it

**Evidence:** Reasoned from SPARQL 1.1, which has no window functions and whose aggregates are defined over a multiset with no order, and Observed for the cost on rdflib 7.6.0, Oxigraph 0.5.13 and Jena ARQ 5.1.0 in memory, 2026-11-11, by the benchmark of TSD-71.

**Context.** A credit never lifts an account above its stated amount, so where a ceiling can bind, an account's balance is a running balance applied in order with each credit clamped, and not a sum.

**What the store does.** No SPARQL 1.1 query computes it. A sum ignores order, and a clamp applied to the sum differs from a clamp applied at each step whenever a credit is clamped and a later debit follows. The form that works on every engine is `SELECT ?pos ?mp ?d ?a WHERE { … } ORDER BY ?pos ?mp`, folded by the caller. Its cost against the plain `SUM` of the same rows, 30% credits, for 1,000, 5,000 and 20,000 postings:

| | rdflib | Oxigraph | Jena ARQ, through a JSON pipe to Python |
|---|---|---|---|
| ordered read and fold | 1.56, 1.57, 1.21 | 1.90, 1.97, 1.95 | 3.43, 4.49, 4.89 |

The Jena figure includes every row crossing a pipe between processes, which a caller in the JVM would not pay, so it is an upper bound until a native run is recorded.

**What it means for a project.** A read that must apply a ceiling step by step moves every row it folds out of the store. On rdflib and Oxigraph that costs about what a filtered aggregate costs. A checkpoint bounds the rows the same way it bounds a sum.

**Addendum, 2026-11-11, a caller in the JVM.** Called natively from Java, the ordered read and fold costs 1.9, 2.4 and 2.6 times the plain sum on Jena ARQ in memory and 1.7, 1.8 and 2.0 times on Jena TDB2, for 1,000, 5,000 and 20,000 postings, about half what the pipe gave. The transfer of rows into Java took 2 to 3 ms at 20,000 postings, and the fold loop under a millisecond on every engine and caller, so the cost is the engine's ordered query.

**Reproduce it.** As TSD-71.

### TSD-73. Oxigraph on disk does not make concurrent updates wait for each other, so a guarded update loses writes, and Oxigraph in memory does

**Evidence:** Observed on pyoxigraph 0.5.13 with Python 3.14.7 on an 8-core arm64 laptop running macOS, 2026-11-12, by two independent probes. Source, from Oxigraph's files at the tag `v0.5.13`, read through a fetch of each file and not line by line. Documented for what the store promises. Not measured: Oxigraph's server over HTTP, any other version, and any other store.

**Context.** A ledger's writer reads the balances on a posting's path and then writes the posting. Two writers that both read and then both write can each act on a stale balance. The guard that closes the race is one revision triple moved by a `DELETE`/`INSERT`/`WHERE` on its expected value, so a stale writer matches nothing and writes nothing. The guard only works if the store either applies update requests one at a time or refuses the second of two that rewrite the same statement. We needed to know whether Oxigraph does.

**How we found it.**

1. *A benchmark's stress.* Threads in one process shared one `pyoxigraph.Store`, and each wrote postings through the guard, confirming a write by asking for its own posting. On the in-memory store 9 of 9 runs (2, 4 and 8 threads, 3 runs each, 1,000 requests) left one commit at each revision. On the on-disk store 8 of 9 left two commits at some revision, 2 to 32 of them a run, and both writers' confirmations succeeded.
2. *A minimal probe, written apart from the benchmark.* A counter triple and nothing else. Each thread reads the counter `N`, sends the guarded update below with its own marker, and asks whether its marker exists. It stops at 1,000. A second form increments in the store, with no read by the caller.
3. *The same probe with one lock.* A `threading.Lock` held around `Store.update` alone, with the reads and the confirmation outside it.
4. *Source.* `lib/oxigraph/src/storage/memory.rs`, `start_transaction`, takes `self.transaction_lock.lock()` and keeps the guard in the transaction until it commits or is dropped, under the comment "We ensure there is only one transaction running". `lib/oxigraph/src/storage/rocksdb_wrapper.rs` opens a plain RocksDB with `rocksdb_open_column_families`, never an optimistic or pessimistic transaction database. Its `start_readable_transaction` takes `rocksdb_create_snapshot` and an indexed write batch, and its `commit` calls `rocksdb_write_writebatch_wi` with no lock, no sequence check and no retry.
5. *Documentation.* The `Store` documentation for 0.5.13 promises the "repeatable read" isolation level and atomic transactions. It says nothing about two writers.

**What the store does.**

```sparql
# stored beforehand: <urn:c> <urn:v> 0 . A writer that read N sends, with its own marker <urn:w:t:i>:
DELETE { <urn:c> <urn:v> N }
INSERT { <urn:c> <urn:v> N+1 . <urn:w:t:i> <urn:at> N+1 }
WHERE  { <urn:c> <urn:v> N }
```

Markers confirmed, where 1,000 is correct, 3 runs a cell.

| Threads | In memory | On disk | On disk, a lock around `update` |
|---|---|---|---|
| 1 | 1,000 | 1,000 | not run |
| 2 | 1,000, 1,000, 1,000 | 1,576, 1,607, 1,539 | 1,000, 1,000, 1,000 |
| 4 | 1,000, 1,000, 1,000 | 1,611, 1,549, 1,542 | 1,000, 1,000, 1,000 |
| 8 | 1,000, 1,000, 1,000 | 1,368, 1,597, 1,308 | 1,000, 1,000, 1,000 |

On disk 270 to 607 revisions a run carried more than one marker. The counter itself ended at 1,000 as one triple, so nothing in the final state shows the loss except the markers.

```sparql
# the increment in the store, 500 times from each thread
DELETE { <urn:k> <urn:v> ?n } INSERT { <urn:k> <urn:v> ?m } WHERE { <urn:k> <urn:v> ?n . BIND(?n + 1 AS ?m) }
```

In memory the counter ended at threads times 500 as one triple, in every run. On disk it ended near 500 whatever the number of threads, and as several triples at once, from 1 to 60 of them, for example 502 and 503 together after two threads.

**Why.** In memory a write transaction holds a lock from its start to its commit, so a second update waits and evaluates its `WHERE` on the committed result of the first. On disk each update evaluates its `WHERE` on a snapshot taken at its start and then writes its batch. Nothing makes a second update wait and nothing compares its snapshot with what was committed since, so two updates that read the same revision both write. Each write is atomic and each read is repeatable, which is what the documentation promises. A lost update is outside that promise.

**What it means for a project.** A guard on an expected value, a counter in the store, and any check-then-write in one request are safe on Oxigraph in memory and unsafe on Oxigraph on disk, unless the caller sends updates one at a time. One lock around the update call restores the guard, and reads need not take it. A caller who treats Oxigraph as a store that serialises its writers is right for the memory store and wrong for the on-disk one at this version. The confirmation step of an optimistic write does not detect the loss, since both writers find their own write. Whether the server behaves the same over HTTP is not measured, and its update handler calls the same `Store`, so treat it as unsafe until probed. This answers the isolation half of TSD-Q4 for Oxigraph. TSD-62 answers it for Fuseki on TDB2 by timing.

**Reproduce it.**

```python
import collections, tempfile, threading
import pyoxigraph as ox

def run(store, threads, target=1000):
    store.update("INSERT DATA { <urn:c> <urn:v> 0 }")
    def work(t):
        i = 0
        while True:
            n = int(next(iter(store.query("SELECT ?n WHERE { <urn:c> <urn:v> ?n }")))[0].value)
            if n >= target:
                return
            i += 1
            store.update(f"DELETE {{ <urn:c> <urn:v> {n} }} INSERT {{ <urn:c> <urn:v> {n + 1} . <urn:w:{t}:{i}> <urn:at> {n + 1} }} WHERE {{ <urn:c> <urn:v> {n} }}")
    pool = [threading.Thread(target=work, args=(t,)) for t in range(threads)]
    [t.start() for t in pool]
    [t.join() for t in pool]
    per = collections.Counter(r[0].value for r in store.query("SELECT ?x WHERE { ?w <urn:at> ?x }"))
    return sum(per.values()), sum(c > 1 for c in per.values())  # markers, revisions with more than one

print("memory", run(ox.Store(), 4))
with tempfile.TemporaryDirectory() as d:
    print("disk", run(ox.Store(d), 4))
```

A correct store prints 1,000 markers and 0 revisions with more than one. The result on disk varies from run to run, so a test of it asserts the memory store and reports the disk store.

## 7. Corrections, and what they taught us

### 7.1 rdflib and the order of an update (TSD-01)

We first said, from reading `evalModify`, that rdflib applies an update one solution at a time. A later probe then reported that rdflib keeps the specification's order, and a dated note said the first statement was wrong. The first statement was right. The probe had happened to run in processes whose hash seeds put the deleting row first. While checking, one of our own traces, which read every binding of each solution, also gave a result we did not reproduce later. We note it rather than explain it.

What we take from it. A probe on rdflib whose answer depends on the order of solutions must fix or sweep `PYTHONHASHSEED`, or it measures luck. A conclusion that contradicts the source needs the source re-read before it is recorded. And a trace should touch as little of the engine's state as it can.

### 7.2 Variable scope (TSD-03)

A static check first let every inner group see the enclosing group's variables. A review then changed the `GRAPH` block to do the same, calling it a fix. A probe on rdflib showed the opposite: no inner group but `NOT EXISTS` sees them. Planning the next step then found that a pattern's solutions *are* joined with the enclosing group, which is a different question from what a `BIND` can see. The check now tracks both. Each step was a claim about SPARQL written from memory, and each was settled only by running a query. That is why every semantic rule the check encodes now has an engine test.

### 7.3 The retry on rdflib

An early spike reported that rdflib and Oxigraph disagreed on a retried append. The cause was two bugs in the first scratch scripts. rdflib does materialise the solutions of a `WHERE` before it changes anything, as its `list(res)` shows. That is a different property from applying the changes in the specification's order, which TSD-01 shows it does not.

### 7.4 Cost claims, and what the first versions got wrong

Four statements about cost were wrong or incomplete when first written, and a probe corrected each.

- We first called a chain of groups "slower unless lazy". A probe that counted lookups (TSD-26) showed that evaluating an operand whole can be cheaper than one lookup per row. It is expensive only when the left side is one row, which is the case for a constraint query, and the record now says that.
- The first reproduction of the guard group (TSD-27) came out backwards, 0.6 times where we expected a gain. The guard group we wrote bound a variable with 25 values, so it multiplied the rows. The real guard group binds one variable. A reproduction has to be checked against the thing it stands for, as the gain is a property of the rejection rate and the group's cardinality.
- "Move the existence check into `FILTER EXISTS`" was first measured on a documentation rule as 1.4 times, from rewriting the count as `NOT EXISTS`. The real cost was the type pattern (TSD-25), found only by removing the query's parts one at a time.
- `FILTER (false)` (TSD-32) was found because a test of a benchmark's own pass criterion did not fail. A guideline had warned against it without a reason.

What we take from it. State what a mechanism does not explain, as TSD-26 does for W8. Time the parts of a slow query separately before rewriting it. And test that a check can fail.

### 7.5 The mechanism of TSD-34

TSD-34 first said that `evalGraph` resets a solution's graph after yielding it, and its step 6 found that removing the reset gave the right answer on 8 seeds. Read that way, the reset looked like the mistake and removing it like the fix. Writing a fix for the upstream report, we re-read the source. The reset comes before the yield, and it is there so that an `EXISTS` outside the block reads the enclosing graph. A probe of `EXISTS` outside a `GRAPH` block confirmed it. Without the reset every such case was wrong on every seed. The defect is that the context being reset is shared, not that it is reset.

What we take from it. A change that makes a failing case pass is not a fix until the cases the changed code exists for have been run. Before calling a line wrong, find out why it is there.

### 7.6 The mechanism of TSD-41

TSD-41 first said that Oxigraph dropped one record's contribution from a `GROUP_CONCAT` over a `UNION` of nested aggregate sub-selects, and guessed at an interaction between the order of the groups and an idiom that leaves a variable unbound. It named nothing smaller than the generated query. Bisecting it removed one construct at a time and found that no construct of the query mattered. The rows entering the aggregate were identical on both engines, and the disagreement reproduced on one `VALUES` row with no `UNION`, no sub-select and no `GROUP BY`. The cause is the aggregate's treatment of a row on which its argument errors, which Oxigraph follows from the specification and rdflib does not.

What we take from it. A divergence found in a large generated query is not explained by the large query's features until a small one reproduces it. The guess about the `UNION` pointed at constructs that were not at fault, while the cause sat in the aggregate. Dump the rows entering the aggregate on each engine first. If they match, the aggregate is the suspect.

### 7.7 TSD-04 on a newer Oxigraph, and the form it tested (TSD-42)

TSD-04 recorded that both stores pass an outer binding of `?g` into a `GRAPH ?g` inside an `OPTIONAL`, and a probe was written to fail if either started to scan. It failed after the Oxigraph version floor moved to 0.5.13 for TSD-40, which is the purpose the probe was written for. The same ratio on 0.5.11, 0.5.12 and 0.5.13 in separate environments showed the engine version alone moved the ratio from about 1 to about 85. The first reading of what that meant for the generator was that 0.5.13 had made the generated replace slower. That was wrong. Measuring the generated replace itself found it linear in the dataset on all three versions, and the cause was in the probe's form, which bound `?g` to a constant IRI where the generator computes it from `$root`. Oxigraph passes the constant in and never the computed name, so TSD-04 had never been true of a generated query on Oxigraph.

What we take from it. A probe stands in for a generated query only if it has the generated query's form, and the form of an expression matters to an optimizer as much as the pattern around it. A cost finding names the patch version it was measured on, and the floor of a dependency is a reason to re-run the cost findings that rest on it. Measure the operation the finding is about before saying what a change means for it.

## 8. Open questions

| # | Question | Why it matters | Next step | Answered by |
|---|---|---|---|---|
| TSD-Q1 | does rdflib order a lazy join's solutions deterministically | only non-lazy joins go through a `set`, so some updates may have a fixed order | read `evalLazyJoin` and probe a lazy form |  |
| TSD-Q2 | what Oxigraph does with initial bindings, if `pyoxigraph` offers them for updates | TSD-12 holds for rdflib only | probe it |  |
| TSD-Q3 | whether the specification's zero-length path includes a constant endpoint in no triple | TSD-08 names one store as right without checking the text. Five engines of six include it (TSD-51's addendum, 2026-11-01), Oxigraph 0.5.13 alone does not | re-read SPARQL 1.1 §18.4 |  |
| TSD-Q4 | the isolation half of request atomicity, and whether inserting an identical statement conflicts | a model of each store's concurrency branches on both | Answered in part. TSD-73 gives the isolation half for Oxigraph 0.5.13, in memory and on disk, and TSD-62 for Fuseki on TDB2. Still to probe with concurrent writers: Oxigraph's server, Jena in memory and TDB2 called in process, rdflib and RDF4J, and whether inserting an identical statement conflicts anywhere |  |
| TSD-Q5 | why Oxigraph turns `xsd:long` into `xsd:integer` | TSD-07 is observed, not explained | read Oxigraph's literal encoding |  |
| TSD-Q11 | whether Oxigraph groups a chain of same-precedence operators from the right in a later version, and why in 0.5.11 | TSD-17 is observed, not explained, and a renderer works around it | read the parser, try the newer releases, and report it upstream if it still holds |  |
| TSD-Q12 | whether Fuseki shows TSD-15 to TSD-18 | they are measured on three in-process stores, and Fuseki adds an HTTP layer and perhaps a different dataset implementation | Narrowed 2026-10-27 (R-X): in-process Jena ARQ 5.1.0 shows none of them departing (TSD-15, TSD-16 in TSD-56, TSD-17 and TSD-18 in TSD-52). run the same probes against Fuseki through its endpoint |  |
| TSD-Q13 | whether Virtuoso applies even one `DELETE`/`INSERT` atomically under `log_enable(2)` or `(2,1)`, and whether Jena TDB2 and Neptune make a multi-operation request one transaction | guarded writes assume a single operation is atomic, and a two-operation form assumes the request is (TSD-06) | Jena ARQ in memory does (TSD-50, 2026-10-27), and TDB2 and Fuseki do (TSD-66, 2026-10-31, with a failing operation and with an abandoned request). Virtuoso and Neptune remain. probe each with a failing operation and a large update interrupted | Jena's half by TSD-66 |
| TSD-Q14 | whether Jena SHACL, TopBraid or another validator runs a SPARQL constraint once per focus node, and what a node shape costs there | TSD-23 and the choice of targets (rule R5 of the query rules) are measured on pySHACL only | run the same shapes on another validator |  |
| TSD-Q15 | why moving W8's identity lookup ahead of its pair enumeration halved its cost | TSD-26 does not explain it | trace the join order and the sizes of the operands |  |
| TSD-Q16 | what `inplace=True` saves, and whether it is safe with shared graphs | TSD-31's 9% is a fixed cost per validation | run the suite with a copy per test and `inplace=True`, and with a guard against mutation on |  |
| TSD-Q17 | why Oxigraph returns no row for `COUNT(*)` over a statically false filter | TSD-33 is observed, not explained | read Oxigraph's query optimiser |  |
| TSD-Q18 | whether Fuseki or a disk-backed Jena shows the sensitivity of TSD-25 to TSD-27. Narrowed 2026-10-27 (R-W): in-process, in-memory Jena ARQ 5.1.0 does not (TSD-57) | the rules in the query rules assume only rdflib is sensitive, which three engines now support | run the query-cost experiments against a Fuseki endpoint, and against TDB2 |  |
| TSD-Q19 | why the sub-select decides TSD-34 when a `UNION` alone does not, and whether Fuseki shows the defect | TSD-34 is isolated on a non-lazy join of a sub-select and a `UNION`, which is rdflib's | Narrowed 2026-10-27 (R-X): in-process Jena ARQ 5.1.0 does not (TSD-34). The part of TSD-Q19 about generated operations is answered under TSD-34. trace which solution comes first and which are kept in TSD-34's query, and run the query on Fuseki. The defect was reported upstream 2026-10-20 ([RDFLib/rdflib#3585](https://github.com/RDFLib/rdflib/issues/3585)) |  |
| TSD-Q20 | why a generated query's `?closures` differed between engines | a compiled query that means one thing on one engine and another on the next | none. An aggregate over an expression that errors on one row of its group is unbound on Oxigraph and skips the row on rdflib, and the `UNION` and the nesting play no part | TSD-41 |
| TSD-Q21 | what `MIN` and `MAX` over a group with an error should give, and whether to report Oxigraph's `COUNT(expr)` upstream. Answered in part 2026-10-27, and narrowed again 2026-10-27 | an aggregate over a row with an error gives a different value on each engine | TSD-43 answers in part. `MIN` is unbound on either reading, which Oxigraph and Jena do and rdflib does not. `MAX` is `2` if an error element is "no value" in §15.1's ordering and unbound otherwise. The SPARQL 1.1 Recommendation and the SPARQL 1.2 working draft define `Max` through `ORDER BY DESC` and say nothing about an error element, so the text does not decide it (R-Y). rdflib answers `2`, Oxigraph and Jena unbound. The `COUNT(expr)` departure is reported as [oxigraph/oxigraph#2009](https://github.com/oxigraph/oxigraph/issues/2009). What remains is `MAX` only, which only the working group can settle, and no report is drafted for it since no engine is clearly wrong |  |
| TSD-Q22 | whether an Oxigraph release after 0.5.13 passes an outer binding into a left join again, and whether it is reported upstream | TSD-42 makes a named-graph replace's sweep read every graph, and the version floor is 0.5.13 (TSD-40) | Updated 2026-10-27 (R-W, R-Y): rdflib and Jena ARQ read one graph (TSD-42), so the departure is Oxigraph's alone. Reported as [oxigraph/oxigraph#2008](https://github.com/oxigraph/oxigraph/issues/2008). The tracker holds related open issues, #1980 and #1883, and no report of this change. run the store probes on each new release, and read the bind-optional and values-optional ratios (near 1 means the binding is passed in again). Follow [oxigraph/oxigraph#2008](https://github.com/oxigraph/oxigraph/issues/2008) |  |
| TSD-Q23 | which of TSD-43's departures to report upstream, and whether a later release removes them | rdflib returns a row for a grouped query over no rows and raises for `SUM` over an IRI. Oxigraph's `COUNT(expr)` and `AVG` over an all-unbound group depart from §18.5.1. Jena's `GROUP_CONCAT(DISTINCT)` over an empty group is unbound | Updated 2026-10-27 (R-Y). Written up and not filed, each with its tracker search: Oxigraph `AVG` over an all-unbound group, Jena `AVG` over the same, Jena `STR` of a blank node, rdflib `STR` of a blank node, rdflib `SUM` over an IRI, rdflib skipping or counting error rows. Already reported: Jena's `GROUP_CONCAT(DISTINCT)` over an empty group (apache/jena#4240, fix proposed in #4243), and rdflib's grouped query over no rows has an open fix, RDFLib/rdflib#3498. file each report once agreed, and re-run the aggregate probes on each release. The same probes against Fuseki (TSD-Q12) if it differs from in-process ARQ |  |
| TSD-Q24 | why Oxigraph 0.5.13 stopped passing a constant `?g` into an `OPTIONAL`, and why it never passes a computed one | TSD-42 is observed, not explained, and a generator's default graph naming pays for it | read the `sparopt` change between 0.5.12 and 0.5.13, and ask upstream |  |
| TSD-Q25 | whether Jena on disk, Fuseki over HTTP, RDF4J's `NativeStore`, GraphDB, Virtuoso or Neptune pass a computed `?g` into an `OPTIONAL` | TSD-42 measured two Jena stores and one RDF4J store, all in memory | run the graph-naming benchmark against each, or its Python form against a SPARQL endpoint |  |
| TSD-Q30 | whether Jena TDB2, and Fuseki over it, rolls back the first operation of a request whose second fails, and whether `CLEAR GRAPH` of an absent graph fails there | TSD-50 is in-memory ARQ only, and TSD-Q13 asks it of TDB2 | run probe P2 against a TDB2 dataset |  |
| TSD-Q31 | whether a Jena release after 5.1.0 makes `CLEAR GRAPH` of an absent graph succeed on a dataset that does not record empty graphs, and whether it is reported upstream | TSD-50 expects the failure, so a fix shows as a test failure | A report is drafted. Searched 2026-10-27 (R-Y): `repo:apache/jena CLEAR GRAPH No such graph` gave 35 results, of which the first 15 were read and none is about this, so no report exists that we found. rerun the standalone reproduction and the conformance run on each new Jena |  |
| TSD-Q32 | whether to file the unfiled upstream reports, in what order, and whether RDFLib/rdflib#3498 covers the grouped query over no rows with a constant projection | a report is a person's to make | Updated 2026-10-27: oxigraph/oxigraph#2008 and oxigraph/oxigraph#2009 are filed. check #3498 against a reproduction of the grouped query over no rows first |  |
| TSD-Q35 | whether the two rdflib departures drafted (TSD-60, TSD-61) are filed upstream, and whether a release after 7.6.0 fixes them | both are written up with a failing test and not filed | Searched 2026-10-27 (R-Y): `repo:RDFLib/rdflib SUM IF aggregate SPARQLTypeError` and `SPARQL comparison string number type error` gave 0 results each, so no report exists that we found. file them with their smallest query, and re-run the probes on each rdflib release. The probes of TSD-60 and TSD-61 fail when one is fixed |  |
| TSD-Q36 | why RDF4J 6.1.0 fills an update's templates with every variable unbound when its `WHERE` holds a path with `?` or `*`, and whether `+`, a later release or an upstream report bears on it | every generated composite operation is wrong on RDF4J (TSD-67's addendum). Isolated 2026-11-01 to the path, with `OPTIONAL` and the transaction ruled out | read `SailUpdateExecutor` and the path evaluation at the tag `6.1.0`, probe `+` and the smallest update that fails, search the upstream tracker, and rerun the JVM benchmark on a later RDF4J |  |
| TSD-Q37 | why an `OPTIONAL` join to a property inside `GRAPH ?g` with `?g` from `VALUES` costs four times a plain read on Oxigraph and a `UNION` six times on rdflib (TSD-71) | a generated read must choose its form per engine until the cause is known | read each engine's plan for the two forms |  |
