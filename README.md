# Learn ADL

Welcome to **Learn ADL** — a lightweight guide and tutorial collection for writing physics analyses using the **Analysis Description Language (ADL)**.

This repository provides:
- A complete ADL syntax reference
- Example ADL files for objects, composites, and regions
- Tutorials to help you get started writing your own analyses

## About ADL

Analysis Description Language (ADL) is a declarative way to write down a physics analysis: which objects you pick, which events you keep, how you select and bin your analysis regions. It is a **bridge between your input and your output**: input being whatever tree or ntuple you read (CMS NanoAOD, ATLAS DAOD, Delphes, a custom flat file), output being the selected objects and regions that go into your final result. ADL captures the *logic* that connects the two, and nothing more.

**The physics objects are not part of ADL.** Names like `jet`, `electron`, `MET`, `pt`, `eta`, `charge` are **not ADL keywords**. They come from your data. If your tree calls them `Jet` and `Muon`, you write `Jet` and `Muon`. ADL is schema-agnostic: it uses whatever names you introduce via `take`, and trusts that the attributes you reference are defined on those inputs.

What ADL *does* define is the vocabulary for expressing analysis logic: block types (`object`, `region`, `composite`, ...), keywords (`take`, `select`, `reject`, `define`, `bin`, `bins`, `this`), operators (`and`, `or`, `not`, `[]`, `][`), reducers (`sum`, `min`, `max`, `any`, `all`), and kinematic functions (`dR`, `dphi`, `deta`). Everything else in an ADL file is either a name borrowed from your input or one you define yourself.

**In short: ADL tells you what to do with the physics; the physics objects themselves come from your data.**

## 📚 Contents
- `adl_syntax_summary.md` — Full syntax overview
- `examples/annotated_snippets.adl` — Example ADL snippets with annotations

## 🚀 Quick Start
If you're new to ADL:
1. Read `adl_syntax_summary.md`
2. Explore the examples
3. Try writing your own analysis!

## ✨ Contributing
This repository is a living resource.  
Contributions, corrections, and new tutorials are welcome!
