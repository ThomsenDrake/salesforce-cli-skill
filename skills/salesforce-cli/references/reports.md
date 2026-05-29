# Salesforce Reports

Use this guide when the user asks for a persistent Salesforce report, not just a local CSV or SOQL answer.

## Choose the Interface

| User intent | Use | Do not use |
|---|---|---|
| Count, list, export, or inspect records | `sf data query` or `sf data export bulk` | Report metadata |
| Create or change a deployable Salesforce report | Metadata API source deploy | sObject CRUD, Tooling API, Apex `insert Report` |
| Run, describe, clone, or create reports through JSON | Reports and Dashboards REST API | Tooling API |
| Verify report output | Reports REST API execute/describe, plus SOQL against `Report` | Unverified deployment output only |

Do not switch interfaces repeatedly. If an approach fails twice, stop, re-read the exact error, retrieve or inspect an existing report, and choose a different validated path.

## Persistent Report via Metadata API

This is the safest default when the user wants a Salesforce report that can be kept in source control or deployed.

### Source Format Layout

Reports are folder-backed metadata. A report source file must be inside a report folder directory.

```text
force-app/main/default/reports/<folderDeveloperName>/<reportDeveloperName>.report-meta.xml
```

Examples:

```text
force-app/main/default/reports/unfiled$public/Open_Opportunities.report-meta.xml
force-app/main/default/reports/Sales_Reports/Open_Opportunities.report-meta.xml
```

The folder must exist in the org. If deployment fails with `Cannot find folder:<name>`, deploy to an existing folder or retrieve/create the `ReportFolder` first.

Find report folders with SOQL when needed:

```bash
sf data query --query "SELECT Id, Name, DeveloperName, Type FROM Folder WHERE Type = 'Report' ORDER BY DeveloperName" --target-org my-org
```

Retrieve an existing folder or report to mirror the org's exact shape:

```bash
sf project retrieve start --metadata ReportFolder:Sales_Reports --target-org my-org
sf project retrieve start --metadata Report:Sales_Reports/Existing_Report --target-org my-org
```

### Minimal Report XML Pattern

Report columns are repeated `<columns>` elements with a nested `<field>`. Do not use `<column>` under `<columns>`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <columns>
        <field>OPPORTUNITY_NAME</field>
    </columns>
    <columns>
        <field>ACCOUNT_NAME</field>
    </columns>
    <columns>
        <field>AMOUNT</field>
    </columns>
    <description>Open opportunities with TCV greater than 5000000</description>
    <filter>
        <criteriaItems>
            <column>CLOSED</column>
            <operator>equals</operator>
            <value>False</value>
        </criteriaItems>
        <criteriaItems>
            <column>Opportunity.TCV__c</column>
            <operator>greaterThan</operator>
            <value>5000000</value>
        </criteriaItems>
    </filter>
    <format>Tabular</format>
    <name>Open Opportunities TCV GT 5M</name>
    <reportType>Opportunity</reportType>
    <scope>organization</scope>
    <showDetails>true</showDetails>
</Report>
```

Field names in report metadata are report-column names, not always sObject API names. For example, an Opportunity report can use `OPPORTUNITY_NAME`, `ACCOUNT_NAME`, `AMOUNT`, and `CLOSED`. Custom fields often use fully qualified names like `Opportunity.TCV__c`. When unsure, retrieve or describe an existing report and reuse the exact names.

### Deploy

Use `--source-dir` for file or directory paths.

```bash
sf project deploy start --source-dir force-app/main/default/reports/Sales_Reports --target-org my-org --wait 10
```

Use `--metadata` only for metadata component names, not paths.

```bash
sf project deploy start --metadata Report:Sales_Reports/Open_Opportunities --target-org my-org --wait 10
```

Do not run:

```bash
sf project deploy start --metadata force-app/main/default/reports/Sales_Reports/Open_Opportunities.report-meta.xml --target-org my-org
```

## Persistent Report via Reports REST API

The Reports and Dashboards REST API can create reports with JSON. Use this when JSON construction is simpler than Metadata XML or when you need report REST behavior. Requests with bodies must be JSON.

```json
{
  "reportMetadata": {
    "name": "Open Opportunities TCV GT 5M",
    "reportType": {
      "type": "Opportunity"
    }
  }
}
```

Send inline JSON:

```bash
sf api request rest /services/data/v66.0/analytics/reports \
  --method POST \
  --header "Content-Type: application/json" \
  --body '{"reportMetadata":{"name":"Open Opportunities TCV GT 5M","reportType":{"type":"Opportunity"}}}' \
  --target-org my-org
```

Send a JSON file with `@`. Without `@`, the CLI sends the filename string as the body.

```bash
sf api request rest /services/data/v66.0/analytics/reports \
  --method POST \
  --header "Content-Type: application/json" \
  --body @create-report.json \
  --target-org my-org
```

## Verify Reports

After creating or deploying a report, verify both that it exists and that it returns the expected shape.

```bash
sf data query --query "SELECT Id, Name, Description FROM Report WHERE Name = 'Open Opportunities TCV GT 5M'" --target-org my-org --json
```

Run or describe the report using its ID:

```bash
sf api request rest /services/data/v66.0/analytics/reports/<report-id> --target-org my-org
sf api request rest /services/data/v66.0/analytics/reports/<report-id>/describe --target-org my-org
```

`sf api request rest` is beta and does not support the normal `--json` flag. It can print beta warnings before JSON output, so do not blindly pipe it to a JSON parser unless you suppress or strip warnings.

## Anti-Patterns

- Do not use `sf data create record --sobject Report` for report creation; `Report` is not a normal createable sObject workflow.
- Do not assume `ReportType` is queryable with SOQL; use Reports REST API report-type endpoints or retrieve existing report metadata.
- Do not use Tooling API for `Report` unless you have confirmed the object exists in that API for the target org.
- Do not invent report XML tags. Validate against retrieved report metadata or a known Metadata API schema.
- Do not claim folder, filter, sorting, or row-count details until verified with SOQL or Reports REST API output.
