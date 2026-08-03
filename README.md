# FAME artifact

Companion artifact for *Construction Before Alignment: Trustworthy Models and
Correspondences for Semantic Interoperability* (Ocansey, Lamo, Rutle, Rabbi).

It contains the two metamodels, the well-formedness conditions as OCL, the
running healthcare scenario of Section 2, the seeded-violation test set, and
tools that decide the conditions intra-resource OCL cannot.

There are two ways to check it, and they check the same conditions.

**In Eclipse — the normative path.** Open `scenario/scenario.xmi` in the sample
Ecore editor, select the root, and choose **Edit → Validate**. Nothing else is
needed: the instance points at `metamodels/fame-delegates.ecore`, which carries
every invariant as an EMF validation delegate. Every file in `tests/seeded/`
points there too, so each can be validated the same way and should report the
condition its filename names. This is the path that verifies what Sections 4.4
and 6.4 of the paper claim. Tested Eclipse/EMF/OCL versions are under "Running
the checks in Eclipse" below.

### The two metamodel files

They declare **the same metamodel**: identical classes, features and
enumerations. They differ only in where the constraints live.

| File | Constraints | Use it when |
|---|---|---|
| `fame.ecore` | none — they live in `constraints/constraints.ocl` | reading the metamodel on its own, or loading the Complete OCL document against it |
| `fame-delegates.ecore` | embedded as `eAnnotations` | validating instances with no setup (**what the instances point at**) |

`constraints.ocl` remains the authoritative statement of the conditions;
`fame-delegates.ecore` is **generated** from it by `tools/make_delegates.py`.
Edit the OCL and regenerate; do not edit the delegates file by hand.

**Do not load `constraints.ocl` against `fame-delegates.ecore`.** The
invariants would then be present twice and every violation would be reported
twice. Use one or the other:

- delegates only (default) — validate the instance directly;
- Complete OCL only — retarget the instance's `xsi:schemaLocation` at
  `fame.ecore`, then load `constraints.ocl`.

Both `.ecore` files declare the same `nsURI`s, so register only one of them in
the EPackage registry if you register either. Left unregistered, EMF resolves
each instance through its `xsi:schemaLocation` and the two coexist without
conflict.

**Without Eclipse — a zero-install cross-check.** Reviewers who would rather
not install Eclipse Modeling Tools can run the same conditions in Python 3.8+
with no dependencies:

```
python3 tests/run_tests.py
```

Expected: `25 passed, 0 failed`.

`tools/fame_validate.py` re-implements every OCL invariant, and the suite
asserts that the two agree on which condition each seeded instance violates. It
is a convenience and a cross-check, not a substitute: where the two disagree,
the OCL is right. It is also the *resolver* the paper refers to, since
conditions E5 and E7 need the participating models loaded and cannot be decided
by intra-resource OCL.

---

## Claim-to-file map

Every checkable claim in the paper, the file that carries it, and the command
that verifies it.

| § | Claim | File | Verify |
|---|---|---|---|
| 4.1–4.3 | Core metamodel: trust-bearing root, structural/behavioural layers, closed vs extensible value spaces | `metamodels/fame.ecore` (subpackage `core`) | open in Ecore editor |
| 4.4 | Conditions A1–A6, B1–B3, C1–C3, T1, D1–D2 as OCL invariants | `constraints/constraints.ocl` | `python3 tools/fame_validate.py scenario/scenario.xmi` |
| 4.4 | Every seedable invariant is rejected with its name reported | `tests/seeded/`, `tests/expected-violations.md` | `python3 tests/run_tests.py` |
| 4.4, Fig. 3 | C2 rejects `ValidatedTest` at confidence 1.0 with a contradicting link | `tests/seeded/C2_ContradictionCapsConfidence.xmi` | `python3 tools/fame_validate.py tests/seeded/C2_*.xmi` |
| 2, Fig. 1 | Three system models M1–M3 and correspondences C1–C5 | `scenario/scenario.xmi` | `python3 tools/fame_validate.py scenario/scenario.xmi` |
| 4.1, B1 | Every element cites resolvable evidence | `artefacts/`, `scenario/scenario.xmi` | `python3 tools/check_provenance.py scenario/scenario.xmi` |
| 6.1–6.2 | Correspondences are `ModelElement`s; n-ary ends; kind-labelled | `metamodels/fame.ecore` (subpackage `correspondence`) | open in Ecore editor |
| 6.3, Fig. 6 | Typed notation; checking a statement and validating the instance are the same judgement | `scenario/correspondences.corr`, `tools/corr_notation.py` | `python3 tools/corr_notation.py check scenario/correspondences.corr` |
| 6.3 | The notation and the metamodel instance agree | both of the above | `python3 tests/run_tests.py` (last two assertions) |
| 6.4 | Conditions E1–E7, G1 | `constraints/constraints.ocl` | `python3 tests/run_tests.py` |
| 6.4, 6.5 | E5 and E7 decided by the resolver | `tools/fame_validate.py` | `python3 tools/fame_validate.py tests/seeded/E5_*.xmi` |
| 6.5 | Invariants also available as EMF validation delegates | `metamodels/fame-delegates.ecore` | **unverified, see below** |

Two rows in Section 6.5 of the paper have **no** entry here, because the
machinery does not exist: the SAT/SMT solver for D3, and the update
operations for T2 and G2. See "Known gaps".

---

## Layout

```
metamodels/
  fame.ecore              core + correspondence metamodels (normative)
  fame-delegates.ecore    generated: same, with OCL validation delegates
constraints/
  constraints.ocl         all invariants, Complete OCL (normative)
scenario/
  scenario.xmi            M1, M2, M3 and the C1..C5 correspondence model
  correspondences.corr    the same correspondences in the Section 6.3 notation
artefacts/                synthetic source corpus cited by provenance links
tests/
  seeded/                 22 instances, one deliberate violation each
  expected-violations.md  what each seed violates, and what is not seedable
  run_tests.py            the suite
tools/
  fame_validate.py        reference validator + the resolver (E5, E7)
  corr_notation.py        notation parser, serialiser, XMI projection
  generate_seeds.py       regenerates tests/seeded/
  make_delegates.py       regenerates fame-delegates.ecore
  check_provenance.py     B1 resolvability check
figures/                  core-mm and corr-mm, vector versions
```

---

## Running the checks in Eclipse

Tested configuration: Eclipse Modeling Tools 2024-03, EMF 2.36, OCL 6.20.

1. Import the repository as an existing project.
2. Open `scenario/scenario.xmi` in the sample Ecore editor.
3. Select the root, then **Edit → Validate**. Expect no diagnostics.
4. Repeat on any file in `tests/seeded/`. Expect the diagnostic to name the
   invariant matching the filename, as listed in
   `tests/expected-violations.md`.

No OCL document needs loading: the instances resolve to
`metamodels/fame-delegates.ecore`, which carries the invariants.

### Reading the conditions instead

To read all conditions in one place, or to check the delegates against their
source, open `constraints/constraints.ocl`. To validate through it rather than
through the delegates, retarget the instance's `xsi:schemaLocation` at
`metamodels/fame.ecore` and load the document with **OCL → Load Document**.
Do not do both at once (see "The two metamodel files" above).

### Regenerating the delegates

`fame-delegates.ecore` is generated. After editing `constraints.ocl`:

```
python3 tools/make_delegates.py
```

Then reopen an instance and validate. If an invariant fails to parse as a
delegate, fix it in `constraints.ocl` and regenerate; the delegate form imposes
slightly tighter syntax than a Complete OCL document, and the `allElements()`
helper is inlined into A1 and A3 by the generator.

---

## The Python tools

`tools/fame_validate.py` re-implements every OCL invariant so the test set can
be checked without Eclipse. The OCL document stays normative; the Python is a
cross-check, and `tests/run_tests.py` asserts the two agree on which condition
each seed violates.

It also *is* the resolver referred to in Sections 6.4 and 6.5. Conditions E5
(endpoint references resolve) and E7 (roles iff directed) need the
participating system models loaded, which intra-resource OCL cannot do. They
are decided here.

`tools/corr_notation.py` parses the Section 6.3 grammar, enforces its typing
rules at parse time, and projects an XMI correspondence model back onto the
notation. The round-trip is asserted by the test suite.

The notation is a **projection**, not a bijection: it carries kind, ends,
roles, compatibility conditions, evidence, confidence and status, but not
`name`, `corroborationCount`, or a condition's `formal` flag. The paper's
claim that each notation clause maps onto one metamodel construct holds in
that direction; it does not claim the reverse, and the reverse is false.

---

## Known gaps

Stated plainly, because parts of the framework described in the paper are
not implemented here.

1. **No SAT/SMT solver.** Condition D3 (satisfiability of a model's formal
   constraint rules) is not decided anywhere in this artifact. Only the
   necessary typing condition D1 is checked.
2. **No update operations.** Conditions T2 (monotone promotion) and G2
   (preservation under update) are properties of operations, not snapshots.
   Neither is implemented, so Theorem 1 is not exercised computationally.
3. **No construction pipeline.** The LLM-based construction, shortlisting,
   validation and repair loop of Section 5 is not in this artifact. It
   belongs to the enabling paper and should be a separate archived record,
   cross-linked from this one.
4. **Provenance resolvability is not provenance faithfulness.**
   `check_provenance.py` confirms that a cited artefact exists. It cannot
   confirm that the artefact supports the element citing it, and nothing
   here calibrates the `confidence` values, which are authored. This is the
   framework's main open problem and should be stated as such in the paper.
5. **The artefact corpus is synthetic.** Written for this artifact, not
   extracted from real clinical systems.

---

## Reproducing derived files

```
python3 tools/generate_seeds.py                    # tests/seeded/
python3 tools/make_delegates.py                    # metamodels/fame-delegates.ecore
python3 tools/corr_notation.py fromxmi scenario/scenario.xmi > scenario/correspondences.corr
```

## Licence

Models, constraints, scenario, artefact corpus and documentation:
CC-BY-4.0. Tools under `tools/` and `tests/`: Apache-2.0. See `LICENSE`.
