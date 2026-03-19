## Sync Status
- MCP data fetched: yes, 2026-03-19 (Mistral platform docs via context7 `/mistralai/platform-docs-public`)
- Delta from training cutoff: No significant changes found. `safe_prompt` flag, `response_format` JSON schema enforcement, and OpenAI-compatible `tools` array for function calling confirmed as current Mistral conventions. No new system prompt engineering patterns identified beyond baseline.

---

## Audit Results

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Structural Integrity | FAIL | Non-canonical `# Minimum outputs` section sits between `# Scope` and `# Instructions`, breaking canonical section flow; plain-text instruction blocks lack XML disambiguation |
| 2 | Instruction Density | FAIL | Grounding rule ("rely strictly on tool outputs… state no results clearly") stated identically in both `# Minimum outputs` and `# Instructions`; output format rules duplicated in both `# Minimum outputs` and `# Output Format` |
| 3 | Ambiguity Elimination | FAIL | `"as appropriate"` (×2), `"Brief overview"`, `"Key findings"`, `"when relevant to the user's question"` — all subjective with no measurable criteria |
| 4 | Modular Layout | FAIL | Toolbox/Tool Use Policy MISSING; Examples MISSING; Scope missing Out-of-Scope list; spurious `# Minimum outputs` not in canonical template |
| 5 | Model Capability Alignment | FAIL | `search_assets_scope` referenced inline with no tool invocation policy (Mistral function calling requires policy in prompt); no Runtime Context XML guards against injection |

### Section Coverage

| Section | Status | Notes |
|---------|--------|-------|
| Role | PRESENT | |
| Objective | PRESENT | |
| Scope | INCOMPLETE | Missing Out-of-Scope list |
| Instructions | INCOMPLETE | No failure handling, no negative constraints; grounding rule duplicated from `# Minimum outputs` |
| Toolbox | MISSING | `search_assets_scope` referenced but no invocation policy, sequencing, or failure handling |
| Output Format | INCOMPLETE | Partially duplicated by non-canonical `# Minimum outputs` |
| Examples | MISSING | |
| Validation Checklist | PRESENT | |
| Special Considerations | PRESENT | |
| Runtime Context | MISSING | No XML wrappers for user input or tool output injection |

---

## Critical Flaws

1. **Missing Tool Use Policy** — `search_assets_scope` is referenced in Instructions and Validation Checklist but has no accompanying policy. Mistral's native function calling (`tools` array at the API layer) defines the schema; the system prompt must define *when* to call the tool, *sequencing*, *what to do on failure*, and *never to fabricate outputs*. None of this is present. [BASELINE]

2. **Non-canonical `# Minimum outputs` section** — This section sits outside the 10-section template and duplicates content belonging in Instructions (grounding rules) and Output Format (response schema). Remove it and redistribute. The label also softens what are actually hard constraints ("minimum" implies optional floor rather than required rules).

3. **Duplicate grounding rules** — "Rely strictly on tool outputs; never invent or assume data" appears in both `# Minimum outputs` and `# Instructions`. Two copies create drift risk if one is updated without the other.

4. **Ambiguous qualifiers without measurable triggers** — `"as appropriate"` (×2), `"Brief overview"`, `"Key findings"`, `"when relevant to the user's question"` give the model no deterministic rule for when to include or omit output elements. Replace with explicit inclusion conditions (e.g., "include findings table when finding count > 0", "include severity breakdown only when the user explicitly requests it").

5. **Missing Out-of-Scope list** — Without explicit out-of-scope constraints, the agent has no guard against drifting into remediation recommendations, compliance scoring, or other adjacent domains it should not address.

6. **Missing Runtime Context with injection guards** — User-provided asset identifiers and tool outputs injected at runtime are not wrapped in XML tags. For Mistral (and all models), this is a security gap — undelimited user input can blend with instructions. [BASELINE]

---

## Citations
- [Mistral Guardrailing Docs](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/guardrailing.mdx) — `safe_prompt` flag prepends system guardrail; avoid manual safety preambles that duplicate it
- [Mistral Structured Output](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/structured-output/custom.mdx) — native `response_format` + JSON schema available; system prompt schema prepend confirmed as default pattern
- [Mistral Function Calling](https://github.com/mistralai/platform-docs-public/blob/main/docs/getting-started/docs_introduction.md) — OpenAI-compatible `tools` array; policy for invocation belongs in the system prompt
