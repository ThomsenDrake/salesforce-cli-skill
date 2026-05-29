# Bulk Operations (Bulk API 2.0)

All bulk commands use **Bulk API 2.0** and work with CSV files.

## Bulk Export

For large exports (millions of records):

```bash
sf data export bulk \
  --query "SELECT Id, Name, Account.Name FROM Contact" \
  --output-file contacts.csv \
  --wait 10 \
  --target-org my-org

# JSON format
sf data export bulk \
  --query "SELECT Id, Name FROM Account" \
  --output-file accounts.json \
  --result-format json \
  --wait 10 \
  --target-org my-org

# Include soft-deleted records
sf data export bulk --query "SELECT Id FROM Account" --output-file out.csv --all-rows --wait 10

# Resume a timed-out export
sf data export resume --job-id <job-id>
sf data export resume --use-most-recent
```

**SOQL limitations with Bulk API 2.0**: No aggregate functions (COUNT, SUM, etc.), no TYPEOF, no nested queries beyond one level.

## Bulk Upsert

```bash
sf data upsert bulk \
  --sobject Contact \
  --file contacts.csv \
  --external-id Id \
  --wait 5 \
  --target-org my-org

# With custom external ID field
sf data upsert bulk --sobject MyObject__c --file data.csv --external-id MyField__c --wait 5

# Resume a timed-out upsert
sf data upsert resume --job-id <job-id>
sf data upsert resume --use-most-recent
```

## Bulk Import (Insert)

```bash
sf data import bulk --file accounts.csv --sobject Account --wait 10 --target-org my-org
sf data import resume --job-id <job-id>
```

## Bulk Update

```bash
sf data update bulk --file accounts.csv --sobject Account --wait 10 --target-org my-org
# CSV must have Id column as first column
```

## Bulk Delete

```bash
sf data delete bulk --sobject Account --file delete_ids.csv --wait 5 --target-org my-org
# CSV must have only one column: "Id"

# Hard delete (skip Recycle Bin — requires system permission)
sf data delete bulk --sobject Account --file delete_ids.csv --hard-delete --target-org my-org
```

## Get Bulk Job Results

```bash
sf data bulk results --job-id <job-id> --target-org my-org
```
Returns: job status, records processed, success/failure counts, and CSV result files.

## CSV Formatting Notes

- **Line endings**: `--line-ending LF` (default on macOS/Linux) or `CRLF` (Windows)
- **Column delimiter**: `--column-delimiter COMMA` (default), also: `PIPE`, `TAB`, `SEMICOLON`, `CARET`, `BACKQUOTE`
- **Setting a field to null**: Use `#N/A` in the CSV cell. **Exception**: this does NOT work for boolean/checkbox fields — omit them or set explicitly to `true`/`false`.
- **External ID for upsert**: Use `--external-id Id` to upsert by Salesforce ID, or `--external-id MyField__c` for a custom external ID field.

## Async Bulk Job Patterns

Bulk commands may time out. The pattern is:
1. Run the command with `--wait N`
2. If it times out, note the job ID
3. Run the corresponding `resume` command: `sf data <operation> resume --job-id <id>`
4. After completion, get detailed results: `sf data bulk results --job-id <id>`
