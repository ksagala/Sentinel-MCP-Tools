# Scenario: Exposure Management

Read the [Graph tools overview and prerequisites](./README.md) before running this scenario.

## Introduction
This scenario provides a modular baseline for prioritizing critical assets and exposures using Microsoft Sentinel graph relationships together with vulnerability and exposure evidence.

Exposure Management answers which assets or exposures should be prioritized, why they matter, and what evidence supports the decision. Graph reachability describes possible capability; it does not prove that compromise, movement, exploitation, or access occurred.

## Expected Outputs
- Bounded inventory of critical assets or exposure candidates.
- Separate vulnerability, exposure, and graph relationship findings.
- Prioritized remediation candidates based only on evidence that was successfully retrieved or supplied.
- Data gaps, evidence freshness, and recommended follow-up investigations.
- Optional SVG relationship map generated from the collected evidence.

## When to use?
- Use when prioritizing vulnerable or internet-exposed critical assets.
- Use when comparing exposure across asset classes or business-critical resources.
- Use when attack-path context should change remediation priority.
- Use named graph scenarios when the question is specifically about blast radius, paths, perimeter, or direct relationships.

## Scenario Flow & Instructions
Use this file as the active scenario module together with the core context template.
Do not merge this file into context.md. Attach/load both files and set this file as the active scenario.

> Runtime-relevant sections (load at runtime): When to use?, Evidence and tool reference, the Exposure Management instructions and investigation flow, Constraints, and Sample Prompts. The remaining sections (Introduction, Expected Outputs, Example Investigation Flow, References) are human-facing README context and can be skipped at runtime.

Suggested modular run input:
- Core context: templates/context.template.md
- Shared constraints: Data Exploration/Graph/README.md
- Active scenario: Data Exploration/Graph/ExposureManagement.md
- Scope: provide the asset class, exposure question, prioritization objective, and available vulnerability or exposure evidence

## Evidence and tool reference

| Evidence or purpose | Source or tool | Use |
|---|---|---|
| Supported labels, properties, and graph capabilities | `get_graph_context` | Call before label/property-driven operations; graph selection is automatic and no parameter selects a graph |
| Critical-asset and exposure candidate inventory | `find_nodes` | Use a valid label, valid properties, and an explicit `resultsLimit`; the response projects only `Id`, `Name`, and normalized `Criticality` |
| Relationships between asset classes | `find_connected_nodes` | Expand relationships only when they can change prioritization |
| Vulnerability evidence | Triage `RunAdvancedHuntingQuery` or Data Exploration `query_lake` | Confirm which source holds the TVM table before querying; keep vulnerability results separate from graph results |
| Enterprise exposure graph evidence | Triage `RunAdvancedHuntingQuery` or Data Exploration `query_lake` | Confirm which source holds `ExposureGraphNodes` and `ExposureGraphEdges` before querying |
| Initiative scores, metric values and weights, and portal recommendations | Analyst-provided Microsoft Security Exposure Management portal or export data | No MCP tool and no advanced hunting table returns these; supply them or configure a separate source |
| Secure Score | Analyst-provided data or the Microsoft Graph API `security/secureScores` endpoint, configured separately | The Microsoft Graph API is unrelated to the Sentinel exposure graph. Treat Secure Score as a separate posture measure, not as an Exposure Management initiative score |
| Named traversals | Built-in graph tool | Use [BlastRadiusEvaluation.md](./BlastRadiusEvaluation.md) for blast radius; use the matching built-in tool for walkable path, exposure perimeter, or connected-nodes questions |

## Exposure Management - Instructions and Investigation Flow
Classify the request
- Decide whether the objective is critical-asset inventory, vulnerability prioritization, exposure posture, class-to-class relationship analysis, or a named graph traversal.
- Use [BlastRadiusEvaluation.md](./BlastRadiusEvaluation.md) for blast radius. Other named traversals use the matching built-in graph tool and the shared prerequisites.

Select the evidence mode before calling any tool
- **Graph only**: supported node labels, relationships, paths, exposure perimeter, blast radius, and normalized criticality.
- **Defender advanced hunting or lake**: CVEs, vulnerability severity, exploitability, TVM evidence, `ExposureGraphNodes`, and `ExposureGraphEdges`. Also required to read the value of any node property other than criticality. Confirm which source holds the table before querying.
- **Data Exploration lake**: observed activity from confirmed Sentinel data lake tables.
- **External or analyst-supplied**: Exposure Management portal initiative scores, metric values and weights, portal recommendations, and Secure Score.
- A broad exposure question is not a graph-only question. State the required modes up front, and report any unavailable mode as a data gap instead of answering from graph evidence alone.

Establish evidence provenance
- Record the source, tenant context, collection date, and freshness for each evidence set.
- Keep Exposure Management initiative scores and metric values, Defender TVM vulnerability results, TVM exposure score, and Microsoft Secure Score as separate measures. Do not present them as interchangeable.
- If portal, export, Triage, a required Defender table, or Secure Score data is unavailable, report the gap and continue with graph evidence only. Do not infer missing values.
- When supplied evidence is older than the period the question asks about, state its collection date and label it as the last verified baseline rather than as a current measurement.

Discover graph capabilities
- Call `get_graph_context` before `find_nodes` or `find_connected_nodes`. Use only labels and properties returned by this call.
- Graph selection is automatic. No tool parameter selects a graph, so do not ask the analyst which graph to use.
- Graph tools accept no `workspaceId` and no time window. Do not ask for either or imply that graph results are time-scoped.
- Split a time-bound question: answer the topology part from the graph as a current snapshot, route the time-bound part to a time-capable source, and label which part is which.

Build a bounded candidate inventory
- Run `find_nodes` with a valid label, valid properties, and an explicit `resultsLimit`.
- Enumerate supported asset labels in priority order when a complete critical-asset inventory is requested. Cap label queries at five per run by default, state the cap and the labels covered, and ask before exceeding it.
- Criticality and internet-exposure property names differ per label. Use the exact names `get_graph_context` returns for the label being queried and do not reuse one label's property name against another label.
- `find_nodes` projects only `Id`, `Name`, and a normalized `Criticality` column. It does not return the value of the properties passed in `validNodeProperties`, so a node returned under an exposure property is a schema-backed candidate, not a confirmed `true`. Never label such results as internet-exposed, customer-facing, or scored without confirming the value in `ExposureGraphNodes`.
- The schema property `criticalityLevel` is returned as the `Criticality` column. Use `criticalityLevel` on input and read `Criticality` on output.
- Retain every populated `criticalityLevel` value, including `criticalityLevel = 0`, which is the highest criticality. Do not filter with `criticalityLevel > 0`.
- Report unset criticality as an explicit unset category rather than dropping those nodes or treating unset as low criticality.
- Not every label carries every property. Report labels that lack a criticality or exposure property as a coverage gap instead of excluding them silently.

Expand relationships selectively
- Use `find_connected_nodes` only when class-to-class relationships can change the priority or explain concentration of risk.
- Set `resultsCountLimit` and avoid speculative fan-out because every graph invocation is metered.
- An empty `find_connected_nodes` or `find_exposure_perimeter` result is not evidence that no relationship or no exposure path exists. Report it as an unresolved query and confirm against `ExposureGraphNodes` and `ExposureGraphEdges` before concluding absence.
- Treat graph relationships as possible reachability or capability, not observed activity.

Correlate and prioritize
- Where available, correlate graph candidates with vulnerability severity, exploitability, internet exposure, privilege impact, RCE impact, attack-path concentration, and evidence freshness.
- `exposureScore` appears in the graph schema on some labels, but `find_nodes` does not return its value. Do not rank on it from graph output; retrieve it from `ExposureGraphNodes` or report it as a data gap.
- In `ExposureGraphNodes`, exposure properties are nested under `NodeProperties.rawData` and an internet-exposure property can be a structured object rather than a Boolean. Sample the property keys for the class before querying, and do not test for a literal `true`.
- Defender TVM is device-scoped while the graph catalog spans endpoint, Azure, AWS, GCP, data, repository, and identity classes. Report non-device classes as outside TVM coverage rather than as free of vulnerabilities.
- State the join key before correlating graph entities with advanced hunting or lake rows. Prefer a stable identifier such as device ID or object ID over a display name, and use the identifier fields returned by the graph node rather than reconstructing one.
- Report entities that could not be matched as unresolved. Do not fuzzy-match names, merge similarly named entities, or assume a graph node and a hunting row are the same asset without a shared identifier.
- The graph node inventory can be narrower than the Defender device inventory. An asset missing from the graph is a graph coverage gap, not an absent or non-critical asset.
- Keep distinct measures in distinct columns. Do not merge initiative percentages, TVM exposure score, and CVE counts into one score, rating, or signal.
- Confirm which source holds a required table before querying it: `search_tables` before a lake query, and `FetchAdvancedHuntingTablesOverview` plus `FetchAdvancedHuntingTablesDetailedSchema` before an advanced hunting query.
- If a table returns no results, treat it as a source or availability question rather than evidence of no exposure.
- Use Data Exploration lake queries or Triage evidence to establish observed activity.
- Prioritize using supported and retrieved evidence. Do not score or rank on unavailable factors; report them as data gaps.

Summarize
- Report an Executive Summary, Key Findings, prioritized remediation actions, data gaps, and provenance.
- Explain which findings are measured exposure or vulnerability evidence and which are graph-based possible relationships.

Generate an SVG relationship map (optional)
- Generate the SVG only when requested or when the analyst selects it as an output.
- Use only collected evidence; do not infer nodes, edges, labels, or property values.
- Adapt columns, node classes, and annotations to the returned evidence.
- Mark missing and unresolved evidence explicitly.
- Do not make additional metered graph calls solely to fill visual gaps.

## Constraints
- Graph labels and properties must come from `get_graph_context`.
- Graph tools have no workspace scoping and no time window.
- UPNs are unsupported for graph identity lookups. Resolve an identity to a display name or object ID before using a graph tool.
- Prefix entity types before names and add `in my graph` when graph-only scope is required.
- Criticality and internet-exposure property names vary by node label; there is no universal property name for either.
- `find_nodes` returns only `Id`, `Name`, and normalized `Criticality`. Property-filtered results are candidates, not confirmed property values.
- The schema property `criticalityLevel` is returned as the `Criticality` column.
- Preserve critical assets with populated `criticalityLevel`, including `criticalityLevel = 0`, which is the highest criticality rather than an absence of criticality. Report unset criticality as its own category.
- Graph tools are preview and metered; keep calls bounded and do not page blindly.
- An empty graph traversal result is an unresolved query, not proof that no relationship or exposure path exists.
- Absence of an asset from the graph is a coverage gap, not evidence that the asset does not exist or is not critical.
- Defender TVM covers devices only. A non-device class without TVM results is outside coverage, not vulnerability-free.
- This scenario cannot retrieve Exposure Management portal initiative scores, metric values or weights, or portal initiative recommendations unless the analyst supplies them or a separate configured source exposes them. Graph-derived recommendations returned by a graph tool are a different output and must be labeled as such.
- Confirm table availability and schema in the chosen source before querying it.
- Exposure Management initiative scores, TVM vulnerability results, TVM exposure score, and Secure Score are different measures.
- Graph output alone does not establish compromise, exploitation, lateral movement, or observed access.
- An SVG must not contain inferred nodes, edges, labels, or property values. Mark missing and unresolved evidence explicitly.
- Defang suspicious indicators in findings and recommendations.

## Sample Prompts
Groups match the evidence modes above. Do not answer a prompt from a lower group using graph evidence alone.

### Graph only
- "Which node labels in my graph carry criticality and internet-exposure properties, and which of those properties can `find_nodes` actually return?"
- "List critical assets in my graph that carry an internet-exposure property, and state clearly that the property values themselves are not returned."
- "Which device classes connect to critical assets in my graph, and how are those relationships concentrated?"

### Graph plus advanced hunting or lake
- "List critical assets in my graph that carry an internet-exposure property, then confirm the actual exposure values from `ExposureGraphNodes`."
- "Assess exposure posture across supported asset classes using graph evidence and Defender vulnerability evidence."
- "Prioritize vulnerable devices connected to critical assets using graph relationships and Defender TVM evidence, and state the join key you used."
- "What evidence supports remediation priority for internet-facing critical assets, including CVE severity and exploitability?"

### Graph plus analyst-supplied data
- "Prioritize posture gaps using graph evidence and the supplied Exposure Management initiative metrics."

### Graph plus advanced hunting or lake plus Data Exploration lake
- "Which posture gaps should be prioritized based on criticality, internet exposure, vulnerability evidence, and observed activity freshness?"

### Output from collected evidence
- "Create an SVG relationship map of the critical assets and connected entity classes returned by this investigation."

## Example Investigation Flow
Figure 1: Add example screenshot or flow summary.

## References
- [Graph tools overview and prerequisites](./README.md)
- [Blast Radius Evaluation](./BlastRadiusEvaluation.md)
- [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool)
- [Triage tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-triage-tool)
- [Microsoft Security Exposure Management](https://learn.microsoft.com/en-us/security-exposure-management/)
- [Query the enterprise exposure graph](https://learn.microsoft.com/en-us/security-exposure-management/query-enterprise-exposure-graph)
- [Exposure insights overview](https://learn.microsoft.com/en-us/security-exposure-management/exposure-insights-overview)
