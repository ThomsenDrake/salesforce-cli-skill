# Authentication

## Interactive Web Login
```bash
sf org login web --alias my-org --set-default
sf org login web --instance-url <instance-url> --set-default
sf org login web --instance-url <sandbox-instance-url>
```

## JWT Bearer Flow (CI/CD)
```bash
sf org login jwt \
  --username user@example.org \
  --jwt-key-file /path/to/server.key \
  --client-id <consumer-key> \
  --alias ci-org \
  --set-default
```
Requires: a connected app with a digital certificate and the user pre-approved.

## SFDX Auth URL (CI/CD)
```bash
# From a file (JSON with sfdxAuthUrl property, or plain text with just the URL)
sf org login sfdx-url --sfdx-url-file authFile.json --alias my-org --set-default

# From stdin
echo "force://<clientId>:<clientSecret>:<refreshToken>@<instanceUrl>" | sf org login sfdx-url --sfdx-url-stdin
```
**Format**: `force://<clientId>:<clientSecret>:<refreshToken>@<instanceUrl>` (no `https://` in instanceUrl).

To generate the auth URL from an existing authorized org:
```bash
sf org display --target-org my-org --verbose --json > authFile.json
```
The `sfdxAuthUrl` is in `result.sfdxAuthUrl`. Only works for web-server-flow authorized orgs (not JWT).

## Access Token
```bash
sf org login access-token --instance-url <instance-url>
# Non-interactive (CI):
SF_ACCESS_TOKEN=<token> sf org login access-token --instance-url <instance-url> --no-prompt
```

## List Authorized Orgs
```bash
sf org list auth          # Fast, cached locally
sf org list               # Live connection check (slower)
```

## Logout
```bash
sf org logout --target-org my-org --no-prompt
```

## CI/CD Best Practices
- JWT flow is preferred for headless environments
- SFDX auth URL from a file is the simplest CI pattern
- Access token auth works but tokens expire
- **Never** commit auth files or tokens to version control
