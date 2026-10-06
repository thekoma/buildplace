# nspady/google-calendar-mcp (npm: @cocal/google-calendar-mcp) publishes to npm
# only, no container image. This repackages the released npm tarball onto a
# distroless Node runtime: no shell, no package manager, no source of ours.

# renovate: datasource=npm depName=@cocal/google-calendar-mcp
ARG VERSION=2.7.0

FROM node:22-slim AS build
ARG VERSION
WORKDIR /app
RUN npm install --omit=dev --no-audit --no-fund "@cocal/google-calendar-mcp@${VERSION}"

FROM gcr.io/distroless/nodejs22-debian12:nonroot
WORKDIR /app
COPY --from=build /app/node_modules /app/node_modules
# Streamable HTTP; bind address, port and tool allowlist come from the pod.
ENTRYPOINT ["/nodejs/bin/node", "/app/node_modules/@cocal/google-calendar-mcp/build/index.js"]
CMD ["start", "--transport", "http"]
