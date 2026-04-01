# Skill Development Guide

Skills in OpenBotStack represent high-level domain capabilities. They are governed compositions of tools and prompts that securely execute logic within the **Execution Plane**.

> **CRITICAL RULE:** Skills must **NOT** contain infinite reasoning loops or open-ended autonomous behavior (like AutoGPT). They must be bounded, deterministic at the control plane, and purely request-scoped.

## 1. Concept and Purpose

A skill encapsulates a specific action or intellectual capability, for example:
- `generate_nursing_note`
- `detect_sepsis_risk`
- `generate_sbar_report`

Instead of giving a generic LLM complete freedom, an OpenBotStack skill binds a strictly defined contextual prompt with specific deterministic **tools**. 

## 2. Structure

A standard skill is packaged with a manifest, its executable code, and its prompt configurations.

```text
skills/generate_nursing_note/
├── manifest.yaml      # Defines metadata, permissions, and timeouts
├── skill.go           # The executable logic (compiled to Wasm)
└── prompts/
    └── system.tmpl    # Prompt configuration
```

### Manifest

The `manifest.yaml` defines the execution boundaries of a skill:

```yaml
name: generate_nursing_note
version: 1.0.0
description: Summarizes vitals and observations into a standard nursing note
entrypoint: action.wasm

# Maximum execution time allowed
timeout: 10s

# Required capabilities from the AI Service Plane
requires:
  - text_generation

# Required deterministic tools
tools:
  - vital_signs_retrieval
  - shift_observations
```

## 3. The Lifecycle of a Skill

1. **Invocation:** The execution plane receives a request to run the skill. The state machine moves to `Running`.
2. **Context Assembly:** The skill retrieves its configured prompts and requested input data.
3. **Tool Execution:** The skill deterministically invokes any necessary tools to gather facts (e.g., calling the `vital_signs_retrieval` tool).
4. **Model Request:** The skill sends the assembled prompt and tool results to the **AI Service Plane** via the internal router. **A skill does NOT communicate with external APIs (like OpenAI) directly.**
5. **Completion:** The AI Service Plane returns the response. The skill formats the output, logs it, and terminates. The state machine transitions to `Completed`.

## 4. Bounded Logic & Prompts

Prompts are configuration, not business logic.

- **Do NOT** embed complex branching logic into your prompts.
- **Do NOT** allow the LLM to decide what tool to call in an unchecked loop.
- **Do** map specific states in your application strictly to specific skills.

If you find yourself writing a "ReAct" loop inside your skill, you are violating the core architectural rules. If complex orchestration is needed, use a **Workflow**.
