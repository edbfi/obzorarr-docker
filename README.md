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
No workflow in this repository builds or publishes images.

Stable 0.1.11 and the historical PR channel are retained separately. Their legacy
workflows remain disabled; do not merge channel branches wholesale or relabel
nightly as stable.
