# Scenario: Blast Radius Evaluation

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

> Runtime-relevant sections (load at runtime): When to use?, the Blast Radius Evaluation instructions and investigation flow, and Sample Prompts. The remaining sections (Introduction, Expected Outputs, Example Investigation Flow, References) are human-facing README context and can be skipped at runtime.

Suggested modular run input:
- Core context: templates/context.template.md
- Active scenario: Data Exploration/Graph/BlastRadiusEvaluation.md
- Scope: provide source entity, time window, and investigation objective

## Blast Radius Evaluation - Instructions and Investigation Flow
Initialize graph context
- Call get_graph_context first to identify supported labels, properties, and graph capabilities.
Find source entity
- Resolve the source entity using graph discovery patterns before blast radius analysis.
Evaluate blast radius
- Run find_blastradius for the source entity and collect impacted paths and critical nodes.
Validate critical paths
- Confirm high-risk paths and identify choke points for containment.
Summarize scope and actions
- Produce impact summary, prioritize containment, and list recommended next steps.

## Sample Prompts
- "What is the blast radius of user <entity> if compromised?"
- "Evaluate downstream impact from device <entity> to critical assets."
- "List high-risk propagation paths from <entity> and recommend containment order."

## Example Investigation Flow
Figure 1: Add example screenshot or flow summary.

<p align="center">
  <img src="../../media/placeholder-blastradius.png" alt="Blast radius analysis placeholder" width="800" />
</p>

## References
- [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool)
- [Graph tools (preview)](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#graph-tools-preview)
- [What is Microsoft Sentinel graph?](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-graph-overview)
