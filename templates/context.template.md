<!-- Template: Unified | Version: 2.1.1 | Last Updated: 2026-07-04 | Maintainer: <name/team> -->
# context.md (Unified Template - MCP-Aware Investigation Playbook)

Use this single template to give GitHub Copilot + Sentinel MCP consistent context for investigations.
Keep secrets out of this file.

## Purpose
- Use the smallest MCP surface for each task.
- Operate with 4 core servers and enable optional servers only on demand.
- Avoid loading unnecessary servers and tools.
- Keep investigation flow consistent across identity, incident, data lake, and graph scenarios.

## Metadata
- Scenario: <Behavioral Analysis | High-Risk User | Incident Triage | Entity Investigation | Graph Investigation | Custom Hunting (e.g., AiTM)>
- Owner: <name/team>
- Last Updated: <YYYY-MM-DD>

## Modular Scenario Contract
- Keep this file as the core playbook (routing, guardrails, deliverables).
- Load exactly one scenario module per run unless the user explicitly asks to combine scenarios.
- Scenario modules live under Data Exploration and contain scenario-specific flow, rules, and prompts.
- If core guidance conflicts with scenario guidance, follow core safety/routing rules first, then scenario workflow.
- Always state which scenario module is active in the response summary.

## Scenario Modules (modular playbooks)

Detailed, scenario-specific flows live in the Data Exploration modules below.
Load the matching module on demand instead of duplicating its steps here.

| Scenario | Module | Primary server(s) | Optional secondary | When to use |
|---|---|---|---|---|
| Behavioral analysis | [Data Exploration/BehavioralAnalysis.md](../Data%20Exploration/BehavioralAnalysis.md) | Data Exploration | Triage | Sign-in behavior analytics, anomaly detection, and IP reputation checks |
| Entity analyzer | [Data Exploration/EntityAnalyzer.md](../Data%20Exploration/EntityAnalyzer.md) | Data Exploration | Triage | Fast AI-assisted risk verdicts for users, URLs, and domains |
| Entity investigation and relationship analysis | [Data Exploration/EntityInvestigation.md](../Data%20Exploration/EntityInvestigation.md) | Data Exploration | Data Exploration graph tools, Triage | Deep relationship mapping, lateral movement tracing, and timeline reconstruction |

**Cross-cutting guardrails for all modules:** defang suspicious IPs, URLs, and domains; filter RFC1918 and tenant-owned indicators; complete Sentinel workspace selection before entity drill-down; wait for user selection before deep analysis.

## Prompt Template (Core + Scenario)
Use this prompt format with your MCP client:

```text
Use the attached core context file as global investigation policy.
Use the attached scenario module as task-specific instructions.

Active scenario module: <scenario file name>
Objective: <one sentence>
Time window: <e.g., last 7d>
Entities in scope: <users/devices/IPs/URLs>

Execution rules:
1) Follow server routing and tool minimization from core context.
2) Follow workflow steps and decision logic from the active scenario module.
3) Add one secondary server only when required by evidence.
4) If required data is unavailable, continue with available evidence and clearly list gaps and fallback path.
5) Return: findings table, evidence summary, and prioritized mitigation actions.
```

## Modular Scenario Loading
- Keep this file as the global orchestration layer (routing, constraints, outputs, and decision rules).
- Load only one scenario module by default, and add a second module only when the investigation requires a pivot.
- Scenario modules are stored separately and should contain scenario-specific workflows, limits, and sample prompts.

### Scenario Module Index
- Behavioral Analysis: Data Exploration/BehavioralAnalysis.md
- Entity Analyzer: Data Exploration/EntityAnalyzer.md
- Entity Investigation and Relationship Analysis: Data Exploration/EntityInvestigation.md

### Instruction Precedence
- Global rules in this file always apply.
- Scenario module instructions override generic workflow wording when there is overlap.
- If a scenario module conflicts with data/tool availability, follow global routing rules and document the fallback.

## Server Responsibilities

### 1) Docs MCP (microsoft.docs.mcp)
- Use for Microsoft product research, guidance, validation, prerequisites, and official references.
- Use first when a task asks how to or what Microsoft recommends.

### 2) Triage (Triage)
- Use for incident-centric investigations.
- Use for active incidents, alert artifacts, severity triage, and incident-driven threat analysis.

### 3) Data Exploration (Data Exploration)
- Use for Sentinel data lake KQL investigations.
- Use for sign-in analytics, anomaly detection, timeline reconstruction, and table-driven evidence gathering.

### 4) Graph tools (part of the Data Exploration collection)
- Use for graph investigations such as blast radius, path discovery, exposure perimeter, and graph relationship hunting.
- Use for entity relationship expansion where graph traversal is required.
- The graph tools are hosted in the Data Exploration collection at `https://sentinel.microsoft.com/mcp/data-exploration`. There is no separate Sentinel Graph MCP endpoint.
- Graph calls are billed against the Microsoft Sentinel graph meter, so keep them bounded.

### 5) Custom XDR Tools (optional / on-demand)
- Use custom tools only when explicitly needed by the investigation.
- Any KQL query saved as a tool in Defender XDR can be used through MCP; AiTM is only one example scenario.

## Tool Routing Rules
- Start with the primary server for the use case.
- Add one secondary server only when needed.
- Use Docs MCP to validate architecture or recommended mitigations before final recommendations.
- Do not use unrelated servers for convenience.

## Suggested Default Execution Order
- If user asks for Microsoft guidance: Docs MCP first.
- If user asks for incidents/alerts: Triage first.
- If user asks for KQL/table evidence or behavior analytics: Data Exploration first.
- If user asks for blast radius/path/exposure: Data Exploration graph tools first.
- If user asks for CVE, exploitability, TVM, or `ExposureGraphNodes`/`ExposureGraphEdges` evidence: Triage advanced hunting or Data Exploration, depending on which source holds the table.
- If user explicitly requests a custom hunting flow (for example AiTM): Custom XDR Tools (optional / on-demand).


## Deliverables
- Findings table: explicit columns per step.
- Markdown summary: key findings, severity, and rationale.
- Mitigation and remediation plan: prioritized actions with owners and due dates.

## Decision Rules
- Use scenario-specific decision rules from the active scenario module.
- If a scenario does not define explicit thresholds, state the threshold assumptions used.
- Label every finding with evidence source, confidence, and recommended next action.

## Acceptance Criteria
- All required steps in the active scenario workflow are executed for the requested scope.
- Each flagged entity has evidence (query results, fields) and a stated reason.
- A single consolidated findings table and clear remediation plan are produced.

## Assumptions and Constraints
- Allowed countries and baseline source are defined (for example HR records, device inventory, policy docs).
- Data sources are accessible and named concretely in the section above.
- Credentials/secrets are provided via environment or keychain; none are stored in this file.

---

## Use Case Mapping

### Behavioral Analysis of User Sign-In Activity
Primary server
- Data Exploration

Optional secondary
- Docs MCP (mitigation guidance references)

Flow
- Retrieve users with most distinct sign-in IP addresses.
- Check country anomalies for top users.
- Check user agent anomalies.
- Verify suspicious IP behavior using available data and related indicators.
- Summarize findings and remediation plan.

### Identify Compromised or High-Risk User Behavior
Primary server
- Data Exploration

Optional secondary
- Triage (if behavior is tied to active incidents)
- Docs MCP (best-practice mitigations)

Flow
- Analyze unusual sign-in behavior for a specific account.
- Detect failure-then-success sign-in patterns.
- Detect impossible travel signals.
- Correlate risky sign-ins with password reset activity.
- Detect authentications from new countries versus baseline.
- Summarize with mitigation and containment actions.

### Incident Triage Workflow
Primary server
- Triage

Optional secondary
- Data Exploration (deep evidence pivot with KQL)
- Docs MCP (response hardening references)

Flow
- Retrieve active incidents and prioritize high and medium.
- Investigate impacted entities and key artifacts.
- Validate reputation and campaign indicators.
- Identify persistence, gaps, and misconfigurations.
- Summarize blast radius and prioritized response plan.

### Entity Investigation and Relationship Analysis
Primary server
- Data Exploration

Optional secondary
- Data Exploration graph tools (when graph traversal is needed)
- Triage (if incidents are involved)

Flow
- Build timeline and entity map from lake tables.
- Retrieve linked alerts/incidents.
- Trace lateral movement indicators.
- Reconstruct file/process activity chains.
- Correlate network communications with threat indicators.
- Summarize risk, scope, and next actions.

### Graph Investigations
Primary server
- Data Exploration (graph tools)

Optional secondary
- Triage (Defender advanced hunting for vulnerability and exposure-table evidence)

Flow
- Blast Radius: enumerate downstream impact from a user/device/entity.
- Path Discovery: find feasible access paths between source and target.
- Exposure Perimeter: identify identities/entities with direct/indirect access.
- Highlight risky intermediaries and overexposed boundaries.

### Entity Analyzer Usage Note
Primary server
- Data Exploration (Entity Analyzer tools: analyze_user_entity, analyze_url_entity, get_entity_analysis).

Optional secondary
- Triage (when the analysis is tied to active incidents).

Fallback
- If entity analyzer output is unavailable or required evidence tables are missing, continue with Data Exploration (search_tables + query_lake) and Triage evidence-based investigation.

## References
- Sentinel Graph overview: https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-graph-overview
- Blast Radius: https://aka.ms/sentinel/graph/docs/incidents
- Graph hunting and path discovery: https://aka.ms/sentinel/graph/docs/hunting
- Entity analyzer tool: https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#entity-analyzer
