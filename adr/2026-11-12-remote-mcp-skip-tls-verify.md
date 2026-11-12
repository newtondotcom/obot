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

Support two opt-in controls that disable TLS verification for Obot's direct remote MCP HTTPS clients:

1. Server-wide: `OBOT_SERVER_MCP_INSECURE_SKIP_VERIFY` / `--mcp-insecure-skip-verify` on `mcp.Options`, stored on `SessionManager`.
2. Per remote server: `skipTLSVerify` on `RemoteRuntimeConfig` / `RemoteCatalogConfig`.

Either setting causes the MCP HTTP and OAuth clients to use `tls.Config.InsecureSkipVerify = true`. Verification remains the default. Both apply only to Obot's direct connection to the remote URL; when an MCP tunnel is used, the tunnel CLI host still performs its own TLS handshake to the destination.

## Rationale

Operators asked for a root-level switch so every internal MCP endpoint with a private CA does not need a per-server toggle. A per-server flag remains useful for narrower exceptions and matches the LLM gateway pattern. Trusting a custom CA would be preferable, but skip-verify is the immediate unblock. Scoping both controls to remote MCP HTTPS keeps hosted MCP containers and unrelated egress clients unchanged.

## Consequences

- Operators can set `OBOT_SERVER_MCP_INSECURE_SKIP_VERIFY=true` (Helm `config.OBOT_SERVER_MCP_INSECURE_SKIP_VERIFY`) for all remote MCP HTTPS from Obot.
- Administrators can also enable **Skip TLS certificate verification** under Advanced Configuration for one remote MCP server or catalog entry.
- Misconfiguration can expose credentials to a man-in-the-middle; the UI warns for the per-server option.
- Tunnel-backed remotes still need TLS trust (or an equivalent skip) on the tunnel CLI machine; these flags do not rewrite the CLI's forwarder.
- A follow-up may add custom CA trust for remote MCP so skip-verify is not the only enterprise path.

## References

- `pkg/safehttp/client.go` — `Options.InsecureSkipVerify`
- `pkg/mcp/http.go` — `HTTPClientForServer` / `OAuthClientForServer`
- `apiclient/types/mcpserver.go` — `RemoteRuntimeConfig.SkipTLSVerify`
- LLM gateway `skip_tls_verify` / `ca_cert_pem` for the prior art on model providers
