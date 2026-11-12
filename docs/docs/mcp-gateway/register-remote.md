---
title: "Add remote MCP servers"
---

Register an existing HTTP MCP endpoint when the server runs outside Obot. Obot proxies the connection; it does not deploy or sandbox that remote service.

1. Open **MCP Servers**, select **Add MCP Server**, and choose **Remote Server**.
2. Enter a name, short description, and the server's MCP URL.
3. Configure required headers or user-supplied values. A remote service must be reachable from Obot, or through a configured [MCP tunnel](./filters-tunnels.md#mcp-tunnels-create-an-mcp-tunnel).
4. If the provider requires a pre-registered OAuth application, enable **Static OAuth** and configure its client credentials using [MCP Server OAuth Configuration](../configuration/mcp-server-oauth-configuration.md).
5. Save the entry and review catalog access policies.
6. Add the entry to a [vMCP](./server-types.md), configure its tools and profiles, and test the resulting connection.

For a complete provider-specific setup, see the [Slack MCP tutorial](../configuration/tutorials/slack-mcp-server.md).

Obot blocks localhost, private-IP, and link-local remote destinations by default. Use a tunnel for private services when appropriate. Do not disable these protections globally just to make an arbitrary endpoint connect. See [Network isolation](../security/isolation.md) for the separate gateway and workload boundaries. A remote server's URL is refused when it resolves to such an address; set `OBOT_SERVER_DISALLOW_PRIVATE_IPMCP=false` (or the `LOCALHOST` and `LINK_LOCAL` equivalents) only when that is intended. Guardrail services are different: they may be on a private network, as described in [Guardrails](../functionality/guardrails.md).

## Remote server {#mcp-servers-remote-server}

MCP Servers that are HTTP Streaming compatible should be configured this way. These servers can be provided by trusted 3rd party vendors. Remote servers also work for MCP servers deployed through existing CI/CD pipeline within the organization.

Choose this type when:

- You have MCP services deployed through traditional application deployment mechanisms
- External partners provide MCP endpoints and you just want to integrate
- You are building MCP servers through existing CI/CD workflows or SaaS services

Remote MCP servers that conform to the MCP spec authentication schema will work out of the box. Servers that do not conform to the spec may not work within the gateway. Please open a GitHub issue if you run into issues with remote servers.

**Configuration**: Specify the remote URL endpoint. Additional options include connection restrictions for unconventional configurations, custom HTTP headers, and configuration values to send to the remote server.

If Obot cannot directly reach a remote server, use an [MCP tunnel](./filters-tunnels.md#mcp-tunnels-create-an-mcp-tunnel) to route requests through a machine on the server's network. Keep the remote server's real HTTP or HTTPS URL and select the tunnel separately in **Advanced Configuration**.

### TLS certificate errors

Obot verifies the remote server's TLS certificate against the trust store of the Obot process. A self-signed certificate or a private CA that Obot does not trust fails with `tls: failed to verify certificate: x509: certificate signed by unknown authority`.

Prefer installing the issuing CA into the Obot container or pod trust store when you can. If you cannot:

- Set `OBOT_SERVER_MCP_INSECURE_SKIP_VERIFY=true` on the Obot server to skip certificate checks for every remote MCP HTTPS connection.
- Or open **Advanced Configuration** for one remote server and enable **Skip TLS certificate verification**.

Both options disable certificate checks only for Obot's direct HTTPS connection. Prefer a trusted CA in production. When the server is reached through an MCP tunnel, TLS to the destination is terminated on the tunnel CLI host, so that host must trust the certificate (or skip verification there) as well.

## Caller attribution headers

Remote MCP servers support [caller attribution headers](./server-configuration.md#caller-attribution-headers). Use them to send selected information about the caller authenticated by Obot to the backend. See the shared configuration guide for setup, available headers, and limits.

## Localhost OAuth callbacks

Some MCP servers require their OAuth clients to use a localhost callback. For these servers, enable **Localhost OAuth callback** in **Advanced Configuration** when adding or editing the remote server.

Leave **Callback path** blank to use `/oauth/callback`, or enter the path required by the server. The path must start with `/` and contain no query string or fragment. `/oauth/obot/callback` is reserved for Obot to notify the CLI that sign-in has finished; it cannot be used for the provider callback. The server must accept an available callback port; fixed ports are not supported.

During authentication, Obot displays an [`obot mcp login` command](../reference/cli-api.md#mcp-localhost-oauth-sign-in) to run on a computer with a web browser. MCP clients continue using their normal Obot HTTP connection URL.
