# V7 Go plugin

Connect your AI assistant to V7 Go to build and run enterprise workflows, inspect results, and search company knowledge. Work with documents, structured data, and automations using your existing V7 Go workspace.

This repository packages V7 Go's hosted MCP connection in the Cursor plugin format for marketplace review. It does not run a local server. Marketplace availability, including Grok Bot, depends on review and publication.

## What you can do

- Inspect workflows, query their data, and read results.
- Run workflows, including workflows with file inputs.
- Create and configure workflows, properties, views, and triggers.
- Ingest documents and query your workspace's knowledge graph.

Available tools and actions depend on the authenticated workspace and permissions.

## Current test status

Local package loading and endpoint discovery have been checked in Cursor. End-to-end use is currently blocked in both regions: V7 Go rejects the native callback URI in Cursor's OAuth client registration. Authentication and tool calls have not passed. See [TESTING.md](TESTING.md) for the observed failure and retest steps.

## Requirements

An existing V7 Go account, access to a workspace, and a client that supports Cursor plugins and remote MCP authentication. Your V7 Go service plan and usage limits still apply. The plugin package is free.

## Connect

Once published, install **V7 Go**. In the Cursor dashboard under **Plugins → Configure**, set **V7 Go workspace region** to the endpoint matching your workspace, then complete OAuth sign-in and select your workspace. Team admins configure plugin variables.

| Workspace region | Configuration value |
| --- | --- |
| EU | `https://mcp.go.v7labs.com` |
| US | `https://mcp.go.us.v7labs.com` |

One plugin supports either endpoint through the required `V7_GO_MCP_URL` configuration variable. No default region is assumed. The two allowed values are declared using JSON Schema `enum`, as supported by the [Cursor plugin reference](https://cursor.com/docs/reference/plugins#variables). Marketplace configuration UI and Grok Bot propagation still require validation; a local folder import does not establish that those dashboard variables have been configured.

Clients that support custom MCP connections can also use the appropriate endpoint directly. These direct connections do not test marketplace configuration. If you need simultaneous access to both regions, configure two custom MCP connections; two concurrent configurations of the marketplace plugin have not been verified.

OAuth is handled by the client and V7 Go. There is no API key or client secret to paste into this repository or a chat.

See the [V7 Go MCP setup guide](https://docs.go.v7labs.com/docs/connect-go-to-claude-chatgpt-and-other-ai-assistants) for current connection instructions.

## Example requests

- “List the workflows in my V7 Go workspace.”
- “Show the latest results from my contract review workflow.”
- “Find documents about this company in my knowledge graph.”
- “Help me configure a workflow to extract key fields from invoices.”

When asking the assistant to run or change a workflow, specify the intended workspace, workflow, inputs, and changes. Review consequential actions using your client's approval controls.

## Test the package locally

A local folder import can check package discovery, but our test did not resolve dashboard-configured variables. Do not assume the schema's values will be supplied automatically by a local import.

To isolate the service connection, make a temporary local copy of `.cursor-plugin/plugin.json`, `mcp.json`, and `assets/` in a test plugin folder. Replace `${V7_GO_MCP_URL}` **in that copy only** with one of the two endpoints above, then load it from `~/.cursor/plugins/local/` and reload Cursor. Local plugin imports must be permitted by your team's policy. This checks the chosen endpoint, not marketplace variable interpolation.

Before submitting or releasing:

1. Validate the manifest and MCP configuration, including both allowed endpoint choices and referenced files.
2. Install the canonical package through a supported marketplace flow; configure each region and verify that the selected URL reaches the client.
3. Complete OAuth with a test workspace and verify the discovered tools.
4. Run a read-only request, such as listing workflows. Verify it accesses only the selected workspace.
5. Test Grok Bot itself, including its region configuration and sign-in flow.
6. Verify permissions and approval controls before testing writes in a disposable workflow.

Do not treat a fixed-URL local test or an unauthenticated endpoint check as a completed marketplace or OAuth compatibility test. See [TESTING.md](TESTING.md).

## Data and permissions

The plugin contains a manifest, a remote MCP configuration, and a logo; the repository also includes documentation. It has no local executable, hooks, analytics, or bundled credentials. The selected remote V7 Go service processes MCP requests and returns results to your AI client. Requests may read workspace data or make changes when the authenticated user and exposed tools permit them.

Review [V7's service terms](https://www.v7labs.com/terms/msa-go), [privacy policy](https://www.v7labs.com/terms/dps), and [Trust Center](https://trust.v7labs.com/), along with your AI client's policies. This repository's MIT license covers the plugin code and documentation; the V7 logo and trademarks remain V7's property, and the hosted service is governed by its own terms.

## Support

For package defects, open a [GitHub issue](https://github.com/v7labs/v7-go-plugin/issues). For account or workspace assistance, use your existing V7 support channel or [contact V7](https://www.v7labs.com/contact). Never include access tokens, customer documents, or private workspace data in public issues.

## Package layout

```text
.cursor-plugin/plugin.json         One V7 Go plugin and its region configuration schema
mcp.json                           Remote MCP connection using the selected endpoint
assets/logo.png                    Plugin and publisher logo
README.md                          Setup and usage
TESTING.md                         Evidence, limitations, and retest steps
SECURITY.md                        Reporting guidance
LICENSE                            Package license
```

## Compatibility note

Version 0.1.1 temporarily split the endpoints into two plugin entries to isolate an OAuth failure during local testing. Version 0.1.2 uses one plugin with a required region choice, following Cursor's documented dashboard variable configuration. The earlier local substitution failure does not demonstrate that marketplace variables are unsupported. Actual marketplace configuration and Grok Bot compatibility remain unverified.
