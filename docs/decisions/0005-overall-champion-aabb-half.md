# ADR 0005: Overall Champion determination by AABB half-game

## Status

Accepted.

## Context

ADR 0004 separated Policy strength governance into two formal families:

```text
Heuristic Champion
Learning Champion
```

and defined `Overall Champion` as a derived designation that requires cross-family formal evidence.

Family-internal promotion and cross-family Overall determination solve different problems. Heuristic and Learning research may use different development gates, diagnostics, sample sizes, or experiment-specific evidence while improving candidates inside each family. Requiring those internal loops to become identical would discard useful existing workflows without making the final cross-family claim more reliable.

The point at which architecture-independent comparability is required is the direct comparison between the current Heuristic Champion and current Learning Champion.

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

Overall superiority is determined only from the locked cross-family event. Family-internal development metrics do not carry into the Overall decision.

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

`Overall Champion` is derived from evidence for the exact pair of current family Champions.

If the predeclared rule clearly supports one side, that family Champion receives the current Overall designation.

If the result is `INCONCLUSIVE`, then for that current pair:

```text
Overall Champion: not established
```

When either family Champion changes, historical Overall evidence remains valid historical evidence but does not directly compare the new pair. The previous Overall designation is therefore not automatically inherited by the new family-Champion pair. A new cross-family formal event is required to establish a current Overall Champion.

`Overall Champion` still does not mean universal superiority across every ruleset, opponent distribution, metric, or environment, and it does not imply production adoption.

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
- Updating either family Champion invalidates automatic carry-forward of the current Overall designation, so Overall may temporarily return to `not established`.
- Exact sample size, rotation schedule, and statistical method remain Arena concerns rather than permanent project-wide architecture constants.
