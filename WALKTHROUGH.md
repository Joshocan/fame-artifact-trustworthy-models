# Walkthrough: the well-formedness conditions, one at a time

This guide accompanies *Construction Before Alignment: Trustworthy Models and
Correspondences for Semantic Interoperability*. It exists so that the paper's
central empirical claim can be checked rather than taken on trust:

> Every condition that a loadable instance can violate is exercised
> negatively, and the violated invariant is reported by name.
> — Sections 4.4 and 6.4

Each file in `tests/seeded/` is `scenario/scenario.xmi` with **exactly one**
edit. The unedited scenario validates clean, so every diagnostic below is
attributable to a single change. Reading time is about ten minutes; running
everything takes about twenty.

**Contents**

- [How to run a check](#how-to-run-a-check)
- [Index of the 21 seeded instances](#index-of-the-21-seeded-instances)
- [What each condition serves](#what-each-condition-serves)
- Conditions in detail: [structural](#structural-conditions-a) ·
  [provenance](#provenance-conditions-b) ·
  [corroboration](#corroboration-conditions-c) ·
  [trust lifecycle](#trust-lifecycle-t) ·
  [logical](#logical-conditions-d) ·
  [correspondence](#correspondence-conditions-e-g)
- [Why cascades happen](#why-cascades-happen)
- [The one condition Eclipse cannot decide](#the-one-condition-eclipse-cannot-decide)
- [Conditions with no seed](#conditions-with-no-seed)
- [Writing your own seed](#writing-your-own-seed)
- [Troubleshooting](#troubleshooting)

---

## How to run a check

**In Eclipse — the normative path.** Import the repository as a single
project, keeping the directory layout intact. Open a seed in the sample Ecore
editor, select the root element, then **Edit → Validate**.

No OCL document needs loading. Every instance resolves through its relative
`xsi:schemaLocation` to `metamodels/fame-delegates.ecore`, which carries the
invariants as EMF validation delegates.

Tested configuration: Eclipse Modeling Tools 2024-03, EMF 2.36, OCL 6.20.

**In Python — a zero-install cross-check.** Version 3.8 or later, no
dependencies.

```
python3 tools/fame_validate.py tests/seeded/C2_ContradictionCapsConfidence.xmi
python3 tests/run_tests.py          # the whole set -> 24 passed, 0 failed
```

`tools/fame_validate.py` re-implements every OCL invariant, and the suite
asserts that the two agree on which condition each seed violates. Where they
disagree, the OCL is normative. The Python tool additionally decides the two
conditions intra-resource OCL cannot; see
[below](#the-one-condition-eclipse-cannot-decide).

---

## Index of the 21 seeded instances

| # | File | Condition | In one line |
|---|---|---|---|
| 1 | `A1_UniqueIds.xmi` | A1 | two elements share an `id` |
| 2 | `A2_NonEmptyName.xmi` | A2 | an element has an empty `name` |
| 3 | `A3_LocalEndpoints.xmi` | A3 | a relationship points outside its own model |
| 4 | `A6_TypeConsistentSpecialization.xmi` | A6 | a `DataConcept` specialises a `Component` |
| 5 | `B1_EverythingJustified.xmi` | B1 | an element cites no evidence at all |
| 6 | `B2_BoundedConfidence.xmi` | B2 | confidence outside $[0,1]$ |
| 7 | `B3_PrimaryEvidence.xmi` | B3 | an element rests on corroboration alone |
| 8 | `C1_ConsistentCounting.xmi` | C1 | the count is below the artefacts actually cited |
| 9 | `C2_ContradictionCapsConfidence.xmi` | C2 | certainty claimed despite contradicting evidence |
| 10 | `C3_HighConfidenceNeedsCorroboration.xmi` | C3 | high confidence on a single source |
| 11 | `T1_TrustedMeansEvidenced.xmi` | T1 | `trusted` without the evidence to justify it |
| 12 | `D1_FormalHasExpression.xmi` | D1 | a formal constraint with no expression |
| 13 | `E1_RelatesTwoModels.xmi` | E1 | a correspondence model relating one model twice |
| 14 | `E2_AtLeastTwoEnds.xmi` | E2 | a correspondence with a single end |
| 15 | `E3_CrossesModels.xmi` | E3 | a correspondence entirely inside one model |
| 16 | `E4_DistinctEndpoints.xmi` | E4 | the same element used as two ends |
| 17 | `E5_EndpointsResolve.xmi` | E5 | an end naming an element that does not exist |
| 18 | `E6_DirectionMatchesKind.xmi` | E6 | a refinement that declares no direction |
| 19 | `E7_RolesIffDirected.xmi` | E7 | a directed end with no role |
| 20 | `E7b_NoRolesIfUndirected.xmi` | E7b | an undirected end carrying a role |
| 21 | `G1_InternalConsistency.xmi` | G1 | the same pair asserted equivalent and mismatched |

Instance 9 is the one shown in **Figure 3** of the paper.

---

## What each condition serves

The conditions are grouped by the trustworthiness criterion each
operationalises, and each criterion answers one level of the construction
uncertainty taxonomy (paper, Section 3).

| Family | Criterion | Uncertainty answered | Paper |
|---|---|---|---|
| **A**, **D** | schema-conformance | structural | §4.4 |
| **B** | provenance-traceability | evidential | §4.4 |
| **C** | multi-source corroboration | semantic | §4.4 |
| **T** | trust lifecycle | — (gates promotion) | §4.4, §5 |
| **E** | schema-conformance, correspondences | structural | §6.4 |
| **G** | correspondence-preservation | validation | §6.4, §7 |

This is the mapping to check if you are verifying that the artifact covers what
the paper claims: each row of the paper's Table 1 is discharged by the
corresponding family below.

---

Throughout, **Baseline** is the value in `scenario/scenario.xmi` and **Seeded**
is what the file under `tests/seeded/` contains. Python messages are quoted
verbatim. Eclipse reports each violation in the form
`The '<invariant>' constraint is violated on '<object>'`.

## Structural conditions (A)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `A1_UniqueIds.xmi` | `M1.authorisedBy` | `id="M1.authorisedBy"` | `id="M1.isFor"` | `A1_UniqueIds` on `M1` | `duplicate id 'M1.isFor'` |
| `A2_NonEmptyName.xmi` | `M1.LabRequest` | `name="LabRequest"` | `name=""` | `A2_NonEmptyName` on `M1.LabRequest` | `name is empty or absent` |
| `A3_LocalEndpoints.xmi` | `M1.isFor` | `target="M1.Patient"` | `target="M2.TestOrder"` | `A3_LocalEndpoints` on `M1` | `target 'M2.TestOrder' outside M1` |
| `A6_TypeConsistentSpecialization.xmi` | `M1.servedBy` | `kind="dataflow"` | `kind="specialization"` | `A6_TypeConsistentSpecialization` on `M1.servedBy` | `DataConcept specialises Component` |

**A3** is the condition that keeps system models self-contained: relationships
stay inside a model, and anything crossing a boundary must be a correspondence.
It is what makes the separation between the core and correspondence metamodels
load-bearing rather than stylistic.

**A6** rewards a pause. `M1.servedBy` runs from `M1.LabRequest`, a
`DataConcept`, to `M1.GPSystem`, a `Component`. As a dataflow that is
perfectly ordinary; as a specialisation it crosses metatypes, which A6 forbids.
The seed changes nothing but the `kind`.

## Provenance conditions (B)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `B1_EverythingJustified.xmi` | `M1.Patient` | 2 `justifiedBy` links | all removed | `B1_EverythingJustified`, `T1_TrustedMeansEvidenced` | `no ProvenanceLink` |
| `B2_BoundedConfidence.xmi` | `M1.Patient` | `confidence="0.97"` | `confidence="1.7"` | `B2_BoundedConfidence`, `T1_TrustedMeansEvidenced` | `confidence=1.7` |
| `B3_PrimaryEvidence.xmi` | `M1.Patient` | one link `role="primary"` | demoted to `corroborating` | `B3_PrimaryEvidence`, `T1_TrustedMeansEvidenced` | `no primary link` |

**B3** is the subtle one. The element still cites two artefacts, so B1 is
satisfied; what it lacks is a *primary* source. Corroboration without a primary
reading is agreement about nothing, and B3 rules it out.

## Corroboration conditions (C)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `C1_ConsistentCounting.xmi` | `M1.LabRequest` | `corroborationCount="3"` | `corroborationCount="1"` | `C1_ConsistentCounting`, `C3_HighConfidenceNeedsCorroboration` | `count=1 < 3 distinct artefacts` |
| `C2_ContradictionCapsConfidence.xmi` | `M2.ValidatedTest` | `confidence="0.86"` | `confidence="1.0"` | `C2_ContradictionCapsConfidence` | `contradicting link with confidence=1.0` |
| `C3_HighConfidenceNeedsCorroboration.xmi` | `M2.ValidatedTest` | `corroborationCount="2"` | `corroborationCount="1"` | `C3_HighConfidenceNeedsCorroboration` | `confidence=0.86 >= 0.8 but count=1` |

**C2 is the strongest single demonstration in the set, and the instance shown
in Figure 3.** `M2.ValidatedTest` carries a `contradicting` provenance link:
one artefact says the laboratory's validated test is a final reportable result,
another treats it as one message segment. The model may record that
disagreement. What it may not do is claim certainty in spite of it, and C2 is
the invariant that says so.

**C3** uses the corroboration threshold τ, fixed at **0.8** in this artifact
(`constraints/constraints.ocl`, line 6, and `TAU` in
`tools/fame_validate.py`). Calibrating τ is future work; the artifact commits
to a value so the condition is checkable.

## Trust lifecycle (T)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `T1_TrustedMeansEvidenced.xmi` | `M1.Practitioner` | `status="trusted"` with a primary link | primary link demoted to `corroborating` | `B3_PrimaryEvidence`, `T1_TrustedMeansEvidenced` | `status=trusted but B/C preconditions unmet` |

T1 is the gate the whole trust lifecycle rests on: nothing reaches
`status="trusted"` without satisfying B1–B3 and C1–C3. It is deliberately
redundant with those conditions, and the redundancy is the point.

## Logical conditions (D)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `D1_FormalHasExpression.xmi` | `M1.OnePatient` | `expression="every LabRequest isFor exactly 1 Patient"` | `expression=""`, `formal="true"` | `D1_FormalHasExpression` | `formal=true with empty expression` |

D1 is the *necessary typing condition* for D3 (satisfiability). D3 itself is
delegated to a solver this artifact does not include; see
[Conditions with no seed](#conditions-with-no-seed).

## Correspondence conditions (E, G)

| Seed | Element | Baseline | Seeded | Eclipse | Python |
|---|---|---|---|---|---|
| `E1_RelatesTwoModels.xmi` | correspondence model | `relates` M1, M2, M3 | second `relates` → `M1` | `E1_RelatesTwoModels` | `relates 3 model(s), refs=['M1','M1','M3']` **+ 5 × E5** |
| `E2_AtLeastTwoEnds.xmi` | `C3` | 2 ends | end `M2.LaboratoryTest` removed | `E2_AtLeastTwoEnds`, `E3_CrossesModels` | `1 end(s)` |
| `E3_CrossesModels.xmi` | `C5` | ends in M2 and M3 | `M3.Observation` → `M2.Specimen` | `E3_CrossesModels` | `all ends in one participating model` |
| `E4_DistinctEndpoints.xmi` | `C2` | ends `M1.LabRequest`, `M2.TestOrder`, `M2.Specimen` | third end → `M2.TestOrder` | `E4_DistinctEndpoints` | `an endpoint appears twice` |
| `E5_EndpointsResolve.xmi` | `C5` | end `M2.ValidatedTest` | end `M2.ValidatedTestX` | **nothing — see below** | `'M2.ValidatedTestX' not found in M2` |
| `E6_DirectionMatchesKind.xmi` | `C1` | `kind="mismatch"`, `directed="false"` | `kind="refinement"` | `E6_DirectionMatchesKind` | `kind=refinement but directed=False` |
| `E7_RolesIffDirected.xmi` | `C2` | first end `role="abstracted"` | role removed | `E7_RolesIffDirected` | `directed end without a role` |
| `E7b_NoRolesIfUndirected.xmi` | `C5` | undirected, no roles | first end given `role="abstracted"` | `E7b_NoRolesIfUndirected` | `undirected end carries a role` |
| `G1_InternalConsistency.xmi` | new `C6` | — | copy of `C5` with `kind="mismatch"` | `G1_InternalConsistency` | `same endpoints asserted equivalence and mismatch` |

**E6 and E7 are a pair**, and worth showing together. E6 says the *kind*
determines whether a correspondence is directed: `refinement`,
`generalization` and `dependency` are directed, the rest are not. E7 says the
ends must then carry roles exactly when the correspondence is directed. E7b is
the converse failure. Between them they make direction a property of the kind
rather than something an author may forget or invent.

**G1** is the condition that keeps a correspondence model from contradicting
itself: the same endpoint set cannot be both an equivalence and a mismatch. The
seed clones `C5` and relabels the clone, which is the smallest possible way to
produce the contradiction.

---

## Why cascades happen

Seven of the 21 seeds trip more than one condition. This is recorded rather
than engineered away, because engineering it away would misrepresent how the
conditions depend on one another.

| Seed | Cascade | Why |
|---|---|---|
| `B1`, `B2`, `B3` | → T1 | `M1.Patient` is `status="trusted"`, and T1 requires a trusted element to satisfy B1–B3 and C1–C3. The cascade is T1 doing its job. |
| `C1` | → C3 | Lowering the count to 1 leaves confidence at 0.92, above τ = 0.8, so C3 fails too. |
| `T1` | → B3 | T1 is violated *by way of* B3, since demoting the primary link is what makes the element unevidenced. |
| `E2` | → E3 | A correspondence reduced to one end trivially has all its ends in one model. |
| `E1` | → 5 × E5 | Collapsing M2 into M1 leaves every correspondence that pointed into M2 pointing into a model without those elements. |

14 of the 21 seeds are isolated. The per-seed breakdown is in
`tests/expected-violations.md`.

---

## The one condition Eclipse cannot decide

**`E5_EndpointsResolve.xmi` validates clean in Eclipse. This is expected.**

A `CorrespondenceEnd` names its endpoint by string (`elementRef`), not by
containment. Deciding whether that string resolves requires the participating
system model to be loaded and searched — something an OCL invariant evaluated
within a single resource cannot do. E5 therefore has no delegate and no
invariant in `constraints/constraints.ocl`. It is decided by the resolver:

```
python3 tools/fame_validate.py tests/seeded/E5_EndpointsResolve.xmi
```

```
FAIL tests/seeded/E5_EndpointsResolve.xmi  (1 violation(s))
  E5_EndpointsResolve   C5   'M2.ValidatedTestX' not found in M2
```

This is the most instructive moment in the artifact. Validate the file in
Eclipse, get a clean result, then run the resolver and get the unresolved
endpoint. It shows concretely why the framework needs a resolution step
*beside* schema validation rather than instead of it, which is the claim
Sections 6.4 and 6.5 of the paper make.

E7 is decided by the resolver for the same reason, though it also has a
delegate form that covers the intra-resource part.

---

## Conditions with no seed

Three conditions cannot be violated by a loadable instance, and three more are
not properties of instances at all.

| Condition | Why no seed exists |
|---|---|
| **A4** acyclic containment | Guaranteed by EMF containment semantics: an object has at most one container and cannot transitively contain itself. Not expressible as a violable instance; no invariant in the OCL. |
| **A5** owned attributes | Guaranteed by the Ecore structure: `attributes` is a containment feature declared only on `DataConcept`, so an `Attribute` has no other possible container. The invariant exists and is vacuously true. |
| **D2** constraints target existing elements | Guaranteed by EMF reference resolution. `ConstraintRule.constrains` is an `EReference` resolved by id, so an instance naming a non-existent target raises `UnresolvedReferenceException` at **load** time and never reaches validation. The invariant exists; no loadable instance can violate it. |
| **D3** satisfiability | Semi-formal. Delegated to a SAT/SMT solver this artifact does not include. Only the necessary typing condition D1 is checked. |
| **T2** monotone promotion | Constrains *operations*, not snapshots. Requires the update operations, which are not implemented here. |
| **G2** preservation under update | Constrains operations, as above. Theorem 1 covers the endpoint-preserving case analytically. |

Section 4.4 of the paper is qualified accordingly: every invariant a
**loadable** instance can violate is seeded, which excludes A4, A5 and D2.

### A note on seeding A1

`ModelElement.id` is declared `iD="true"`, so EMF resolves every cross-reference
through it. Duplicating the id of a *referenced* element — `M1.Patient`, say —
orphans every reference to it, and the instance fails to load rather than
failing to validate. The seed therefore duplicates the id of
`M1.authorisedBy`, a relationship no other element names, so the duplicate is
visible to A1 without breaking resolution.

The general rule, if you are writing seeds of your own: **mutations to
reference-valued features break loading; mutations to attributes do not.**

---

## Writing your own seed

Seeds are generated, not hand-edited, so that all 21 stay in step with
`scenario/scenario.xmi`. Add a mutation function and a `SEEDS` entry in
`tools/generate_seeds.py`:

```python
def s_A2(r):
    find_id(r, "M1.LabRequest").set("name", "")

SEEDS = [
    ...
    ("A2_NonEmptyName", s_A2, "M1.LabRequest name emptied"),
]
```

The mutation receives the parsed scenario root and edits it in place.
`find_id(root, "M1.LabRequest")` locates an element by id; `corr(root, "C2")`
and `cm(root)` reach a correspondence and the correspondence model.

Then regenerate and check:

```
python3 tools/generate_seeds.py     # -> 21 seeded instances written
python3 tests/run_tests.py
```

The script wipes `tests/seeded/*.xmi` before writing, so rerunning it is safe
and removes any stale file. Refresh the Eclipse project afterwards.

Two constraints on what a seed may do. It must produce a **loadable** instance,
so do not rename an id that something references, and do not point an
`EReference` at a non-existent target. And it should isolate one condition
where the conditions permit; where they do not, record the cascade in
`tests/expected-violations.md` rather than contriving around it.

---

## Troubleshooting

**`PackageNotFoundException: Package with uri 'http://fame/core/1.0' not
found`.** The instance is not where its relative `xsi:schemaLocation` expects
it to be. Instances locate the metamodel relatively — `../metamodels/` from
`scenario/`, `../../metamodels/` from `tests/seeded/` — so importing a
subfolder on its own, or moving a file between directories, breaks resolution.
Import the repository as a single project with the layout intact.

**`UnresolvedReferenceException` on a seed you just wrote.** The mutation
touched a reference-valued feature. See [the note on seeding
A1](#a-note-on-seeding-a1).

**Eclipse reports the old result after regenerating.** Eclipse validates its
cached workspace copy. Close the editor tab, select the project, press **F5**,
reopen.

**Every violation reported twice.** `constraints.ocl` has been loaded against
`fame-delegates.ecore`, so the invariants are present twice. Use one or the
other: validate through the delegates (the default), or retarget the instance's
`xsi:schemaLocation` at `metamodels/fame.ecore` and load the Complete OCL
document.

**A seed reports nothing at all.** If it is `E5_EndpointsResolve.xmi`, that is
[expected](#the-one-condition-eclipse-cannot-decide). Otherwise, check that the
instance's `xsi:schemaLocation` points at `fame-delegates.ecore` and not
`fame.ecore` — the latter carries no invariants.

---

## Where to go next

- `README.md` — layout, the two metamodel files, claim-to-file map, known gaps
- `constraints/constraints.ocl` — all conditions in one place, normative
- `tests/expected-violations.md` — per-seed violation table, including cascades
- `scenario/correspondences.corr` — the same correspondences in the textual
  notation of Section 6.3