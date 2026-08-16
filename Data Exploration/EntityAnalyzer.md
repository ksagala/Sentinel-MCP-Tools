# Scenario: Entity Analyzer - Analyze a specific entity’s relationships and signals
This guide shows you how to use Microsoft Sentinel's Entity Analyzer tools to perform AI-powered risk assessment and analysis of user accounts and URLs through MCP clients.

> [!NOTE]
> The Entity Analyzer belongs to the **data exploration tool collection**
> (`https://sentinel.microsoft.com/mcp/data-exploration`). It is **not** a graph tool and
> there is **no separate "Sentinel Graph" MCP server endpoint**. For blast radius, path
> discovery, exposure perimeter and node discovery, see the
> [Graph tools guide](./Graph/README.md) — those tools have
> different constraints (no `workspaceId`, no time window, no UPN support).

## Introduction

The **Entity Analyzer** is a tool in Microsoft Sentinel's Data Exploration MCP collection that uses AI to analyze your organization's data and provide intelligent verdicts and detailed insights on URLs, domains, and user entities. This powerful capability eliminates the need for manual data collection and complex integrations typically required for enriching and investigating entities.

### What Entity Analyzer Does

Entity Analyzer leverages artificial intelligence to reason over multiple data sources in your Microsoft Sentinel data lake:

- **`analyze_user_entity`**: Analyzes user authentication patterns, behavioral anomalies, activity within your organization, alert associations, and more to provide a comprehensive verdict and risk analysis.
- **`analyze_url_entity`**: Evaluates URLs and domains using Microsoft threat intelligence, your custom threat intelligence from Sentinel's Threat Intelligence Platform (TIP), click/email/connection activity, and presence in watchlists to deliver a verdict and analysis.
- **`get_entity_analysis`**: Retrieves the results of an analysis operation initiated by either of the above tools.

The entity analyzer runs asynchronously and may require a few minutes to generate results. The workflow involves starting an analysis (which returns a job identifier) and then retrieving the results using that identifier.

> [!NOTE]
> Some tenants also surface **`analyze_application_entity`**. This tool is **not yet
> covered by the public Microsoft Learn documentation** — treat it as unverified/preview
> and do not build production flows on it until documented.

### Key Benefits

- **Automated Enrichment**: No manual data gathering or complex integrations needed
- **AI-Powered Insights**: Intelligent reasoning over multiple data sources simultaneously
- **Verdict and Analysis**: Clear risk assessment with detailed supporting evidence
- **Time-Saving**: Eliminates hours of manual investigation and correlation
- **Standardized Output**: Consistent, automation-friendly analysis format
- **Fast Triage**: Quickly assess entity risk during incident response


## When to use?
- **Entity Analyzer**: AI-powered asynchronous analysis for specific users (Microsoft Entra object ID or UPN) or URLs. Returns verdict and narrative analysis based on authentication patterns, behavioral analytics, threat intelligence, and activity data. Requires specific tables (SigninLogs, AlertEvidence, CloudAppEvents, IdentityInfo) with a 7-day window limit for users. Best for fast triage, standardized risk assessment, and automated entity enrichment.
- **Manual Investigation** (search_tables + query_lake): Analyst-driven synchronous KQL queries across any tables for broad discovery of multiple entity types (users, devices, IPs, file hashes, URLs). Suited for relationship mapping, lateral movement analysis, custom queries, and investigations beyond the 7-day window or across entity types not supported by Entity Analyzer.
- **Graph tools** ([Graph](./Graph/)): Relationship and reachability questions — downstream impact, lateral movement feasibility, who can reach a critical asset. Use these *after* the analyzer establishes that an entity is risky, to scope the impact.


## Using Entity Analyzer

Use this file as the active scenario module together with the core context template.
Do not merge this file into context.md. Attach/load both files and set this file as the active scenario.

> Runtime-relevant sections (load at runtime): When to use?, Tool reference, the Entity Analyzer instructions flow, Requirements and Limitations, Sample Prompts, and Tips. The remaining sections (Introduction, What Entity Analyzer Does, Key Benefits, figures) are human-facing README context and can be skipped at runtime.

Suggested modular run input:
- Core context: templates/context.template.md
- Active scenario: Data Exploration/EntityAnalyzer.md
- Scope: provide objective, time window (**≤ 7 days for users**), and entities (user object ID / UPN, or URL/domain)

**Supported entity types:**
- Users (via Microsoft Entra object ID or User Principal Name (UPN)). On-premises Active Directory–only users are not supported.
- URLs and domains

**What you get:**
- User analysis: Authentication patterns, behavioral anomalies, alert associations, activity within your organization, and risk verdict
- URL analysis: Microsoft threat intelligence, custom TIP indicators, click/email/connection activity, watchlist presence, and risk verdict

## Tool reference

| Tool | Parameters |
|---|---|
| `analyze_user_entity` | Microsoft Entra object ID **or UPN** (required), `startTime` (required), `endTime` (required), `workspaceId` (optional) |
| `analyze_url_entity` | URL (required), `startTime` (required), `endTime` (required), `workspaceId` (optional) |
| `get_entity_analysis` | `analysisId` (required) |

> [!IMPORTANT]
> Unlike the graph tools, the entity analyzer **does** accept an optional `workspaceId`
> and **requires** an explicit time window. It also **does** accept UPNs — the "no UPN"
> restriction applies only to graph identity lookups.

**Workflow:**
1. Call `analyze_user_entity` or `analyze_url_entity` with entity identifier, startTime, endTime, and optional workspaceId
2. Receive an `analysisId` in response
3. Call `get_entity_analysis` with the `analysisId` to retrieve the verdict and detailed analysis. The tool self-polls for a few minutes, but its internal timeout may be insufficient — **re-invoke it several times** for long-running analyses rather than treating the first non-result as a failure


## Entity Analyzer – Use Sentinel Data Exploration MCP server
Enrich and assess user entities
- When asked to understand, analyze, or assess a user, call `analyze_user_entity` with the Microsoft Entra object ID (or UPN) and a time window of **at most 7 days** to get an AI-generated risk verdict, behavioral insights, and activity analysis
Identify at-risk users
- When asked to find users at risk or identify risky users, call `analyze_user_entity` for relevant users and rank them by risk verdict to surface the top at-risk accounts
Investigate alert-associated users
- When investigating security alerts (password spray, suspicious sign-ins, etc.), call `analyze_user_entity` for each implicated user to determine if they are compromised or at risk
Enrich and analyze URLs and domains
- When asked to analyze a URL or domain, call `analyze_url_entity` to get comprehensive threat intelligence, activity data, and a risk verdict from Microsoft and your organization's data
Analyze IOCs from threat intelligence
- When provided with threat reports or IOC lists containing URLs, extract and analyze each URL with `analyze_url_entity` to understand everything Microsoft knows about them
Poll for analysis results
- After starting an analysis, call `get_entity_analysis` with the returned analysisId; if results aren't ready, poll multiple times as needed
Handle missing prerequisites
- If `analyze_user_entity` returns an error listing missing tables, **report the named tables and fall back to `search_tables` + `query_lake` evidence** instead of retrying the analyzer
Render unprocessed results
- Include "render the results as returned exactly from the tool" in prompts to ensure the analyzer's output is provided without additional MCP client processing. Do not re-reason over or restate the verdict
Defang indicators
- Defang all malicious/suspicious URLs and domains in prompts and output (`hxxps://evil[.]com/path`, `evil[.]com`)
Escalate to graph when scope matters
- Once an entity is assessed as risky, hand the resolved entity to
  [Blast Radius Evaluation](./Graph/BlastRadiusEvaluation.md) or use the exposure
  perimeter tool described in the [Graph tools guide](./Graph/README.md) to determine impact.
  Resolve the UPN to a display name / object ID first — graph tools do not accept UPNs


## Requirements and Limitations
- **MCP access roles**: Access to the Sentinel MCP tools requires at least one of the following roles: **Security Administrator**, **Security Operator**, or **Security Reader**.
- **Security Copilot required**: The Entity Analyzer consumes Security Compute Units (SCUs), so it requires the **Security Copilot Contributor** role to run analyses. The **Security Copilot Owner** role is optional and only needed to view and monitor SCU usage. See [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#entity-analyzer).
- **Cost**: Every analysis consumes SCUs and counts against the entity analyzer's [preview thresholds](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-billing#microsoft-sentinel-entity-analyzer-tool-1). Scope entity lists before running batches.
- User analysis requires: AlertEvidence, SigninLogs, CloudAppEvents, IdentityInfo (`IdentityInfo` is populated only for tenants with Microsoft Defender for Identity, Microsoft Defender for Cloud Apps, or Microsoft Defender for Endpoint P2); works best with AADNonInteractiveUserSignInLogs and BehaviorAnalytics
- URL analysis works best with: EmailUrlInfo, UrlClickEvents, ThreatIntelIndicators, Watchlist, DeviceNetworkEvents
- **Graceful degradation**: If required user tables are missing, `analyze_user_entity` returns an error listing the missing tables and onboarding links. For URLs, `analyze_url_entity` still returns a verdict with a disclaimer noting any missing tables.
- Only users with a Microsoft Entra object ID are analyzed; on-premises Active Directory–only users are not supported.
- Maximum 7-day time window for user analysis to ensure accuracy
- Avoid running more than 5 concurrent analyses to prevent timeouts and increased latency
- **Asynchronous by design**: `get_entity_analysis` may need to be invoked multiple times before results are available
- **Undocumented tool**: `analyze_application_entity` may appear in the tool list but is absent from public documentation — treat as unverified


## Sample Prompts
- "Help me understand if the user `<Microsoft Entra object ID>` is compromised"
- "Find the top three users that are at risk and explain why they're at risk"
- "Investigate users with a password spray alert in the last seven days and tell me if any of them are compromised"
- "Find all the URL IOCs from `<threat analytics report>` and analyze them to tell me everything Microsoft knows about them"
- "Analyze the URL `hxxps://suspicious-domain[.]com` for threats"
- "What is known about this user `<object ID>` in terms of security risk?"


## Tips
- Add "render the results as returned exactly from the tool" to prompts for unprocessed analyzer output
- Keep user analysis time window within 7 days for best accuracy
- Limit to 5 concurrent analyses to avoid timeouts
- Re-invoke `get_entity_analysis` rather than assuming a single poll is enough
- Defang URLs and domains in both prompts and output

Figure 1: Behavioral analysis of a user entity in VS Code using the Sentinel Data Exploration MCP tools.

<p align="center">
  <img src="../media/EntityAnalyzer-1.png" alt="URL analysis by Entity Analyzer through VS Code" width="600" />
 </p>

<p align="center">
  <img src="../media/EntityAnalyzer-url.png" alt="URL analysis by Entity Analyzer through VS Code" width="600" />
 </p>

## References
- [Data exploration tool collection in Microsoft Sentinel MCP server](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool)
- [Entity analyzer](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-data-exploration-tool#entity-analyzer)
- [Understand authentication in Microsoft Security Copilot](https://learn.microsoft.com/en-us/copilot/security/authentication)
- [Microsoft Sentinel MCP billing — entity analyzer](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-billing#microsoft-sentinel-entity-analyzer-tool-1)
