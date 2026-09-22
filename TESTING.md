# Client compatibility test

Tested September 21, 2026, using Cursor 3.21.16 on macOS arm64. The live client tests below used version 0.1.1's temporary fixed-URL packages. Version 0.1.2 consolidates distribution into one configurable plugin; its marketplace configuration flow has not been tested end to end.

## Results

| Check | Result |
| --- | --- |
| Local discovery of version 0.1.1 | PASS: Cursor loaded both temporary regional plugin entries. |
| Version 0.1.2 manifest and MCP configuration | PASS: JSON, component paths, declared placeholder, and two allowed HTTPS endpoints checked. |
| Marketplace configuration support | DOCUMENTED: Cursor supports dashboard-configured plugin variables and the JSON Schema `enum` keyword. Actual region UI, substitution, and propagation to Grok Bot remain NOT TESTED. |
| EU and US OAuth discovery | PASS: both publish resource and authorization-server metadata, including PKCE S256. |
| EU fixed-URL client connection | BLOCKED: dynamic client registration returned HTTP 400 before sign-in. |
| US fixed-URL client connection | BLOCKED: same registration error. |
| Completed OAuth, tool discovery, read-only tool request | NOT TESTED: blocked by registration. |
| Grok Bot marketplace installation (September 22) | BLOCKED before installation: no V7 listing; the test account lacks team marketplace import controls. OAuth and tool calls NOT TESTED in Grok Bot. |

## Region configuration findings

Version 0.1.0 declared an endpoint variable. In a local Cursor folder import, it remained literal and the client attempted to contact `${V7_GO_MCP_URL}`. Version 0.1.1 used two fixed endpoints to isolate subsequent connection behavior.

The [Cursor plugin reference](https://cursor.com/docs/reference/plugins#variables) documents that team admins set variables at installation or through dashboard Plugins → Configure. It explicitly accepts `enum` in the variables schema. A local folder import without a configured dashboard value is insufficient evidence that marketplace variables are unsupported.

Version 0.1.2 therefore has one `v7-go` plugin, with `V7_GO_MCP_URL` required and restricted to the EU and US URLs. There is no default, so the workspace region must be selected explicitly. We have not verified how the dashboard renders that choice or whether every target client receives it correctly. The current team's dashboard Add button led to the public catalogue; it did not expose a repository-import test flow in the inspected page.

## Remaining service compatibility blocker

With fixed endpoint values, Cursor reaches the selected endpoint and receives the expected HTTP 401 authentication challenge. OAuth dynamic client registration then returns HTTP 400 with:

```json
{"code":"invalid_mcp_oauth_request","message":"Redirect URIs must use HTTPS or localhost HTTP"}
```

Cursor's registration includes a native callback according to [Cursor staff's compatibility report](https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877). The observed error is consistent with rejection of `cursor://anysphere.cursor-mcp/oauth/callback`; our test did not capture the outgoing registration body directly. The service's validator allows only HTTPS or localhost HTTP URIs and validates every URI in the registration.

Cursor also reports that the error body lacks the OAuth `error` field. It subsequently falls back to SSE and displays HTTP 406; that is secondary to registration failing.

The rejection occurs in V7 Go's MCP registration handler before the authorization flow redirects to Auth0. Changing Auth0 callback settings does not resolve this validation failure. A backend compatibility fix must preserve exact registered redirect matching and PKCE; registration errors should also use OAuth-compatible response fields. Backend code and Auth0 settings have not been changed as part of this package work.

This does not establish that Grok Bot's cloud path behaves identically. That client needs its own end-to-end test.

## Grok Bot marketplace check — September 22, 2026

Opened Marketplace in the actual Grok Bot desktop app and searched for `V7`: the app reported no matching results. Its catalogue displayed a Team Plugins section with an existing team integration, confirming that team-distributed entries are visible in this client.

The current Cursor account is a team Member. Dashboard → Plugins & MCPs did not expose Team Marketplaces / Import from Repo controls, and Team MCP editing controls were disabled. No V7 plugin was installed or authorized, and no Grok Bot OAuth registration request was captured. This is an installation/access blocker, not a reproduced OAuth failure in Grok Bot.

To unblock a pre-publication test, a team admin should import this repository into the team's existing marketplace and keep installation **Default Off**. The [official team marketplace guide](https://cursor.com/docs/plugins#add-a-team-marketplace) documents repository import and installation modes. Then install V7 Go through Grok Bot's Team Plugins entry and test configuration, OAuth, tool discovery, and a read-only call. A public catalogue test remains unavailable until review and publication.

The proposed backend callback exception remains on hold. Cursor staff describes the native callback as legacy and plans to remove it from registration. Capture the actual Grok Bot registration behavior before deciding whether a service compatibility exception is necessary.

## Retest

1. Validate the canonical package through a supported marketplace installation; set each allowed region and confirm the resolved endpoint.
2. Confirm OAuth registration succeeds and opens V7 sign-in and consent.
3. Authorize a test workspace, verify tool discovery, and run a read-only workflow-list request.
4. Repeat for the other region with an account that has access there.
5. Test the actual package in Grok Bot before public submission.

Do not disable redirect validation or place account tokens in the package to bypass the blocker.
