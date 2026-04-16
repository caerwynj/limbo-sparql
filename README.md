# limbo-sparql

A SPARQL query engine and Turtle RDF parser written in [Limbo](http://www.vitanuova.com/inferno/limbo.html) for the [Inferno](http://www.vitanuova.com/inferno/) operating system.

## Overview

This project implements:

- **Turtle parser** (`turtle.b`) -- reads RDF data serialized in [Turtle](https://www.w3.org/TR/turtle/) format, producing a list of subject-predicate-object triples.
- **SPARQL parser** (`sparql.y`) -- a yacc grammar that parses a subset of [SPARQL](https://www.w3.org/TR/sparql11-query/) `SELECT ... WHERE` queries.
- **Triple store** -- two back-end implementations:
  - `query.b` -- in-memory store backed by the Turtle parser. Loads triples from a `.ttl` file and evaluates queries via backtracking search.
  - `triplestore.b` -- SQLite-backed store. Loads triples from a database and generates SQL queries against a `triple(s, p, o)` table.

Query evaluation uses a backtracking algorithm that joins triple patterns by substituting bound variables across patterns at each level.

## Files

| File | Description |
|---|---|
| `sparql.y` | Yacc grammar for SPARQL SELECT queries |
| `turtle.b` / `turtle.m` | Turtle RDF parser module |
| `query.b` / `query.m` | In-memory triple store with backtracking evaluator |
| `triplestore.b` / `triplestore.m` | SQLite-backed triple store |
| `ddl.sql` | Schema for the SQLite triple table |
| `samp[1-4]` | Sample N-Triples data files |
| `sample.ttl` | Sample Turtle data with prefixes, blank nodes, collections |
| `sample*.sparql` | Sample SPARQL queries |
| `mkfile` | Build rules (Inferno `mk`) |
| `mashfile` | Mash shell helpers for building and testing |

## Building

Requires an Inferno installation with `limbo`, `mk`, and `yacc` (via mash).

```sh
# Generate the SPARQL parser from the yacc grammar
mash yacc sparql.y

# Compile all modules
mk all

# Install dis files and module interfaces
mk install
```

## Usage

Run a SPARQL query against a Turtle data file:

```sh
# sparql <data-file> <query-file>
emu sparql samp1 < sample.sparql
```

### Example

Given `samp1`:
```
<http://example.org/book/book1> <http://purl.org/dc/elements/1.1/title> "SPARQL Tutorial" .
```

And `sample.sparql`:
```sparql
SELECT ?title
WHERE
{
    <http://example.org/book/book1> <http://purl.org/dc/elements/1.1/title> ?title
}
```

The query returns:
```
"SPARQL Tutorial",
```

Queries can use `PREFIX` declarations and join multiple triple patterns:

```sparql
PREFIX ex: <http://example.org/book/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT ?title ?name
WHERE
{
    ?book dc:title  ?title .
    ?book dc:url ?url .
    ?url dc:title ?name
}
```

## SPARQL subset supported

- `PREFIX` and `BASE` declarations
- `SELECT` with named variables or `*`
- `WHERE` clause with basic graph patterns (triple patterns joined by `.`)
- Property paths with `;` (shared subject) and `,` (shared predicate)
- The `a` shorthand for `rdf:type`
- Variables: `?var` and `$var`

## Turtle features supported

- `@prefix` and `@base` directives
- Blank nodes (`_:label` and `[ ... ]`)
- Collections (`( ... )`)
- Multi-line quoted strings (`"""..."""` and `'''...'''`)
- `a` shorthand for `rdf:type`
- Comments (`#`)

## License

See repository for license information.
