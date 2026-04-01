# Application Development Guide

OpenBotStack is an enterprise AI execution infrastructure designed to provide governance, bounded execution, and unified capabilities. It does **not** implement business applications directly. Instead, domain-specific apps (e.g., healthcare, finance) live in the **Application Plane**, external to the core OpenBotStack repositories.

This guide explains how to architect, structure, and build applications on top of OpenBotStack.

## Standard Application Layout

Applications must live outside the core repositories (e.g., in `openbotstack-apps/<your-app>/`). Each application represents a specific domain use case and generally follows this standard layout:

```text
apps/nursing-ai/
├── tools/          # Deterministic, auditable external system integrations
├── skills/         # Governed domain capabilities combining tools and prompts
├── workflows/      # State-machine driven orchestrations of skills
├── ui/             # Web, mobile, or API interfaces
└── config/         # App-specific configuration and profiles
```

---

## 1. Tools

Tools provide deterministic, strictly bounded integrations with external systems. They do not contain any LLM or reasoning logic.

**Characteristics:**
- **Deterministic:** Always behave predictably given the same inputs.
- **Auditable:** Every invocation must log its inputs and outputs.
- **Stateless:** Tools execute an action (e.g., fetch data, write record) and return.

**Examples:**
- EMR query
- Vital signs retrieval
- Lab results lookup
- Medication list retrieval

**Structure:**
```text
tools/
└── emr_query/
    ├── manifest.yaml   # Defines inputs, outputs, and requirements
    └── tool.go         # Implementation
```

*For more details, see the [Tool Development Guide](TOOL_DEVELOPMENT_GUIDE.md).*

---

## 2. Skills

Skills are governed compositions of tools and prompts. They represent domain capabilities and expose high-level operations that the Control Plane can manage.

**Characteristics:**
- **Boundaries:** Skills operate within strict execution timelines and resource limits.
- **No Infinite Loops:** Skills must NOT contain infinite reasoning loops or autonomous reflection cycles.
- **Configuration over Code:** Prompts are treated as configuration, not business logic.

**Examples:**
- `generate_nursing_note`
- `detect_sepsis_risk`
- `generate_sbar_report`

**Structure:**
```text
skills/
└── generate_nursing_note/
    ├── manifest.yaml   # Defines capabilities and dependencies
    ├── skill.go        # Execution logic invoking tools/LLMs
    └── prompts/        # Contextual prompt configurations
```

*For more details, see the [Skill Development Guide](SKILL_DEVELOPMENT_GUIDE.md).*

---

## 3. Workflows

Workflows orchestrate multiple skills deterministically to achieve multi-step domain processes.

**Characteristics:**
- **State-Machine Driven:** Workflows should be modeled as state machines rather than open-ended sequences.
- **Auditable:** Step transitions must be tracked and logged.
- **Governed:** Execution can be paused, validated, or resumed according to OpenBotStack policies.

**Examples:**
- `nurse_shift_handover.yaml`
- `sepsis_screening.yaml`

**Structure:**
```text
workflows/
├── nurse_shift_handover.yaml
└── sepsis_screening.yaml
```

*For more details, see the [Workflow Development Guide](WORKFLOW_DEVELOPMENT_GUIDE.md).*

---

## 4. UI (Client Plane Integration)

The application’s user interface (UI) sits in the Client Plane and exposes the application to the end users. 

**Characteristics:**
- **Decoupled:** The UI communicates with OpenBotStack via APIs or SDK.
- **Platform Agnostic:** Can be Web, Mobile, or Voice.

**Examples:**
- Nursing dashboard
- ICU copilot mobile app
- Shift handover tablet interface

---

## Example Global Execution Flow

1. **UI:** A nurse requests a shift handover report.
2. **API Gateway:** Request is routed through the Access Plane for auth and rate limits.
3. **Policy Engine (Control Plane):** Validates the user's RBAC and active policies.
4. **Execution Plan:** A state-machine orchestrated `nurse_shift_handover` workflow is selected.
5. **Runtime Executor (Execution Plane):** Kicks off the required skills.
6. **Skill Execution:** `vital_signs` tool and `emr_query` tool execute (fetching deterministic data). Then, a generation request is sent to the **AI Service Plane**.
7. **External System:** EMR responds to the tools safely, the AI securely generates the text, and the final response propagates back to the UI.
