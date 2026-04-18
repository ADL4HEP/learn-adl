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
| composite | Build candidate objects from multiple instances |
| region  | Define event-level selection, possibly with bins |
| table   | Generic block for tabular information (e.g., efficiencies) |

### Keywords
| Keyword | Purpose | Example | Block Type |
|:--------|:--------|:--------|:-----------|
| take    | Take an input object, region, or composite | `take slimmedJets` | object, composite, region |
| select  | Apply selection criteria | `select pT > 30` | object, composite, region |
| reject  | Apply rejection criteria | `reject abs(eta) > 2.4` | object, composite, region |
| bin     | Define a search bin based on conditions | `bin HT [] 500 800` | region |
| bins    | Define multiple bins along a variable | `bins MET 300 500 700` | region |

### Special Statements
| Statement | Purpose | Example |
|:--------|:--------|:--------|
| define  | Define a new attribute inside a block or a global event variable outside of blocks | `define HT = sum(pT(goodJets))` |
| this    | Refers to the current instance inside object or composite blocks. Used only when the object itself, not an attribute of it, needs to be referred to (e.g. angular separation calculations) | `select dR(this, jets) > 0.4` |

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
| union(a, b, c, ...) | Merge n collections | `take union(electrons, muons, taus)` |
| sort(collection, expression, direction) | Sort a collection by a custom expression | `take sort(jets, pT, descend)` |
| sum(expression) | Sum over a collection | `define HT = sum(pT(jets))` |
| min(expression) | Minimum over a collection | `select min(dphi(jets, MET)) > 0.4` |
| max(expression) | Maximum over a collection | `select max(pT(jets[:3])) > 100` |
| any(expression) | Check if any instance satisfies a condition | `select any(dR(this, jets) > 0.4)` |
| all(expression) | Check if all instances satisfy a condition | `select all(pT(jets) > 30)` |

### Mathematical and HEP-Specific Functions
| Function | Purpose |
|:---------|:--------|
| abs(x), sin(x), cos(x), tan(x), log(x), sqrt(x) | Standard mathematical functions |
| dR(obj1, obj2), dphi(obj1, obj2), deta(obj1, obj2) | HEP-specific kinematic functions |

### Slicing
| Syntax | Purpose |
|:-------|:--------|
| jets[i:j] | Select from index i to j |
| jets[:i] | Select from start up to index i |
| jets[i:] | Select from index i to the end |

**Example:**
```
select min(dphi(jets[:3], METLV[0])) > 0.5
```

---

### Writing Principles and Syntax Guidelines

- ADL is **declarative**: you specify *what* to do, not *how*.
- Each block has a consistent, keyword-driven structure.
- Object and event selections are clearly separated.
- No explicit loops: iteration over collections is **implicit**.
- `select` and `reject` apply conditions **per instance** in collections.
- Functions like `size`, `any`, `all`, `sum`, `min`, `max` are used to reduce over a collection.
- Logical operators must be `and`, `or`, `not` — avoid `&&`, `||` symbols.
- Use **slicing** to operate on subgroups within a collection.
- Bins defined within a region must be **disjoint**.
- Use `this` to refer to the current object instance inside object and composite blocks.

---
