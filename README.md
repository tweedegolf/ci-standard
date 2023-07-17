# TG CI Standard

This repository contains the standard CI used for TG's Rust projects. The aim is to provide projects with a basic CI without having to spend time engineering it.

See [CI Standard](https://tgrep.nl/tweedegolf/ci-standard) for the GitLab version.


## Usage

To use it, copy [.github/workflows/ci.yml](https//github.com/tweedegolf/ci-standard/blob/main/.github/workflows/ci.yml) into your GitHub project. For some steps you might want to include extra steps, see [customising](#customising) for examples.


## Steps included

The following steps are included in the CI. The steps cache whatever makes sense to cache for Rust projects (see [caching](#caching) for more details).

### Linting
- `cargo format`

### Build
- `cargo build`
- `cargo clippy`
- `cargo test`

### Code quality
- `cargo deny`
- `cargo outdated`
- `cargo udeps` (nightly)

## Caching

Caching is stable over multiple jobs using [Swatinem/rust-cache](https://github.com/Swatinem/rust-cache) on the following directories:
```yaml
path: |
  .cargo/bin/
  .cargo/registry/index/
  .cargo/registry/cache/
  .cargo/git/db/
  target
```

## Customising
No project is the same, therefore, feel free to customise your setup. Here are some examples:

### `cargo-deny`

[`cargo deny`](https://github.com/EmbarkStudios/cargo-deny) expects a [`deny.toml`](https://github.com/tweedegolf/ci-standard/blob/main/deny.toml) in order to be able to check which licenses are allowed. Add it using `cargo deny init` and configure it as described in [the book](https://embarkstudios.github.io/cargo-deny/checks/cfg.html).

On the GitHub version of TG standard CI, `cargo deny` performs all steps (`advisories`, `bans`, `licenses` and `sources`), but it is also possible to only perform a subset, for example:
```diff
deny:
  runs-on: ubuntu-latest
  needs: build
  steps:
  - uses: actions/checkout@v3
  - uses: Swatinem/rust-cache@v2
    with:
      prefix-key: cargo
      shared-key: build
  - uses: tweedegolf/ci-standard/.github/actions/cargo-deny@main
+    with:
+      command: check advisories bans sources
```


### `cargo test`

In order to be able to run `cargo test`, one might need some extra services. For example, if you need a postgres container, the following might work:
```diff
test:
  runs-on: ubuntu-latest
  needs: build
+  services:
+    postgres:
+      image: postgres
+      env:
+        POSTGRES_PASSWORD: postgres
+      options: >-
+        --health-cmd pg_isready
+        --health-interval 10s
+        --health-timeout 5s
+        --health-retries 5
+      ports:
+        - 5432:5432
  steps:
  - uses: actions/checkout@v3
  - uses: Swatinem/rust-cache@v2
    with:
      prefix-key: cargo
      shared-key: test
  - run: cargo test --all-features --all-targets
```


## Contributing

Feel free to contribute via a PR if you think we should include more stuff or do things differently.
