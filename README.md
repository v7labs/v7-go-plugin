# V7 Go plugin

Connect your AI assistant to V7 Go to build and run enterprise workflows, inspect results, and search company knowledge. Work with documents, structured data, and automations using your existing V7 Go workspace.

This repository packages V7 Go's hosted MCP connection in the Cursor plugin format for marketplace review. It does not run a local server. Marketplace availability, including Grok Bot, depends on review and publication.

## What you can do

- Inspect workflows, query their data, and read results.
- Run workflows, including workflows with file inputs.
- Create and configure workflows, properties, views, and triggers.
- Ingest documents and query your workspace's knowledge graph.

Available tools and actions depend on the authenticated workspace and permissions.

## Requirements

An existing V7 Go account, access to a workspace, and a client that supports Cursor plugins and remote MCP authentication. Your V7 Go service plan and usage limits still apply. The plugin package is free.

## Connect

Once the plugin is published, find V7 Go in your client's marketplace and install it. Select **V7 Go region** in the plugin's configuration, then complete the OAuth sign-in in your browser and select your workspace.

| Workspace region | Endpoint |
| --- | --- |
| EU (default) | `https://mcp.go.v7labs.com` |
| US | `https://mcp.go.us.v7labs.com` |

The package declares `V7_GO_MCP_URL` as a configuration value, with EU as the default. Choose the endpoint matching your existing workspace before authenticating. Do not configure both regions merely to connect one workspace. If your client does not expose plugin configuration, use its custom MCP connection flow with the appropriate endpoint instead.

OAuth is handled by the client and V7 Go. There is no API key or client secret to paste into this repository or a chat.

See the [V7 Go MCP setup guide](https://docs.go.v7labs.com/docs/connect-go-to-claude-chatgpt-and-other-ai-assistants) for current connection instructions.

## Example requests

- “List the workflows in my V7 Go workspace.”
- “Show the latest results from my contract review workflow.”
- “Find documents about this company in my knowledge graph.”
- “Help me configure a workflow to extract key fields from invoices.”

When asking the assistant to run or change a workflow, specify the intended workspace, workflow, inputs, and changes. Review consequential actions using your client's approval controls.

## Test the package locally

For a local Cursor installation, copy this repository's contents into `~/.cursor/plugins/local/v7-go/`, including `.cursor-plugin/`, and reload Cursor. Check that the plugin and its MCP connection appear in Customize. Local plugin imports must be permitted by your team's policy.

Before submitting or releasing an update:

1. Confirm the manifest and `mcp.json` parse as JSON and the logo path exists.
2. Confirm the client resolves `V7_GO_MCP_URL` to the selected endpoint.
3. Complete OAuth with a test workspace and verify the discovered tools.
4. Run a read-only request, such as listing workflows. Verify it accesses only the selected workspace.
5. Verify the appropriate permissions and approval flow before testing a write in a disposable workflow.
6. Repeat region-specific connection checks for each region supported by the release.

Do not treat an unauthenticated endpoint check as a completed OAuth or client compatibility test.

## Data and permissions

The plugin contains a manifest, a remote MCP configuration, documentation, and a logo. It has no local executable, hooks, analytics, or bundled credentials. The selected remote V7 Go service processes MCP requests and returns results to your AI client. Requests may read workspace data or make changes when the authenticated user and exposed tools permit them.

Review [V7's service terms](https://www.v7labs.com/terms/msa-go), [privacy policy](https://www.v7labs.com/terms/dps), and [Trust Center](https://trust.v7labs.com/), along with your AI client's policies. This repository's MIT license covers the plugin code and documentation; the V7 logo and trademarks remain V7's property, and the hosted service is governed by its own terms.

## Support

For package defects, open a [GitHub issue](https://github.com/v7labs/v7-go-plugin/issues). For account or workspace assistance, use your existing V7 support channel or [contact V7](https://www.v7labs.com/contact). Never include access tokens, customer documents, or private workspace data in public issues.

## Package layout

```text
.cursor-plugin/plugin.json  Plugin metadata and region configuration
mcp.json                    Hosted MCP connection
assets/logo.png             V7 logo
README.md                   Setup and usage
SECURITY.md                 Reporting guidance
LICENSE                     Package license
```
