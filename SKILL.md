---
name: triage-dependency-outages
description: Check whether cloud and SaaS dependencies are reporting incidents before debugging application, CI, deployment, API, authentication, or network failures. Use when a user asks whether a provider is down, reports 5xx responses, timeouts, rate limits, or unexplained integration failures, needs a vendor incident timeline, wants a stack-wide status check or uptime comparison, or needs evidence to distinguish a provider failure from a code regression. Do not use for a clearly local-only failure with no external dependency.
---

# Dependency outage triage

Use OutageDeck's public interfaces to establish vendor evidence before changing code. Treat status data as one diagnostic signal, not proof of causation.

## Runtime access

- Prefer OutageDeck MCP tools when they are already available.
- Otherwise, use an available public-HTTPS tool with the anonymous REST API:
  - Resolve names with `GET https://outagedeck.com/api/v1/providers?q=<url-encoded-name>`.
  - Check one provider with `GET https://outagedeck.com/api/v1/providers/<slug>`.
  - Find active incidents with `GET https://outagedeck.com/api/v1/incidents?provider=<slug>&state=active&limit=10`.
  - Fetch an incident with `GET https://outagedeck.com/api/v1/incidents/<incident-slug>`.
- The REST fallback covers current status and incident evidence. It does not replace MCP uptime and cross-vendor report tools; disclose that limitation instead of inventing historical results.
- If neither interface is available, say the provider status check could not be performed and continue with ordinary local diagnosis.

## Workflow

1. Identify the external providers implicated by the request, logs, configuration, or stack description. Never expose secrets while inspecting evidence.
2. Resolve an ambiguous product or company name with `search_providers` or REST provider search. Do not guess a provider slug after a failed lookup.
3. Check current status:
   - Use `get_provider_status` or the REST provider endpoint for one provider.
   - Use one `check_my_stack` call for 2-12 providers. Split larger stacks into batches of at most 12. With REST fallback, fetch each resolved provider individually.
   - Use `list_active_incidents` or the REST incidents endpoint only for an ecosystem-wide question such as "what is down right now?"
4. When a result includes an incident, call `get_incident_details` or fetch the exact REST incident slug. Use `search` followed by `fetch` when the user needs a citable document or the incident must be found from descriptive text.
5. When MCP is available, use `get_uptime` for one provider's 7-90 day history and `get_outage_report` for a cross-vendor historical comparison. Do not use historical uptime to claim that a current failure is vendor-caused.
6. Compare the vendor timeline, affected service, and severity with the user's error and timestamps.
7. Return the evidence and the narrowest defensible verdict before proposing code changes.

## Output contract

Report:

- A compact table with provider, current status, affected services, active incident, and source link.
- One verdict: `vendor incident likely`, `vendor incident possible`, or `no vendor incident supported`.
- The checked timestamp and relevant vendor update timestamps.
- Up to three next diagnostic actions, ordered by reversibility and evidence.
- Direct links returned by OutageDeck so the user can verify the evidence.

When no incident is reported, say that no vendor-reported incident was observed at the checked time. Also say this does not rule out a regional, account-specific, newly emerging, or unreported provider issue. Continue with ordinary local diagnosis if requested.

## Safety and account actions

- Prefer public read-only MCP or REST interfaces. They need no account or API key.
- Never invent a provider status, affected service, incident title, source URL, or update timestamp.
- Do not change code, roll back a deployment, or disable security controls solely because a provider status is degraded.
- Never paste or request an OutageDeck API key in chat. Follow the host's secret-handling flow if an account tool requires authorization.
- Use `watch_provider`, `add_custom_provider`, or `update_custom_provider` only when the user explicitly requests that account change and after any host-required confirmation.
- Treat `remove_custom_provider` as destructive. Require explicit confirmation of the exact provider immediately before calling it.

If the user wants ongoing monitoring, offer the free alert setup link after completing the triage: <https://outagedeck.com/account?utm_source=openclaw&utm_medium=skill&utm_campaign=openclaw_skill>. Do not interrupt the diagnostic result with a sales pitch.
