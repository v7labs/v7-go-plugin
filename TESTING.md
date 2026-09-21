# Client compatibility test

Tested September 21, 2026, using Cursor 3.21.16 on macOS arm64.

## Results

| Check | Result |
| --- | --- |
| Local plugin discovery | PASS: Cursor loads both regional plugin entries from its local plugin directory. |
| Manifest and MCP configuration | PASS: JSON, component paths, unique plugin names, and fixed regional URLs validated. |
| EU and US OAuth discovery | PASS: both publish resource and authorization-server metadata, including PKCE S256. |
| EU client connection | BLOCKED: dynamic client registration returns HTTP 400 before sign-in. |
| US client connection | BLOCKED: same registration error. |
| Completed OAuth, tool discovery, read-only tool request | NOT TESTED: blocked by registration. |
| Grok Bot end-to-end connection | NOT TESTED. |

## Defect corrected in this repository

Version 0.1.0 declared a plugin variable for the endpoint. In the local Cursor client, the plugin loaded, but the variable remained literal and the client attempted to contact `${V7_GO_MCP_URL}`. Version 0.1.1 replaces that configuration with separate `v7-go` (EU) and `v7-go-us` (US) entries, each with a fixed HTTPS endpoint.

## Remaining service compatibility blocker

With the corrected packages, Cursor reaches the selected endpoint and receives the expected HTTP 401 authentication challenge. OAuth dynamic client registration then returns HTTP 400 with:

```json
{"code":"invalid_mcp_oauth_request","message":"Redirect URIs must use HTTPS or localhost HTTP"}
```

Cursor includes a native `cursor://anysphere.cursor-mcp/oauth/callback` URI in its registration. The service rejects that scheme. Cursor also reports that the returned error does not match the OAuth error schema because it lacks an `error` string. Cursor subsequently falls back to SSE and displays HTTP 406; that is secondary to the registration failure.

This result does not establish that Grok Bot's cloud path behaves identically. That client needs its own end-to-end test after the registration compatibility issue is resolved.

## Retest

1. Load the appropriate `plugins/` subdirectory as described in the README.
2. Confirm OAuth registration succeeds and opens the intended V7 sign-in and consent flow.
3. Authorize a test workspace, then verify tool discovery and a read-only workflow-list request.
4. Repeat for the other region with an account that has access there.
5. Test the actual package in Grok Bot through a supported private/team marketplace before public submission.

Do not work around this by disabling redirect validation or placing account tokens in the package. Fix and review service/client compatibility, then rerun the failed checks.
