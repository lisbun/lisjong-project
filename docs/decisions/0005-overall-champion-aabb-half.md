# ADR 0005: Overall Champion determination by AABB half-game

## Status

Accepted.

## Context

ADR 0004 separated Policy strength governance into two formal families:

```text
Heuristic Champion
Learning Champion
```

and defined `Overall Champion` as a derived designation whose winning family is established from cross-family formal evidence.

Family-internal promotion and cross-family Overall determination solve different problems. Heuristic and Learning research may use different development gates, diagnostics, sample sizes, or experiment-specific evidence while improving candidates inside each family. Requiring those internal loops to become identical would discard useful existing workflows without making the final cross-family claim more reliable.

Architecture-independent comparability is required when Overall is first established, when the winning family may change, or when a changed opposing family Champion must be evaluated. Those cases require a direct comparison between the current Heuristic Champion and current Learning Champion. This ADR also defines the narrower governance case in which the already-winning family may replace its own representative without claiming a new direct cross-family statistical comparison.

For this final layer, single-round evidence is cheaper but does not capture all longer-horizon effects that matter to match strength, including placement, score carry-over, dealer continuation, honba, and riichi-stick state across rounds. The ecosystem already has a generic Arena comparison contract whose normal game mode is `4p-red-half` and which can realize symmetric AABB seat exposure through deterministic rotation.

## Decision

### Preserve family-internal promotion

This ADR does not redesign or unify Heuristic / Learning family-internal promotion.

```text
Heuristic candidate
    -> existing Heuristic family promotion
    -> Heuristic Champion

Learning candidate
    -> existing Learning family promotion
    -> Learning Champion
```

Historical family-promotion evidence is not reinterpreted by this decision.

### Overall participants

An Overall determination event compares exactly the current family Champions:

```text
A = current Heuristic Champion
B = current Learning Champion
```

If either family has `Champion: not established`, `Overall Champion` remains `not established`. The other family Champion is not promoted to Overall by absence of an opponent.

### Canonical matchup shape

Cross-family Overall determination uses half-game evaluation:

```text
game mode = 4p-red-half
matchup   = AABB
```

Every evaluated hanchan contains two seats from the current Heuristic Champion and two seats from the current Learning Champion. Across the locked evaluation population, the seat-assignment plan must be symmetric: A and B receive equal total exposure to each seat position.

The exact ordered seed population, number of rotations per seed, and concrete rotation order are **Arena-owned purpose-specific protocol details**, not project-wide invariants. Arena's existing generic `ComparisonPlan` / comparison semantics should be reused where they satisfy the AABB half-game and seat-symmetry contract. Its current four cyclic rotations are a natural implementation candidate, but this ADR does not freeze that concrete rotation schedule into project architecture.

A second equivalent AABB runner should not be created merely for Champion governance.

### Evidence boundary

DIRECT Overall superiority is determined only from the locked cross-family event. Family-internal development metrics do not become cross-family superiority evidence.

```text
family Champion status
    !=
Overall superiority evidence
```

The formal event must bind at least:

- exact Heuristic Champion identity;
- exact Learning Champion identity;
- execution software / runtime / dependency provenance;
- pre-result seed population and sample size;
- pre-result seat-assignment / rotation plan;
- pre-result primary statistic and classification rule;
- strict-read or immutable evaluation artifact;
- fail-closed handling of partial or invalid execution.

After result exposure, seed extension, replacement population, rotation-plan replacement, rerun rescue, or criterion changes are not permitted within the same Overall event.

The same statistical criterion must be applied to both families. Architecture-specific favorable thresholds or metrics are forbidden.

### Arena-owned concrete statistics and execution plan

Project-wide architecture fixes the AABB half-game comparison shape and symmetric seat exposure, not the concrete statistical or rotation design of every Overall event.

The purpose-specific Arena contract owns and locks before execution:

```text
ordered seeds
sample size / precision target
rotation count and concrete seat-assignment order
primary statistic
confidence interval or test method
classification threshold
artifact schema
parallel execution details
```

These values may evolve between future protocol revisions, but cannot be changed after exposure of a given event's result.

### Overall designation lifecycle

`Overall Champion` is a derived project-governance designation. Its current record must distinguish two modes.

```text
DIRECT
    canonical cross-family formal event directly compared
    the exact current Heuristic / Learning Champion pair

INHERITED
    current Overall-winning family validly promoted its own
    exact Champion while the opposing family Champion remained unchanged
```

A `DIRECT` designation is created only when the locked cross-family event for the exact current pair supports one side under the predeclared classification rule.

If the result is `INCONCLUSIVE`, then for that current pair:

```text
Overall Champion: not established
```

An `INHERITED` designation is permitted only when all of the following hold:

1. a current Overall Champion already exists and its winning family is known;
2. that same winning family validly promotes an exact successor Champion through its accepted family-internal promotion process;
3. the opposing family Champion is exactly unchanged from the direct cross-family basis;
4. the predecessor / successor relationship and exact identities are unambiguous;
5. the family-promotion evidence and the current direct Overall basis remain valid;
6. no implementation substitution, stale provenance, withdrawn evidence, or other explicit invalidation exists.

When those conditions hold:

```text
Overall Champion = new Champion of the already-winning family
designation mode = INHERITED
```

The rule is symmetric. It applies equally when Heuristic is the current Overall-winning family and when Learning is the current Overall-winning family.

Winning-family inheritance is **project-governance succession, not a statistical-transitivity claim**. In particular, this ADR does not claim:

```text
H2 > H1
H1 > L1
therefore H2 > L1 is mathematically proven
```

Mahjong Policy strength may be non-transitive, and family-internal promotion protocols may differ from the Overall protocol. An inherited Overall Champion therefore need not have direct cross-family evidence against the unchanged opponent. The designation instead means that the already-winning family retained the project-level Overall role while replacing its own representative through an accepted promotion.

If the current Overall-losing family Champion changes, the old direct basis does not compare the winning-family representative against the new opponent. The previous Overall designation is not inherited across that change:

```text
H1 > L1
L1 -> L2
=> fresh cross-family Overall event required
```

If both family Champions change before a fresh direct Overall event, a fresh cross-family event is also required. This remains true even if a winning-family promotion temporarily produced an `INHERITED` designation before the opposing family changed.

Historical direct evidence remains valid historical evidence for its exact locked pair. Inheritance must not rewrite or relabel that evidence.

Every current Overall record must preserve enough provenance to distinguish:

- current Overall Champion exact identity;
- winning family;
- designation mode: `DIRECT` or `INHERITED`;
- direct cross-family basis: exact pair, Arena event / protocol, and result identity;
- for `INHERITED`: predecessor Overall Champion, exact successor identity, family-promotion evidence, and confirmation that the opposing Champion is unchanged.

If any required identity, predecessor/successor relation, promotion validity, or direct-basis validity is ambiguous, fail closed:

```text
Overall Champion: not established for the current pair
fresh cross-family Overall event required
```

`Overall Champion` still does not mean universal superiority across every ruleset, opponent distribution, metric, or environment, and it does not imply production adoption.

### Current-state migration

The first canonical Overall event completed in `lisbun/lisjong-arena#281` remains unchanged historical evidence.

```text
Heuristic Champion
    targeted-honor-release-terminal-progression

Learning Champion
    exact #211 Arm Y

classification
    HEURISTIC CHAMPION SUPERIOR

result identity
    d9beb3f837684f9e29426d60d1c1f1d6ead77ee5b89a0010729627641a17810f
```

Therefore the current Overall designation remains:

```text
Overall Champion
    targeted-honor-release-terminal-progression

winning family
    Heuristic

designation mode
    DIRECT

direct basis
    lisbun/lisjong-arena#281
```

A future valid Heuristic Champion promotion may inherit this designation only while the current Learning Champion remains the same exact #211 Arm Y identity and the other inheritance conditions above continue to hold.

## Repository responsibility

- `lisjong-project` owns the cross-family semantics described by this ADR.
- `lisjong-arena` owns the concrete Overall evaluation protocol, seed / rotation plan, execution, statistics, artifact, and provenance checks.
- `lisjong` continues to own stable Policy identity / current role according to existing repository boundaries.

Champion registry / metadata placement remains a separate follow-up concern and is not decided here.

## Consequences

- Existing Heuristic and Learning development loops can continue without forced protocol unification.
- Overall claims use a common, architecture-independent high-fidelity AABB half-game matchup.
- Family Champions can be selected cheaply or purpose-specifically while the final cross-family claim is based on half-game performance.
- Seat exposure must be symmetric across the locked Overall evaluation population, while the exact rotation schedule remains an Arena contract.
- Inconclusive evidence remains a valid terminal result instead of being rescued by post-result sampling.
- A valid promotion inside the current Overall-winning family may carry the designation forward as `INHERITED` while the opposing Champion is unchanged; a losing-family change or both-family change requires fresh direct cross-family evidence.
- Exact sample size, rotation schedule, and statistical method remain Arena concerns rather than permanent project-wide architecture constants.
