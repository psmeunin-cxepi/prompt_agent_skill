# AGENTS.md

Project-level context for AI coding agents (Claude Code, OpenAI Codex, GitHub Copilot, etc.).

## What this repo is

A prompt engineering workbench for auditing, refactoring, and versioning AI agent system prompts.
It contains:
- A reusable **Prompt Auditor** agent skill
- A canonical **system prompt template and guide**
- A **prompts library** with versioned system prompt iterations

---

## Repository Structure

```
prompt-review-agent/
├── AGENTS.md                          # This file
├── system-prompt-guide.md             # Canonical 10-section system prompt template guide
├── system_prompt_template.txt         # Blank template to copy when writing new prompts
│
├── prompts/                           # System prompts under review or development
│   ├── <prompt-name>.md               # Source prompt (current working version)
│   ├── <prompt-name>_v0_analysis_<model>.md   # Audit report (initial review)
│   └── <prompt-name>_vN_<model>.md            # Refactored prompt (version N, target model)
│
├── .agents/
│   └── skills/
│       └── prompt-auditor/            # Agent Skills-format skill (cross-client)
│           ├── SKILL.md               # Skill entry point
│           └── references/
│               ├── model-baselines.md       # Per-model capability baselines (OpenAI/Claude/Mistral)
│               └── system-prompt-guide.md   # Portable copy of the guide (for standalone skill use)
│
└── .github/
    ├── copilot-instructions.md        # GitHub Copilot workspace instructions
    └── agents/
        └── prompt-auditor.agent.md    # GitHub Copilot custom agent (legacy; skill takes precedence)
```

---

## Key Conventions

### Prompt versioning

Files in `prompts/` follow a strict naming convention:

| Pattern | Meaning |
|---|---|
| `<name>.md` | Active source prompt |
| `<name>_v0_analysis_<model>.md` | Audit report for the first review cycle (no prior versioned prompt existed) |
| `<name>_vN_<model>.md` | Refactored prompt, version N, for target model |
| `<name>_vN_analysis_<model>.md` | Audit report for version N |

`<model>` is one of: `openai`, `claude`, `mistral`.

**Do not create new versioned files manually.** The Prompt Auditor skill manages versioning after user confirmation.

### System prompt structure

All prompts in this repo follow the 10-section template defined in `system-prompt-guide.md`:
Role → Objective → Scope → Instructions → Tool Use Policy → Output Format → Examples → Validation Checklist → Special Considerations → Runtime Context.

When editing a source prompt, preserve this structure. Do not add sections not defined in the guide without updating the guide first.

### Prompt Auditor skill

The canonical auditor is the Agent Skills version at `.agents/skills/prompt-auditor/SKILL.md`.
The `.github/agents/prompt-auditor.agent.md` is a legacy GitHub Copilot agent kept for backward compatibility during transition. If both are present and conflict, the skill takes precedence.

---

## What agents should NOT do in this repo

- Do **not** modify versioned files (`_vN_*`) — they are immutable audit artifacts.
- Do **not** edit `references/system-prompt-guide.md` inside the skill directly — sync changes from `system-prompt-guide.md` at the repo root.
- Do **not** create new `.md` files in `prompts/` outside the versioning convention above without being asked.
