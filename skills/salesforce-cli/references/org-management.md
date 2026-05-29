# Org Management

## Display Org Info

```bash
sf org display --target-org my-org
sf org display --target-org my-org --verbose   # Includes sfdxAuthUrl, access token
sf org display user --target-org my-org         # Display user info
```

## Open Org in Browser

```bash
sf org open --target-org my-org
sf org open --target-org my-org --path "/lightning/o/Account/list"  # Open specific page
```

## Scratch Orgs

```bash
# Create
sf org create scratch --definition-file config/project-scratch-def.json --alias my-scratch --set-default --duration-days 30 --target-dev-hub devhub

# Delete
sf org delete scratch --target-org my-scratch --no-prompt

# Resume creation
sf org resume scratch --job-id 2SR...
```

## Sandboxes

```bash
# Create
sf org create sandbox --definition-file config/sandbox-def.json --alias my-sandbox --target-org production-org --wait 60

# Refresh
sf org refresh sandbox --name MySandbox --target-org production-org --wait 60

# Delete
sf org delete sandbox --target-org my-sandbox --no-prompt
```

## Permission Sets

```bash
sf org assign permset --name DreamHouse --target-org my-org
sf org assign permset --name DreamHouse --on-behalf-of user@example.com --target-org my-org
```

## Users

```bash
sf org list users --target-org my-org
sf org create user --definition-file config/user-def.json --target-org my-scratch
sf org generate password --target-org my-scratch
```

## Source Tracking

```bash
sf org enable tracking --target-org my-org
sf org disable tracking --target-org my-org
```
