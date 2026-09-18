# ACC Implementation Ladder: Discovery, `@acc/prism`, `@acc/common`

## Status

**Stage 0 (this document): filed. Stage 1: landed on `alpha`.**
Stages 2-5 below remain *planned, not built* -- no ACC package code
exists yet in this repo. This is the same role
`docs/Implementation/JS-VOCABULARY-ADDENDUM-v0.070.md` plays on
`alpha`: turning "should we?" into "here's the landing order," one
stage at a time, each stage small enough to land and be verified on
its own.

_Written against ARKlight `alpha` as cloned for reference -- treated
as **read-only**. Nothing in this ladder edits `alpha` directly. Stage
1 was filed as a proposal against that repo rather than work this repo
could complete unilaterally, and has since landed there as
`arklight/capabilities.py`, with `tests/test_capabilities.py` passing
(9/9). The real discovery path now exists on `alpha`; Stages 3-5 are
no longer confined to designed-and-tested-in-isolation code exercised
only through direct calls._

## Why these two tracks, together, first

The person driving this project asked for two things in the same
breath: user-defined components for common use cases, and the single
most valuable Python-ecosystem tool to integrate first. They turn out
to be the same proof: a capability package (`@acc/prism`) demonstrates
the packaging/discovery/diagnostics machinery end to end using a real,
high-value dependency, and a components package (`@acc/common`) then
*consumes* that capability the way a real project would -- proving the
two ACC package classes `acc-foundational-design.md` §5 already
distinguishes (capabilities vs. components) actually compose.

## Which tool, and why

**[Pygments](https://pygments.org)** -- syntax highlighting.

Chosen over alternatives (`Pillow`/image processing, `pandas`/data
tables, `matplotlib`/charts -- all valid future `@acc/*` candidates,
none picked first) on three grounds, checked rather than assumed:

1. **It's the de facto standard for exactly this build-time job.**
   Pelican and Nikola -- the two other Python-authored static site
   generators -- both ship Pygments-based code highlighting as a
   core, non-optional feature, not a bolt-on. Confirmed against
   Pelican's own project description directly rather than assumed
   from reputation.
2. **It's still a live, heavily-depended-on project**, not legacy
   dead weight: PyPI lists it at 68 releases with a release roughly
   every 3-4 months, most recently a few months prior to this
   writing, and it sits among the top tier of PyPI packages by
   monthly download volume with well over a thousand direct
   dependents. Verified directly against current PyPI/package-index
   data rather than relying on Pygments' general reputation.
3. **Its output shape fits ARKlight's closed-vocabulary boundary
   exactly**, with zero tension: Pygments' primary output mode is
   static HTML with CSS classes (or a fully static, pre-generated CSS
   stylesheet for a given style). Nothing about highlighting a code
   block needs to run in the browser -- it's a pure function of
   `(source text, language) -> static markup`, computed once at
   `arklight build` time and never touched again. This is the exact
   pattern `BUILD-TIME-ECOSYSTEM.md` §2 already describes generically;
   Pygments is simply the cleanest possible instance of it, cleaner
   even than the `matplotlib` example in that document, since there's
   no image encoding step at all -- just HTML and CSS, natively.

Every code block in every one of these documents (this one included)
is itself a small, live argument for why this matters: ACC's own docs
would look better and be more useful with this capability, and have
none today.

## The five stages

| Stage | Delivers | Lives in | Status |
| --- | --- | --- | --- |
| 0 | This document -- ladder, tool choice, acceptance criteria. | `ACC` (this repo) | Filed |
| 1 | Minimal capability-discovery hook | `ARKlight` core (**upstream dependency, not this repo's to build**) | **Landed** |
| 2 | ACC package skeleton + capability metadata shape | `ACC` (this repo) | Not started |
| 3 | `@acc/prism` -- the Pygments capability | `ACC` (this repo) | Not started |
| 4 | `@acc/common` -- first common-use components, consuming Stage 3 | `ACC` (this repo) | Not started |
| 5 | Installability: `arklight install/list/info` against a local package | `ARKlight` core + `ACC` (**joint, upstream-gated**) | Not started |

### Stage 1 -- capability-discovery hook (upstream, **landed on alpha**)

Nothing past Stage 0 can run without this, and it was deliberately
**not** ACC's to build unilaterally -- `acc-foundational-design.md`
§39 already draws ACC as sitting *beside* the compiler, and a
compiler-side discovery hook is, definitionally, compiler-side. It has
since landed on `alpha` as `arklight/capabilities.py`, matching every
point below. What Stage 1 needed, precisely, filed as a proposal
against `alpha` rather than guessed at by this repo:

- An entry-point group name ARKlight scans for -- `acc-foundational-
  design.md` §10 already proposes `arklight.capabilities`; Stage 1
  should either confirm that name or supersede it in one place.
- A minimal, closed registration contract: what shape of object an
  entry point may return, and what the compiler does with it
  (register the capability identity from `acc-foundational-design.md`
  §9; nothing else). No execution beyond importing the declared entry
  point and calling one documented registration function.
- Exactly the diagnostics `acc-foundational-design.md` §27 already
  requires: capability not found, duplicate capability with no
  multi-provider support declared, malformed metadata. All at build
  time, all as a normal ARKlight error, never a silent skip.
- No compatibility-version solver, no lockfile, no registry client --
  `acc-foundational-design.md` §35 already scopes those out of ACC's
  own alpha goals, and Stage 1 should be smaller still: it only needs
  to prove the compiler *can* see a capability that exists outside
  its own source tree, at all, for the first time.

Acceptance for Stage 1: a throwaway local package with one dummy
entry point is discoverable by an `alpha` build without editing
`arklight`'s own source for that package specifically. **Landed**,
with a nuance: `tests/test_capabilities.py` (9/9 passing) exercises
this by monkeypatching `importlib.metadata.entry_points` rather than
installing a real throwaway distribution -- an approach this same
paragraph pre-authorized for anything built before an end-to-end
package exists. The registration contract, entry-point group name,
and all three required diagnostics (`CapabilityError` for a bad/failed
entry point, a missing or invalid identity, and a duplicate identity
without `allow_multi` declared) are implemented and covered. A
`require_capability()` lookup helper also shipped, ahead of schedule,
for future consumers such as Stage 4's `CodeBlock`. Stages 3-5 are no
longer confined to designed-and-tested-in-isolation code exercised
only through direct calls -- the real discovery path now exists on
`alpha`.

### Stage 2 -- ACC package skeleton + capability metadata shape

The first code in *this* repo. Concretely:

- One example/reference package directory under a new `packages/`
  tree in this repo (not published anywhere yet), following the
  layout `acc-foundational-design.md` §37 already sketches
  (`pyproject.toml`, `src/<pkg>/capability.py`, `docs/README.md`).
- A settled, minimal metadata shape answering the subset of §8's list
  that Stage 1's registration contract actually needs -- package
  name, version, the capability identity string(s) it provides, and
  its ARKlight/ACC compatibility floor. Everything else in §8 stays
  deferred, per §35.
- No registry, no installer -- a package "installed" for Stage 2's
  purposes just means "present on `sys.path` and importable," exactly
  matching Stage 1's acceptance bar.

Acceptance for Stage 2: the reference package's metadata can be
statically validated (required fields present, capability identity
well-formed) by a small, pure-Python checker function in this repo,
independent of Stage 1 having landed.

### Stage 3 -- `@acc/prism`: the Pygments capability

The flagship capability from "Which tool, and why" above, built
against Stage 2's shape:

- Declares one capability, `code.highlight`, taking source text and a
  language identifier and returning static markup -- HTML with CSS
  classes -- plus a way to obtain the matching static stylesheet for
  a chosen Pygments style, computed once and reusable across every
  call in a build rather than regenerated per code block.
- Depends on `pygments` as an ordinary Python dependency, declared in
  the package's own metadata -- never a compiler dependency. A
  project that never calls `code.highlight` never needs `pygments`
  installed, per `acc-foundational-design.md` §6's closed-vocabulary
  guarantee: installing this ACC package doesn't make `pygments` an
  implicitly trusted part of the compiler language, only an
  implementation detail behind one declared capability.
- A clear, build-time diagnostic (per §27) if `pygments` isn't
  installed when `code.highlight` is actually called -- not a bare
  `ImportError` several frames deep, per `BUILD-TIME-ECOSYSTEM.md`
  §3's stated problem.
- No runtime JS, no client-side highlighting library shipped, ever --
  this is precisely the "browser never executes Python, and doesn't
  need to execute anything extra either" case `BUILD-TIME-ECOSYSTEM.md`
  §2 describes.

Acceptance for Stage 3: given a source string and a language name,
`code.highlight` returns markup and a stylesheet that render
correctly with no additional runtime dependency, verified by a direct
unit test against the capability function -- independent of whether
Stage 1's discovery path exists yet, exactly as Stage 1's own
acceptance note anticipates.

### Stage 4 -- `@acc/common`: first common-use components

The "components (user-defined ones like for common use cases)" half
of the original ask, and the first package that *consumes* another
ACC package's capability rather than only providing one:

- A small, deliberately unglamorous starting set -- a `CodeBlock`
  component that calls `@acc/prism`'s `code.highlight` capability
  being the obvious first one, since it's the one piece of "everyone
  hand-rolls this per project" friction `BUILD-TIME-ECOSYSTEM.md` §3
  names directly for exactly this kind of capability.
- Built as ordinary `component(...)` registrations per `alpha`'s own
  `USER-DEFINED-COMPONENTS.md` -- ACC does not invent a second
  component system; it distributes components that use the one
  `alpha` already has.
- A declared dependency on `@acc/prism`'s capability identity (not a
  Python import of its internals), exercising
  `acc-foundational-design.md` §14's ACC-package-depends-on-ACC-
  package model for the first time.

Acceptance for Stage 4: `CodeBlock(source=..., lang=...)` renders the
same output `@acc/prism` alone would, through the ordinary component
call syntax, with a clear compiler diagnostic (not a crash) if
`@acc/prism` isn't present.

### Stage 5 -- installability (joint, upstream-gated)

Wires Stages 2-4 into the actual `arklight install/list/info` surface
`acc-foundational-design.md` §12 sketches. Explicitly **not** started
until Stage 1 has actually landed on `alpha` -- there is no CLI
surface to build against before then, and speculative CLI code
written against an unlanded hook is exactly the kind of premature
surface `acc-foundational-design.md` §35's alpha-scope list already
warns off.

## Non-goals for this ladder

Per `acc-foundational-design.md` §35 and `BUILD-TIME-ECOSYSTEM.md` §4:
no registry service, no lockfile solver, no package signing, no ACC
SDK for third parties (that's downstream of Stage 5, not part of it),
and no second or third capability package until `@acc/prism` and
`@acc/common` are both real and used. One clean vertical slice first;
breadth after.
