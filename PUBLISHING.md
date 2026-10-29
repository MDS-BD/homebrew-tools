# Publishing a tool to this tap

Tools are released by [GoReleaser](https://goreleaser.com) from their own repository's GitHub Actions: a `v*` tag
builds the binaries, publishes a GitHub release, signs and notarizes the macOS builds, and pushes a cask to
`Casks/` here. Nobody edits this repository by hand.

[`MDS-BD/aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) is the reference setup: copy its `.goreleaser.yaml` and
`.github/workflows/release.yml`, then change the names as below. The examples here are for Go; GoReleaser also
builds Rust, Zig, Bun and Deno projects, and prebuilt binaries.

## License and release-archive requirements

The tap's own material is licensed under the root [MIT License](LICENSE).

Application archives are separate distributions maintained by each tool repository. Every release archive for every supported OS/architecture must contain the tool's `LICENSE`, a `LICENSES/` bundle with applicable third-party license texts and notices (including nested dependency licenses), and a versioned inventory/provenance record identifying the app and dependency versions and the origin/version of each notice. Include complete applicable source and license for any third-party source snapshot distributed with the app. Do not substitute an SPDX-only list for the required license texts. The Go toolchain license must match the compiler used, and the archive must include the runtime vendor notices actually shipped with the binary. The tap does not bundle or replace those application notices.

For Go applications, use `actions/setup-go`'s `go-version-file: go.mod` so the workflow selects the Go version declared by that repository. Record the actual compiler selected in CI; verify the precise compiler version used for a release from the running workflow and built binary, not by assuming that `go.mod` identifies the patch compiler. Configure GoReleaser to use the installed `go` tool and pass `-mod=readonly`; do not run `go mod tidy` during release, so builds use committed dependency versions. Refresh the Go license and compiler provenance for the toolchain actually used.

For additional per-target inspection details and copyable rerun commands, see [Rerunning the license audit](#rerunning-the-license-audit) below. Earlier toolchain and dependency findings in that procedure are historical evidence; re-check actual versions and applicability before a release.

## Rerunning the license audit

Prerequisites: the Go toolchain selected for the audit/release, `jq`, `tar`, `find`, and GoReleaser on `PATH`; network access for Go to fetch the pinned scanner and any uncached modules. The commands below use standard macOS/Linux shell utilities.

Run this procedure from the root of each Go application repository (for example, `aws-tunnel` or `ecr-mirror`) before a release and after changing dependencies, build tags, target platforms, or the Go toolchain. It audits the four configured release targets (`darwin`/`linux` × `amd64`/`arm64`) for the `./cmd` release binary. Replace the sample executable below with the actual executable name. Earlier audit reports are snapshots, not current release evidence: the Go 1.27.3 host observation records the compiler then used for that audit only, not a pin or a guarantee about a later release.

```sh
set -euo pipefail
AUDIT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/license-audit.XXXXXX")"
mkdir -p "$AUDIT_DIR/bin"
go version | tee "$AUDIT_DIR/compiler-version.txt"
go env GOVERSION GOROOT GOTOOLCHAIN GOOS GOARCH | tee "$AUDIT_DIR/compiler-environment.txt"
for os in darwin linux; do
  for arch in amd64 arm64; do
    printf '\n== %s/%s ==\n' "$os" "$arch"
    GOOS="$os" GOARCH="$arch" CGO_ENABLED=0 go list -mod=readonly -deps -json ./cmd \
      > "$AUDIT_DIR/$os-$arch-go-packages.json"
    jq -r 'select(.Module != null and (.Module.Main != true)) | [.Module.Path, .Module.Version] | @tsv' \
      "$AUDIT_DIR/$os-$arch-go-packages.json" \
      | sort -u | tee "$AUDIT_DIR/$os-$arch-modules.tsv"
    jq -r 'select(.Module == null and (.ImportPath | startswith("vendor/"))) | .ImportPath' \
      "$AUDIT_DIR/$os-$arch-go-packages.json" \
      | sort -u | tee "$AUDIT_DIR/$os-$arch-go-vendor-imports.txt"
  done
done
GOBIN="$AUDIT_DIR/bin" go install github.com/google/go-licenses@v1.6.0
for os in darwin linux; do
  for arch in amd64 arm64; do
    GOOS="$os" GOARCH="$arch" CGO_ENABLED=0 GOFLAGS=-mod=readonly \
      "$AUDIT_DIR/bin/go-licenses" report ./cmd > "$AUDIT_DIR/$os-$arch-report.csv"
    GOOS="$os" GOARCH="$arch" CGO_ENABLED=0 GOFLAGS=-mod=readonly \
      "$AUDIT_DIR/bin/go-licenses" save ./cmd --save_path="$AUDIT_DIR/$os-$arch-licenses"
  done
done
printf 'Audit output: %s\n' "$AUDIT_DIR"
```

Use a temporary, version-pinned `go-licenses` installation. Version `v1.6.0` is an observed upstream Go module version, not guessed; update it only after resolving and reviewing an actual published module version. Each per-target report and saved collection is evidence to review, not a legal or source audit by itself.

Review every report warning and every file in the saved collections. Compare module/version inventories across all four targets, then inspect the exact-version dependency source and package-level license files—not just module-root names—including nested `LICENSE`, `COPYING`, `NOTICE` and `PATENTS` files, embedded source headers, vendored/forked code, assembly and any scanner `Unknown` result. Preserve complete applicable license texts and NOTICE attributions in the release bundle. The scanner can miss package-level exceptions or licenses without recognizable filenames, and non-Go-source warnings require manual source review; neither an empty warning list nor a successful `check` proves completeness. Do not reject a legitimate MPL dependency as if its license were unknown or forbidden: review its terms and satisfy its separate source obligations.

Earlier `aws-tunnel` audits found mixed-license AWS SDK/Smithy and Kubernetes package subpaths, and an Apache-licensed Prometheus `procfs` module whose license was not detected by the scanner. Earlier `ecr-mirror` audits found mixed-license Smithy and Docker notices and nested BSD/MIT terms in `klauspost/compress`. Re-check whether those paths and versions remain in fresh target inventories; carry forward every applicable full text and attribution. Earlier `aws-tunnel` inventories also contained `github.com/hashicorp/go-version@v1.9.0` under MPL-2.0. If it or another source-available/copyleft dependency remains, include its full license and satisfy its source offer: provide a complete copy of the exact-version Source Code Form, or a durable, recipient-accessible retrieval reference to that exact-version source; retain it if upstream access cannot be relied upon. Check the license's recipient instructions and any modifications' obligations separately. A report row or license copy alone is insufficient.

For Go itself, use the recorded compiler's `GOROOT` (from `go env GOROOT`) to identify and review its root `LICENSE` and licenses/notices for vendored runtime packages listed by the target inventory under `GOROOT/src/vendor`. Use that exact installation's `GOROOT/src/vendor/modules.txt` and package source directories to establish the selected vendor module versions and notice origins; record the actual compiler version, `GOROOT`/distribution provenance, each dependency and vendor module version, and each notice's source/version in the release provenance record. Retain the exact compiler's complete Go license and every applicable runtime-vendor notice; prior audit evidence for an older Go installation is not a substitute. If the Go distribution does not have `LICENSE` at `GOROOT/LICENSE`, locate the license from that exact installed distribution (for Homebrew, inspect the installed formula/cellar contents); do not copy a license from an unrelated Go version.

Finally create a release snapshot using the normal GoReleaser configuration, inspect every archive and binary, and retain the inspection with the audit:

```sh
TOOL=aws-tunnel  # use TOOL=ecr-mirror in that repository
goreleaser release --snapshot --clean --skip=publish
go version
for archive in dist/*.tar.gz; do
  printf '\n== %s ==\n' "$archive"
  tar -tzf "$archive" | sort | tee -a "$AUDIT_DIR/archive-contents.txt"
  (sha256sum "$archive" 2>/dev/null || shasum -a 256 "$archive") | tee -a "$AUDIT_DIR/archive-sha256.txt"
  archive_dir="$AUDIT_DIR/$(basename "$archive" .tar.gz)"
  mkdir -p "$archive_dir"
  tar -xzf "$archive" -C "$archive_dir"
  find "$archive_dir" -type f -name "$TOOL" -exec go version -m {} \;
done
```

Before publishing, require fresh evidence for every target: inspect each extracted archive's root `LICENSE`, complete `LICENSES/` texts, applicable dependency, NOTICE, Go toolchain and runtime-vendor notices, versioned inventory/provenance, and any required third-party source. Verify the actual toolchain on the release runner and use the matching Go toolchain to run `go version -m` on each archive's binary; compare its build information and target with the recorded toolchain/environment. Resolve missing files, unexplained scanner warnings, changed licenses, target-specific differences, or mismatched compiler evidence before publishing. Archive file listings alone do not verify text contents or compiler provenance.

The procedure does not grant or establish project copyright ownership and is not legal advice. Confirm that the project's own `LICENSE` is authorized and included; escalate unclear source-license obligations before distribution.

## What is already in place

These secrets are set at **organization level**, so any MDS-BD repository can use them in GitHub Actions without
further setup:

| Secret | Used for |
| ------ | -------- |
| `HOMEBREW_RELEASER_APP_ID` | GitHub App allowed to push to this repository only |
| `HOMEBREW_RELEASER_APP_PRIVATE_KEY` | private key of that app |
| `MACOS_SIGN_P12` | *Developer ID Application* certificate (base64 `.p12`) |
| `MACOS_SIGN_PASSWORD` | password of that certificate |
| `MACOS_NOTARY_ISSUER_ID`, `MACOS_NOTARY_KEY_ID`, `MACOS_NOTARY_KEY` | App Store Connect API key for notarization |

If a new repository cannot see them, an org admin has to add it to each secret's repository access list
(*Organization settings → Secrets and variables → Actions*).

## Steps

### 1. Release workflow

`.github/workflows/release.yml` in the tool repository. For Go, use `actions/setup-go`'s `go-version-file: go.mod` and record the actual compiler version used by the release; do not add a patch-level pin or `check-latest`.

```yaml
name: release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: actions/setup-go@v7
        with:
          go-version-file: go.mod

      - name: Record selected Go toolchain
        run: go version && go env GOVERSION GOROOT GOTOOLCHAIN

      # Short-lived token from the org GitHub App, allowed to push the cask to the tap repo only.
      - uses: actions/create-github-app-token@v3
        id: tap-token
        with:
          client-id: ${{ secrets.HOMEBREW_RELEASER_APP_ID }}
          private-key: ${{ secrets.HOMEBREW_RELEASER_APP_PRIVATE_KEY }}
          owner: MDS-BD
          repositories: homebrew-tools
          permission-contents: write

      - uses: goreleaser/goreleaser-action@v7
        with:
          distribution: goreleaser
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          HOMEBREW_TAP_GITHUB_TOKEN: ${{ steps.tap-token.outputs.token }}
          MACOS_NOTARY_ISSUER_ID: ${{ secrets.MACOS_NOTARY_ISSUER_ID }}
          MACOS_NOTARY_KEY: ${{ secrets.MACOS_NOTARY_KEY }}
          MACOS_NOTARY_KEY_ID: ${{ secrets.MACOS_NOTARY_KEY_ID }}
          MACOS_SIGN_P12: ${{ secrets.MACOS_SIGN_P12 }}
          MACOS_SIGN_PASSWORD: ${{ secrets.MACOS_SIGN_PASSWORD }}
```

### 2. GoReleaser config

Copy [`aws-tunnel/.goreleaser.yaml`](https://github.com/MDS-BD/aws-tunnel/blob/main/.goreleaser.yaml) and replace
every `aws-tunnel` with your tool's name. The parts that matter for this tap:

- **`builds`**: `goos: [darwin, linux]`, `goarch: [amd64, arm64]`, `CGO_ENABLED=0` so the binaries run anywhere.
  Keep the build `id`: `notarize.macos.ids` refers to it.

For Go applications, use GoReleaser's `tool: go` build setting, which selects the `go` executable installed by `actions/setup-go`. Configure GoReleaser build flags to include `-mod=readonly` and retain required environment such as `CGO_ENABLED=0`; do not run `go mod tidy` during release, so builds use committed dependency versions. Record the selected compiler with `go version` in CI and inspect released binaries with `go version -m`; `go.mod` does not establish the precise compiler version used for a release.
- **`notarize.macos`**: signs and notarizes the darwin binaries with the org Apple credentials, from the Linux
  runner. Keep it: unsigned binaries are blocked by Gatekeeper.
- **`release.github`**: `owner: MDS-BD`, `name: <your repo>`.
- **`homebrew_casks`**:
  - `name`, `binaries`, `homepage`, `description` for your tool;
  - `repository: {owner: MDS-BD, name: homebrew-tools, branch: main, token: "{{ .Env.HOMEBREW_TAP_GITHUB_TOKEN }}"}`
    and `directory: Casks`;
  - `url.template` pointing at your tool repository's public release assets, for example
    `https://github.com/MDS-BD/<tool>/releases/download/{{ .Tag }}/{{ .ArtifactName }}`. Keep the normal Homebrew downloader: do not set `url.using` or define a `custom_block`. Release assets must be publicly downloadable for anonymous Homebrew installs.

Check it locally before tagging (installs with `go install github.com/goreleaser/goreleaser/v2@latest`):

```sh
goreleaser check
goreleaser release --snapshot --clean --skip=publish   # builds everything, writes dist/homebrew/Casks/<tool>.rb
```

### 3. Release

```sh
git tag v0.1.0 && git push origin v0.1.0
```

The workflow takes about 15 minutes, most of it Apple notarization. When it is done, `Casks/<tool>.rb` appears here
in a `feat: <tool> v0.1.0` commit by goreleaserbot.

### 4. List it

Add the tool to the table in [README.md](README.md), then check the install as a user would:

```sh
brew update
brew install --cask mds-bd/tools/<tool>
codesign -dv "$(which <tool>)" 2>&1 | grep Authority   # Developer ID Application: RTI spa
```

## Troubleshooting

- **`Invalid keyData`** in *create-github-app-token*: `HOMEBREW_RELEASER_APP_PRIVATE_KEY` is not a valid `.pem`.
  Set it from the file itself (`gh secret set ... < key.pem`), never by pasting.
- **Resource not accessible** when pushing the cask: the GitHub App is not installed on this repository, or lacks
  *Contents: write*.
- **Empty secrets** (`client-id` must be set, or the notarize step is skipped): the repository is not in the org
  secrets' access list.
- **`Calling postflight is deprecated!`** on install: GoReleaser still generates the old stanza
  ([goreleaser#6870](https://github.com/goreleaser/goreleaser/issues/6870)). Harmless; it goes away with a
  GoReleaser release that ships the fix.
