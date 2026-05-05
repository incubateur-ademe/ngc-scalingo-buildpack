# NGC Scalingo Buildpack

Custom Scalingo buildpack for deploying pnpm monorepo workspace packages using `pnpm deploy`.

## How it works

Uses `PROJECT_DIR` to locate the app within the monorepo, navigates up to the monorepo root for `pnpm install` and `pnpm deploy --prod`, then wipes the monorepo and recreates `PROJECT_DIR` from only the production deployment. The final image contains only the deployed package + a Node.js binary.

**Node.js version**: read from `engines.node` in the root `package.json`. The buildpack resolves the latest matching patch release (e.g. `"22.14"` → latest `22.14.x`), downloads it, and bundles it in the image.

## Setup

Each app needs a `.buildpacks` file in its directory pointing to this buildpack, and `PROJECT_DIR` set to the app's path:

```
# apps/server/.buildpacks (and apps/site/.buildpacks)
https://github.com/incubateur-ademe/ngc-scalingo-buildpack
```

```bash
scalingo -a nosgestesclimat-server env-set PROJECT_DIR=apps/server
scalingo -a nosgestesclimat-site   env-set PROJECT_DIR=apps/site
```

No other environment variables are needed — the buildpack reads the package name from `package.json` and the Node version from the root `package.json`.

## Procfile

Each app must have a `Procfile` in its `PROJECT_DIR`:

- **Server**: `web:  node dist/src/index.js`
- **Site**: `web: node .next/standalone/apps/site/server.js`

## Testing locally

```bash
docker run --pull always --rm -it \
  --env STACK=scalingo-22 \
  --volume /path/to/ngc-scalingo-buildpack:/buildpack \
  --volume /path/to/nosgestesclimat-app:/build \
  scalingo/scalingo-22:latest bash

# Inside the container — simulate PROJECT_DIR=apps/server:
mkdir -p /tmp/cache /tmp/env
/buildpack/bin/compile /build/apps/server /tmp/cache /tmp/env
```
