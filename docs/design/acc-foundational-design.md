# ACC — Foundational Design Document

ARKlight Components Collection
ACC
Foundational Design Document

ARKlight Alpha Compiler
Status: Proposal
Scope: Foundational
Audience: ARKlight contributors and component authors


## 1. Overview

ACC, the ARKlight Components Collection, is the package and distribution
system for ARKlight capabilities.

ARKlight is a compiler framework with Python-based authoring, a
batteries-included developer workflow, and compilation targets that include
static Web artifacts and cross-platform wrapped applications.

As ARKlight's authoring vocabulary grows, users need a way to distribute
reusable ARKlight capabilities without treating every capability as part of
the core compiler.

ACC provides that mechanism.

An ACC package may provide components, authoring functions, actions,
derivations, predicates, compiler-recognized vocabulary, styles, templates,
static assets, or other ARKlight-specific extensions.

ACC is not intended to replace Python's package ecosystem.

Python packages remain Python packages.

ACC defines how those packages become discoverable and usable as ARKlight
capabilities.


## 2. The Problem

ARKlight's authoring model is intentionally Python-based.

This makes ordinary Python packaging immediately useful for distributing
Python code.

However, an ARKlight capability is not necessarily ordinary Python code.

A capability may represent a semantic operation understood by the ARKlight
compiler and lowered into ARKlight IR.

For example, an authoring function might conceptually look like:

    chart(data)

but the important part is not that Python executes `chart`.

The important part is that the compiler recognizes the operation and
produces the appropriate ARK AST / IR representation.

This creates a distinction:

    Python package
        |
        v
    Python objects

versus:

    ACC package
        |
        v
    ARKlight capability
        |
        v
    ARK AST
        |
        v
    ARK IR
        |
        v
    compilation target

Python packaging solves the first problem.

ACC exists to solve the second.


## 3. Definition

ACC is ARKlight's distributable collection of third-party and first-party
authoring capabilities, components, and compiler extensions.

An ACC package is a distributable unit which declares one or more
ARKlight-specific capabilities and the metadata required for ARKlight to
discover, validate, install, and use them.

ACC packages may be implemented using normal Python packaging mechanisms.

ACC does not require a separate Python language or execution environment.


## 4. Relationship With Python Packaging

ACC should build on Python packaging rather than reinvent it.

Python already defines standardized mechanisms for package metadata,
dependencies, versions, and entry points.

Entry points allow an installed Python distribution to advertise components
which another application can discover. They are explicitly designed for
plugin-style extension and discovery.

ARKlight can therefore use Python's packaging infrastructure as the
distribution layer while defining an ARKlight-specific interface on top of
it.

Conceptually:

    Python Distribution
          |
          +-- Python metadata
          +-- Python dependencies
          +-- ACC metadata
          +-- ACC entry point(s)
          |
          v
    ARKlight compiler

ACC therefore becomes an ARKlight package system rather than a competing
replacement for pip or the Python packaging ecosystem.

The compiler should reuse established packaging mechanisms wherever they are
appropriate.


## 5. What an ACC Package Can Provide

An ACC package may provide one or more of the following:

    Components
    Authoring functions
    Actions
    Computed / Derive operations
    Predicates
    State-related vocabulary
    Compiler-recognized constructs
    ARK AST extensions
    ARK IR extensions
    CSS / styling resources
    Static assets
    Templates
    Documentation
    Environment-specific implementations
    Other explicitly supported ARKlight capabilities

Not every ACC package needs to provide all of these.

A package should declare exactly what it provides.

The initial ACC implementation should keep the supported capability classes
small and explicit.

New capability classes can be added as the compiler architecture matures.


## 6. Closed Vocabulary

ACC must not undermine ARKlight's closed-vocabulary architecture.

Installing an ACC package must not mean that arbitrary Python code becomes
an implicitly trusted part of the compiler language.

This distinction is fundamental.

Bad model:

    install package
        ->
    compiler imports arbitrary Python
        ->
    arbitrary Python execution becomes part of compilation

Preferred model:

    install package
        ->
    compiler discovers declared ACC capability
        ->
    capability is validated
        ->
    capability maps to known ARKlight semantics
        ->
    ARK AST / IR
        ->
    backend

The package may contain Python implementation code.

However, ARKlight should only expose the portions of that package which are
declared and supported by the ACC interface.

ACC extends the vocabulary deliberately.

It does not turn Python imports into an unrestricted compiler escape hatch.


## 7. Authoring Functions

An ACC package may expose a Python function as an ARKlight authoring
construct.

The function is an authoring interface.

It should not be assumed to be an ordinary runtime function.

For example:

    from acc.charts import chart

    chart(data)

may cause ARKlight to construct an internal semantic representation of a
chart.

The browser does not receive the Python function.

The compiler does not need to ship Python.

The final artifact contains the result of compilation according to the
capability's supported semantics.

This preserves the distinction between:

    Python authoring

and:

    ARKlight execution / generated artifacts.


## 8. Package Metadata

An ACC package requires metadata sufficient for the compiler to determine
what the package is and whether it can be used.

At minimum, metadata should eventually describe:

    Package name
    Package version
    ACC specification version
    Supported ARKlight versions
    Provided capabilities
    Required ARKlight capabilities
    Python requirements
    Supported compilation environments
    Package dependencies
    License information
    Documentation information

The exact metadata format is intentionally left open at this stage.

The metadata should preferably live alongside normal Python project metadata
rather than creating an entirely separate packaging format.


## 9. Capability Declaration

A capability should have an explicit identity.

Conceptually:

    capability = "charts"

or:

    capability = "charts.plot"

The capability identity is not necessarily the Python import path.

This allows the compiler to distinguish:

    Python implementation

from:

    ARKlight semantic capability

That distinction becomes important when multiple environments or compiler
backends eventually implement the same semantic capability differently.


## 10. Discovery

ACC packages should be discoverable through standard Python package
metadata.

Python entry points are a natural mechanism for this purpose because they
allow installed distributions to advertise components for discovery.

ARKlight can define its own entry-point group, for example:

    arklight.capabilities

The exact group name is subject to final specification.

An ACC package could then advertise its ARKlight capabilities through normal
Python package metadata.

The compiler discovers installed capabilities rather than scanning arbitrary
Python modules.

This provides a deterministic boundary between the Python package ecosystem
and the ARKlight compiler.


## 11. Package Names

ACC packages should have a recognizable ARKlight identity.

The proposed user-facing namespace is:

    @acc

Examples:

    arklight install @acc/charts
    arklight install @acc/gallery
    arklight install @acc/forms

The namespace is conceptual and does not require Python distribution names
to use the same syntax.

The CLI is responsible for translating ACC identifiers into package
resolution requests.


## 12. CLI

The initial user-facing interface should be simple.

Install:

    arklight install @acc/charts

Remove:

    arklight remove @acc/charts

Update:

    arklight update @acc/charts

List installed packages:

    arklight list

Search:

    arklight search @acc

Inspect:

    arklight info @acc/charts

The precise command structure may change as the package manager develops.

The important principle is that ACC operations should feel like part of
ARKlight's developer workflow rather than requiring users to understand the
underlying Python packaging machinery.


## 13. Installation

Installing an ACC package should perform the following conceptual steps:

    resolve package
        ->
    resolve dependencies
        ->
    verify compatibility
        ->
    install distribution
        ->
    register / discover capabilities
        ->
    validate ACC metadata
        ->
    make capability available to compiler

Installation should not silently modify ARKlight's core compiler source.

ACC packages remain external distributions.

The compiler discovers them through the supported extension interface.


## 14. Dependencies

ACC packages may depend on other ACC packages.

For example:

    @acc/gallery
        requires
            @acc/images

Dependencies should use version constraints.

Python packaging already provides a standardized dependency-specifier model
for naming distributions, constraining versions, and expressing conditional
dependencies.

ACC should reuse that machinery where possible.

The ACC layer may add ARKlight-specific compatibility requirements on top.

For example:

    package dependency
        +
    ARKlight compatibility
        +
    capability compatibility

must all be satisfied before the package is usable.


## 15. ARKlight Compatibility

An ACC package must declare which ARKlight compiler versions it supports.

A package may depend on a particular compiler API, AST structure, IR
feature, or capability interface.

The compiler must detect incompatible packages before compilation proceeds.

An incompatible package should produce a compiler/package diagnostic.

It should not be silently ignored.

It should not be rewritten into something approximately compatible.

ARKlight's compiler philosophy applies here too:

    explicit failure is preferable to semantic guessing.


## 16. Versioning

ACC packages have their own versions.

ARKlight itself has its own version.

The ACC specification has its own compatibility level.

These are separate concepts.

Conceptually:

    ACC package version
        !=
    ARKlight compiler version
        !=
    ACC specification version

A package may therefore declare:

    requires ARKlight >= X
    requires ACC >= Y

The exact version syntax should follow established Python packaging
conventions where possible.


## 17. Local and Offline Packages

ACC should not require a network connection for every installation.

The package system should eventually support:

    registry packages
    local packages
    local archives
    source distributions
    prebuilt distributions
    cached packages

This is particularly important for compiler development, testing, CI, and
restricted environments.

A package installed from a local source should still pass the same metadata
and compatibility validation as a registry package.


## 18. Registry

ACC requires a package index or registry for convenient discovery.

The registry is a distribution service, not part of the compiler itself.

The compiler should be capable of consuming packages from configured
sources.

The initial implementation does not need to define a global public registry
immediately.

A local or development registry can be sufficient for early alpha testing.

The long-term registry should provide:

    package discovery
    versions
    metadata
    dependency information
    package artifacts
    integrity information
    documentation references
    publication information


## 19. Trust and Security

ACC introduces an extension boundary into the compiler.

That boundary must be treated as a security boundary.

Installing an arbitrary Python package already means trusting that package's
installation and build behavior.

ACC adds another concern:

    what compiler semantics is this package allowed to introduce?

An ACC package must therefore declare its capabilities.

The compiler should validate those declarations.

The package system should not silently grant arbitrary compiler privileges.

Capabilities which can access files, execute commands, communicate with
external services, modify the build environment, or otherwise cross a
security boundary require additional design and explicit policy.

ACC should start with capabilities which remain within well-defined
ARKlight compiler semantics.


## 20. No Arbitrary Compiler Execution

ACC should not become:

    "pip, but the compiler imports everything."

The compiler must not treat package-provided Python as an unrestricted
execution surface.

An ACC extension should operate through an explicit ARKlight extension API.

The extension API determines what the package can contribute.

This preserves:

    closed vocabulary
    predictable compilation
    compiler diagnostics
    deterministic behavior where possible
    backend independence
    security boundaries


## 21. Compilation Model

An ACC capability should ultimately participate in the normal ARKlight
compiler pipeline.

Conceptually:

    Python authoring
        |
        v
    Python AST
        |
        v
    ARK AST
        |
        v
    normalization
        |
        v
    validation
        |
        v
    Website / ARK IR
        |
        v
    backend / environment
        |
        v
    generated artifact

ACC should extend this pipeline rather than bypass it.

A capability should not need to invent its own parallel rendering system
unless explicitly supported by the compiler architecture.


## 22. Backend Independence

ACC capabilities should distinguish semantic definitions from backend
implementations.

A capability may be usable by one backend and unsupported by another.

For example:

    capability
        |
        +-- Web implementation
        |
        +-- Android implementation
        |
        +-- Desktop implementation

The capability itself represents the semantic operation.

The backend implementation represents how that operation is compiled.

If a package does not support a target, ARKlight should diagnose the
unsupported use at compile time.


## 23. Environments

ARKlight has an evolving distinction between compilation mechanisms and
execution contexts.

ACC should not hard-code assumptions about future target environments.

A package may eventually declare support for particular ARKlight targets or
environments.

For the initial ACC design, this is metadata and compatibility information,
not a requirement to implement a complete multi-environment extension
system.

Future environment support can build on the same semantic capability model.


## 24. Components

Components are one of the most obvious ACC use cases.

A package may provide reusable ARKlight components which can be consumed by
an authoring project.

For example:

    @acc/gallery

could provide:

    Gallery(...)
    ImageGrid(...)
    Lightbox(...)

The component declarations remain compiler-facing constructs.

The compiler may expand or lower them according to ARKlight's component
architecture.

The final generated artifact does not need to contain the ACC package or
the component's Python implementation.


## 25. Assets

An ACC package may also provide static resources.

Possible resources include:

    CSS
    images
    icons
    fonts
    templates
    JavaScript generated by the package's supported compiler interface

Asset handling must remain explicit.

Installing a package should not automatically inject unused assets into
every generated site.

Unused package resources should remain outside the output unless referenced
by the compiled program.


## 26. Documentation

ACC packages should be able to expose documentation.

Documentation may eventually integrate with ARKlight's local documentation
retrieval system.

This would allow an installed capability to become discoverable through
ARKlight's own tooling.

For example:

    arklight search @acc/charts

could expose:

    package metadata
    capability list
    usage documentation
    compatibility
    examples

This is particularly useful once ARKlight's documentation retrieval and
assistant systems become more mature.


## 27. Compiler Diagnostics

ACC failures should use normal ARKlight diagnostics.

Examples:

    ACC package not installed
    capability not found
    incompatible ARKlight version
    incompatible ACC specification
    missing dependency
    unsupported target
    duplicate capability
    malformed package metadata
    unsupported extension interface
    conflicting capability provider

Diagnostics should identify the package and capability involved.

The compiler should not silently select an arbitrary provider when two
packages claim the same capability.


## 28. Duplicate Capabilities

Capability identity must be deterministic.

If two installed packages provide the same exclusive capability, ARKlight
should report a conflict unless the capability explicitly supports multiple
providers.

The compiler must not silently choose:

    first installed
    newest installed
    alphabetically first
    whatever Python happened to import

Package resolution should be explicit.

Humans have spent decades discovering that "whatever happened first" is not
a dependency-management strategy.


## 29. First-Party and Third-Party Packages

ACC should support both first-party and third-party packages.

First-party packages may be maintained alongside ARKlight itself.

Third-party packages are independently distributed.

Both should use the same public capability interface.

A package being first-party does not remove the need for metadata and
compatibility validation.


## 30. Core vs ACC

Not every ARKlight capability should become an ACC package.

The core compiler should contain foundational language and compiler
semantics.

ACC should contain capabilities which are useful extensions but do not need
to be permanently embedded in the compiler core.

A useful boundary is:

    Core
        fundamental compiler semantics

    ACC
        distributable extensions to those semantics

The boundary should remain conservative during alpha development.


## 31. Installation State

The package manager should maintain enough local state to answer:

    What ACC packages are installed?
    Which version is installed?
    Where did the package come from?
    Which capabilities does it provide?
    Which dependencies caused it to be installed?
    Which ARKlight versions does it support?

The exact lockfile or state format is not defined by this document.

A future lockfile may be desirable for reproducible projects.

Until then, package state should remain inspectable and recoverable.


## 32. Reproducibility

ACC should support reproducible compilation.

A project should eventually be able to describe the exact ACC package versions
required to reproduce its build.

This is particularly important because an ACC capability can affect compiler
output.

A project depending on:

    @acc/charts 1.x

should not unexpectedly compile differently merely because the registry
returned a newer incompatible implementation.

Exact resolution and lockfile semantics can be introduced after the basic
package manager is operational.


## 33. Update Behavior

Updates should be explicit.

The package manager should not silently replace compiler extensions during
an ordinary build.

Conceptually:

    arklight build

uses the project's resolved package state.

While:

    arklight update

changes that state.

This separation keeps compilation predictable.


## 34. Removal

Removing an ACC package should remove its installed distribution and make
its capabilities unavailable.

If the project still references a removed capability, compilation should
fail with a clear diagnostic.

ARKlight should not attempt to silently replace the capability.


## 35. Alpha Scope

The first ACC implementation should remain deliberately small.

Initial goals:

    package discovery
    package installation
    package removal
    installed-package listing
    package metadata
    capability declaration
    ARKlight version compatibility
    dependency resolution
    compiler capability discovery
    basic diagnostics

The first implementation does not need:

    a sophisticated public registry
    automatic trust scoring
    complex package signing
    native capability bridges
    arbitrary compiler plugins
    automatic code transformation
    a complete lockfile solver
    cross-environment implementation negotiation

Those can be addressed when the underlying compiler extension model is
stable.


## 36. Suggested Initial Commands

The initial CLI may provide:

    arklight install <package>
    arklight remove <package>
    arklight list
    arklight search <query>
    arklight info <package>
    arklight update <package>

Examples:

    arklight install @acc/charts

    arklight install @acc/gallery

    arklight list

    arklight info @acc/charts

The exact syntax remains subject to CLI conventions.


## 37. Example Package Model

A conceptual ACC package might contain:

    acc-charts/
        pyproject.toml
        README.md
        LICENSE
        src/
            acc_charts/
                __init__.py
                capability.py
                compiler.py
        docs/
            README.md

Its Python project metadata declares the package normally.

Its ARKlight metadata declares the capability.

The compiler discovers the capability through the supported extension
interface.

The exact metadata schema is intentionally deferred until the compiler-side
extension API is defined.


## 38. Design Principles

ACC follows several principles.

1. Python remains the authoring language.

2. Python packaging remains the underlying distribution ecosystem.

3. ACC adds ARKlight-specific capability discovery and lifecycle.

4. Capabilities are explicit.

5. The compiler remains the authority over compilation semantics.

6. ACC must not become an arbitrary Python execution mechanism.

7. Compatibility failures are compiler diagnostics.

8. Backend support is explicit.

9. Unused capabilities should not affect generated output.

10. Package state should remain inspectable.

11. Installation and compilation are separate operations.

12. The core compiler should remain smaller than the ecosystem around it.


## 39. Architectural Summary

The intended relationship is:

    Python
      |
      | authoring
      v
    ARKlight
      |
      | discovers
      v
    ACC capability
      |
      | declares
      v
    ARK AST / IR semantics
      |
      | lowered by
      v
    ARKlight compiler
      |
      +-------------------+
      |                   |
      v                   v
    target A            target B


ACC therefore sits beside the compiler rather than inside it.

The compiler defines the language.

ACC distributes extensions to that language.

Python provides the authoring ecosystem.

The generated artifact remains the result of ARKlight compilation.


## 40. Status

ACC is an ARKlight alpha proposal.

This document defines the foundational intent and boundaries of the ACC
system.

It does not define the final package metadata schema, registry protocol,
dependency solver, signing system, lockfile format, or complete compiler
extension API.

Those should be specified only after the corresponding compiler interfaces
are stable.

The central decision is:

    ARKlight needs a native distribution mechanism for ARKlight capabilities.

That mechanism is ACC.

ACC should use the Python ecosystem where Python already solved the problem,
and add ARKlight semantics only where ARKlight actually has a different
problem to solve.


## 41. Non-Goals

ACC is not:

    a replacement for pip
    a replacement for Python packaging
    a Python runtime for generated sites
    an arbitrary plugin execution system
    a general native API
    a substitute for the ARKlight compiler
    a second programming language
    a requirement that every ARKlight project use third-party packages

ACC exists to make ARKlight extensible without making the compiler
architecturally uncontrolled.


## 42. Closing

ARKlight's Python authoring model makes reusable Python distributions
possible from the beginning.

ACC gives those distributions an ARKlight-native identity.

The distinction is small at the surface:

    Python package

becomes:

    ARKlight capability package

but architecturally it matters.

The package provides the implementation and metadata.

The compiler decides what that capability means.

The IR carries the semantics.

The target compiler produces the artifact.

That is the boundary ACC should preserve.
