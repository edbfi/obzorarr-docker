<p align="center">
  <img src="docs/assets/obzorarr-docker-logo.svg" alt="obzorarr-docker logo" width="160" height="158">
</p>
<h1 align="center">obzorarr-docker</h1>
<p align="center"><strong>Multi-architecture Docker packaging for Obzorarr</strong></p>
<p align="center">
  <a href="https://github.com/edbfi/obzorarr-docker/blob/release/LICENSE"><img src="https://img.shields.io/badge/packaging_license-GPL--3.0-blue?style=flat" alt="Docker packaging license: GPL-3.0"></a>
  <a href="https://github.com/moby/moby" title="Docker Engine source (Moby)"><img src="https://img.shields.io/badge/Docker-2496ED?style=flat&amp;logo=docker&amp;logoColor=white" alt="Docker"></a>
  <a href="https://git.savannah.gnu.org/cgit/bash.git/" title="GNU Bash source"><img src="https://img.shields.io/badge/Bash-4EAA25?style=flat&amp;logo=gnubash&amp;logoColor=white" alt="Bash"></a>
  <a href="https://github.com/edbfi/obzorarr-docker/actions/workflows/call-build.yml?query=branch%3Arelease"><img src="https://img.shields.io/github/actions/workflow/status/edbfi/obzorarr-docker/call-build.yml?branch=release&amp;event=push&amp;label=CI&amp;style=flat" alt="CI"></a>
</p>

Docker packaging for [Obzorarr](https://github.com/edbfi/obzorarr).

[Documentation and examples](https://web.edb.fi/containers/obzorarr/).

<p align="center">Release channel</p>

## Releases

The release channel follows the app's latest published, non-prerelease GitHub Release with a plain `X.Y.Z` tag (release titles may say `v`). Publish only an owner-approved, tested commit from `main`; never move a published tag. A bare tag does not publish anything.

## Environment variables

Set `ORIGIN` to the address people open in the browser, for example `ORIGIN=http://192.168.1.10:3000` (in Compose, `- ORIGIN=http://192.168.1.10:3000`). It is **required when serving plain HTTP**: without it Obzorarr assumes `https://<Host>`, so signing in and saving changes fail. Leave it unset only behind an HTTPS reverse proxy that passes the original `Host`. It must be a bare origin (no path, query or credentials), or the app does not start.

Behind a reverse proxy, set `ADDRESS_HEADER=x-forwarded-for` (and `XFF_DEPTH` to the number of proxies, default `1`) only when every request goes through that proxy, so the app sees the real client address. `PROTOCOL_HEADER` and `HOST_HEADER` are for setups without `ORIGIN`, behind a trusted proxy.

`SHUTDOWN_TIMEOUT` is the number of seconds the app waits for open requests when the container stops. The image sets `5` so a plain `docker stop` (10 s) finishes cleanly; if you raise it, raise the stop timeout too (`docker stop -t`, `stop_grace_period`).

## Building

Images are built and published by the Hotio workflows in `edbfi/base-image`. `.github/workflows/call-build.yml` runs on every push (except to a branch named `workflows`) and builds linux/amd64 and linux/arm64, smoke-tests each architecture on `test_url` (`test_amd64`, `test_arm64`), then publishes `ghcr.io/edbfi/obzorarr-docker:<branch>`, `<branch>-<commit>` and `<branch>-<version>` (for a release version also `latest` and `<branch>-v<major>`, `<branch>-v<major>.<minor>` and `<branch>-v<version>`). `.github/workflows/call-update.yml` runs hourly: it evaluates the `__command` keys in `meta.json` (the latest obzorarr release tag as `version`, the current `alpinevpn` base image as `upstream_tag_sha`) and commits any change, which triggers a new build. The `nightly` branch builds the latest `main` commit the same way.

To build locally, run `./build.sh amd64` or `./build.sh arm64` from the repository root (needs `docker` and `jq`); `./build.sh update` refreshes `meta.json` the way the hourly workflow does.

## License

- Docker packaging: [GPL-3.0 license](LICENSE).
- Application: [AGPL-3.0-or-later source repository](https://github.com/edbfi/obzorarr).
