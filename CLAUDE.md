# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this checkout is

Packaging-only repo for the `obzorarr` Docker image; no application source. Both Dockerfiles
download `https://github.com/engels74/obzorarr/archive/${VERSION}.tar.gz` at build time, so
app behaviour changes belong upstream, not here.

This is the `release` branch: a retained, frozen channel (stable `0.1.11`). Its
`call-build` and `call-update` workflows are disabled in GitHub, so a push here builds and
publishes nothing, and `meta.json` no longer auto-updates. The maintained channel is
`origin/nightly`, which has a different toolchain (`tools/*.py`, `ghcr.io/edbfi/base-image`,
digest- and checksum-pinned `meta.json`, its own `ci.yml`).

| Question | → `release` (this branch) | → `nightly` |
| --- | --- | --- |
| Fixing or changing the image people should run? | | ✅ |
| Keeping stable 0.1.11 packaging reproducible or documented? | ✅ | |
| Copying a Dockerfile, `build.sh`, or `meta.json` pattern? | only within this branch | only within nightly |

Never merge channel branches wholesale or cherry-pick nightly's tooling here; README, `meta.json`,
workflows and Dockerfiles legitimately differ (`git diff release origin/nightly --stat`). The
`pr` channel branch is also historical and carries its own `update-versions.sh`.

## Commands

No tests, linter, or typechecker exist on this branch. Validation is an image build:

```sh
./build.sh amd64   # or arm64; needs docker + jq; tags "<repo-dir-name>-amd64"
jq -r 'to_entries[] | [(.key | ascii_upcase),.value] | join("=")' < meta.json  # build args it passes
```

Smoke check matches `meta.json` `test_url`: run the image and `curl -fsSL http://localhost:3000`.

## meta.json (legacy update contract)

- Every key becomes an uppercase `--build-arg`; the Dockerfiles consume only `VERSION`,
  `UPSTREAM_IMAGE`, `UPSTREAM_TAG_SHA`, and `IMAGE_STATS` (not in `meta.json`; the external CI
  supplied it).
- `*__command` values are shell snippets the (now disabled) update workflow evaluated, writing
  the output back to the key without the suffix. To change discovery, edit the `__command`;
  `version`, `upstream_tag_sha`, and `packages_hash` are bot-owned and would be overwritten.
- `packages.txt` is bot-generated and intentionally empty; don't hand-edit it.

## Runtime contract

`APP_DIR`, `CONFIG_DIR`, `UMASK`, the `hotio` user, `/etc/s6-overlay/scripts/bash-functions`,
and the `init-setup` / `init-wireguard` services come from the base image
(`ghcr.io/engels74/base-image:alpinevpn`), not this repo. Use the variables; don't hardcode
`/app` or `/config`. Persistent state is `${CONFIG_DIR}/data`: the Dockerfiles symlink
`${APP_DIR}/data` to it and `init-setup-app/run` chowns it to `hotio`. Services drop privileges
with `exec s6-setuidgid hotio ...` (see `service-obzorarr/run`).

## Adding an s6 service

1. `root/etc/s6-overlay/s6-rc.d/<name>/type` containing `oneshot` or `longrun`.
2. `<name>/run` starting with `#!/command/with-contenv bash`; a `oneshot` also needs `up` holding
   the absolute in-container path of its `run` (copy `init-setup-app/up`).
3. Ordering: empty file `<name>/dependencies.d/<dependency>`.
4. Enable: empty file `root/etc/s6-overlay/user-bundles.d/user/contents.d/<name>`. Not
   `s6-rc.d/user/contents.d/`: the service silently never starts on s6-overlay >= 3.2.3.1
   (commit `b8a1b6c`).
5. No `chmod` needed; both Dockerfiles end with a `find ... -name "run*" ... chmod +x`.

## Gotchas

- Edit `linux-amd64.Dockerfile` and `linux-arm64.Dockerfile` together. They are identical except
  arm64's builder installs `build-base python3`; divergence breaks one architecture.
- Keep `# check=skip=InvalidDefaultArgInFrom` at the top of both; `FROM ${UPSTREAM_IMAGE}:...`
  trips that BuildKit check otherwise.
- Changing the port touches: `ENV PORT=` and `WEBUI_PORTS=` in both Dockerfiles, the
  `!= "3000"` guard in `init-setup-app/run`, and `test_url` in `meta.json`.
- Human commits use Conventional Commits. `Modified: <file>` commits are the bot's format;
  don't imitate it.
