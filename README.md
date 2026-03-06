# IdeaArena (n8n VC Incubation Tournament)

IdeaArena is an n8n workflow system that runs a head-to-head startup incubation competition between two intentionally different autonomous teams.

- Team A: `inventor_lab` (breakthrough-first, science-push, category creation)
- Team B: `operator_forge` (market-gap, distribution realism, execution focus)

Each team generates and hardens one final concept packet. `Entrepreneur Arena` judges both packets, applies confidence gates and deterministic tie-break logic, and logs results to Google Sheets.

## Repository structure

- `Entrepreneur Arena`: orchestrator workflow
- `Team A`: Team A pipeline export (no `.json` extension)
- `Team B`: Team B pipeline export (no `.json` extension)

## v2.1 highlights

- Distinct team brains with different ideation and ranking prompts.
- Persona engine with rotating non-overlapping persona pairs (1 named + 1 archetype per team).
- Signal scout stage before ideation (science, trend/pattern, competition signals).
- Hybrid dedup (normalized key + semantic similarity heuristic).
- Deterministic mutation selection using run seed.
- Neutralize-persona stage before final packet output.
- Arena confidence gate (`NO_WINNER_LOW_CONFIDENCE`) and deterministic tie-break chain.
- Credential IDs removed from workflow exports.

## Workflow overview

### Arena orchestration (`Entrepreneur Arena`)

1. `Manual Trigger`
2. `Set Arena Config` (code node): initializes run seed, constraints, rubric, persona pool, and team configs.
3. `Prepare Team A Context` / `Prepare Team B Context`: inject team-specific strategy + personas.
4. `Execute Team A Runner` / `Execute Team B Runner`
5. `Merge Team Packets1`
6. `Arena Judge (LLM)`
7. `Parse & Gate Verdict1`:
   - strict parse
   - confidence threshold gate
   - deterministic tie-break (`feasibility -> originality -> distribution -> market -> confidence -> stable default`)
8. `Winner Packager`
9. `Prep Winners Row1`
10. `Write to Winners1`
11. `Log Success`

### Team pipeline (`Team A` and `Team B`)

1. Load prior ideas from `Idea Bank`.
2. `Signal Scout1` gathers emerging signals.
3. `Parse Signal Theses1` validates and normalizes signal theses.
4. Dual-model ideation with team-specific prompts and persona lenses.
5. Compile + hybrid dedup.
6. Write novel ideas to `Idea Bank`.
7. Research phase with web tools.
8. Parse and validate research payload.
9. Team-specific ranking.
10. `Parse Ranked Winner1` for strict winner extraction.
11. Concept build, critique, repair, deterministic mutation, VC attack/response.
12. `Neutralize Persona Voice1` removes persona-style language.
13. Final packet compilation with extended metadata fields.
14. `Prep Final Concept Row` strict parse and normalization.
15. `Write to Final Concepts`.

## Extended contracts

### Team packet additions

- `teamStrategy` (`inventor_lab|operator_forge`)
- `personaSet` (JSON string in sheet row)
- `personaRationale`
- `signalTheses` (JSON string in sheet row)
- `noveltyScore`
- `confidenceScore`
- `horizonBucket`
- `evidenceCoverage`
- `decisionRiskFlags` (JSON string in sheet row)
- `parseStatus`

### Arena verdict additions

- `decisionStatus` (`WINNER_SELECTED|NO_WINNER_LOW_CONFIDENCE|INVALID_VERDICT`)
- `tieBreakTrace`
- `confidenceGateReason`
- `teamAConfidence`
- `teamBConfidence`

## Setup checklist

- Import all three workflows into n8n.
- Configure credentials:
  - Google Sheets OAuth2
  - OpenRouter
  - Brave Search
- Configure workflow IDs in environment variables (recommended):
  - `TEAM_A_WORKFLOW_ID`
  - `TEAM_B_WORKFLOW_ID`
- Ensure Google Sheet tabs exist:
  - `Idea Bank`
  - `Final Concepts`
  - `Winners`
- Ensure `Execute Team A Runner` and `Execute Team B Runner` can resolve the imported team workflow IDs.

## Suggested validation run

1. Trigger `Entrepreneur Arena` manually.
2. Confirm `Set Arena Config` emits team contexts with personas.
3. Confirm each team executes `Signal Scout1` and logs one final concept row.
4. Confirm `Winners` row includes `decisionStatus`, confidence fields, and tie-break trace when applicable.
5. Re-run with same seed context and compare deterministic mutation behavior.
