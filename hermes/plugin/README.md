# AgentEarth for Hermes

AgentEarth adds 31 English-language skills for web research, e-commerce, social
data, financial research, media creation, and related workflows. Hermes loads the bundled
skills and discovers tools from the remote AgentEarth MCP server.

## Install and connect

Use Hermes Agent 0.23.1 or later and an AgentEarth account. After the plugin is
listed in the Hermes Marketplace, run the following in the profile you use for
chat:

```bash
hermes plugins install agentearth --enable
hermes mcp add agentearth-plugin --url https://agentearth.ai/mcp-server/r/FbtaiJMv --auth oauth
```

The package declares `agentearth-plugin` in `mcp.json`. The sign-in command
adds a same-named OAuth entry to the active profile, which takes precedence
over the package's server entry. Complete AgentEarth sign-in in the browser.
Hermes discovers the server's tools and manages its OAuth tokens.

Use `hermes mcp login agentearth-plugin` to sign in again and
`hermes mcp test agentearth-plugin` to check the connection. Start a new chat
after setup. Complete authorization before running scheduled or unattended
tasks.

[Hermes MCP OAuth setup](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp#oauth-authenticated-http-servers)

## Try it

After connecting AgentEarth, try these prompts in Hermes.

- **Video generation.** “Refine this prompt, then use Seedance 2.5 to create a
  realistic four-second video at 720p. A fictional adult woman puts on sunglasses,
  then turns toward the camera.”
- **Product research.** “Find three home-office product opportunities in the US
  market using data from Amazon and TikTok Shop. Compare demand and competition,
  and use customer reviews to identify how each product could be improved.”
- **US stock comparison.** “Compare NVIDIA and AMD on this week's stock price
  performance, latest reported revenue and profit margins, valuation, and recent
  news. Include dated source links.”

The remote MCP server provides `GetAccountOverview`,
`GetUsageRecords`, `RecommendTools`, `ListTools`, `GetToolDetail`, and
`ExecuteTool`. Tool availability, schemas, and prices can change.

## Data, credentials, and credits

Hermes connects to AgentEarth over HTTPS. Search queries and tool inputs are
sent to AgentEarth. Execution may send the inputs needed for the selected tool
to its upstream provider. Account and credit history are scoped to the signed-in
AgentEarth account.

Hermes handles browser authorization, token storage, and token refresh. The
package contains metadata, the MCP address, and Skill documents. It adds no
local executable code, third-party dependencies, or background processes.
Tool execution may consume AgentEarth credits. Review the tool and its price
before confirming execution. If an asynchronous call returns a task ID, check
that task before retrying a paid operation.

[AgentEarth privacy policy](https://agentearth.ai/privacy.html)

## Update

When a new version is approved in the catalog, use `hermes plugins update
agentearth` or the Marketplace update action. The update installs the approved
commit, including its bundled skills. Start a new session to load the update.

[Hermes Plugin Catalog](https://hermes-agent.nousresearch.com/docs/user-guide/features/plugin-catalog)
