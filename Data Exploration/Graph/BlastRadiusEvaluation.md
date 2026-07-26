# Scenario: Blast Radius Evaluation

> [!IMPORTANT]
> Graph tools are in **preview** and are **metered**. Read
> [_GraphToolPrerequisites.md](./_GraphToolPrerequisites.md) before running this scenario.

## Introduction
This scenario provides a modular baseline for evaluating the potential blast radius of a compromised entity using Sentinel graph tools.

Use this file as an instruction module template and tailor it with environment-specific scope, prioritization rules, and output expectations.

## Expected Outputs
- Ranked list of impacted entities and critical assets.
- Propagation paths from the source entity.
- Risk summary and containment priorities.
- Recommended mitigation actions.

## When to use?
- Use when a user, device, service principal, or workload is suspected to be compromised.
- Use to estimate downstream risk before containment planning.
- Use during incident triage when impact scope is unclear.

## Scenario Flow & Instructions
Use this file as the active scenario module together with the core context template.
Do not merge this file into context.md. Attach/load both files and set this file as the active scenario.

> Runtime-relevant sections (load at runtime): When to use?, Tool reference, the Blast Radius Evaluation instructions and investigation flow, Constraints, and Sample Prompts. The remaining sections (Introduction, Expected Outputs, Example Investigation Flow, References) are human-facing README context and can be skipped at runtime.

Suggested modular run input:
- Core context: templates/context.template.md
- Shared constraints: Data Exploration/Graph/_GraphToolPrerequisites.md
- Active scenario: Data Exploration/Graph/BlastRadiusEvaluation.md
- Scope: provide the **source entity** and the investigation objective

> [!NOTE]
> Do **not** ask for a time window or a workspace. `find_blastradius` accepts neither.

## Tool reference

| Tool | Parameters |
|---|---|
| `find_blastradius` | `sourceName` (required) — the **only** parameter |
| `find_nodes` (entity resolution, optional) | `validNodeLabel` (required), `validNodeProperties`, `resultsLimit` |
| `get_graph_context` (only if resolution is ambiguous) | — |

## Blast Radius Evaluation - Instructions and Investigation Flow
Resolve the source entity
- Prefix the entity type before the name (`user Sipa`, `device DC01`).
- If only a UPN or email is available, resolve it first — **graph identity lookups do not support UPNs**. Use `find_nodes` (call `get_graph_context` first to get valid labels/properties), or resolve the object ID outside the graph.
- Confirm the exact node with the analyst when multiple candidates match.

Evaluate blast radius
- Run `find_blastradius` with `sourceName` and collect impacted paths and critical nodes.
- Add `in my graph` to the prompt to keep results graph-scoped.

Validate critical paths
- Confirm high-risk paths and identify choke points for containment.
- State explicitly that paths represent **possible** propagation, not observed activity.

Corroborate (optional)
- Use `query_lake` for evidence that a path was actually exercised, or the triage collection for open incidents on the same entities.

Summarize scope and actions
- Produce an impact summary, prioritize containment, and list recommended next steps.

## Constraints
- Single parameter only: no time window, no result limit, no workspace scoping.
- UPNs unsupported for identities; entity type goes before the name.
- Preview feature; each invocation is billed against the graph meter.

## Sample Prompts
- "What is the blast radius of user <display name> if compromised, in my graph?"
- "Evaluate downstream impact from device <hostname> to critical assets in my graph."
- "List high-risk propagation paths from <entity> and recommend containment order."

## Example Investigation Flow
Figure 1: Add example screenshot or flow summary.

<p align="center">
  <img src="../../media/BlastRadius-1.png" alt="Blast radius analysis analysis" width="800" />
</p>


<p align="center">
  <img src="../../media/BlastRadius-2.png" alt="Blast radius analysis analysis figure" width="150" />
</p>

## References
- [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool)
- [Graph tools (preview)](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#graph-tools-preview)
- [What is Microsoft Sentinel graph?](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-graph-overview)
