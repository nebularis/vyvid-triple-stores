---
name: shacl-testing
description: Writing and speeding up tests that validate RDF graphs against SHACL shapes with pySHACL. Use when adding a test that validates a graph against shapes, when sharing parsed graphs or validation reports across test modules with pytest fixtures, when a suite makes many pyshacl.validate calls, or when a cache of graphs may hide a mutation.
---

# Testing SHACL shapes

## Parse once, copy, never share a mutable graph

Session-scoped fixtures parse a document once per process, and a test that changes a graph changes a copy. A session-wide translation parses and translates each query text once (H1 of the query rules). A pure function that parses SPARQL is memoised on its input and returns a fresh value.

## The parse and validation cache

Two session-scoped fixtures, in the suite's shared `{{shared_fixtures}}`, are available to every test:

- `graph_cache(*sources)`: memoises a parsed or merged `rdflib.Graph` by its source arguments (each a `Path`, parsed as a file, or a `str`, parsed as inline Turtle), order-independent (sources are sorted before use as the key, so two modules building "the same layer stack" in a different literal order still share one object).
- `validated(data_graph, *, shacl_graph, **options)`: a drop-in replacement for `pyshacl.validate` (same parameter names, so an existing call site needs no change once substituted in), memoising the whole report.

**How a module adopts it**, without rewriting every call site's signature:

```python
@pytest.fixture(scope="module", autouse=True)
def _cached_graphs(request: pytest.FixtureRequest, graph_cache, validated) -> None:
    module = request.module
    module.MODEL = graph_cache(MODEL_DIR / "core.ttl", EXTRA_TURTLE)
    module.SHAPES = graph_cache(SHAPES_DIR / "structural.ttl", SHAPES_DIR / "constraints.ttl")
    module.validate = validated
```

The existing `MODEL` and `SHAPES` names, and every test and helper that already references them, keep working unchanged: this assigns onto the module's own namespace, so a bare `MODEL` or a call to `validate(...)` resolves to the cached version from here on.

## Pitfalls found by building this, not predicted in advance

- **`request.module` needs function, class or module scope, never session.** A first draft used `scope="session"` for the per-module adoption fixture and failed immediately with `AttributeError: module not available in session-scoped context`. Use `scope="module"`: it still runs once per module, and the *cache underneath* (`graph_cache` and `validated` themselves) stays session-scoped, so sharing across modules is unaffected.
- **A default argument that reads a cached name breaks at import time.** `def _results(data, shapes=SHAPES): ...` evaluates `SHAPES` the moment the `def` line runs, before any fixture has assigned it. Change it to `def _results(data, shapes=None): shapes = SHAPES if shapes is None else shapes`, which defers the lookup into the function body (by which point, for any test actually calling it, the fixture has already run).
- **Key the validation cache on graph *content*, never on Python identity alone.** A real test builds a graph, validates it, mutates the *same* object in place (`data.remove(...)`), then validates it again expecting a different answer. An `(id(data), id(shapes), options)` key would return the first, stale report for the second call, which was caught because that exact test failed under the first design. Fix: key on `hash(frozenset(graph))` instead. Costs one linear pass per call, far cheaper than the `validate()` call it may save, and correctly treats two independently-built graphs with identical triples as one cache entry too.
- **`pyshacl`'s own `advanced=True` mode mutates the shapes graph you give it**, once, idempotently (confirmed empirically: it adds a couple of fixed RDFS/OWL compatibility axioms the first time it sees a given shapes graph, never again after). A guard that asserts a cached graph never changes size will trip on this unless it re-snapshots the graph's size immediately after a real `validate()` call. It is not a test bug, it is `pyshacl`'s own documented-by-behaviour side effect.
- **A cache must still catch a real test mutating shared state.** A fixture that runs once, at session end, asserts no cached graph's triple count drifted from what it was when built (aside from the one pySHACL exception above). Probe it for real before trusting it: add a throwaway test that does `graph_cache(...).add((s, p, o))` and confirm the session fails at teardown, then delete the probe.

## Writing a new SHACL-validating test

1. Does an existing module already parse the same layer stack or shape files? If so, reuse `graph_cache` rather than a fresh `Graph().parse(...)`, and check the cache's own tests for the exact call pattern.
2. Never mutate a graph `graph_cache` returned. Build a fresh one for a changed example (a fresh parse, or `cached_graph + data`, which is already how every mutation helper in this suite works: `Graph.__add__` returns a new graph, never mutates either operand).
3. Call `validate(...)` exactly as `pyshacl.validate` is documented (`shacl_graph=`, not a bare positional second argument) if the module has substituted `validated` in for it. This is what makes the substitution a true drop-in.
4. If the test will shell out for a repository-wide check, use the suite's text-scan helper instead, not a fresh `subprocess.run(["git", "grep", ...])` call.

## Reporting quirks of validators

- **Jena SHACL reports the shape that carries the failing constraint.** A constraint nested as `sh:property [ … ]` under a named node shape reports the blank node as its source, not the named shape. Where a report's shape identifier matters, give each property constraint its own named, targeted shape.
- **Jena 5 SHACL's report entries** are `org.apache.jena.shacl.validation.ReportEntry`, with `focusNode()`, `source()`, `severity()` and `message()`. There is no nested `ValidationReport.Entry`.
