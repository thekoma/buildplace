# excalidraw/excalidraw-mcp publishes neither an image nor an npm package, only
# a Claude Desktop bundle and the Vercel deployment behind mcp.excalidraw.com.
# This builds the same server from source. Its dist/index.js is a
# self-contained bun bundle, so the runtime needs no node_modules.
#
# One build-time change: export_to_excalidraw hardcodes excalidraw.com's share
# backend. Both URLs become env overrides that default to upstream's values,
# so a deployment can point exports at its own excalidraw-storage-backend:
#   EXCALIDRAW_EXPORT_POST_URL   where the encrypted scene is POSTed
#   EXCALIDRAW_EXPORT_SHARE_URL  prefix of the returned link, before <id>,<key>
# The greps fail the build if an upstream change moves either string.

# renovate: datasource=git-refs depName=https://github.com/excalidraw/excalidraw-mcp currentValue=main
ARG COMMIT=157aa23ceb1976008aadc89eb05e3444060f09d6

FROM node:26-slim AS build
ARG COMMIT
ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
# BuildKit fetches the exact commit itself: no git in the image.
ADD https://github.com/excalidraw/excalidraw-mcp.git#${COMMIT} /src
WORKDIR /src
RUN sed -i \
      -e 's|"https://json.excalidraw.com/api/v2/post/"|(process.env.EXCALIDRAW_EXPORT_POST_URL ?? "https://json.excalidraw.com/api/v2/post/")|' \
      -e 's|`https://excalidraw.com/#json=${id},${jwk.k}`|`${process.env.EXCALIDRAW_EXPORT_SHARE_URL ?? "https://excalidraw.com/#json="}${id},${jwk.k}`|' \
      src/server.ts \
 && grep -q 'EXCALIDRAW_EXPORT_POST_URL' src/server.ts \
 && grep -q 'EXCALIDRAW_EXPORT_SHARE_URL' src/server.ts
# pnpm comes from packageManager in package.json; postinstall fetches the bun
# binary the build script bundles with.
RUN corepack enable \
 && pnpm install --frozen-lockfile \
 && pnpm run build

FROM gcr.io/distroless/nodejs24-debian13:nonroot
WORKDIR /app
COPY --from=build /src/dist ./dist
# Without --stdio the server speaks stateless Streamable HTTP on /mcp.
# Checkpoints are written under os.tmpdir() (/tmp): mount it writable.
ENV PORT=3001
EXPOSE 3001
CMD ["dist/index.js"]
