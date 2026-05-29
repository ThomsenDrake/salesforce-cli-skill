---
name: salesforce-cli
description: Comprehensive reference for operating the Salesforce CLI (`sf`) from coding agents. Covers authentication, SOQL/SOSL queries, record CRUD, bulk data operations, metadata deployment/retrieval, org management, Apex execution, object inspection, and raw API calls. Use this skill whenever a task involves interacting with Salesforce orgs via the command line.
metadata:
  version: "1.0.0"
  sf_cli_version: ">=2.123.1"
  salesforce_api_version: "66.0"
  tags: "salesforce, cli, devops, data, soql"
---

# Salesforce CLI (`sf`) Reference for Coding Agents

This skill provides everything a coding agent needs to operate the Salesforce CLI v2 (`sf`). The `sfdx` executable is deprecated — always use `sf`.

All examples in this skill must use placeholders or public example values. Do not add real usernames, org IDs, record IDs, auth URLs, tokens, keys, passwords, or customer-specific domains to these files.

## Detailed References

For in-depth coverage of specific topics, see the reference files:

- **[Authentication](references/authentication.md)**: Web login, JWT, SFDX auth URL, access token, CI/CD patterns
- **[Bulk Operations](references/bulk-operations.md)**: Bulk API 2.0 upsert/import/update/delete/export, CSV formatting, async patterns
- **[Org Management](references/org-management.md)**: Org display, scratch orgs, sandboxes, users, permission sets, source tracking
- **[Metadata Deployment](references/metadata-deployment.md)**: Deploy/retrieve source, manifests, test levels, project commands
- **[Reports](references/reports.md)**: Create, deploy, run, describe, and verify Salesforce reports
- **[Apex](references/apex.md)**: Execute Apex, run tests, logs, code generation
- **[Raw API Requests](references/api-requests.md)**: REST API, GraphQL API

---

## Global Conventions

### Operating Protocol

- Classify the user's intent before choosing commands: data answer, metadata deploy, REST API operation, Apex execution, or org administration.
- Use one Salesforce interface for the task unless the error proves it is the wrong interface. Do not bounce between SOQL, Metadata API, Tooling API, Apex, and REST by trial and error.
- If the same approach fails twice, stop and inspect authoritative shape: command help, `sf sobject describe`, `sf org list metadata-types`, retrieved metadata, or the relevant REST describe endpoint.
- For persistent metadata such as reports, flows, fields, and permission sets, prefer source-format files plus `sf project deploy start --source-dir` or `--metadata Type:Name`.
- For raw REST calls, remember `sf api request rest` is beta, has no normal `--json` flag, and requires `--body @file.json` to send a file as the request body.
- Verify before claiming success. Use SOQL, retrieve, deploy output, or REST describe/run results to confirm the created or changed artifact.

### Target Org

Almost every command requires a target org. Specify it with:
- `--target-org <alias-or-username>` (short: `-o`)
- Or set a default: `sf config set target-org=my-alias`

### JSON Output

Add `--json` to any command to get machine-readable JSON output. Always returns:
```json
{ "status": 0, "result": { ... }, "warnings": [] }
```
Parse with `jq` or language-native JSON parsing. `status`: 0 = success, 1 = error.

### Common Flag Patterns

| Flag | Short | Purpose |
|------|-------|---------|
| `--target-org` | `-o` | Target org alias or username |
| `--target-dev-hub` | `-v` | Dev Hub org (for scratch org commands) |
| `--json` | | JSON output |
| `--api-version` | | Override API version |
| `--wait` | `-w` | Minutes to wait for async operations |

---

## Authentication (Summary)

```bash
# Browser login
sf org login web --alias my-org --set-default

# JWT (CI/CD)
sf org login jwt --username X --jwt-key-file server.key --client-id KEY --alias ci-org

# SFDX auth URL (CI/CD — simplest pattern)
sf org login sfdx-url --sfdx-url-file authFile.json --alias my-org

# Access token
SF_ACCESS_TOKEN=<token> sf org login access-token --instance-url <instance-url> --no-prompt

# List / logout
sf org list auth
sf org logout --target-org my-org --no-prompt
```

See **[references/authentication.md](references/authentication.md)** for full details.

---

## Data: SOQL Queries

### `sf data query`

```bash
# Basic query
sf data query --query "SELECT Id, Name FROM Account LIMIT 10" --target-org my-org

# JSON output for scripting
sf data query --query "SELECT Id, Name FROM Account" --target-org my-org --json

# CSV output to file
sf data query --query "SELECT Id, Name FROM Account" --result-format csv --output-file accounts.csv --target-org my-org

# From a file
sf data query --file query.soql --target-org my-org

# Include deleted records
sf data query --query "SELECT Id FROM Account" --all-rows --target-org my-org

# Tooling API objects
sf data query --query "SELECT Name FROM ApexTrigger" --use-tooling-api --target-org my-org
```

**Key flags:**

| Flag | Short | Description |
|------|-------|-------------|
| `--query` | `-q` | SOQL query string |
| `--file` | `-f` | File containing the SOQL query |
| `--result-format` | `-r` | `human` (default), `csv`, `json` |
| `--output-file` | | Write results to a file |
| `--use-tooling-api` | `-t` | Query Tooling API objects |
| `--all-rows` | | Include soft-deleted records |

**Limits**: For queries returning >10,000 records, use `sf data export bulk` instead (see [references/bulk-operations.md](references/bulk-operations.md)).

### `sf data export bulk` (Bulk API 2.0)

```bash
sf data export bulk --query "SELECT Id, Name FROM Contact" --output-file contacts.csv --wait 10 --target-org my-org
sf data export resume --use-most-recent   # Resume timed-out export
```

---

## Data: SOSL Search

```bash
sf data search --query "FIND {Anna Jones} IN Name Fields RETURNING Contact (Name, Phone)" --target-org my-org
sf data search --file search.sosl --result-format csv --target-org my-org
```

---

## Data: Single Record CRUD

### Create
```bash
sf data create record --sobject Account --values "Name='Universal Containers' Website=www.example.com" --target-org my-org
```

### Read
```bash
sf data get record --sobject Account --record-id <record-id> --target-org my-org
sf data get record --sobject Account --where "Name=Acme" --target-org my-org
```

### Update
```bash
sf data update record --sobject Account --record-id <record-id> --values "Name='New Acme'" --target-org my-org
sf data update record --sobject Account --where "Name='Old Acme'" --values "Name='New Acme'" --target-org my-org
```

### Delete
```bash
sf data delete record --sobject Account --record-id <record-id> --target-org my-org
sf data delete record --sobject Account --where "Name=Acme" --target-org my-org
```

### Field Value Format
- Format: `<fieldName>=<value>`, all pairs in one double-quoted string, space-delimited
- Values with spaces: use single quotes inside: `Name='My Company'`
- Dates: `StartDate=2024-01-01T00:00:00.000+0000`

---

## Data: Tree Import/Export

```bash
# Export with plan
sf data export tree --query "SELECT Id, Name, (SELECT Name FROM Contacts) FROM Account" --plan --output-dir export-dir --target-org my-org

# Import (order matters: parent objects before children)
sf data import tree --plan Account-Contact-plan.json --target-org my-org
sf data import tree --files Account.json,Contact.json --target-org my-org
```

Maximum: 2,000 records per export.

---

## Object & Schema Inspection

```bash
# Describe an object (returns fields, record types, picklists, relationships)
sf sobject describe --sobject Account --target-org my-org
sf sobject describe --sobject MyObject__c --target-org my-org

# List objects
sf sobject list --sobject all --target-org my-org
sf sobject list --sobject custom --target-org my-org

# Record counts
sf org list sobject record-counts --target-org my-org
sf org list sobject record-counts --sobject Account --sobject Contact --target-org my-org

# Metadata types & org limits
sf org list metadata-types --target-org my-org
sf org list metadata --metadata-type CustomObject --target-org my-org
sf org list limits --target-org my-org

# Generate schema files
sf schema generate field
sf schema generate sobject
sf schema generate platformevent
sf schema generate tab
```

---

## Configuration & Aliases

```bash
# Set defaults
sf config set target-org=my-org
sf config set target-org=my-org --global
sf config set target-dev-hub=devhub

# View / unset
sf config list
sf config get target-org
sf config unset target-org

# Aliases
sf alias set my-org=user@example.com
sf alias list
sf alias unset my-org
```

**Config variables**: `target-org`, `target-dev-hub`, `org-api-version`, `org-instance-url`, `org-max-query-limit`

---

## Common Pitfalls

1. **SOQL quoting**: Use outer double quotes, inner single quotes: `--query "SELECT Id FROM Account WHERE Name = 'Acme'"`. For complex queries, use `--file`.

2. **Bulk API SOQL limitations**: No aggregate functions (`COUNT`, `SUM`), no `TYPEOF`, no deeply nested subqueries. Use `sf data query` for aggregates.

3. **Boolean fields in CSV**: `#N/A` nulls most field types but NOT booleans. Omit them or set to `true`/`false`.

4. **Large query results**: `sf data query` hits ~10,000 record limit. Use `sf data export bulk` for large datasets.

5. **Tooling API**: Some objects only exist in Tooling API (`ApexCodeCoverage`, `TraceFlag`). Add `--use-tooling-api` flag.

6. **Async bulk jobs**: If `--wait` times out, use `sf data <op> resume --job-id <id>` then `sf data bulk results --job-id <id>`.

7. **Deploy selector confusion**: Use `--source-dir` for local paths and `--metadata` for type/name selectors like `ApexClass:MyClass`. Do not pass file paths to `--metadata`.

8. **REST body files**: Use `--body @file.json` for request bodies from files. `--body file.json` sends the literal string `file.json`.

9. **Reports are special**: Salesforce reports are folder-backed metadata and also have Reports REST API resources. See [references/reports.md](references/reports.md) before creating or modifying reports.

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Login (browser) | `sf org login web --alias X --set-default` |
| Login (JWT/CI) | `sf org login jwt --username X --jwt-key-file Y --client-id Z` |
| Login (auth URL) | `sf org login sfdx-url --sfdx-url-file X` |
| SOQL query | `sf data query -q "SELECT ..." -o org` |
| SOQL query (JSON) | `sf data query -q "SELECT ..." -o org --json` |
| SOSL search | `sf data search -q "FIND {...} RETURNING ..." -o org` |
| Create record | `sf data create record -s Object -v "Field=Value" -o org` |
| Get record | `sf data get record -s Object -i <id> -o org` |
| Update record | `sf data update record -s Object -i <id> -v "Field=Value" -o org` |
| Delete record | `sf data delete record -s Object -i <id> -o org` |
| Bulk upsert | `sf data upsert bulk -s Object -f file.csv -i ExternalId -o org` |
| Bulk import | `sf data import bulk -s Object -f file.csv -o org` |
| Bulk update | `sf data update bulk -s Object -f file.csv -o org` |
| Bulk delete | `sf data delete bulk -s Object -f ids.csv -o org` |
| Bulk export | `sf data export bulk -q "SELECT ..." --output-file out.csv -o org` |
| Bulk job results | `sf data bulk results -i <job-id> -o org` |
| Describe object | `sf sobject describe -s Object -o org` |
| List objects | `sf sobject list -s all -o org` |
| Record counts | `sf org list sobject record-counts -o org` |
| Org limits | `sf org list limits -o org` |
| Metadata types | `sf org list metadata-types -o org` |
| Deploy source | `sf project deploy start -d force-app -o org` |
| Deploy metadata | `sf project deploy start -m ApexClass -o org` |
| Deploy manifest | `sf project deploy start -x package.xml -o org` |
| Deploy report source | `sf project deploy start -d force-app/main/default/reports/<folder> -o org` |
| Deploy report by name | `sf project deploy start -m Report:<folder>/<reportName> -o org` |
| Retrieve source | `sf project retrieve start -d force-app -o org` |
| Retrieve metadata | `sf project retrieve start -m ApexClass -o org` |
| Run Apex | `sf apex run -f script.apex -o org` |
| Run tests | `sf apex run test --class-names Test -o org -w 10` |
| View logs | `sf apex get log --number 5 -o org` |
| Tail logs | `sf apex tail log -o org` |
| REST API | `sf api request rest 'services/data/v66.0/...' -o org` |
| REST API body file | `sf api request rest 'services/data/v66.0/...' -X POST -b @body.json -o org` |
| GraphQL | `sf api request graphql --body query.gql -o org` |
| Open org | `sf org open -o org` |
| Org info | `sf org display -o org` |
| List authed orgs | `sf org list auth` |
| Set config | `sf config set target-org=my-org` |
| Set alias | `sf alias set my-org=user@example.com` |
