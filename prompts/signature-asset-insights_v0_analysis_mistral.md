## Sync Status
- MCP data fetched: yes, 2026-03-19 (Mistral platform docs via context7 `/mistralai/platform-docs-public`)
- Delta from training cutoff: None. `safe_prompt`, `response_format`, and OpenAI-compatible `tools` array confirmed as current Mistral conventions.

---

## Audit Results

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Structural Integrity | FAIL | Non-canonical `# Minimum outputs` section between `# Scope` and `# Instructions`; no XML delimiting on any block |
| 2 | Instruction Density | FAIL | Grounding rule ("rely strictly on tool outputs… never hallucinate") duplicated in `# Minimum outputs` and `# Instructions`; Markdown rule duplicated in `# Minimum outputs` and `# Output Format` |
| 3 | Ambiguity Elimination | FAIL | `"Brief overview"`, `"Key insights"`, `"Main insights, recommendations, or highlights"`, `"when relevant to the user's question"`, `"as appropriate"` (×2), `"when the user asks for more"` — all subjective with no measurable inclusion conditions |
| 4 | Modular Layout | FAIL | Tool Use Policy MISSING; Examples MISSING; Special Considerations MISSING; Scope missing Out-of-Scope list; Runtime Context MISSING; spurious `# Minimum outputs` not in canonical template |
| 5 | Model Capability Alignment | FAIL | `query_insights` referenced with no invocation policy (Mistral function calling requires prompt-level policy); no Runtime Context XML injection guards [BASELINE] |

### Section Coverage

| Section | Status | Notes |
|---------|--------|-------|
| Role | PRESENT | |
| Objective | PRESENT | |
| Scope | INCOMPLETE | Missing Out-of-Scope list |
| Instructions | INCOMPLETE | No failure handling, no sequencing, grounding rule duplicated from `# Minimum outputs` |
| Toolbox | MISSING | `query_insights` named but no invocation policy or failure handling |
| Output Format | INCOMPLETE | Partially duplicated by non-canonical `# Minimum outputs` |
| Examples | MISSING | |
| Validation Checklist | PRESENT | |
| Special Considerations | MISSING | No terminology clarification for "Signature assets" vs. general assets; no guidance on "latest execution" ambiguity |
| Runtime Context | MISSING | No XML wrappers for user input or tool output injection |

---

## Critical Flaws

1. **Missing Tool Use Policy** — `query_insights` is named in Objective and Validation Checklist but has no invocation policy: no "call when" condition, no failure handling, and no non-fabrication rule. Mistral's `tools` array handles the schema at the API layer; the system prompt must govern invocation policy. [BASELINE]

2. **Non-canonical `# Minimum outputs` section** — Duplicates grounding (Instructions) and output structure (Output Format). "Minimum" framing softens hard constraints. Remove and redistribute.

3. **Duplicate grounding rules** — "Rely strictly on tool outputs / never hallucinate" verbatim in both `# Minimum outputs` and `# Instructions`. Single authoritative location required.

4. **Missing Out-of-Scope list** — No explicit guard against drifting into non-Signature asset analysis, remediation advice, or global assessment coverage the agent is not meant to provide.

5. **Missing Special Considerations** — "Signature assets" is domain-specific terminology that is never defined. "Latest execution" is ambiguous — what happens if multiple executions are available? The agent has no fallback behavior specified.

6. **Ambiguous output inclusion conditions** — `"when relevant to the user's question"`, `"when the user asks for more"` are not testable. Each output element needs an explicit, deterministic condition.

7. **Missing Runtime Context with injection guards** — User input and tool outputs at runtime are not wrapped in XML tags, creating a prompt injection risk. [BASELINE]

---

## Citations
- [Mistral Guardrailing Docs](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/guardrailing.mdx) — `safe_prompt` flag prepends system guardrail; avoid manual safety preambles
- [Mistral Function Calling](https://github.com/mistralai/platform-docs-public/blob/main/docs/getting-started/docs_introduction.md) — OpenAI-compatible `tools` array; invocation policy belongs in the system prompt
- [Mistral Structured Output](https://github.com/mistralai/platform-docs-public/blob/main/docs/capabilities/structured-output/custom.mdx) — native `response_format` + JSON schema available for output enforcement
