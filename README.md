# buildplace

Container images that exist only because nobody upstream publishes them.

One repo, one workflow, N images. Adding an image is adding a directory —
there is no CI to write.

## What belongs here

| | |
|---|---|
| **Yes** | A `Dockerfile` that recomposes other people's published images or binaries. No source of ours. |
| **No** | Anything with our own source code. That image is built in the repo that holds the code (`hermes-iris`, `vaultsync`, `fredy`). |

`gog-mcp` is the archetype: two `FROM` lines and a `COPY`, needed only because
`gog mcp` speaks stdio and `mcp-proxy` does not ship it.

Pre-existing single-image repos (`agentmemory-image`, `happy-images`,
`gpodder-docker`) would fit the rule but are **not** being migrated: they work,
they carry their own upstream-tracking logic, and moving them buys nothing.
This repo is for the next ones.

## Adding an image

```
images/<name>/Dockerfile
```

That is the whole procedure. On push to `main` the workflow detects which
directories under `images/` changed and builds exactly those. The directory
name is the image name.

Optional `images/<name>/build.env` overrides the defaults:

```sh
PLATFORMS=linux/amd64,linux/arm64   # default: linux/amd64
TAGS=latest,v1                      # default: latest
```

## Where images go

`zot.zot.svc.cluster.local:5000/<name>:<tag>` — the in-cluster registry.
It is **not** reachable from outside the cluster, which is why the builds run
on the self-hosted runner and not on GitHub's.

## Builds

Builds run on the Odin cluster: an ARC runner scale set named `buildplace`
drives a shared rootless BuildKit. Neither the source nor the layers leave the
network, and the layer cache is local.

The registry credential is mounted into the runner from Vault by the ARC
ApplicationSet — there is **no** `docker/login-action` step and no GitHub
secret. See `infrastructure/odin/arc/NOTES.md` in `asgard-k8s`.

```bash
gh workflow run build              # everything
gh workflow run build -f image=gog-mcp
```

## Multi-arch

Defaults to the builder's architecture (amd64). BuildKit here has no QEMU
emulators registered, so a `PLATFORMS` value naming a foreign arch will fail
until they are. Not needed yet, so not set up.

## Renovate

`renovate.json` tracks the pinned `FROM` tags in every `images/*/Dockerfile`.
Pin them: an unpinned base defeats the point of building the image at all.
