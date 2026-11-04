<!-- SPDX-License-Identifier: MIT -->

# Vyvid Triple Stores Domain

Вивід: домен сховищ триплетів (графових баз даних RDF)

This domain holds what we found out about working with RDF triple stores and SPARQL engines, kept so that any project can use it. It has three parts.

- `findings.md` is the record. Each finding `TSD-nn` says what a store, engine or validator does, how we found out, the evidence with engine and version, and any report made upstream. An open question is `TSD-Qn`.
- `query-rules.md` holds the rules for writing SPARQL queries and SHACL shapes that run in a reasonable time, the method for finding and changing a cost, and the measurements behind them.
- `query-standards.md` is the same rules as a register, `RDF-1` to `RDF-30`, each row pointing at its findings and at the section of `query-rules.md` that gives its reason.

A finding holds for the versions it names and no others, until it is checked again. Read its first lines for the evidence, which is one or more of Source (we read the code at a named version), Observed (we ran a probe), Documented (the tool's own documentation says so) and Reasoned (a hypothesis not yet observed). An Observed finding names a run record, `R-A` and so on, in the table in section 1 of `findings.md`, with the kind of machine, the Python version and the store versions. A statement about cost says whether it holds on any engine or only on the one measured.

[MIT](LICENSE)
