# Client compatibility test

Tested September 21, 2026, using Cursor 3.21.16 on macOS arm64. The Cursor local tests below used version 0.1.1's temporary fixed-URL packages. Version 0.1.2 consolidates distribution into one configurable plugin; its team-marketplace installation, EU authorization, and a read-only workflow-list call were tested in Grok Bot 0.57.1 on September 22, 2026.

## Results

| Check | Result |
| --- | --- |
| Local discovery of version 0.1.1 | PASS: Cursor loaded both temporary regional plugin entries. |
| Version 0.1.2 manifest and MCP configuration | PASS: JSON, component paths, declared placeholder, and two allowed HTTPS endpoints checked. |
| Marketplace configuration support | PASS: imported one plugin from the repository into the Default team marketplace as Default Off. Both dashboard Configure and Grok Bot install render the EU/US dropdown. EU selection completes authorization and a read-only tool test. US authorization and a read-only workflow-list call also passed. |
| EU and US OAuth discovery | PASS: both publish resource and authorization-server metadata, including PKCE S256. |
| Cursor local EU fixed-URL connection | BLOCKED: dynamic client registration returned HTTP 400 before sign-in. |
| Cursor local US fixed-URL connection | BLOCKED: same registration error. |
| Cursor local completed OAuth and tool calls | NOT TESTED: blocked by registration. |
| Grok Bot team-marketplace installation (September 22) | PASS after admin access was granted: repository imported, region selected, plugin installed, Authenticate opened the EU V7 workspace consent page. |
| Grok Bot EU authorization and tool discovery | PASS: owner approved all offered permissions; Grok Bot showed Connected and 46 of 46 tools enabled. |
| Grok Bot EU read-only tool call | PASS: a dedicated test bot reported `list_workflows` succeeded and returned five workflows. |
| Grok Bot EU connection reuse | PASS: repeated `list_workflows` calls succeeded without another login. Restoring EU and reopening Grok Bot also preserved the connection and a fresh five-item list call passed. Token-expiry refresh has not been verified. |
| Grok Bot EU workflow-result query | FAIL: successful `workflow_querying_info` followed by `query_workflows` returned MCP -32000 requiring that prerequisite; reproduced with a fresh explicitly sequential retry. No entity was read. |
| Grok Bot US authorization and tool call | PASS: selected US in installed plugin setup, completed owner-approved full-scope consent, and observed Connected. Grok Bot reported `list_workflows` succeeded with zero workflows and no error in the US test workspace. |
| Grok Bot write operations, public listing | NOT TESTED. |

## Region configuration findings

Version 0.1.0 declared an endpoint variable. In a local Cursor folder import, it remained literal and the client attempted to contact `${V7_GO_MCP_URL}`. Version 0.1.1 used two fixed endpoints to isolate subsequent connection behavior.

The [Cursor plugin reference](https://cursor.com/docs/reference/plugins#variables) documents that team admins set variables at installation or through dashboard Plugins → Configure. It explicitly accepts `enum` in the variables schema. A local folder import without a configured dashboard value is insufficient evidence that marketplace variables are unsupported.

Version 0.1.2 therefore has one `v7-go` plugin, with `V7_GO_MCP_URL` required and restricted to the EU and US URLs. There is no default, so the workspace region must be selected explicitly. The dashboard renders the enum as a dropdown. Grok Bot also prompts for its own region selection during installation; the EU selection successfully reached the EU consent page. The US selection also completed OAuth and a successful workflow-list call. Other client surfaces remain unverified.

## Earlier Cursor local compatibility blocker

With fixed endpoint values, Cursor reaches the selected endpoint and receives the expected HTTP 401 authentication challenge. OAuth dynamic client registration then returns HTTP 400 with:

```json
{"code":"invalid_mcp_oauth_request","message":"Redirect URIs must use HTTPS or localhost HTTP"}
```

Cursor's registration includes a native callback according to [Cursor staff's compatibility report](https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877). The observed error is consistent with rejection of `cursor://anysphere.cursor-mcp/oauth/callback`; our test did not capture the outgoing registration body directly. The service's validator allows only HTTPS or localhost HTTP URIs and validates every URI in the registration.

Cursor also reports that the error body lacks the OAuth `error` field. It subsequently falls back to SSE and displays HTTP 406; that is secondary to registration failing.

The Cursor local-test rejection occurs in V7 Go's MCP registration handler before the authorization flow redirects to Auth0. Changing Auth0 callback settings does not resolve that validation failure. Any backend compatibility fix must preserve exact registered redirect matching and PKCE; registration errors should use OAuth-compatible response fields. No backend fix or Auth0 change was deployed as part of this package work.

The successful Grok Bot EU team-marketplace test below demonstrates different behavior from this earlier local Cursor test. The underlying registration payloads were not captured, so the cause of the difference remains unverified.

## Grok Bot team-marketplace check — September 22, 2026

Initial access check: the app's Marketplace search for `V7` returned no results. Team plugins were visible, but the test account was a Member and could not import the package. The public listing remains unpublished.

After the account received team admin access, we used Dashboard → Plugins & MCPs → Default → Add to Marketplace → Plugin, pasted this repository URL, and saved the discovered `v7-go` package. It remains **Default Off**, so other teammates are not automatically enrolled. The existing marketplace and its other plugin were preserved.

The dashboard Configure UI displayed `V7 Go workspace region` with both HTTPS endpoints. Grok Bot 0.57.1 then found `v7-go` as a Team plugin. Its Add flow separately prompted for the region using the same dropdown. We selected EU and installed it. Returning from the setup screen showed the installed entry with an account marked Needs auth.

Clicking Authenticate opened `https://mcp.go.v7labs.com/oauth/consent` in the browser, displaying Authorize Cursor, a workspace selector, and permission checkboxes. Therefore this team-marketplace EU flow passed the registration/sign-in steps that blocked the earlier Cursor local test. No native-callback compatibility fix from this task was deployed. We did not capture the underlying registration request, so the reason for the different behavior is not yet established.

After the account owner explicitly requested reconnection with all permissions, we reset the plugin account and completed consent with every offered permission selected for the chosen EU workspace. Grok Bot then showed **Connected** and **46 of 46 tools enabled**.

A separate test bot was instructed to use only this plugin for one read-only workflow-list call, limited to five results, and to stop afterward. It reported **Succeeded**, tool **`list_workflows`**, and **5 workflows returned**. This evidence is the Grok Bot UI result; raw MCP request/response traces were not captured. No write operation was requested or tested. Private workspace identifiers and workflow data are omitted from this report.

The proposed backend callback exception remains on hold: it was not needed for this successful EU team-marketplace test. The US workflow-list test subsequently passed; the public-marketplace listing still requires testing.

## Additional pre-submission checks — September 22, 2026

The existing EU account successfully repeated `list_workflows` with `page_size: 5`, returning five workflows without another login. A bounded search for demo/test workflows also succeeded. A later check restored EU after US testing, quit/reopened Grok Bot, and successfully repeated a fresh five-item workflow-list call without another login. Token-expiry refresh remains untested.

The demo-result check failed before retrieving any entity: `workflow_querying_info` succeeded, then `query_workflows` returned MCP error `-32000`: “You must call `workflow_querying_info` tool before using this tool”. A second attempt explicitly called the prerequisite first, waited for success, then queried one demo row; the same error occurred. Evidence is the Grok Bot UI report, not a captured raw transport trace. Backend source inspection and a local transport probe reproduced this error when prerequisite and query calls do not share a surviving MCP session; same-session calls passed. Actual Grok Bot session headers and lifecycle were not captured, so the production transport cause is unverified. A focused backend fix is being prepared to make schema guidance advisory while preserving authorization and complexity controls. No fix has been deployed; do not treat full workflow-result querying as verified.

Grok Bot's Edit Values form states that setup values apply to the plugin's connectors. Switching the endpoint to US reset the account to Needs auth. Authenticate then opened the US V7 consent page with available workspaces. The owner then approved all offered permissions for the US test workspace. Consent completed, Grok Bot showed Connected, and a fresh `list_workflows` call limited to five succeeded with zero workflows and no error. We restored EU afterward; it showed Connected and 46 of 46 enabled tools. A fresh EU list call after reopening the app returned five workflows successfully.

## Retest

1. Validate the canonical package through a supported marketplace installation; set each allowed region and confirm the resolved endpoint.
2. Confirm OAuth registration succeeds and opens V7 sign-in and consent.
3. Authorize a test workspace, verify tool discovery, and run a read-only workflow-list request.
4. Repeat for the other region with an account that has access there.
5. Test the actual package in Grok Bot before public submission.

Do not disable redirect validation or place account tokens in the package to bypass the blocker.
