# Milestone 1: custom frontend staging

This repository is the deployment half of the PressF Taiga customization. The
frontend source lives in the `taiga-front` Git submodule and is built directly
by Docker Compose, so no external registry or GitHub Actions minutes are
required.

## Staging flow

1. Push the frontend `develop` branch and update the `taiga-front` submodule
   pointer in this repository.
2. Create a Git-based Docker Compose application in Coolify from this
   repository's `develop` branch during initial testing.
3. Copy the keys from `.env.coolify.example` into the Coolify environment and
   provide all secret values there.
4. Enable **Preserve Repository During Deployment** because the gateway mounts
   `taiga-gateway/taiga.conf` from the checked-out repository.
5. Under **Advanced**, enable Git submodules so the frontend build context is
   checked out before Docker Compose starts.
6. Assign the public domain only to `taiga-gateway` on internal port `80`.
7. Use new volumes for staging. Do not attach the production database or media
   volumes.

The base Compose file intentionally does not publish a host port. For local or
native-server testing, add the local override:

```sh
docker compose -f docker-compose.yml -f docker-compose.local.yml up -d
```

## Pin production images

The current server runs Taiga images tagged `latest`. Before production is
moved, record the immutable images on that server:

```sh
docker inspect taiga-docker-taiga-front-1 --format '{{json .RepoDigests}}'
docker inspect taiga-docker-taiga-back-1 --format '{{json .RepoDigests}}'
docker inspect taiga-docker-taiga-events-1 --format '{{json .RepoDigests}}'
docker inspect taiga-docker-taiga-protected-1 --format '{{json .RepoDigests}}'
```

Put the resulting `image@sha256:...` values into the corresponding Coolify
variables. A staging deployment must be verified before reusing production
data or changing the production domain.

## Rollback

Rollback the frontend by moving the `taiga-front` submodule pointer to a
previous release commit and redeploying. Backend migrations are outside this
milestone; the official backend remains unchanged.
