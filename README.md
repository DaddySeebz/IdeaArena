# IdeaArena (n8n VC Incubation Tournament)

IdeaArena is an n8n workflow system that runs a **head-to-head startup incubation competition** between two autonomous VC-style teams (`Team A` and `Team B`).

Each team independently generates, researches, stress-tests, and hardens ideas. They each submit one final concept packet, then the `Entrepreneur Arena` workflow judges both submissions, selects a winner, and logs results to Google Sheets.

## Repository structure

- `Entrepreneur Arena` – Orchestrator workflow that runs both teams, judges outcomes, packages the winner, and logs winners/errors.
- `Team A` – Team workflow for ideation through final packet submission.
- `Team B` – Parallel team workflow with the same pipeline shape.

> These files are exported n8n workflow JSON (stored without `.json` extension).

## How the system works

## 1) Arena orchestration (`Entrepreneur Arena`)

1. **Manual trigger** starts a run.
2. **Set Arena Config** initializes run context.
3. **Execute Team A Runner** and **Execute Team B Runner** call each team workflow.
4. Team outputs are merged in **Merge Team Packets1**.
5. **Arena Judge (LLM)** applies a weighted rubric (market, originality, feasibility, differentiation, distribution) and returns scores + winner.
6. **Winner Packager** creates an executive summary for the winning concept.
7. **Prep Winners Row1** normalizes/parses outputs into a single logging row.
8. **Write to Winners1** appends the winner record to Google Sheets (`Winners` tab).
9. **Log Success** / **Workflow Error Trigger + Log Error** provide observability.

## 2) Team pipeline (`Team A` and `Team B`)

Each team runs the following sequence:

1. **Read & load prior ideas** from `Idea Bank` in Google Sheets.
2. **Dual-model ideation** (`Generate Ideas — Model A/B`) to produce diverse options while avoiding duplicate summaries.
3. **Compile + dedupe/normalize** generated ideas.
4. **Append novel ideas** back into `Idea Bank` (memory across runs).
5. **Research phase** with web search tooling + LLM synthesis.
6. **Ranking phase** chooses the best candidate idea using weighted constraints (including buildability under 6 months / $25k).
7. **Concept expansion** into structured startup packet fields.
8. **Self-critique + repair loop** to fix weaknesses and fatal assumptions.
9. **Mutation step** (e.g., ICP/pricing/channel/scope/keyFeature) to probe for stronger variants.
10. **VC assassination + response** adversarial challenge and defense of the concept.
11. **Finalize packet** with rubric scoring and summary.
12. **Write to Final Concepts** sheet and return packet to arena orchestrator.

## Data outputs

The workflows use a single Google Sheet document with (at least) these tabs:

- **Idea Bank** – deduplicated historical idea memory (`normalizedKey`, `oneLiner`, metadata).
- **Final Concepts** – team-level final packets per run.
- **Winners** – arena-level winning concept + score breakdown + summary/justification.

## What is already strong

- Multi-stage adversarial refinement (critic → repair → VC attack → defense).
- Persistent memory via Idea Bank to reduce repeated concepts.
- Explicit scoring rubric at both team and arena levels.
- Run-level IDs and timestamps to improve traceability.

## Improvement opportunities

1. **Version and lock prompt contracts**
   - Centralize expected JSON schemas and validate every LLM output before downstream parsing.
   - Add explicit fallback/retry when parse fails (currently parsing is regex-based and permissive).

2. **Harden structured output handling**
   - Use stricter n8n JSON/structured-output nodes (or schema-enforced parser code) instead of `match(/\{[\s\S]*\}/)` extraction.
   - Log malformed payloads into a dedicated sheet for debugging.

3. **Introduce deterministic tie-breaking**
   - If `teamAScore === teamBScore`, add explicit tie-break policy (e.g., higher feasibility wins; then originality).

4. **Calibrate randomness for reproducibility**
   - Mutation selection is random; consider storing seed/config so runs are replayable and experiments are comparable.

5. **Separate credentials and IDs from export docs**
   - Keep README instructions for required credentials (`OpenRouter`, `Google Sheets`, `Brave Search`) and avoid hardcoding IDs in shared exports when possible.

6. **Add basic quality metrics dashboard**
   - Track per-run novelty score, parse-failure count, winner distribution, and idea reuse rate over time.

7. **Create a quickstart for local import**
   - Document exact import order and required n8n community nodes/plugins.

## Suggested operating procedure

1. Trigger `Entrepreneur Arena` manually.
2. Confirm both team executions return packet outputs.
3. Review `Winners` row for score consistency and rubric rationale.
4. Periodically inspect `Idea Bank` for quality drift and prune low-value ideas.

## Quick setup checklist

- n8n instance with workflows imported.
- Credentials configured:
  - Google Sheets OAuth2
  - OpenRouter chat model credentials
  - Brave Search tool credentials/node
- Shared Google Sheet with expected tabs/columns.
- Workflow IDs in `Execute Team A Runner` and `Execute Team B Runner` pointed to the imported team workflows.

---

If you want, the next improvement pass can include a **schema-validation node set** + **retry policy design** directly in these workflow JSON files so malformed model output can never silently pass downstream.
