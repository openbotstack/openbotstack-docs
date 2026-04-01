# Tool Development Guide

In OpenBotStack, **Tools** provide deterministic integrations with external systems. Tools live in the **Application Plane** and are executed during a skill's lifecycle.

## 1. What is a Tool?

A tool is a predictable, stateless piece of code that retrieves data from or writes data to an external system.

**Tools are NOT intelligent.** They do not use LLMs, they do not reason, and they do not orchestrate.

Examples of good tools:
- `emr_query`: Queries a medical database for a patient record.
- `vital_signs`: Retrieves current heart rate and blood pressure from an API.
- `lab_results`: Fetches a JSON array of recent lab tests.

## 2. Structure

A tool consists of a manifest and its implementation.

```text
tools/vital_signs/
├── manifest.yaml
└── tool.go
```

### Manifest

The manifest defines the exact inputs the tool requires, and the structure of its deterministic output.

```yaml
name: vital_signs_retrieval
version: 1.0.0
description: Retrieves vital signs for a given patient ID

inputs:
  patient_id:
    type: string
    required: true
    description: Unique patient identifier

outputs:
  type: json
  schema:
    heart_rate: int
    blood_pressure: string
    temperature_c: float
```

## 3. Strict Determinism and Auditability

To satisfy the global governance rules, every tool must be:

1. **Deterministic:** If you feed a tool the same input parameters and the external system state hasn't changed, the output must be exactly the same.
2. **Auditable:** The Execution Plane automatically wraps tool invocations. All inputs sent to the tool and all outputs returned by the tool are intercepted and saved to the audit log.
3. **Bounded:** Tools must execute quickly. Long-running tasks must be handled asynchronously by an external system, not by keeping an OpenBotStack tool execution blocking.

## 4. Writing a Tool

When writing the `tool.go` implementation (compiled to Wasm or executed natively via plugin), focus on pure I/O:

```go
// Example Tool Output
func execute() {
    input := tool.GetInput()
    patientID := input.GetString("patient_id")
    
    // Deterministic API call to external database
    vitals, err := externalAPI.GetVitals(patientID)
    if err != nil {
        tool.Error("Failed to fetch vitals: " + err.Error())
        return
    }
    
    // Tools return raw structured facts, NOT natural language
    tool.SetOutputJSON(vitals)
}
```

By keeping tools deterministic and stupid, we reduce hallucinations and ensure strong auditability across the entire enterprise stack.
