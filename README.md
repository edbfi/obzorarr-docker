# Obzorarr Docker Image (Nightly)

## For full documentation

Installation instructions, Docker Compose examples and published image tags are available in the [Obzorarr container documentation](https://web.edb.fi/containers/obzorarr/).


## Building

The nightly image pins the reviewed application revision and archive checksum in
`meta.json`, along with both native base-image digests. Bun 1.4.2 is pinned to the
same image digest for build and runtime. Dependency lockfiles and Hotio's s6/data
layout are retained.

Run `./build.sh amd64` or `./build.sh arm64` from the repository root to build the
image locally. It needs `docker` and `jq`, passes the `meta.json` keys as build
arguments, and the Dockerfiles verify the source archive against `source_sha256`.
Pushes to `nightly` also build the image on GitHub runners: `.github/workflows/build-nightly.yml`
calls the reusable workflow in `edbfi/base-image`, which builds linux/amd64 and linux/arm64,
smoke-tests each architecture using `test_url`, `test_amd64` and `test_arm64` from `meta.json`,
and publishes the images to `ghcr.io/edbfi/obzorarr-docker`. The caller is kept identical to the
one in `edbfi/base-image` apart from its file name and `name`, so it runs on a push to any branch except `workflows` and the
published tags are named after the branch. `./build.sh` stays for local builds.

Stable 0.1.11 and the historical PR channel are retained separately. Their legacy
workflows remain disabled; do not merge channel branches wholesale or relabel
nightly as stable.
