# Paseo Relay image

Unofficial container image of [Paseo Relay](https://github.com/getpaseo/paseo-relay), built from the upstream Dockerfile at a pinned commit. Not affiliated with Paseo. Report relay bugs [upstream](https://github.com/getpaseo/paseo-relay/issues); this repository only covers packaging.

Differences from an upstream build:

- `/app/LICENSE` is included (Apache-2.0).
- `RELEASE_DISTRIBUTION=none` is set by default (see [Security](#security)).
- `LANG=C.UTF-8` is set so the Erlang VM uses UTF-8 filename encoding instead of latin1.

## Usage

```sh
docker run --rm -p 4000:4000 --user 65534:65534 --read-only \
  ghcr.io/malsius/paseo-relay:latest
```

The relay serves plain HTTP on port 4000: WebSocket on `/ws`, plus `/health`, `/ready` and `/metrics`. Put it behind a reverse proxy that terminates TLS, expose only `/ws` publicly, and keep the other endpoints internal.

Run a single replica. Multiple nodes require a rerouting adapter; see upstream's [Fly deployment](https://github.com/getpaseo/paseo-relay/tree/main/deployment/fly).

## Tags and versions

The upstream commit is set by `UPSTREAM_REF` in [`build.yml`](.github/workflows/build.yml). Tags are `sha-<first 12 characters of the upstream commit>` and `latest`. Both are mutable: upstream does not pin its base images or packages, so a rebuild of the same commit produces a new digest. Pin deployments by digest.

Verify an image's provenance:

```sh
gh attestation verify oci://ghcr.io/malsius/paseo-relay@sha256:<digest> --repo Malsius/paseo-relay-image
```

## Security

The image runs as root unless you set a user; the example above runs it as `65534` with a read-only root filesystem, as tested in CI. Erlang distribution is disabled because the release cookie is generated at build time and readable by anyone who pulls the image. To re-enable distribution, set your own secret `RELEASE_COOKIE`.

## License

Apache-2.0, like [upstream](https://github.com/getpaseo/paseo-relay/blob/main/LICENSE).
