## Sync Status
- MCP data fetched: yes, 2026-03-19 (Mistral platform docs via context7 `/mistralai/platform-docs-public`)
- Delta from training cutoff: None. `safe_prompt` flag, `response_format` JSON schema enforcement, and OpenAI-compatible `tools` array for function calling confirmed as current Mistral conventions.

---

## Audit Results

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Structural Integrity | FAIL | Non-canonical `# Minimum outputs` section between `# Scope` and `# Instructions`; plain-text blocks not XML-delimited |
| 2 | Instruction Density | FAIL | Grounding rule ("rely strictly on tool outputs… never hallucinate") duplicated verbatim in `# Minimum outputs` and `# Instructions`; Markdown/structure rule duplicated in `# Minimum outputs` and `# Output Format` |
| 3 | Ambiguity Elimination | FAIL | `"as appropriate"` (×2), `"Brief overview"`, `"Key findings or violations"`, `"when relevant to the user's question"`, `"when useful"` — all subjective with no measurable conditions |
| 4 | Modular Layout | FAIL | Tool Use Policy MISSING; Examples MISSING; Special Considerations MISSING; Scope missing Out-of-Scope list; Runtime Context MISSING; spurious `# Minimum outputs` not in canonical template |
| 5 | Model Capability Alignment | FAIL | `search_assets_scope` referenced without invocation policy (Mistral function calling requires prompt-level policy); no Runtime Context XML guards against injection [BASELINE] |

### Section Coverage

| Section | Status | Notes |
|---------|--------|-------|
| Role | PRESENT | |
| Objective | PRESENT | |
| Scope | INCOMPLETE | Missing Out-of-Scope list |
| Instructions | INCOMPLETE | No failure handling; no negative constraints; grounding rule duplicated from `# Minimum outputs` |
| Toolbox | MISSING | `search_assets_scope` mentioned but no invocation policy, sequencing, or failure handling |
| Output Format | INCOMPLETE | Partially duplicated by non-canonical `# Minimum outputs` |
| Examples | MISSING | |
| Validation Checklist | PRESENT | |
| Special Considerations | MISSING | No rule identifier terminology normalization; no disambiguation guidance for name collisions |
| Runtime Context | MISSING | No XML wrappers for user input or tool output injection |

---

## Critical Flaws

1. **Missing Tool Use Policy** — `search_assets_scope` is referenced conditionally in Instructions and Validation Checklist but has no invocation policy. The prompt also references unspecified rule query tools with no policy at all. Mistral's native function calling (`tools` array) handles the schema at the API layer; the system prompt must define when to call each tool, sequencing, failure handling, and the non-fabrication rule. [BASELINE]

2. **Non-canonical `# Minimum outputs` section** — Duplicates grounding rules (belonging in Instructions) and output structure (belonging in Output Format). The "minimum" label softens hard constraints. Remove and redistribute.

3. **Duplicate grounding rules** — "Rely strictly on tool outputs / never hallucinate" stated identically in both `# Minimum outputs` and `# Instructions`. Single authoritative location required to prevent drift.

4. **Missing Out-of-Scope list** — No guard against the agent drifting into remediation guidance, compliance scoring, or asset-only analysis not tied to a rule target.

5. **Missing Special Considerations** — Rule identifiers ("rule ID", "rule name", descriptive constraint) are never normalized. No disambiguation guidance when a rule name matches multiple rules — the agent has no fallback behavior specified.

6. **Missing Runtime Context with injection guards** — User-provided rule targets and tool outputs injected at runtime are not wrapped in XML tags. Undelimited user input can blend with instructions. [BASELINE]

7. **Ambiguous output inclusion conditions** — `"when relevant to the user's question"`, `"when useful"`, `"when multiple"` give the model no deterministic trigger. Each output element needs an explicit, testable condition.

---

## Citations
- [Mistral Guardrailing Docs](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/guardrailing.mdx) — `safe_prompt` flag prepends system guardrail; avoid manual safety preambles that duplicate it
- [Mistral Function Calling](https://github.com/mistralai/platform-docs-public/blob/main/docs/getting-started/docs_introduction.md) — OpenAI-compatible `tools` array; invocation policy belongs in the system prompt
- [Mistral Structured Output](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/structured-output/custom.mdx) — native `response_format` + JSON schema available for enforcing output structure
