# Raw API Requests

## REST API

`sf api request rest` is beta and does not support the normal `--json` flag. It prints the raw HTTP response body, and can print beta warnings before JSON. Do not blindly pipe output to a JSON parser unless you account for warning lines.

```bash
# GET
sf api request rest 'services/data/v66.0/limits' --target-org my-org

# POST (create record)
sf api request rest /services/data/v66.0/sobjects/Account \
  --body '{"Name":"New Account","ShippingCity":"Boise"}' \
  --method POST \
  --target-org my-org

# POST with a JSON body from a file. The @ prefix is required.
sf api request rest /services/data/v66.0/sobjects/Account \
  --body @new-account.json \
  --method POST \
  --target-org my-org

# PATCH (update record)
sf api request rest '/services/data/v66.0/sobjects/Account/<id>' \
  --body '{"BillingCity":"San Francisco"}' \
  --method PATCH \
  --target-org my-org

# With custom headers
sf api request rest '/services/data/v66.0/limits' --header 'Accept: application/xml' --target-org my-org

# Save output to file with HTTP headers
sf api request rest 'services/data/v66.0/limits' --stream-to-file output.txt --include --target-org my-org

# From a request definition file
sf api request rest --file myRequest.json --target-org my-org
```

### Body Files vs Request Files

| Need | Flag | Format |
|---|---|---|
| Send a raw JSON body from a file | `--body @body.json` | File contents are sent as the request body |
| Send inline JSON | `--body '{"Name":"New Account"}'` | String is sent as the request body |
| Define URL, method, headers, and body together | `--file request.json` | Postman-like request definition |

Do not use `--body body.json`; that sends the literal text `body.json`, not the file contents.

For JSON POST/PATCH APIs, include a content type header when the endpoint requires it:

```bash
sf api request rest /services/data/v66.0/analytics/reports \
  --method POST \
  --header "Content-Type: application/json" \
  --body @create-report.json \
  --target-org my-org
```

## GraphQL API

```bash
sf api request graphql --body "query { uiapi { query { Account { edges { node { Id Name { value } } } } } } }" --target-org my-org

# From file
sf api request graphql --body query.graphql --target-org my-org
```
