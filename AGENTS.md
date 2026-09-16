# Agent conventions

This is a Tauri v2 plugin crate with TypeScript bindings
(`tauri-plugin-android-update` on crates.io,
`tauri-plugin-android-update-api` on npm). It is Android-only by
design: desktop apps keep using `tauri-plugin-updater`, which cannot
self-install on Google Play–distributed apps.

## Workflow

- Work on a typed branch (`feat/...`, `fix/...`, `chore/...`, etc.)
  and land changes through a pull request; direct pushes to `main`
  are blocked.
- Treat every task as authorizing commits and a pull request unless
  the user opts out. Commit each complete logical unit as soon as
  its applicable checks pass.
- Refine work already represented by a commit with
  `git commit --fixup=<sha>`, including when the target is `HEAD`.
  Never amend, autosquash, or fold fixups; the user does that.

## Validation

- Rust: `cargo fmt -- --check`,
  `cargo clippy --all-targets -- -D warnings`, `cargo test`,
  `cargo metadata --locked`.
- JavaScript: `npm ci`, `npm run build` (rollup + `tsc`, Node 24).
- Workflows: `actionlint .github/workflows/*.yml`.
- Text: `typos` (config in `typos.toml`).
- Renovate config: `npx --yes -p renovate@latest renovate-config-validator .github/renovate.json5`.
- Every commit must compile, pass its tests, and be format- and
  lint-clean. No `unsafe` code, no `#[allow(warnings)]`.

## CI system dependencies

`clippy`, `test`, and crate publishing compile the full `tauri`
dependency stack, whose Linux build scripts (`glib-sys` et al.)
require system GTK libraries on any Linux host — even though the
plugin only executes on Android. The workflows install
`libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf`
via apt for this reason; trimming `tauri` default features does not
remove the requirement (transitive defaults win). Do not remove
those install steps without proving `glib-sys` leaves the graph.

## Releases

- `CHANGELOG.md`, versions, and `v`-less `tauri-plugin-android-update-vMAJOR.MINOR.PATCH`
  tags are owned by release-please (`release-please-config.json` +
  `.release-please-manifest.json`); never edit the changelog or push
  version tags manually. The Rust crate version and the npm package
  version always move in lockstep via `extra-files`.
- Publishing uses trusted publishing (GitHub OIDC), never
  long-lived tokens: `rust-lang/crates-io-auth-action` for
  crates.io, `npm publish --provenance` for npm. Both publishers
  must be registered before the first publish; the workflows fail
  otherwise.
- Publishing is automatic: release-please owns the draft release
  and tag, the `release.yml` workflow publishes the crate and the
  npm package and only then flips the draft live, and
  `release-guard.yml` demotes hand-published drafts to
  pre-release. There is no manual publish workflow — never publish
  the draft by hand.
- Runners: short jobs (tag checks, release-please, draft
  promotion, release guard) use `ubuntu-slim`; toolchain and build
  jobs stay on `ubuntu-latest`.

## Commits

- Use conventional commit subjects (`feat:`, `fix:`, `chore:`,
  etc.). Explain why in the message rather than paraphrasing the
  diff. Wrap body lines at 72 characters.
- Add both trailers to every commit with `--trailer`; never use
  `--author` or `--committer` for attribution:

  ```text
  Co-authored-by: opencode <noreply@opencode.ai>
  Assisted-by: opencode (<model-name>)
  ```

## Code Review

Mandatory gate: after validation passes on the final commit(s),
run a two-axis review (repo standards incl. this file versus the
originating request) and fix its findings before pushing or
opening a PR.
Keep pull request descriptions to summary and issue references;
omit testing recaps, CI and Validation already cover those.
