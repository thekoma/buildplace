# gog mcp speaks stdio only -- no HTTP transport flag exists -- and mcp-proxy
# does not ship a gog build. This image is the bridge: mcp-proxy's runtime with
# the gog binary dropped in, so the MCP server can be exposed over HTTP to the
# gateway.

FROM ghcr.io/openclaw/gogcli:v0.43.0 AS gog

FROM ghcr.io/sparfenyuk/mcp-proxy:v0.12.0
COPY --from=gog /usr/local/bin/gog /usr/local/bin/gog
