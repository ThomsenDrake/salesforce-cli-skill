# Metadata Deployment & Retrieval

## Deploy

```bash
# Deploy a source directory
sf project deploy start --source-dir force-app --target-org my-org --wait 10
sf project deploy start --source-dir force-app/main/default/classes/MyClass.cls --target-org my-org --wait 10

# Deploy specific metadata types
sf project deploy start --metadata ApexClass --target-org my-org
sf project deploy start --metadata ApexClass:MyClass --target-org my-org

# Deploy from a manifest (package.xml)
sf project deploy start --manifest manifest/package.xml --target-org my-org

# Deploy with test execution
sf project deploy start --source-dir force-app --test-level RunLocalTests --target-org my-org
sf project deploy start --source-dir force-app --test-level RunSpecifiedTests --tests MyTest --target-org my-org

# Validate only (don't actually deploy)
sf project deploy validate --source-dir force-app --target-org my-org --test-level RunLocalTests

# Quick deploy (after successful validation)
sf project deploy quick --job-id 0Af... --target-org my-org

# Ignore conflicts with source tracking
sf project deploy start --source-dir force-app --ignore-conflicts --target-org my-org

# Preview (dry run)
sf project deploy preview --source-dir force-app --target-org my-org
```

**Test levels**: `NoTestRun`, `RunSpecifiedTests`, `RunLocalTests`, `RunAllTestsInOrg`

### Deploy Flag Semantics

`--source-dir`, `--metadata`, `--manifest`, and `--metadata-dir` are mutually exclusive ways to select deployment contents.

| Flag | Use for | Example |
|---|---|---|
| `--source-dir` / `-d` | Local source-format file or directory paths | `-d force-app/main/default/classes/MyClass.cls` |
| `--metadata` / `-m` | Metadata type names or `Type:MemberName` specs | `-m ApexClass:MyClass` |
| `--manifest` / `-x` | A source-format `package.xml` manifest | `-x manifest/package.xml` |
| `--metadata-dir` | Metadata API format directories or ZIP roots | `--metadata-dir mdapi-output` |

Do not pass a file path to `--metadata`. This is wrong:

```bash
sf project deploy start --metadata force-app/main/default/classes/MyClass.cls --target-org my-org
```

Use `--source-dir` for paths:

```bash
sf project deploy start --source-dir force-app/main/default/classes/MyClass.cls --target-org my-org
```

### Folder-Backed Metadata

Some metadata types are folder-backed. Their source files must live under a folder directory, and their metadata names include the folder.

| Type | Source path pattern | Metadata selector pattern |
|---|---|---|
| `Report` | `force-app/main/default/reports/<folder>/<name>.report-meta.xml` | `Report:<folder>/<name>` |
| `Dashboard` | `force-app/main/default/dashboards/<folder>/<name>.dashboard-meta.xml` | `Dashboard:<folder>/<name>` |

If deployment says `Cannot find folder:<name>`, the folder name in the source path is not a deployable folder in the target org. Retrieve or create the folder metadata first, or deploy into an existing folder.

## Retrieve

```bash
# Retrieve by source directory
sf project retrieve start --source-dir force-app --target-org my-org

# Retrieve specific metadata
sf project retrieve start --metadata ApexClass --target-org my-org
sf project retrieve start --metadata ApexClass:MyClass --target-org my-org

# Retrieve from manifest
sf project retrieve start --manifest manifest/package.xml --target-org my-org

# Preview retrieval
sf project retrieve preview --target-org my-org
```

## Other Project Commands

```bash
# Generate a new DX project
sf project generate --name my-project

# Generate a manifest (package.xml)
sf project generate manifest --source-dir force-app --name package.xml

# Convert between formats
sf project convert mdapi --root-dir mdapi-output --output-dir force-app
sf project convert source --root-dir force-app --output-dir mdapi-output

# Delete source from org and project
sf project delete source --metadata ApexClass:OldClass --target-org my-org
```
