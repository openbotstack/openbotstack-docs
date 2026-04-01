# ADR 011: Model Layer Isolation and 6-Plane Architecture

## Status
Accepted

## Context
As OpenBotStack evolved, the usage of LLMs became ubiquitous across various components (agents, planners, executors). Previously, the `llm` package in `openbotstack-runtime` directly interfaced with LLM providers like OpenAI. This created several issues:
1. **Coupling:** The Execution Plane was overly tightly coupled to specific providers.
2. **Missing Multi-Tenant Features:** We lacked centralized enforcement of tenant-level quotas, routing policies, or cost controls.
3. **Capability Abstraction:** Skills and agents need capabilities (e.g., text generation, tool calling, vision) rather than specific models.

## Decision
We decided to extract and centralize all model interactions into a distinct **Model Plane** within the `openbotstack-core` module, migrating the system from a 4-plane to a **6-Plane Architecture**. 

### The Model Plane consists of:
1. **Capability Router:** Dynamically routes requests to the appropriate `ModelProvider` based on required capabilities (`CapTextGeneration`, `CapJSONMode`, etc.) and constraints (latency, cost, tenant policies).
2. **Middleware Pipeline:** Extensible hook points for logging, tracing, metrics, retry logic, and rate-limiting.
3. **Stateless Providers:** Provider adapters (`OpenAIProvider`, `ClaudeProvider`, `OllamaProvider`, `VLLMProvider`) that map the internal `ModelProvider` interface to external APIs.
4. **Token Accounting:** Interfaces measuring usage and propagating telemetry up to the Management Plane.

## Consequences
- **Positive:** Execution logic (agents and skills) no longer knows about LLM vendors. It only requests capabilities.
- **Positive:** Hard enforcement of tenant quotas, privacy policies, and rate limits is achieved consistently.
- **Positive:** Future integrations (e.g., new local models or cloud APIs) only require writing a new `ModelProvider` in the core module.
- **Negative:** Increased initial complexity due to the router abstraction and interface definitions overhead.

## Migration
The deprecated `llm` package in `openbotstack-runtime` has been completely deleted. All runtime components (such as `agent.NewLLMPlanner` and the WebAssembly Host API) now accept the abstract `ModelRouter` interface from `openbotstack-core/model`.
