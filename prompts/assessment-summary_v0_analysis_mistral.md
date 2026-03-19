# Audit: assessment-summary.md — Target Model: Mistral

## Sync Status
- MCP data fetched: yes, March 19, 2026
- Delta from training cutoff: `safe_prompt` flag confirmed as the canonical guardrailing mechanism; system prompt prepending for structured outputs uses `{{ json_schema }}` template; function calling uses OpenAI-compatible `tools` array. No breaking changes found.

## Audit Results

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Structural Integrity | FAIL | Non-standard `# Minimum outputs` section sits between `# Scope` and `# Instructions`, creating a fourth container for output rules alongside `# Instructions`, `# Output Format`, and `# Validation Checklist`. The section label doesn't match any canonical template section. |
| 2 | Instruction Density | FAIL | Grounding rule ("never invent data / state no results clearly") is restated verbatim across four sections: Objective, Minimum outputs, Instructions, and Validation Checklist. |
| 3 | Ambiguity Elimination | FAIL | "as appropriate" (Minimum outputs + Output Format) — no criteria for when to use list vs. table vs. inline; "directly support" (Instructions) — no decision rule; "relevant" (Minimum outputs) — no trigger condition; "Brief overview" — no length bound; "clearly" — used 3×, subjective. |
| 4 | Modular Layout | FAIL | Tool Use Policy section entirely absent despite 3 tool categories referenced. Scope has no Out-of-scope list. Examples and Runtime Context missing. |
| 5 | Model Capability Alignment | FAIL | Prompt manually encodes safety via repeated grounding rules — redundant when `safe_prompt: true` is set. Tabular outputs use ad-hoc Markdown rather than leveraging Mistral's native `response_format` structured output. No Tool Use Policy for the `tools` array binding. |

### Section Coverage

| Section | Status | Notes |
|---------|--------|-------|
| Role | PRESENT | |
| Objective | PRESENT | |
| Scope | INCOMPLETE | Out-of-scope list absent |
| Instructions | INCOMPLETE | Two lines; no decision logic, no negative constraints |
| Toolbox | MISSING | Agent references tools in 3 categories but has no use policy |
| Output Format | PRESENT | |
| Examples | MISSING | |
| Validation Checklist | PRESENT | |
| Special Considerations | MISSING | |
| Runtime Context | N/A | Tool outputs come through conversation roles, not system-prompt injection |

## Critical Flaws

1. **Missing Tool Use Policy** — The prompt explicitly names "findings-oriented tools," "asset tools," and "rule tools" but provides zero guidance on when to call each, sequencing, or failure handling. For Mistral function calling via the `tools` array, the system prompt is the only place to encode call policy. *Fix: add a dedicated Tool Use Policy section with sequencing, failure handling, and non-fabrication rule.*

2. **Non-standard `# Minimum outputs` section** — This block mixes output format specs with grounding rules, creating four separate containers for overlapping concerns. *Fix: dissolve it. Move the grounding rule to Instructions (one authoritative location), move the expected-output schema to Output Format.*

3. **Missing Out-of-scope list** — Without explicit out-of-scope items, Mistral has no boundary to refuse off-topic questions (remediation advice, historical trends, pricing). *Fix: add Out-of-scope bullets to Scope.*

4. **Ambiguous format trigger `"as appropriate"`** — Neither the model nor evaluators can verify compliance without discrete format-selection criteria. *Fix: replace with a format-decision table keyed to answer type (single metric → inline, ranked list → numbered list, multi-attribute → table, no data → literal string).*

5. **Redundant grounding rule** — The rule "rely on tool outputs, state no results clearly" appears in Objective, Minimum outputs, Instructions, and Validation Checklist. Repetition dilutes authority and wastes tokens. *Fix: one authoritative statement in Instructions; Validation Checklist retains a one-line verification.*

6. **No Examples** — No few-shot examples to demonstrate the "no results" path, which is the primary edge case for this agent. *Fix: add one positive and one empty-results example.*

## Citations
- [Guardrailing — Mistral AI Docs](https://docs.mistral.ai/capabilities/guardrailing) — `safe_prompt: true` prepends a canonical safety system prompt; manual safety preambles in the system prompt are redundant when this flag is set.
- [Custom Structured Output — Mistral AI Docs](https://docs.mistral.ai/capabilities/structured-output/custom_structured_output) — System prompt is automatically prepended with `Your output should be an instance of a JSON object following this schema: {{ json_schema }}` when `response_format` is used; native alternative to Markdown table instructions.
- [Function Calling — Mistral AI Docs (Pixtral reference)](https://docs.mistral.ai/models/pixtral-12b-24-09) — OpenAI-compatible `tools` array with `name`, `description`, `parameters`; `tool_choice: "auto"` or specific selection.
