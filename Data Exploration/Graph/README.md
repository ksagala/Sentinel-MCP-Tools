# Microsoft Sentinel Graph Tools

> Shared human-facing setup for the built-in Microsoft Sentinel graph tools. Keep
> agent-specific execution policies outside this public guide.

## Status

> [!IMPORTANT]
> The Microsoft Sentinel graph tools are in **preview**.

## Where the tools live

All graph tools are part of the **data exploration tool collection**, hosted at
`https://sentinel.microsoft.com/mcp/data-exploration`. There is **no separate "Sentinel
Graph" MCP server endpoint** — do not add one to `mcp.json` and do not route scenarios
to one.

The **entity analyzer** (`analyze_user_entity`, `analyze_url_entity`,
`get_entity_analysis`) is in the *same* collection but is **not a graph tool** and has
different rules — it accepts an optional `workspaceId`, requires an explicit time window,
and does accept UPNs. See [EntityAnalyzer.md](../EntityAnalyzer.md).

The tools reason over the Microsoft Sentinel **exposure, hunting and blast radius / data
risk graphs**.

## Prerequisites

- Microsoft Sentinel data lake and graph.
- Microsoft Sentinel data lake **onboarded to the Defender portal** — required for graph
  tools specifically, in addition to normal lake onboarding.
- Roles: at least **Security Reader**, **Security Operator** or **Security Administrator**.
- **At least read-only access in Microsoft Security Exposure Management** to read graph
  data in the Defender portal.
- A supported MCP client: Security Copilot, Copilot Studio, Microsoft Foundry or
  Visual Studio Code.

## Cost

Installing and configuring the graph tool collection carries no cost. **Invoking** the
tools triggers the Microsoft Sentinel **graph billing meter**.

- Hunting graph and blast radius visualizations used *inside* the Defender portal (and
  IRM / Data Security Investigations in Purview) are free — reaching those same graphs
  **through the MCP graph tool collection is billable**.
- Graph queries use **6 vCores** with a **minimum query execution time of one minute**.

Scope queries before running them and avoid speculative fan-out.

## Query constraints

| Constraint | Detail |
|---|---|
| Scope to graph | Add **`in my graph`** to prompts to restrict results to graph data. |
| Identities | **UPNs are not supported.** Resolve `user@contoso.com` to a display name / object ID (for example with `find_nodes`) before calling any graph tool. |
| Entity naming | Put the **entity type before the name**: `user John`, `device DC01`, `group Tier0 Admins`. |
| Workspace | Graph tools accept **no `workspaceId`**. Do not gate a graph scenario on workspace selection. |
| Time window | Graph tools accept **no time range**. Never request or imply one. |
| Semantics | Graph results describe **possible** relationships, not observed activity. Corroborate with `query_lake` when a scenario needs evidence of actual use. |

> [!NOTE]
> The first three rows are documented by Microsoft. The **Workspace**, **Time window**
> and **Semantics** rows are operational guidance derived from the published tool
> parameter surface (no `workspaceId` or time parameters exist on any graph tool).

## When graph tools are not enough

Graph tools answer topology questions. They do not carry CVE, exploitability or portal
posture data, so most real exposure questions need a second source.

| Evidence | Route to | Note |
|---|---|---|
| Node labels, relationships, paths, perimeter, blast radius, criticality | Graph tools | Bounded and metered |
| The value of any node property other than criticality | **Triage** `RunAdvancedHuntingQuery` or `query_lake` against `ExposureGraphNodes` | `find_nodes` filters on properties but does not return their values; the values live under `NodeProperties.rawData` and are not always Boolean |
| CVEs, severity, exploitability, TVM assessments | **Triage** `RunAdvancedHuntingQuery` or `query_lake` | Confirm which source holds the table before querying |
| `ExposureGraphNodes`, `ExposureGraphEdges` | **Triage** `RunAdvancedHuntingQuery` or `query_lake` | Confirm which source holds the table before querying |
| Observed activity | Data Exploration `search_tables` + `query_lake` | Confirmed Sentinel data lake tables only |
| Initiative scores, metric weights, portal recommendations, Secure Score | Analyst-supplied or separately configured source | No tool in this collection retrieves them |

Confirm availability in the source you intend to use before querying: `search_tables` for
the data lake, and `FetchAdvancedHuntingTablesOverview` plus
`FetchAdvancedHuntingTablesDetailedSchema` for advanced hunting.

## Graph context

`get_graph_context` returns supported labels, properties and graph capabilities.

- Call it before the label/property-driven tools: `find_nodes`, `find_connected_nodes`.
  Use only labels and properties returned by this call.
- **Optional** for the keyword-name tools: `find_blastradius`, `find_walkable_paths`,
  `find_exposure_perimeter`. Calling it anyway adds a metered round trip — only do so
  when entity resolution is ambiguous.

## Tool parameter reference

| Tool | Parameters |
|---|---|
| `get_graph_context` | — |
| `find_nodes` | `validNodeLabel` (required), `validNodeProperties`, `resultsLimit` |
| `find_blastradius` | `sourceName` (required) — the only parameter |
| `find_walkable_paths` | `sourceName` (required), `targetName` (required) |
| `find_exposure_perimeter` | `targetName` (required), `minPathLength`, `maxPathLength`, `resultsCountLimit` |
| `find_connected_nodes` | `sourceNodeLabel` (required), `sourceNodeProperties`, `targetNodeLabel` (required), `targetNodeProperties`, `resultsCountLimit` |

`find_nodes` projects only `Id`, `Name`, and a normalized `Criticality` column, and the
schema property `criticalityLevel` is returned as `Criticality`. Properties passed in
`validNodeProperties` act as selection criteria; their values are not returned, so a
returned node is a candidate rather than a confirmed `true`.

## Example scenario

See [Blast Radius Evaluation](./BlastRadiusEvaluation.md) for an end-to-end example.

## References

- [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool)
- [Graph tools (preview)](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#graph-tools-preview)
- [Triage tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-triage-tool)
- [What is Microsoft Sentinel graph?](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-graph-overview)
- [Plan costs and understand Microsoft Sentinel pricing and billing — graph charges](https://learn.microsoft.com/en-us/azure/sentinel/billing#graph-charges)
