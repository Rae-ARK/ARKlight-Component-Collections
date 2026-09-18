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
| [`IMPLEMENTATION-LADDER.md`](design/IMPLEMENTATION-LADDER.md) | **Stage 0 of 5, filed.** The staged plan for ACC's first real capability + component packages: a minimal upstream discovery hook (Stage 1, blocked on `alpha`), the ACC package skeleton (Stage 2), `@acc/prism` -- a Pygments-backed syntax-highlighting capability chosen and justified as the highest-value first Python-ecosystem integration (Stage 3), `@acc/common` -- the first common-use component package consuming it (Stage 4), and CLI installability (Stage 5). |
