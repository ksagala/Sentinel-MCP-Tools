# Sentinel-MCP-Tools

Sentinel MCP Tools is a guideline repository for Microsoft Sentinel Model Context Protocol (MCP) tool collections designed to enhance automation, enrichment, and investigation workflows. The goal of this repository is to provide modular, easy-to-extend instructions for using Sentinel MCP tool collections with AI agents or custom SOC automation frameworks.

Most examples use VS Code as the MCP client, but the scenarios also work in other MCP-enabled frameworks and clients.

## Context.md
Use `context.md` to give GitHub Copilot + Sentinel MCP a small, consistent core brief in VS Code. Keep it as the global orchestration layer, then load one scenario module on demand.

- What it is: a lightweight, non-sensitive core playbook (no secrets).
- Where to keep it: in your workspace as `context.md`, typically created from `templates/context.template.md`.
- Scenario modules: keep scenario-specific instructions in separate files under `Data Exploration/` (for example `BehavioralAnalysis.md`, `EntityAnalyzer.md`, `EntityInvestigation.md`).
- How to use it in VS Code: load both files for a run: core `context.md` and one active scenario module.
- Secrets: store credentials in environment variables or your keychain. Do not place tokens in `context.md` or scenario files.


## Benefits

 **Consistency & Tool-Aware Guidance**:
 - Shared scope, terms, and steps keep MCP responses aligned.

 **Reproducibility & Auditability**:
 - Versioned context yields repeatable runs and clear review trails.

 **Determinism**:
 - Reduces variance across prompts and analysts for stable outcomes.

 **Speed & Focus**:
 - Cuts setup time by front-loading scope, filters, and preferred queries.

 **Secret Hygiene**:
 - Non-sensitive file; keep credentials in env vars or keychain.

 **Modularity & Onboarding**:
 - Small per-scenario files scale well and ramp new analysts faster.


## How to Use Context.md
1. Create `context.md` from `templates/context.template.md` and set metadata (objective, time window, entities in scope).
2. Choose one scenario module from `Data Exploration/` that matches the investigation.
3. Start the MCP server(s) required by the selected module and routing rules in `context.md`.
4. In Copilot Chat (with MCP enabled), attach/reference both files: `context.md` and the active scenario module.
5. Run the prompt template from `context.md` and execute the investigation.
6. If needed, pivot to one additional scenario module and document the pivot and rationale.
7. Capture findings and decisions for handoffs and post-incident reviews.

## Prerequisites
- VS Code with GitHub Copilot and MCP client enabled for this profile.
- Access to your Microsoft Sentinel workspace and sign-in/identity telemetry.
- Sentinel Data Lake enabled and configured.
- Sentinel MCP server installed and accessible locally/remote. Means practically local MCP client configuration (e.g., `mcp.json`) pointing to your server.
