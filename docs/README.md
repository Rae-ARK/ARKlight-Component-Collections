# ARKlight-Component-Collections Docs

## Design

Reference material for the ACC system as a whole -- permanent,
updated in place as the design firms up, not removed once its
immediate purpose is served. This mirrors how `alpha`'s own
`docs/Foundational/` treats its permanent design record.

| File | Covers |
| --- | --- |
| [`acc-foundational-design.md`](design/acc-foundational-design.md) | The foundational design document: what ACC is, the package/capability model, discovery via Python entry points, versioning, trust boundaries, and alpha scope. |
| [`BUILD-TIME-ECOSYSTEM.md`](design/BUILD-TIME-ECOSYSTEM.md) | Why ACC exists as an `npm`-for-`arklight`-shaped extension ecosystem beside the compiler rather than inside it, grounded in `alpha`'s actual compile-time/run-time boundary (`arklight/parser/loader.py`) -- and why that boundary is currently unpackaged, hand-rolled, and "janky" without ACC. |
