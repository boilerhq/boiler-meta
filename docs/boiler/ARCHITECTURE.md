# Boiler Architecture

Copyright (c) 2026 Pavel Mikheev (altsignum)  
Licensed under Creative Commons Attribution 4.0 International (CC BY 4.0)

Boiler is a contract-based architectural model for defining and composing
guarantees over a runtime.

## Specification Metadata

- Title: Boiler Architecture Specification
- Author: Pavel Mikheev (altsignum)
- License: Creative Commons Attribution 4.0 International (CC BY 4.0)
- Version: 0.8.9
- Status: Stable
- Last Updated: 2026-02-01
- Canonical Source: https://github.com/boilerhq/boiler-meta

## Goals

The system exists to provide a contract layer that guarantees reproducible and composable properties of development and
execution runtimes across heterogeneous toolchains.

It solves the following class of problems:

- Inconsistent and non-reproducible local environments.
- Hidden assumptions about tooling and runtime state.
- Tool and workflow capture that prevents reuse and evolution.
- Inability to compose environment guarantees across modules and platforms.
- Independence from operating system distributions, tooling, packaging systems, container runtimes, and virtualization
  mechanisms.

## Non-Goals

The system intentionally does NOT attempt to solve:

- Defining or orchestrating developer workflows.
- Replacing language ecosystems or package managers.
- Acting as a task runner, build system, or pipeline engine.
- Managing infrastructure, containers, or virtual machines.
- Providing a global environment or dependency manager.
- Enforcing a single toolchain, shell, or IDE.

These concerns are intentionally excluded to preserve composability, independence,
and coexistence with existing tools.

## Design Goals

The following concerns are part of the document:

- Convention-aligned: the system does not establish a separate convention domain and prefers widely recognized defaults.
- Self-contained: remain operable from repository contents, with no mandatory external services.
- Transparent: the system does not rely on hidden state and makes its behavior and state observable to users.
- Low entry barrier: require minimal prerequisites to begin using the system.

The following concerns are not part of the document:

- User-facing workflows
- CLI UX
- IDE integrations
- Implementation details
- Packaging and distribution mechanisms
- Hosting and deployment environments
- Operational processes and policies

## Table of Contents

- [Boiler Architecture](#boiler-architecture)
  - [Specification Metadata](#specification-metadata)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
  - [Design Goals](#design-goals)
  - [Table of Contents](#table-of-contents)
- [Terminology](#terminology)
- [Canonical References](#canonical-references)
  - [Reference Implementation](#reference-implementation)
  - [Identity Markers](#identity-markers)
- [Contracts](#contracts)
  - [Module Contract](#module-contract)
  - [Runtime Contract](#runtime-contract)
  - [Version Contract](#version-contract)
  - [Version Criterion Contract](#version-criterion-contract)
  - [Platform Contract](#platform-contract)
  - [Context Contract](#context-contract)
  - [Filesystem Layout Contract](#filesystem-layout-contract)
  - [Activation Contract](#activation-contract)
  - [Guarantee Contract](#guarantee-contract)
  - [Predicate Contract](#predicate-contract)

# Terminology

- **Implementation**: A concrete realization of the system that satisfies the contracts.
- **Repository**: A versioned collection of files and directories treated by the system as a single unit of operation.
- **Module**: A directory recognized by the system as a functional unit.
- **Runtime**: An execution environment in which the system operates.
- **Version**: A selector of a variation of a referenced entity.
- **Version Criterion**: A selector that expresses an intent to choose a Version for a referenced entity.
- **Platform**: an element of a Runtime classification system.
- **Context**: A selector that expresses how a Runtime is considered.
- **Filesystem Layout**: The structural organization of directories within a Repository.
- **Boiler Root**: A dedicated directory located at the Repository root, reserved for Boiler use.
- **Module Discovery Root**: A directory that defines a search root for Modules.
- **Activation**: An operation that establishes that the Implementation is executable for the Repository.
- **Guarantee**: A necessary condition that a Runtime must satisfy.
- **Predicate**: An operational contract that provides and verifies a single Guarantee over a Runtime.

# Canonical References

The following decisions give concrete form to the canonical realization, without introducing additional requirements
beyond the contracts.

## Reference Implementation

This reference implementation is non-normative and does not introduce additional requirements beyond the contracts.

The reference realization is expressed in GNU Bash compatible with version 4.4.

Rationale:

- Bash is treated as a widely available, well-understood baseline for repository-local automation. It supports minimal
  prerequisites, self-containment, and transparency.
- Bash 4.4 is selected as the minimal baseline that provides a stable and sufficiently expressive feature set (including
  associative arrays, reliable error handling, and consistent behavior across platforms), without depending on newer,
  less universally available language revisions.
- Bash 4.4 or a compatible version is widely available in contemporary operating system ecosystems targeted by typical
  Runtime realizations, either as a preinstalled component or as a trivially installable system package, without
  introducing additional language runtimes or external toolchains.
- Fixing a concrete baseline version establishes a deterministic interpretive reference for examples, tests,
  and reference behavior.

## Identity Markers

- **Boiler Root Directory Name**: `.boiler`
- **Module Configuration Directory Name**: `.boiler.m`

# Contracts

## Module Contract

Invariants:

- A Module MUST be a distinct Repository entity that exists as a directory.
- A directory qualifies as a Module only if it contains a dedicated configuration directory named
  `<Module Configuration Directory Name>`.
- Module composition MAY be expressed by dependency relations between Modules.
- Module composition MAY be expressed by containment relations through directory nesting.

Rationale:

- Module is defined as a Repository entity that exists as a directory to make module boundaries observable and
  verifiable within the Runtime filesystem, without introducing external registries or non-local identity schemes.
- The qualification marker directory is required to avoid treating arbitrary directories as Modules and to provide a
  stable, explicit criterion for Module recognition, while also serving as the dedicated location for Module-specific
  configuration within the Repository.
- Module composition is expressed only via dependency relations and directory containment to reflect the two
  fundamental ways in which real systems are structured: logical coupling between components and structural
  grouping within a Repository.

## Runtime Contract

Invariants:

- The Runtime MUST provide a filesystem with stable, hierarchical paths.
- The Runtime MUST provide read and write access to the Repository filesystem.
- The Runtime MUST provide a notion of a current working directory.
- The Runtime MUST provide a process environment that can carry environment variables.
- The Runtime MUST permit execution of the Implementation as required to satisfy model contracts.

## Version Contract

Invariants:

- A Version MUST be meaningful only relative to a referenced entity.
- A Version MUST select exactly one variation of a referenced entity when such a Version exists.
- A Version MUST NOT assume any universal format across different referenced entities.
- The set of meaningful Versions MUST be defined by the referenced entity.

Rationale:

- Version is not defined as a global scheme to avoid introducing a universal notion of versioning, since no
  cross-entity semantic equivalence of Versions is assumed.
- The domain of meaningful Versions is owned by the referenced entity, since only that entity defines what
  constitutes a semantic variation of its condition.

## Version Criterion Contract

Invariants:

- Version Criterion MUST express intent to choose a Version for a referenced entity.
- Version Criterion MUST denote a single Version when such a Version exists for the referenced entity.
- Version Criterion MUST be a closed set via Definitions.

Definitions:

- LTS — denotes the long-term supported Version for the referenced entity.
- latest — denotes the most recent Version for the referenced entity.

Rationale:

- Version Criterion is defined as an intent selector, not as a Version itself, to separate user intent from the resolved
  semantic variation.
- Version Criteria are closed to keep the intent domain finite and explicitly enumerable, ensuring that Version
  selection remains specifiable and verifiable, and preventing unbounded semantic labels from entering the model.

## Platform Contract

Invariants:

- A Platform MUST denote a classification of Runtime instances.
- A Platform MUST denote a single classification value, not a set of values.
- A Platform MUST denote a terminal classification of Runtime instances.
- A Platform MUST NOT determine or imply a Version.
- A Platform MUST be subject to Version selection.
- A Platform MUST constrain the applicability of contracts that declare Platforms.
- A Platform MUST constrain only which assumptions are admissible, not which states are guaranteed.

Canonical Naming:

- Platform identifiers are lowercase ASCII strings.
- Each Platform MUST have exactly one canonical representation.
- The canonical representation MUST be used consistently wherever the Platform is referenced.

Rationale:

- Platform is required to be terminal to prevent hierarchical or umbrella classifications from being treated as concrete
  Platforms. This ensures that a Platform always denotes a concrete and maximal classification of Runtime instances,
  rather than an intermediate category that can be further refined. Keeping Platforms terminal avoids ambiguous
  applicability and prevents coarse-grained labels from being used where concrete runtime constraints are required.
- Platform is required to be subject to Version selection to reflect that concrete runtime platforms evolve over time
  and admit meaningful semantic variations. Separating Platform from Version preserves orthogonality between
  classification and variation, ensuring that platform identity remains stable while its concrete realizations are
  selected explicitly via Version.
- Platform constrains only applicability and admissible assumptions to avoid introducing platform-specific guarantees.
  This keeps Platform as a classification axis that limits where contracts may apply, without coupling it to concrete
  runtime states or turning it into an implicit bundle of guarantees.
- Platform does not include processor architecture, because such properties affect artifact construction and
  toolchain selection, not Runtime semantics. Processor architecture is derivable or contract-specific concerns
  that become relevant only for specialized guarantees, and therefore do not belong to the core classification axis
  used for applicability.
- Canonical naming is defined to align Platform representations with widely accepted industry terminology, in order to
  avoid ambiguity, informal aliases, and project-specific shorthand when referring to external runtime platforms.

## Context Contract

Invariants:

- A Context MUST select exactly one interpretation of Runtime for a Module.
- A Context MUST be a closed set via Definitions.

Definitions:

- Development — denotes the interpretation of Runtime as a construction environment.
- Execution — denotes the interpretation of Runtime as an operation environment.

Rationale:

- The set of Contexts is closed because any interpretation of Runtime in practice reduces to one of two cases:
  treating Runtime as an environment in which a Module is constructed, or treating Runtime as an environment in
  which a Module is operated. Construction-oriented interpretations include environments where the Runtime is
  considered for producing, validating, or preparing a Module (such as local development or automated builds).
  Operation-oriented interpretations include environments where the Runtime is considered for running, hosting,
  or using a produced Module (such as deployed services or end-user execution). These variations do not introduce
  additional Contexts, but are realizations of the same two interpretations.

## Filesystem Layout Contract

Invariants:

- The Filesystem Layout MUST be defined relative to the Repository root.
- The Repository MUST contain a Boiler Root located at the Repository root.
- The system recognizes the following directory roles within a Repository:
  - Boiler Root
  - Module Discovery Roots
- The Boiler Root MUST be represented by exactly one directory and MUST NOT be configurable.
- Module Discovery Roots MAY be represented by one or more directories and MAY be configurable.
- If a configurable directory role is used by the Repository and no explicit configuration is provided, the canonical
  baseline MUST be used.

Definitions:

- Boiler Root: intended to serve as a reserved namespace for system-owned state and configuration.
- Module Discovery Root: intended to define the search roots for Modules maintained within the Repository.

Canonical baseline:

- `<Boiler Root Directory Name>/` — Boiler Root
- `src/` — primary Module Discovery Root

Rationale:

- The canonical layout is based on widely recognized Repository structure conventions.
- Canonical defaults provide a predictable baseline while allowing explicit deviation when required.
- The system promotes consistent Repository structure without enforcing it rigidly.

## Activation Contract

Invariants:

- Activation MUST be an operation that establishes that the Implementation is executable for the Repository.
- Activation MUST be Repository-local.
- Activation MUST be idempotent for a given Repository.
- Activation MAY be performed automatically; automation MUST NOT be required for correct operation.
- The Implementation MUST be executable at the point it is first required by the system; the system MUST NOT require the
  Implementation to be executable before that point.

Rationale:

- Activation is defined as a single operation to separate availability of execution from any integration sequencing.
- Automation of Activation is optional and exists solely to improve developer experience; it is not required for
  correct system operation.

## Guarantee Contract

Invariants:

- A Guarantee MUST represent a necessary property of Runtime state.
- A Guarantee MUST be a declarative assertion over Runtime state.
- A Guarantee MUST have an identifier.
- The same Guarantee identifier MUST always denote the same asserted property.
- A Guarantee MUST be verifiable by observing Runtime state.
- For any Runtime instance within scope, a Guarantee MUST evaluate to either satisfied or violated.
- A Guarantee MUST describe required properties, not a complete Runtime state.
- A Guarantee MAY be subject to Version selection.
- A Guarantee MUST be either unversioned or versioned.
- An unversioned Guarantee MUST be evaluable without Version.
- A versioned Guarantee MUST be evaluable only with Version.
- A versioned Guarantee MUST define the set of Versions meaningful for that Guarantee.
- A Version provided to evaluate a versioned Guarantee MUST select exactly one variation of that Guarantee.
- A Version MUST NOT be accepted by an unversioned Guarantee.

Rationale:

- Guarantees are modeled as state properties to separate required conditions from establishment mechanisms.
- Total satisfied/violated evaluation is required to avoid implicit "unknown" outcomes inside the contract layer.
- Splitting Guarantees into unversioned and versioned removes ambiguity about when Version is required.
- The Version domain is owned by the referenced entity to preserve Version orthogonality and avoid universal schemes.

## Predicate Contract

Invariants:

- A Predicate MUST represent exactly one Guarantee.
- A Predicate MUST have an identifier that is equal to the identifier of its Guarantee.
- A Predicate MAY declare a Variant of itself; if a Predicate does not declare a Variant, it MUST be interpreted as the
  canonical Variant.
- All Variants of a Predicate MUST represent the same asserted property of the Guarantee.
- A Predicate MUST declare whether its Guarantee is unversioned or versioned.
- A Predicate MUST declare exactly one target Platform.
- A Predicate MUST declare a minimum supported Version of the target Platform.
- A Predicate MUST declare a maximum supported Version of the target Platform.
- A Predicate MUST NOT be applicable outside its declared target Platform and Version range.
- A Predicate MAY declare dependencies on other Predicates by referencing their identifiers.
- A Predicate MUST NOT rely on undeclared dependencies.
- A Predicate MUST be applicable only when all of its declared dependencies are applicable.
- A Predicate MUST expose exactly one operation for each defined Context.
- A Predicate MUST expose a Check operation.
- A Predicate MUST expose a Version Resolution operation if its Guarantee is versioned.

Context Operations:

- A Context operation MUST accept Version if the Guarantee is versioned.
- A Context operation MUST NOT accept Version if the Guarantee is unversioned.
- A Context operation MUST target a Runtime state where the Guarantee evaluates to satisfied.
- A Context operation MUST rely on the same observation criteria as the Check operation to determine whether the
  Guarantee is satisfied.
- Reapplying the same Context operation with the same inputs MUST NOT change the targeted result.

Check Operation:

- A Check operation MUST evaluate the Guarantee for a given Runtime instance.
- A Check operation MUST return either satisfied or violated.
- A Check operation MUST have no side effects.
- A Check operation MUST accept Version if the Guarantee is versioned.
- A Check operation MUST NOT accept Version if the Guarantee is unversioned.
- For a versioned Guarantee, a Check operation MUST treat the provided Version as selecting exactly one variation.

Version Resolution Operation:

- A Version Resolution operation MUST accept Version Criterion.
- A Version Resolution operation MUST return either a Version or Unsupported.
- A Version Resolution operation MUST have no side effects.
- A Version Resolution operation MUST be defined if and only if the Guarantee is versioned.

Rationale:

- Variant allows multiple ways to provide the same Guarantee without introducing multiple Guarantees or identifiers,
  preserving semantic uniqueness while making realization choices explicit. This enables alternative realization
  strategies for the same asserted property of Runtime state, such as providing it via a system installation,
  via a cached distribution, or via an internal registry mirror, while keeping the semantic contract of the
  Guarantee unchanged.
- A finite supported Platform Version range is required to keep Predicate applicability decidable and to define a
  bounded domain within which the Predicate can be meaningfully realized.
- Idempotence of Context operations is required to allow repeated application without introducing implicit sequencing
  or workflow semantics.
