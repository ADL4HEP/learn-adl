## About ADL

Analysis Description Language (ADL) is a declarative way to write down a physics analysis: which objects you pick, which events you keep, how you bin your signal regions. It is a **bridge between your input and your output**: input being whatever tree or ntuple you read (CMS NanoAOD, ATLAS DAOD, Delphes, a custom flat file), output being the selected events, regions, and candidates that go into your final result. ADL captures the *logic* that connects the two, and nothing more.

**The physics objects are not part of ADL.** Names like `jet`, `electron`, `MET`, `pt`, `eta`, `charge` are **not ADL keywords**. They come from your data. If your tree calls them `Jet` and `Muon`, you write `Jet` and `Muon`; if it calls them `jets` and `muons`, you write those. ADL is schema-agnostic: it uses whatever names you introduce via `take`, and trusts that the attributes you reference are defined on those inputs.

What ADL *does* define is the vocabulary for expressing analysis logic: block types (`object`, `region`, `composite`, ...), keywords (`take`, `select`, `reject`, `define`, `bin`, `bins`, `this`), operators (`and`, `or`, `not`, `[]`, `][`), reducers (`sum`, `min`, `max`, `any`, `all`), and kinematic functions (`dR`, `dphi`, `deta`). Everything else in an ADL file is either a name borrowed from your input or one you define yourself.

**In short: ADL tells you what to do with the physics; the physics objects themselves come from your data.**

---

## ADL Syntax Summary

### ADL Block Structure

ADL comprises of blocks with a keyword-expression structure:

```
blocktype blockname
  # general comment
  keyword1 expression1
  keyword2 expression2
  keyword3 expression3  # comment about value3
```

Blocks allow a clear separation of analysis components.

Blocks used in core analysis algorithm description:

### Core Blocks
| Block   | Purpose |
|:--------|:--------|
| info    | Set metadata about the analysis and ADL file |
| object  | Define a filtered or derived object collection |
| composite | Build candidate tuples from named bindings of one or more collections (e.g. Z candidates from lepton pairs). See the Composite Blocks section below. |
| region  | Define event-level selection, optionally with bins. Can inherit another region's cuts using `take <regionName>`. |
| table   | Generic block for tabular information (e.g., efficiencies) |

### Keywords
| Keyword | Purpose | Example | Block Type |
|:--------|:--------|:--------|:-----------|
| take    | Bring in an input collection, region, or named binding. In an `object` block, takes an input collection (e.g. `take slimmedJets`) or a combinator over collections (e.g. `take union(...)`). In a `region` block, takes another region to inherit its selections (e.g. `take baseline`). In a `composite` block, takes named bindings over collections (e.g. `take disjoint(leptons l1, leptons l2)`). | `take slimmedJets` (object), `take baseline` (region) | object, composite, region |
| select  | Apply selection criteria. In `object` and `composite` blocks, applied **per instance** (or per candidate tuple) of the block's collection. In `region` blocks, applied **per event**. | `select pt > 30` | object, composite, region |
| reject  | Apply rejection criteria (opposite of `select`). Same per-instance / per-event semantics as `select` depending on block type. | `reject abs(eta) > 2.4` | object, composite, region |
| bin     | Define a search bin based on conditions. The bin name is optional: when given, it is a quoted string placed immediately after `bin`. | `bin HT [] 500 800` (unnamed), `bin "SR1" HT [] 500 800` (named) | region |
| bins    | Define multiple bins along a variable (splits the variable range at the listed edges). | `bins HT 300 500 700` | region |

### Special Statements
| Statement | Purpose | Example |
|:--------|:--------|:--------|
| define  | Define a new attribute inside a block or a global event variable outside of blocks | `define HT = sum(pt(goodJets))` |
| this    | Refers to the current instance of the collection in an **object** block. Used only inside `select`, `reject`, or `define`, and only when the object instance itself (not an attribute of it) needs to be passed to a function (e.g. `dR(this, electrons)`). Attribute names on their own (`pt`, `eta`, ...) already mean "attribute of the current instance", so `this` is not needed for attribute access. Not used in `region` blocks, and not used in `composite` blocks. Composites refer to their members by names introduced in `take` (see the composite section). When `take` is a function like `union(...)`, `this` is an instance of the resulting combined collection. | `select dR(this, electrons) > 0.4` |

### Operators
| Operator | Purpose |
|:---------|:--------|
| >, <, >=, <=, == | Standard comparisons |
| []         | Inclusive range check |
| ][         | Exclusive range check |
| +, -, *, /, ^ | Arithmetic operations |
| and, or, not | Logical operators |

### Collection and Reducer Functions
| Function | Purpose | Example Usage |
|:---------|:--------|:--------------|
| union(a, b, c, ...) | Merge n collections into one | `take union(electrons, muons, taus)` |
| sort(collection, expression, direction) | Sort a collection by a custom expression | `take sort(jets, pt, descend)` |
| disjoint(a name1, b name2, ...) | Build unordered tuples across named bindings of one or more collections, with all bindings required to point to distinct instances (used in `take` inside `composite` blocks) | `take disjoint(leptons l1, leptons l2)` |
| cartesian(a name1, b name2, ...) | Build all ordered tuples across named bindings of the given collections (used in `take` inside `composite` blocks) | `take cartesian(electrons e, jets j)` |
| size(collection) | Count instances in a collection (returns a scalar) | `select size(jets) >= 3` |
| sum(expression) | Sum over a collection | `define HT = sum(pt(jets))` |
| min(expression) | Minimum over a collection | `select min(dphi(jets, MET)) > 0.4` |
| max(expression) | Maximum over a collection | `select max(pt(jets[:3])) > 100` |
| any(expression) | Check if any instance satisfies a condition | `select any(dR(this, jets) > 0.4)` |
| all(expression) | Check if all instances satisfy a condition | `select all(pt(jets) > 30)` |

### Mathematical and HEP-Specific Functions
| Function | Purpose |
|:---------|:--------|
| abs(x), sin(x), cos(x), tan(x), log(x), sqrt(x) | Standard mathematical functions |
| dR(obj1, obj2), dphi(obj1, obj2), deta(obj1, obj2) | HEP-specific kinematic functions |

### Slicing

Collections are **0-indexed**, and slicing is **Python-style (half-open)**: the start index is included, the end index is not. For `jets[i:j]`, the resulting sub-collection contains the instances at indices `i, i+1, ..., j-1` (a total of `j - i` instances).

| Syntax | Meaning |
|:-------|:--------|
| jets[i:j] | Instances at indices i, i+1, ..., j-1 |
| jets[:n] | The first n instances (indices 0 to n-1) |
| jets[i:] | Instances from index i to the end |

**Example:**
```
select min(dphi(jets[:3], METLV[0])) > 0.5
```
(Uses the first 3 jets, i.e. indices 0, 1, 2.)

---

### Composite Blocks

A **composite** block builds candidate tuples (e.g. Z candidates, top candidates, W candidates) from one or more existing collections. Each line inside the block applies independently to each candidate tuple.

The general shape is:

```
composite blockname
  take <combinator>(coll1 name1, coll2 name2, ...)
  select <condition on name1, name2, ...>
  object <candidateName> = <expression combining the bindings>
  select <condition on candidateName>
```

Key elements:

1. **Named bindings in `take`.** Unlike in an `object` block, `take` in a composite introduces one or more *named bindings* (e.g. `l1`, `l2`, `e`, `j`) which refer to individual instances within each candidate tuple. These names are the only way to refer to the candidate's members inside the block. `this` is **not** used in composite blocks.
2. **Combinators.** The two common combinators used with `take` are:
    * `disjoint(coll1 name1, coll2 name2, ...)` forms unordered tuples where all bindings point to distinct instances. Used when order does not matter and the same instance must not appear twice (e.g. picking two different leptons to form a Z).
    * `cartesian(coll1 name1, coll2 name2, ...)` forms all ordered pairs/tuples, including repeats (across different collections). Used when every combination is meaningful.
3. **Derived candidate.** `object candidateName = expr` defines a derived candidate from the bindings (typically a 4-vector sum like `l1 + l2`). The candidate can then be used like any other object within the block (e.g. `mass(Z)`, `pt(Z)`).
4. **Range checks.** `select <expr> [] low high` is an inclusive range check; `select <expr> ][ low high` is exclusive. These are convenient for mass-window and similar cuts.

**Example (Z candidates from opposite-sign same-flavor leptons):**

```
composite Zcands
  take disjoint(leptons l1, leptons l2)
  select l1.charge + l2.charge == 0
  object Z = l1 + l2
  select mass(Z) [] 80 100
```

Here `Zcands` is a collection of candidate tuples, each exposing `l1`, `l2`, and the derived `Z`. Downstream `object`, `region`, or `composite` blocks can `take Zcands` and reason about its instances just like any other collection.

---

### Writing Principles and Syntax Guidelines

- ADL is **declarative**: you specify *what* to do, not *how*.
- Each block has a consistent, keyword-driven structure.
- Object and event selections are clearly separated.
- No explicit loops: iteration over collections is **implicit**.
- `select` and `reject` apply conditions **per instance** (or per candidate tuple) inside `object` and `composite` blocks, and **per event** inside `region` blocks.
- Functions like `size`, `any`, `all`, `sum`, `min`, `max` reduce a collection to a scalar or boolean. `union`, `sort`, `disjoint`, `cartesian` produce new collections and are used as arguments to `take`.
- A region can inherit another region's cuts using `take <regionName>`, which is equivalent to copying all of the inherited region's `select` and `reject` statements into the new one.
- Logical operators must be `and`, `or`, `not` (avoid the `&&`, `||` symbols).
- Use **slicing** to operate on subgroups within a collection.
- Bins defined within a region must be **disjoint**. Bin names are optional; when present they are quoted strings.
- Use `this` only inside `select`, `reject`, or `define` statements of an **object** block, and only when the object instance itself (not an attribute of it) must be referenced. `this` is not used in `region` blocks or `composite` blocks. In composites, members are referred to by the names you give them in `take`.
- Inside an object block, a plain name (like `pt` or `eta`) is read as an attribute of the current instance, which is the object introduced by `take`. Names of other collections, global `define`s, or other blocks must always be written out explicitly, and do not shadow attributes. For instance, in `reject dR(this, electrons) < 0.4`, `electrons` is another object collection (referred to by name), while `this` is the current instance.

---

### Info Blocks

`info` blocks attach human-readable metadata to the analysis and the ADL file itself. They are **not used during execution**: tools may read and display them, but they do not affect which objects or events are selected. The field names are self-describing, and the accepted set can grow over time as analyses record more provenance information, so the snippet below is illustrative rather than exhaustive.

```
info analysis
  title     Search for supersymmetry in proton-proton collisions at 13 TeV in final states with jets and missing transverse momentum
  experiment CMS
  id         SUS-19-006
  sqrtS      13.0
  lumi       137
  publication JHEP 10 (2019) 244
  arXiv      1908.04722
  hepdata    https://www.hepdata.net/record/ins1749379
  doi        10.1007/JHEP10(2019)244

info adl
  inputformat  <the input data format used by the analysis, e.g. NanoAODv9, MiniAODv2, ...>
  adlauthor    <name of the person who wrote the ADL file>
```

Each line inside an `info` block is a single field: one keyword followed by its value (free text to end of line). Fields can be omitted if not applicable, and additional fields may be added as conventions evolve.

---
