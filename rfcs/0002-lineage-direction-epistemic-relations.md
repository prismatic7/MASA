# RFC 0002: Lineage direction for epistemic relations

- Status: proposed
- Authors: Chris Wenn (techne-tools)
- Created: 2026-08-08
- Target MASA version: 0.2.0

## Problem and use cases

MASA 0.1.0 defines a rich epistemic vocabulary in its ontology: `masa:listened-as` ("The subject was approached as the object under a declared listening pass", category listening), `masa:measured-from` (evidence), `masa:speculates-about` (epistemic), `masa:supports` (epistemic), and `masa:attributed-to` (responsibility). These predicates are registered, validated, and preserved in records, and `matter.inspect` reports them.

The lineage tool (`matter.trace_lineage`, `packages/core/src/lineage.ts`) only traverses relations whose predicate carries a registered `lineageDirection`. In 0.1.0 that registry is exclusively the material-derivation set: `derived-from`, `derivation-of`, `version-of`, `revision-of`, `captured-from`, `transduced-from`, `segmented-from`, `isolated-from`, `granulated-from`, `mapped-from`, `rendered-as`, `performed-as`, `parent-of`, `child-of`, `variation-of`, `continuation-of`. None of the epistemic predicates carry a direction, so a listening-pass graph is valid and inspectable but its causal edges are invisible to lineage queries.

Use cases that need traversable epistemic edges:

- A researcher traces the ancestry of an `inferred` claim back through its listening pass, aperture, and representation to ask *how* a claim was produced, not just *from what material*.
- An agent audits whether a `speculative` claim is grounded in a measured representation or floats free of any evidence chain.
- A practitioner compares two listening passes over the same representation and asks which claims each pass generated, as a graph rather than a flat list.

## Theoretical and terminology status

The listening vocabulary follows the AKOÚŌ operational listening contract (emeisazam / Sonic Field Labs, v0.9.1), which reserves `heard` for reports by an embodied listener and keeps model, sensor, and prompt outputs in `measured`, `inferred`, `interpreted`, or `undetermined`. The epistemic distinction between *material derivation* (a representation is granulated from another) and *epistemic production* (a claim is produced by a listening pass) follows the auditum tradition in sound studies: the object of listening isn't the physical signal but the perceptual and interpretive event (Ihde, *Listening and Voice*; Voegelin, *Listening to Noise and Silence*). MASA's material-derivation relations describe the former; the epistemic relations describe the latter. This RFC doesn't collapse the two: it makes the epistemic edges traversable while keeping their semantics distinct from derivation.

## Proposed contract

1. Register `lineageDirection` for three epistemic predicates in the ontology:

   | Predicate | Direction | Rationale |
   |---|---|---|
   | `masa:listened-as` | `object-is-descendant` | The subject (representation) is approached by the object (listening pass); the pass is the later epistemic event and its claims descend from it. |
   | `masa:measured-from` | `subject-is-descendant` | The subject (measurement) used the object (representation) as evidence; the measurement is the descendant. |
   | `masa:speculates-about` | `subject-is-descendant` | The subject (speculative claim) is a proposition about the object (representation); the claim is the descendant. |

2. Leave `masa:supports` and `masa:attributed-to` unregistered for now. `supports` is a claim-to-claim evidence relation whose direction is context-dependent (a supporting claim can precede or follow the claim it supports in time); `attributed-to` is a responsibility relation, not a causal one, and registering it would conflate accountability with derivation. Both remain valid, preserved, and inspectable.

3. No schema change. `lineageDirection` is an ontology-registry property; the Relation schema already accepts any predicate matching the protocol pattern.

## Alternatives

- Registering all epistemic predicates at once: rejected — `supports` and `attributed-to` have direction semantics that aren't causal, and registering them would make lineage graphs imply derivation where none exists.
- Introducing a separate epistemic lineage tool that walks all relations regardless of direction: rejected — it would duplicate the traversal machinery and blur the protocol's causal discipline; registering directions on the existing tool keeps one lineage semantics.
- Leaving the registry as-is and documenting the limitation: rejected — the use cases above are real and current; the vocabulary already exists, so the cost of registering directions is a registry edit plus conformance fixtures.

## Effects on implementations

Additive for readers and writers: no schema or record changes, no existing record invalidated. The reference implementation's lineage tool gains three traversable predicates; its conformance claims are unchanged in kind. Adapters that emit epistemic relations (for example the AKOÚŌ→MASA adapter in techne-tools/hermes-akouo-plugin) gain working lineage graphs without code changes beyond regenerating records. Hosts that audit claim provenance can now walk epistemic ancestry with the existing tool.

## Privacy, rights, accessibility, and ecological effects

Lineage traversal exposes the structure of how claims were produced, which is already present in records and already returned by `matter.inspect`. Registering directions doesn't add disclosure surface: the same relations were always readable; only the traversal changed. No new data is collected, and no accessibility or ecological effects follow from a registry edit.

## Migration and conformance

Added in the unreleased 0.2.0 line; no released record is invalidated. Fixtures: one valid listening record whose lineage trace returns the listening pass, aperture, and representation as ancestors of a claim; one valid record asserting that a `supports` relation is preserved but not traversed. Exercised by the reader and transformer conformance classes.

## Objections and unresolved questions

- Whether `masa:listened-as` should be `object-is-descendant` or `subject-is-descendant` depends on whether the listening pass is read as an event that happens *to* the representation (pass descends from representation) or as the frame that *produces* the representation-as-listened (representation descends from pass). This RFC takes the former: the representation exists before the pass, and the pass is the later epistemic act. The direction is a semantic commitment and should be confirmed against the ontology's intended reading before acceptance.
- Whether `masa:supports` deserves a direction after all, for claim-to-claim evidence chains in multi-pass studies: deferred to a future RFC with implementation evidence from a multi-pass corpus.
- Whether the lineage tool should expose a mode that traverses *all* relations with a warning for unregistered predicates, as a diagnostic aid: deferred; the strict registry keeps lineage semantics unambiguous.
