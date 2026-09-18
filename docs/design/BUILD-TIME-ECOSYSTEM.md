# ACC and the Build-Time Python Ecosystem

_A grounding document for `docs/design/`. Written against ARKlight's
own `alpha` branch source (`arklight/parser/loader.py`), not from the
pitch alone -- the same verification standard `alpha`'s own
`docs/Foundational/WHAT-ARKLIGHT-IS.md` holds itself to. Cross-check
that source directly before treating any claim below as still
accurate if `alpha` has moved since this was written._

Status: **reference architecture, not yet implemented.** This
document exists to record *why* ACC is shaped the way it is before
any of it is built -- the same role `acc-foundational-design.md`
already plays for the package system generally. See that document
for the capability-declaration and discovery model this one builds
on top of.

## 1. ACC is an extension ecosystem beside the compiler, not a module inside it

The relationship to keep fixed in mind is the one `npm` has to
Node.js, not the one a compiler plugin has to a compiler:

    Node.js            ARKlight
        |                   |
        | ships stdlib      | ships primitives + closed vocabulary
        v                   v
      node               arklight
        |                   |
        | separate tool     | separate tool
        v                   v
       npm                 ACC
        |                   |
        | installs          | installs
        v                   v
    node_modules/       discovered capabilities

`npm` does not get compiled into `node`; it manages packages that
`node` then loads at run time through ordinary, already-existing
mechanisms (`require`, entry points, `package.json`). ACC does the
same job for ARKlight: it resolves, installs, and versions packages,
and hands the compiler a set of declared capabilities to *discover*
through the entry-point mechanism `acc-foundational-design.md` §10
already describes. Nothing about installing an ACC package changes
`arklight`'s own source, adds a new compiler stage, or requires the
core compiler to know an ACC package exists until that package is
actually installed and its capability is actually called. This is
the same boundary §39's "ACC sits beside the compiler rather than
inside it" already states -- this section just names the more
familiar analogy for it.

One difference from `npm` worth being explicit about: `npm` packages
are trusted to run arbitrary JS at install and require time. ACC
packages are not automatically the same -- §6, §19, and §20 of
`acc-foundational-design.md` already draw that line (installing a
package must not make arbitrary Python an implicitly trusted part of
the compiler *language*). The analogy is about **distribution
architecture** -- a separate tool, a separate namespace, a separate
lifecycle from the compiler's own release cycle -- not about the
trust model.

## 2. The compile-time / run-time boundary this depends on

ARKlight draws a hard line between two execution contexts, and it's
worth stating precisely because ACC's whole reason to exist lives on
one side of it:

- **Compile time** -- `arklight build` runs once, on the developer's
  machine, in a real CPython process. `arklight/parser/loader.py`
  confirms this directly: it does `compile(source, ..., mode="exec")`
  followed by `exec(code, module.__dict__)` against the actual
  interpreter -- no restricted-execution wrapper, no import
  allowlist. A page function is an ordinary Python module.
- **Run time** -- what a browser (or Android/desktop shell) actually
  receives is generated from IR: plain HTML/CSS and a closed-
  vocabulary JS runtime. `docs/README.md`'s Philosophy section on
  `alpha` states this as "the browser never executes Python," backed
  by "no eval, no new Function, no string ever executed as code"
  anywhere in the shipped runtime.

Nothing in that closed-vocabulary guarantee restricts *compile time*.
It restricts what the compiler is allowed to *emit*. That means the
full Python ecosystem -- `matplotlib`, `pandas`, `Pillow`, `requests`,
whatever a package needs -- is already usable inside a page function
today, with no ACC involved, as long as its output is lowered into
something the closed runtime vocabulary can actually express:

```python
import matplotlib.pyplot as plt
import io, base64

def stats():
    fig, ax = plt.subplots()
    ax.plot(get_data())
    buf = io.BytesIO()
    fig.savefig(buf, format="png")
    encoded = base64.b64encode(buf.getvalue()).decode()
    return Image(src=f"data:image/png;base64,{encoded}")
```

`matplotlib` here plays the same role a C++ build's `constexpr`/
code-generation step plays: the generator is unrestricted because
nothing about *how the constant was produced* survives into what the
runtime actually executes. The browser only ever sees a static
`<img>`. The closed-vocabulary invariant is never in the same
conversation as the library that produced the pixels.

## 3. Why this needs ACC even though it already works

Nothing above needs a package manager to be true -- it's already true
of `alpha` today, package or no package. What's missing is everything
*around* it:

- Every site that wants this today hand-rolls its own
  import-render-encode-embed plumbing, per capability, per project.
- There's no declared identity for "the thing that turns `data` into
  a chart" the way `component(...)` gives a declared identity to a
  reusable node type (`acc-foundational-design.md` §9's capability
  identity, `alpha`'s `USER-DEFINED-COMPONENTS.md` for the component
  side of the same idea). Two sites solving the same problem produce
  two independent, undiscoverable, unversioned solutions.
- Nothing validates ARKlight-version or capability compatibility for
  this kind of code the way `acc-foundational-design.md` §15 already
  requires for any other ACC capability.
- Nothing distinguishes "this needs `matplotlib` installed" from
  "this needs nothing but stdlib," so failures show up as a bare
  `ImportError` deep in a page function instead of the compiler
  diagnostic `acc-foundational-design.md` §27 calls for.

That gap -- real, but entirely about packaging, discovery, and
diagnostics, not about compiler architecture -- is what "janky and
finicky" means in practice, and it's exactly ACC's job to close it.
An ACC package can wrap the pattern in §2 behind a declared capability
(`chart(data)`, per `acc-foundational-design.md` §7) with its own
dependency on `matplotlib`, its own ARKlight-version compatibility
range, and its own compiler diagnostic when something's missing --
none of which requires the compiler core to know `matplotlib` exists.

## 4. Where this sits relative to the compiler core

`alpha`'s compiler core works with stdlib alone and stays that way on
purpose -- it isn't the core's job to care whether a capability needs
a heavyweight third-party dependency, only that whatever a capability
lowers to fits the closed IR/AST it already defines
(`acc-foundational-design.md` §21). A build-time-ecosystem capability
is squarely a **non-goal for the core, and squarely ACC's territory**:
the compiler stays small and auditable; ACC is where "you don't have
to hand-roll this yourself" capabilities accumulate, first-party and
third-party alike (`acc-foundational-design.md` §29-30).

Once the capability-discovery interface this depends on is stable, ACC
can expose an SDK for third parties to package and share their own
build-time capabilities the same way -- an ACC-layer concept, not a
compiler feature, and not in scope until the plumbing in §3 actually
ships (`acc-foundational-design.md` §35's alpha-scope list already
excludes "arbitrary compiler plugins" and "native capability bridges"
for the same reason: sequence the foundation before the surface built
on it).

## 5. Non-claims

This document does not claim:

- that any of this is implemented -- it is not; see Status above.
- that ACC changes the closed-vocabulary *runtime* guarantee -- it
  doesn't touch it, by construction (§2).
- that ACC packages get elevated trust merely by being ACC packages
  -- §1's caveat and `acc-foundational-design.md` §19-20 still apply
  in full.
