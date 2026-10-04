# Obzorarr Docker Image (Nightly)

## For full documentation

Installation instructions, Docker Compose examples and published image tags are available in the [Obzorarr container documentation](https://web.edb.fi/containers/obzorarr/).

## Environment variables

Set `ORIGIN` to the address people open in the browser, for example `ORIGIN=http://192.168.1.10:3000` (in Compose, `- ORIGIN=http://192.168.1.10:3000`). It is **required when serving plain HTTP**: without it Obzorarr assumes `https://<Host>`, so signing in and saving changes fail. Leave it unset only behind an HTTPS reverse proxy that passes the original `Host`. It must be a bare origin (no path, query or credentials), or the app does not start.

Behind a reverse proxy, set `ADDRESS_HEADER=x-forwarded-for` (and `XFF_DEPTH` to the number of proxies, default `1`) only when every request goes through that proxy, so the app sees the real client address. `PROTOCOL_HEADER` and `HOST_HEADER` are for setups without `ORIGIN`, behind a trusted proxy.

`SHUTDOWN_TIMEOUT` is the number of seconds the app waits for open requests when the container stops. The image sets `5` so a plain `docker stop` (10 s) finishes cleanly; if you raise it, raise the stop timeout too (`docker stop -t`, `stop_grace_period`).

## Building

The nightly image pins the reviewed application revision and archive checksum in
`meta.json`, along with both native base-image digests. Bun 1.4.2 is pinned to the
same image digest for build and runtime. Dependency lockfiles and Hotio's s6/data
layout are retained.

Run `./build.sh amd64` or `./build.sh arm64` from the repository root to build the
image locally. It needs `docker` and `jq`, passes the `meta.json` keys as build
arguments, and the Dockerfiles verify the source archive against `source_sha256`.
Pushes to `nightly` build and publish the image on GitHub runners: `.github/workflows/build-nightly.yml`
calls the reusable workflow in `edbfi/base-image`, which builds linux/amd64 and linux/arm64,
smoke-tests each architecture using `test_url`, `test_amd64` and `test_arm64` from `meta.json`,
and publishes the images to `ghcr.io/edbfi/obzorarr-docker` with `nightly` tags. Only pushes to
`nightly` (or a manual run) start it; pushes to other branches build and publish nothing.
`./build.sh` stays for local builds.

Stable 0.1.11 and the historical PR channel are retained separately. Their legacy
workflows remain disabled; do not merge channel branches wholesale or relabel
nightly as stable.
