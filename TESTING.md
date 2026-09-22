# Client compatibility test

Tested September 21, 2026, using Cursor 3.21.16 on macOS arm64. The Cursor local tests below used version 0.1.1's temporary fixed-URL packages. Version 0.1.2 consolidates distribution into one configurable plugin; its team-marketplace installation and EU consent-page handoff have now been tested in Grok Bot 0.57.1. Completed authorization and tool calls remain pending.

## Results

| Check | Result |
| --- | --- |
| Local discovery of version 0.1.1 | PASS: Cursor loaded both temporary regional plugin entries. |
| Version 0.1.2 manifest and MCP configuration | PASS: JSON, component paths, declared placeholder, and two allowed HTTPS endpoints checked. |
| Marketplace configuration support | PASS: imported one plugin from the repository into the Default team marketplace as Default Off. Both dashboard Configure and Grok Bot install render the EU/US dropdown. EU selection reaches the EU consent page. US connection remains NOT TESTED in Grok Bot. |
| EU and US OAuth discovery | PASS: both publish resource and authorization-server metadata, including PKCE S256. |
| Cursor local EU fixed-URL connection | BLOCKED: dynamic client registration returned HTTP 400 before sign-in. |
| Cursor local US fixed-URL connection | BLOCKED: same registration error. |
| Cursor local completed OAuth and tool calls | NOT TESTED: blocked by registration. |
| Grok Bot team-marketplace installation (September 22) | PASS after admin access was granted: repository imported, region selected, plugin installed, Authenticate opened the EU V7 workspace consent page. |
| Grok Bot completed authorization and tool call | PENDING: stopped at consent with only workflow/data read access selected, awaiting the account owner’s approval. |

## Region configuration findings

Version 0.1.0 declared an endpoint variable. In a local Cursor folder import, it remained literal and the client attempted to contact `${V7_GO_MCP_URL}`. Version 0.1.1 used two fixed endpoints to isolate subsequent connection behavior.

The [Cursor plugin reference](https://cursor.com/docs/reference/plugins#variables) documents that team admins set variables at installation or through dashboard Plugins → Configure. It explicitly accepts `enum` in the variables schema. A local folder import without a configured dashboard value is insufficient evidence that marketplace variables are unsupported.

Version 0.1.2 therefore has one `v7-go` plugin, with `V7_GO_MCP_URL` required and restricted to the EU and US URLs. There is no default, so the workspace region must be selected explicitly. The dashboard renders the enum as a dropdown. Grok Bot also prompts for its own region selection during installation; the EU selection successfully reached the EU consent page. Other client surfaces and the Grok Bot US connection remain unverified.

## Earlier Cursor local compatibility blocker

With fixed endpoint values, Cursor reaches the selected endpoint and receives the expected HTTP 401 authentication challenge. OAuth dynamic client registration then returns HTTP 400 with:

```json
{"code":"invalid_mcp_oauth_request","message":"Redirect URIs must use HTTPS or localhost HTTP"}
```

Cursor's registration includes a native callback according to [Cursor staff's compatibility report](https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877). The observed error is consistent with rejection of `cursor://anysphere.cursor-mcp/oauth/callback`; our test did not capture the outgoing registration body directly. The service's validator allows only HTTPS or localhost HTTP URIs and validates every URI in the registration.

Cursor also reports that the error body lacks the OAuth `error` field. It subsequently falls back to SSE and displays HTTP 406; that is secondary to registration failing.

The Cursor local-test rejection occurs in V7 Go's MCP registration handler before the authorization flow redirects to Auth0. Changing Auth0 callback settings does not resolve that validation failure. Any backend compatibility fix must preserve exact registered redirect matching and PKCE; registration errors should use OAuth-compatible response fields. No backend fix or Auth0 change was deployed as part of this package work.

This does not establish that Grok Bot's cloud path behaves identically. That client needs its own end-to-end test.

## Grok Bot team-marketplace check — September 22, 2026

Initial access check: the app's Marketplace search for `V7` returned no results. Team plugins were visible, but the test account was a Member and could not import the package. The public listing remains unpublished.

After the account received team admin access, we used Dashboard → Plugins & MCPs → Default → Add to Marketplace → Plugin, pasted this repository URL, and saved the discovered `v7-go` package. It remains **Default Off**, so other teammates are not automatically enrolled. The existing marketplace and its other plugin were preserved.

The dashboard Configure UI displayed `V7 Go workspace region` with both HTTPS endpoints. Grok Bot 0.57.1 then found `v7-go` as a Team plugin. Its Add flow separately prompted for the region using the same dropdown. We selected EU and installed it. Returning from the setup screen showed the installed entry with an account marked Needs auth.

Clicking Authenticate opened `https://mcp.go.v7labs.com/oauth/consent` in the browser, displaying Authorize Cursor, a workspace selector, and permission checkboxes. Therefore this team-marketplace EU flow passed the registration/sign-in steps that blocked the earlier Cursor local test. No native-callback compatibility fix from this task was deployed. We did not capture the underlying registration request, so the reason for the different behavior is not yet established.

All optional scopes were explicitly unchecked. Only the mandatory workflow/data read permission remains selected. Authorization is pending the account owner's approval; no tool call has been made. The proposed backend callback exception remains on hold. The US flow and eventual public-marketplace listing still require testing.

## Retest

1. Validate the canonical package through a supported marketplace installation; set each allowed region and confirm the resolved endpoint.
2. Confirm OAuth registration succeeds and opens V7 sign-in and consent.
3. Authorize a test workspace, verify tool discovery, and run a read-only workflow-list request.
4. Repeat for the other region with an account that has access there.
5. Test the actual package in Grok Bot before public submission.

Do not disable redirect validation or place account tokens in the package to bypass the blocker.
