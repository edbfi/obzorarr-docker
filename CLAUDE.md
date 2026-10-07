# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this checkout is

Packaging-only repo for the `obzorarr` Docker image; no application source. Both Dockerfiles
download `https://github.com/edbfi/obzorarr/archive/${VERSION}.tar.gz` at build time, so
app behaviour changes belong upstream, not here.

This is the `release` branch: it builds obzorarr's latest published, non-prerelease GitHub
Release with a plain `X.Y.Z` tag (now `0.1.11`; `version__command` reads `releases/latest`, so a
bare tag publishes nothing). Publish a backport release with "Set as the latest release"
unchecked, or `release` moves back to that version.
`call-build` builds and publishes every push to this branch (it skips only a branch named
`workflows`, so use that name for PR branches), and the hourly `call-update` bumps `meta.json`
and pushes, which builds again. `origin/nightly` builds the latest obzorarr `main` commit with
the same workflows.

Change both channels together: callers, `build.sh`, `.gitignore`, Dockerfiles and `root/` are
the same on `release` and `nightly`. They differ only in `meta.json`'s channel values
(`description`, `latest`, `version`, `version__command`) and, until obzorarr publishes its first
SvelteKit 3 release, in release's `scripts/serve.ts*` wildcard copy and `bun start` run file
(`0.1.11` has no `scripts/serve.ts`). `release` also carries `README.md` details, and only
`release` has `pullfrog.yml`, `immortality.yml`, `AGENTS.md` and this file
(`git diff origin/nightly release`). The `pr` channel branch keeps an older layout
and carries its own `update-versions.sh`.

## Commands

No tests, linter, or typechecker exist on this branch. Validation is an image build:

```sh
./build.sh amd64   # or arm64; needs docker + jq; tags "<repo-dir-name>-amd64"
jq -r 'to_entries[] | [(.key | ascii_upcase),.value] | join("=")' < meta.json | grep -v '__COMMAND'  # build args it passes
./build.sh update  # evaluates the __command keys like the hourly workflow
```

Smoke check matches `meta.json` `test_url`: run the image and `curl -fsSL http://localhost:3000`.

## meta.json (update contract)

- Every key except the `*__command` ones becomes an uppercase `--build-arg`; the Dockerfiles
  consume only `VERSION`, `UPSTREAM_IMAGE`, `UPSTREAM_TAG_SHA`, and `IMAGE_STATS` (not in
  `meta.json`; the workflow supplies it).
- `*__command` values are shell snippets the hourly update workflow evaluates, writing the
  output back to the key without the suffix. To change discovery, edit the `__command`;
  `version` and `upstream_tag_sha` are bot-owned and would be overwritten.
- `packages.txt` is bot-generated and intentionally empty; don't hand-edit it.

## Runtime contract

`APP_DIR`, `CONFIG_DIR`, `UMASK`, the `hotio` user, `/etc/s6-overlay/scripts/bash-functions`,
and the `init-setup` / `init-wireguard` services come from the base image
(`ghcr.io/edbfi/base-image:alpinevpn`), not this repo. Use the variables; don't hardcode
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
