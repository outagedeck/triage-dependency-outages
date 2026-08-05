# Dependency Outage Triage

An installable agent skill for checking whether a cloud or SaaS dependency has a live incident before you roll back a deployment, rotate credentials, or change working code.

It uses [OutageDeck](https://outagedeck.com/?utm_source=github&utm_medium=repository&utm_campaign=openclaw_skill) as a read-only evidence source for current status, affected services, incident timelines, and source links across 170+ providers. Public checks need no account or API key.

## Install

For agents that support the Agent Skills format:

```bash
npx skills add https://github.com/outagedeck/triage-dependency-outages --skill triage-dependency-outages
```

For OpenClaw:

```bash
openclaw skills install git:outagedeck/triage-dependency-outages@v0.1.0
```

The OpenClaw command pins the tested `v0.1.0` release. The generic installer reads the skill directly from this public repository.

## What the skill does

When an application, CI job, deployment, API, authentication flow, or network call starts failing, the skill:

1. identifies the external providers implicated by the evidence;
2. checks service-scoped status and active incidents through OutageDeck MCP or its public REST API;
3. compares vendor timestamps and affected services with the observed failure;
4. returns a compact evidence table and one conservative verdict;
5. recommends reversible next diagnostics without treating provider status as proof of causation.

It explicitly distinguishes a vendor-reported incident from regional, account-specific, newly emerging, and local failures.

## Example prompts

```text
Is GitHub Actions down, or did our deployment workflow regress?

Check OpenAI, Anthropic, AWS, and Cloudflare before I change this retry logic.

Build an incident timeline for the providers implicated by these 503 errors.
```

## Runtime access

The skill prefers an already configured OutageDeck MCP server. Without MCP, it falls back to the keyless public endpoints documented in [`SKILL.md`](./SKILL.md), including:

```text
GET https://outagedeck.com/api/v1/providers?q=<name>
GET https://outagedeck.com/api/v1/providers/<slug>
GET https://outagedeck.com/api/v1/incidents?provider=<slug>&state=active&limit=10
```

The public API is suitable for lightweight checks. Teams that want ongoing email, Slack, Teams, Discord, or webhook notification can [configure provider alerts](https://outagedeck.com/account?utm_source=github&utm_medium=repository&utm_campaign=openclaw_skill).

## Safety

- No OutageDeck key is required for public checks.
- The skill never asks users to paste credentials into chat.
- It does not change code, roll back deployments, or disable security controls based only on external status.
- Account-changing OutageDeck tools require an explicit user request; provider removal also requires confirmation.

## License

[MIT](./LICENSE)
