# 2026-11-12: Optional TLS skip-verify for remote MCP servers

- **Status:** Accepted
- **Date:** 2026-11-12
- **Supersedes:** None
- **Superseded by:** None

## Related issues

None.

## Related ODPs

None.

## Context

Remote MCP servers are reached over HTTPS by Obot's Go HTTP client (`pkg/mcp` → `pkg/safehttp`). That client verifies certificates against the process trust store. Enterprise and lab deployments often expose MCP endpoints with self-signed certificates or a private CA that is not installed in the Obot container, which surfaces as `tls: failed to verify certificate: x509: certificate signed by unknown authority` when listing tools or configuring a server. Model providers already expose a per-provider skip-verify control; remote MCP had no equivalent.

## Decision

Add an optional `skipTLSVerify` boolean on `RemoteRuntimeConfig` / `RemoteCatalogConfig`. When set, Obot's MCP HTTP and OAuth clients for that server build a transport with `tls.Config.InsecureSkipVerify = true`. Verification remains the default. The setting applies only to Obot's direct connection to the remote URL; when an MCP tunnel is used, the tunnel CLI host still performs its own TLS handshake to the destination.

## Rationale

A per-server flag matches the existing LLM gateway pattern and avoids a global insecure mode. Trusting a custom CA would be preferable, but operators who need an immediate unblock for private endpoints asked to disable verification for a specific remote server. Scoping the flag to the remote runtime keeps hosted MCP containers and unrelated egress clients unchanged.

## Consequences

- Administrators can enable **Skip TLS certificate verification** under Advanced Configuration for a remote MCP server or catalog entry.
- Misconfiguration can expose credentials to a man-in-the-middle on that server's path; the UI warns when the option is on.
- Tunnel-backed remotes still need TLS trust (or an equivalent skip) on the tunnel CLI machine; this flag does not rewrite the CLI's forwarder.
- A follow-up may add custom CA trust for remote MCP so skip-verify is not the only enterprise path.

## References

- `pkg/safehttp/client.go` — `Options.InsecureSkipVerify`
- `pkg/mcp/http.go` — `HTTPClientForServer` / `OAuthClientForServer`
- `apiclient/types/mcpserver.go` — `RemoteRuntimeConfig.SkipTLSVerify`
- LLM gateway `skip_tls_verify` / `ca_cert_pem` for the prior art on model providers
