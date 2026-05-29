# Salesforce CLI Skill

An [Agent Skill](https://agentskills.io/home) that gives coding agents a comprehensive, battle-tested reference for operating the **Salesforce CLI (`sf`)** from the command line. It follows the open Agent Skills standard, so it works with any skills-compatible agent — Claude Code, Cursor, Codex, Gemini CLI, OpenCode, GitHub Copilot, and [many more](https://agentskills.io/clients).

It covers authentication, SOQL/SOSL queries, record CRUD, bulk data operations, metadata deployment/retrieval, org management, Apex execution, schema inspection, reports, and raw REST/GraphQL API calls — with an execution-first operating protocol designed to keep agents from flailing between Salesforce interfaces.

> Built for Salesforce CLI v2 (`sf` ≥ 2.123.1). The legacy `sfdx` executable is deprecated — this skill always uses `sf`.

## What's inside

```
skills/salesforce-cli/
├── SKILL.md                          # Entry point: conventions, protocol, quick reference
└── references/
    ├── authentication.md             # Web login, JWT, SFDX auth URL, access token, CI/CD
    ├── bulk-operations.md            # Bulk API 2.0 upsert/import/update/delete/export
    ├── org-management.md             # Org display, scratch orgs, sandboxes, users, perms
    ├── metadata-deployment.md        # Deploy/retrieve source, manifests, test levels
    ├── reports.md                    # Create, deploy, run, describe, verify reports
    ├── apex.md                       # Execute Apex, run tests, logs, codegen
    └── api-requests.md               # Raw REST API and GraphQL API calls
```

`SKILL.md` is the index an agent loads first; it links out to the topic-specific reference files so detail is pulled into context only when relevant.

## Capabilities at a glance

- **Authentication** — browser, JWT, SFDX auth URL, and access-token flows, including CI/CD patterns.
- **Data / SOQL & SOSL** — `sf data query`, `sf data search`, single-record CRUD, and tree import/export.
- **Bulk data (Bulk API 2.0)** — upsert, import, update, delete, and export of large datasets with async/resume patterns.
- **Schema inspection** — `sf sobject describe`, object/record-count listing, metadata types, org limits.
- **Metadata** — deploy and retrieve source, manifest (`package.xml`) workflows, and test levels.
- **Reports** — folder-backed report metadata plus the Reports REST API.
- **Apex** — run anonymous Apex, execute tests, and pull/tail debug logs.
- **Raw API** — REST and GraphQL requests via `sf api request`.

## Prerequisites

- [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) v2 (`sf`) version **2.123.1 or newer**
- Access to a Salesforce org (production, sandbox, or scratch org)
- Default Salesforce API version targeted by this skill: **66.0**

Verify your install:

```bash
sf version
```

## Installation

The skill lives at `skills/salesforce-cli/`, following the open Agent Skills layout. Pick whichever installer matches your workflow.

### Using the `skills` CLI ([agentskills.io](https://agentskills.io/home))

The `skills` CLI auto-detects which coding agents you have installed and syncs the skill to each.

```bash
# Install into the current project
npx skills add ThomsenDrake/salesforce-cli-skill

# Or install globally for all projects
npx skills add ThomsenDrake/salesforce-cli-skill -g
```

### Using the `openskills` CLI

```bash
npx openskills install ThomsenDrake/salesforce-cli-skill
npx openskills sync
```

Add `--global` to the `install` command to make it available across all projects.

### Manual

Copy the skill directory into your agent's skills folder:

```bash
git clone https://github.com/ThomsenDrake/salesforce-cli-skill.git

# Claude Code (project-local)
cp -R salesforce-cli-skill/skills/salesforce-cli ./.claude/skills/salesforce-cli

# Claude Code (user-global)
cp -R salesforce-cli-skill/skills/salesforce-cli ~/.claude/skills/salesforce-cli
```

## Usage

Once installed, the skill activates automatically whenever a task involves interacting with a Salesforce org via the CLI. A few representative commands it documents:

```bash
# Log in to an org and set it as the default
sf org login web --alias my-org --set-default

# Run a SOQL query as JSON
sf data query --query "SELECT Id, Name FROM Account LIMIT 10" --target-org my-org --json

# Bulk upsert from a CSV using an external ID
sf data upsert bulk --sobject Account --file accounts.csv --external-id ExternalId__c --target-org my-org

# Deploy metadata from source
sf project deploy start --source-dir force-app --target-org my-org

# Run anonymous Apex
sf apex run --file script.apex --target-org my-org
```

See `skills/salesforce-cli/SKILL.md` for the full quick-reference table and operating protocol.

## Operating principles

The skill encodes a few hard-won conventions for agents:

- **Classify intent first** (data answer vs. metadata deploy vs. REST vs. Apex vs. admin) and commit to one interface rather than trial-and-error across them.
- **Inspect authoritative shape** (`--help`, `sf sobject describe`, retrieved metadata, REST describe) when an approach fails twice — don't guess.
- **Verify before claiming success** using SOQL, deploy output, or REST results.
- **Use placeholders only** — examples never contain real usernames, org IDs, record IDs, auth URLs, tokens, or customer domains.

## License

No license file is currently included. Add one if you intend others to reuse this work.
