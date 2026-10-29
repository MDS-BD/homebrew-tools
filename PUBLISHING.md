# Public package publication playbook

This is the canonical, company-agnostic procedure for publishing a Go command-line tool through a public GitHub release and a public Homebrew tap. It is designed for projects public from their first commit: use only public, synthetic examples, configuration, test fixtures and dependencies throughout development. Do not introduce company endpoints, accounts, customer data, real customer fixtures, credentials or secrets into source, examples, tests, commits, tags, release artifacts, logs or public issue reports. Use reserved `example.com` names and clearly synthetic identities such as `example-account` in examples. If a real project or its history already contains sensitive material, stop and resolve the exposure through the repository/security owner before making it public; publishing this guide does not sanitize history.

This repository is one deployment profile, not a prerequisite: examples use synthetic names such as `example-org`, `example-tool`, `example-cmd`, `example.com`, and the public `example-org/homebrew-tools` tap. Substitute the tool's actual public repository, tap owner/repository, command entrypoint and binary name. A Go executable may deliberately use distinct names for its Go import module, Go main-package path, GoReleaser project/build ID, output binary, Homebrew cask token, and public repository; map and verify each separately. For example, `example-org/example-tool` may build `./cmd/real-cli-name` into the `example-tool` binary and cask. The `terraform-migrator` path is an additional repository-specific name-mapping example only. Do not mechanically derive one identifier from another.

The project's complete source, release metadata and downloadable assets must be public. The release archive must be downloadable without credentials by an anonymous consumer using Homebrew; no private-download strategy, custom token flow or credentials in the cask. Publication credentials authorize only the publisher's operations and never gate user downloads. Confirm the actual legal copyright owner and licensing authority for the tool and every embedded notice; do not copy an example copyright holder. Likewise, use a valid Developer ID Application identity authorized for this project and confirm the certificate identity actually used. Notarization/signing evidence, source and license compliance, and release availability are distinct checks; do not claim them from a successful build or tap commit alone.

## End-to-end checklist
Follow these gates in order for initial publication and each release. The detailed [license archive requirements](#license-and-release-archive-requirements), [canonical license audit rerun](#rerunning-the-license-audit), and [publication workflow](#universal-publication-workflow-and-release-configuration) below define the full evidence and configuration. Do not treat old audit output as a new release audit.

1. **Public project setup.** Establish public project and tap repositories, approved project naming, canonical Go entrypoint and output binary, homepage, owner-approved copyright/license text, supported OS/architecture targets, and any runtime requirements. Keep module, build package, binary, cask and repository names distinct where appropriate. Document user-installed runtime tools, if any; do not bundle or imply bundling of external programs without explicit configuration and artifact evidence.
2. **Ownership, license and runtime inventory.** Confirm authority to publish the source under the project's chosen license and identify the correct copyright owner. Inventory exactly the packages compiled into the release binary for every configured target, source and notice exceptions, Go runtime/vendor notices, and any third-party source distributed alongside the executable. Review actual license texts and source terms, not only automated scanner output. Complete the audit procedure below; resolve every unknown or obligation before release.
3. **Configure build and publisher.** Add the target matrix, `CGO_ENABLED=0` where supported by the tool, GoReleaser build `main` that matches the audited package, output `binary` matching the intended CLI/cask executable, readonly module mode, full archives and a cask pointed at public release downloads. Configure a tag-triggered workflow. Keep publishing credentials in CI secret storage and scope the release token to its source repository, and the separate tap token to only the tap repository. No credential belongs in project files or casks.
4. **Audit and snapshot.** With the Go version selected from the project's `go.mod`, rerun the per-target license audit, verify that dependencies are unchanged (`go mod verify`), validate the GoReleaser config (`goreleaser check`), and create a non-publishing snapshot (`goreleaser release --snapshot --clean --skip=publish`). Inspect every archive and binary, verify target names, expected CLI help, notices, provenance, checksums, signing state and build information. This builds cross-platform artifacts; it does not execute foreign-OS binaries.
5. **Release.** On the actual publishing project, only after completing the audit/snapshot and providing each license-required exact-version source offer and recipient notice, review source changes and publisher credentials, create a new semantic-version tag on the approved source commit, and push that tag to the public origin. Follow CI to confirm all archives/checksums are uploaded, macOS signing and notarization ran successfully with the authorized certificate, and the tap update used the scoped token. Treat tags as immutable: never move/reuse a public tag or overwrite assets; fix failures in source/configuration, then release a new version.
6. **Verify the public release and tap.** Inspect anonymous access to all four release archive URLs and checksums. Verify Linux archives on a Linux host and macOS artifacts, signature identity and Gatekeeper/notarization status on macOS; record which platform checks were actually run. Verify every cask URL/checksum maps to the just-published target archive and use the public install/CLI check below on supported host platforms. The publisher additionally checks CI job status, API/tap-write permissions and server/release logs; an end-user install or HTTP success alone proves none of those publisher-side states.
7. **Close out and maintain.** Retain audit inputs/reports, actual compiler and environment evidence, archive listings/content review, hashes, build metadata, CI run, signature/notarization result/log reference, anonymous-download evidence, and tap/install verification with the release record. Update the README's package listing when introducing a tool. For every dependency, build-tag, target, compiler, archive, workflow or publication-identity change, rerun the audit and snapshot checks. Each report/provenance record is version-specific evidence keyed to source revision, dependency inventory and selected compiler; never relabel or reuse it for a newer release.

### Example placeholder conventions

Commands below are copied from repository-root context in a **tool project**, unless identified as public-consumer or macOS-only checks. They use `OWNER`, `REPO`, `TAP_OWNER`, `TAP_REPO`, `ENTRYPOINT`, `TOOL`, and `VERSION` as deliberate shell variables; replace each with the corresponding public value. `ENTRYPOINT` is the Go package path passed to Go commands and GoReleaser, while `TOOL` is the binary name and `REPO` is the GitHub repository name. Example URI values use reserved `example.com` and synthetic repositories. Do not mistake these placeholders for literal credentials or production identities.

```sh
OWNER=example-org
REPO=example-tool          # GitHub source repository name
PROJECT=example-tool       # GoReleaser project/archive name
TAP_OWNER=example-org
TAP_REPO=homebrew-tools     # GitHub repository name, used in the cask YAML
TAP=tools                   # Homebrew tap namespace, used by `brew tap`
ENTRYPOINT=./cmd/example-cmd  # exact package used in GoReleaser builds[].main
TOOL=example-tool             # output executable name
CASK=example-tool             # cask token; can differ from TOOL, PROJECT and REPO
VERSION=0.1.0
TAG="v$VERSION"
DOWNLOAD_DIR="$(mktemp -d "${TMPDIR:-/tmp}/release-check.XXXXXX")"
```

The examples are not a promise that shell variable values configure GoReleaser, workflows, or a project automatically. Replace the corresponding literal values in project YAML. For a command named `terraform-migrator`, set `ENTRYPOINT=./cmd/terraform-migrator`, `TOOL=terraform-migrator`, and use the real public repository identifier independently. Consult the [GoReleaser Go builder reference](https://goreleaser.com/customization/builds/builders/go/), [Homebrew Casks reference](https://goreleaser.com/customization/publish/homebrew_casks/), [GoReleaser release quick start](https://goreleaser.com/getting-started/quick-start/), and [Homebrew Cask Cookbook](https://docs.brew.sh/Cask-Cookbook) when adapting configuration.

## License and release-archive requirements

The tap's own material is licensed under the root [MIT License](LICENSE).

Every public tool archive for each supported OS/architecture is a distribution maintained by that tool's project repository. Each archive must include the tool's actual owner-authorized project `LICENSE`, a `LICENSES/` bundle containing complete applicable third-party license texts and notices (including nested dependency/data and vendor-runtime licenses), and a versioned dependency/license/provenance record identifying the source revision, dependency versions, compiler and origin/version of each notice. Include source-form offers or source access required by applicable licenses. Include all license/source notices required for third-party source snapshots or fixtures actually distributed with the application. A bare SPDX identifier/list is not a substitute for license texts, notices, or required source availability. The tool repository's license and the tap's own license are independent; neither substitutes for the other.

For Go applications, `actions/setup-go` must use `go-version-file: go.mod` so CI installs the Go version declared by the project repository; do not add a fixed compiler patch or `check-latest`. Record the actual compiler selected in the workflow and match the Go root/runtime notices to that installation. `go.mod` does not identify which patch compiler ultimately built a release. Configure GoReleaser to use the installed `go` on `PATH` and pass `-mod=readonly`; do not run `go mod tidy` during release. Refresh compiler-license and dependency/provenance evidence against each release's actual selected toolchain.

For copyable target-by-target inventory and re-audit commands, see the single canonical [Rerunning the license audit](#rerunning-the-license-audit) procedure below. Its earlier findings are historical evidence only; verify applicability against current sources before release.


## Rerunning the license audit

Prerequisites: the Go toolchain selected for the audit/release, `jq`, `tar`, `find`, and GoReleaser on `PATH`; network access for Go to fetch the pinned scanner and any uncached modules.

Run this procedure from the root of the public Go tool repository. Before release and after changing dependencies, build tags, targets, Go toolchain or release entrypoint, set `ENTRYPOINT` to the exact Go main-package path configured as GoReleaser `builds[].main` (for example, `./cmd/example-cmd`; use `./cmd/terraform-migrator` for that entrypoint). It audits the four intended release targets (`darwin`/`linux` × `amd64`/`arm64`) for that package. Earlier audit reports are historical evidence only: no compiler observation in them pins a later release. The compiler identity below is intentionally obtained from the toolchain selected for this run; use the same setup-go `go-version-file: go.mod` and selected `go` tool as the release, with no patch-level pin or `check-latest`.

```sh
set -euo pipefail
ENTRYPOINT=./cmd/example-cmd  # exact GoReleaser builds[].main package
AUDIT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/license-audit.XXXXXX")"
mkdir -p "$AUDIT_DIR/bin"
go version | tee "$AUDIT_DIR/compiler-version.txt"
go env GOVERSION GOROOT GOTOOLCHAIN GOOS GOARCH | tee "$AUDIT_DIR/compiler-environment.txt"
for os in darwin linux; do
  for arch in amd64 arm64; do
    printf '\n== %s/%s ==\n' "$os" "$arch"
    GOOS="$os" GOARCH="$arch" CGO_ENABLED=0 go list -mod=readonly -deps -json "$ENTRYPOINT" \
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
      "$AUDIT_DIR/bin/go-licenses" report "$ENTRYPOINT" > "$AUDIT_DIR/$os-$arch-report.csv"
    GOOS="$os" GOARCH="$arch" CGO_ENABLED=0 GOFLAGS=-mod=readonly \
      "$AUDIT_DIR/bin/go-licenses" save "$ENTRYPOINT" --save_path="$AUDIT_DIR/$os-$arch-licenses"
  done
done
printf 'Audit output: %s\n' "$AUDIT_DIR"
```

Use a temporary, version-pinned `go-licenses` installation. Version `v1.6.0` is an observed upstream Go module version, not guessed; update it only after resolving and reviewing an actual published module version. Each per-target report and saved collection is evidence to review, not a legal or source audit by itself.

Review every report warning and every file in the saved collections. Compare module/version inventories across all four targets, then inspect the exact-version dependency source and package-level license files—not just module-root names—including nested `LICENSE`, `COPYING`, `NOTICE` and `PATENTS` files, embedded source headers, vendored/forked code, assembly and any scanner `Unknown` result. Preserve complete applicable license texts and NOTICE attributions in the release bundle. The scanner can miss package-level exceptions or licenses without recognizable filenames, and non-Go-source warnings require manual source review; neither an empty warning list nor a successful `check` proves completeness. Do not reject a legitimate MPL dependency as if its license were unknown or forbidden: review its terms and satisfy its separate source obligations.

For each module in the exact release package inventory, use `go list -f '{{.Dir}} {{.Version}}' -m <module-path>` to locate its selected module source and inspect license-bearing files in that exact version, including nested package/submodule notices and source headers. This locates one module root and does not crawl nested files; follow the actual imported package paths. Exclude modules not used by the release entrypoint from the shipped-code inventory, while separately considering any tooling or source snapshots that the distribution actually includes.

For every MPL-2.0 module or other source-available dependency in the compiled closure, include its complete full-text license notice and satisfy applicable recipient source obligations. Mozilla Public License §3.2 for covered executable-form software requires making its corresponding Source Code Form available and telling recipients how to obtain it by reasonable means in a timely manner. Publish and test anonymous retrieval for exact-version source copies before the tagged release and include recipient-facing source-offer information with the distribution. Ask the project's rights holder to review which exact-version module files are covered, the terms/notice and any MPL-covered modifications. Preserve upstream source-file licensing notices; do not remove them.
The canonical project procedure applies to any Go main package. As one evidence-backed illustration (not a current allowlist), the target-specific audit for `./cmd/terraform-migrator` included three compiled HashiCorp MPL-2.0 modules (`go-version@v1.9.0`, `hcl/v2@v2.25.0`, `terraform-json@v0.29.0`), each requiring exact-version Source Code Form access and a reasonable, timely recipient notice; `github.com/agext/levenshtein@v1.2.1` with an Apache-2.0 `NOTICE`; nested Unicode UCD licensing for `go-textseg/v17@v17.0.1`; and the actual Go toolchain's `golang.org/x/net` runtime-vendor BSD license and `PATENTS`. Re-audit current versions and files for each release. The example binary invokes `terragrunt` as an external executable; Terraform/OpenTofu are user-installed runtime dependencies, not Go packages bundled by this entrypoint on this evidence.

For Go itself, use the recorded compiler's `GOROOT` (from `go env GOROOT`) to identify and review its root `LICENSE` and licenses/notices for vendored runtime packages listed by the target inventory under `GOROOT/src/vendor`. Use that exact installation's `GOROOT/src/vendor/modules.txt` and package source directories to establish selected vendor module versions and notice origins; record the actual compiler version, `GOROOT`/distribution provenance, each dependency and vendor module version, and each notice's source/version in the release provenance record. Retain the exact compiler's complete Go license and every applicable runtime-vendor notice; prior audit evidence for an older Go installation is not a substitute. If that Go distribution does not have `LICENSE` at `GOROOT/LICENSE`, locate it from that exact installed distribution; do not copy a license from an unrelated Go version.


Finally create a release snapshot using the normal GoReleaser configuration, inspect every archive and binary, and retain the inspection with the audit:
```sh
set -euo pipefail
ENTRYPOINT=./cmd/terraform-migrator  # release package in the worked example
TOOL=terraform-migrator             # GoReleaser binary in the worked example
AUDIT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/license-audit.XXXXXX")"
mkdir -p "$AUDIT_DIR/bin"
go version
go env GOVERSION GOROOT GOTOOLCHAIN GOOS GOARCH
go list -mod=readonly -deps -json "$ENTRYPOINT" > "$AUDIT_DIR/snapshot-host-packages.json"
for archive in dist/*.tar.gz; do
  printf '\n== %s ==\n' "$archive"
  tar -tzf "$archive" | sort | tee -a "$AUDIT_DIR/archive-contents.txt"
  (sha256sum "$archive" 2>/dev/null || shasum -a 256 "$archive") | tee -a "$AUDIT_DIR/archive-sha256.txt"
  archive_dir="$AUDIT_DIR/$(basename "$archive" .tar.gz)"
  mkdir -p "$archive_dir"
  tar -xzf "$archive" -C "$archive_dir"
  find "$archive_dir" -type f -name "$TOOL" -exec go version -m {} \\;
done
```

Before publishing, require fresh evidence for every target: inspect each extracted archive's root `LICENSE`, complete `LICENSES/` texts, applicable dependency, NOTICE, Go toolchain and runtime-vendor notices, versioned inventory/provenance, and any required third-party source. Verify the actual toolchain on the release runner and use the matching Go toolchain to run `go version -m` on each archive's binary; compare its build information and target with the recorded toolchain/environment. Resolve missing files, unexplained scanner warnings, changed licenses, target-specific differences, or mismatched compiler evidence before publishing. Archive file listings alone do not verify text contents or compiler provenance.

The procedure does not grant or establish project copyright ownership and is not legal advice. Confirm that the project's own `LICENSE` is authorized and included; escalate unclear source-license obligations before distribution.


## Universal publication workflow and release configuration

### 1. Release workflow

Add a release workflow to the **tool** repository, triggered by a new semantic-version `v*` tag. This illustrative job uses placeholders and GitHub Actions interfaces already in this deployment; select maintained action versions, review their permissions and pin policies, and configure your own public source repository and tap. For a Go project, `actions/setup-go` must use `go-version-file: go.mod` with no manually pinned Go patch or `check-latest`; record the actual selected compiler with the source revision in CI. Keep the default `GITHUB_TOKEN`'s write permission on the tool repository limited to what publishes its release. Mint a separate short-lived GitHub App token for the tap, configured for **only** that tap repository with `contents: write`; do not pass the source-release token to the tap. Keep all private keys/API credentials in Actions secrets, never in YAML, repository config, release notes or artifacts.

```yaml
name: release
on:
  push:
    tags: ["v*"]

permissions:
  contents: write

jobs:
  release:
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
      - name: Verify committed dependencies
        run: go mod verify
      # Run the canonical per-target license audit and inspect the GoReleaser snapshot
      # as required before tagging; do not run go mod tidy as part of release.
      - uses: actions/create-github-app-token@v3
        id: tap-token
        with:
          client-id: ${{ secrets.TAP_APP_ID }}
          private-key: ${{ secrets.TAP_APP_PRIVATE_KEY }}
          owner: ${{ vars.TAP_OWNER }}
          repositories: ${{ vars.TAP_REPO }}
          permission-contents: write
      - uses: goreleaser/goreleaser-action@v7
        with:
          distribution: goreleaser
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          HOMEBREW_TAP_GITHUB_TOKEN: ${{ steps.tap-token.outputs.token }}
          MACOS_SIGN_P12: ${{ secrets.MACOS_SIGN_P12 }}
          MACOS_SIGN_PASSWORD: ${{ secrets.MACOS_SIGN_PASSWORD }}
          MACOS_NOTARY_ISSUER_ID: ${{ secrets.MACOS_NOTARY_ISSUER_ID }}
          MACOS_NOTARY_KEY_ID: ${{ secrets.MACOS_NOTARY_KEY_ID }}
          MACOS_NOTARY_KEY: ${{ secrets.MACOS_NOTARY_KEY }}
```

The sample App token action needs an app actually installed for the one named tap repository; its IDs and Apple credentials are project-owned secrets, not universal names or credentials. A different public tap, org, signing process, or action-version policy must be configured to match its owner and reviewed separately. Never skip macOS signature/notarization verification just because an optional workflow step did not run; when distributing signed macOS artifacts, confirm the authorized holder, successful service result, and notary log. Apple documents [Developer ID signing and notarization](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution).

### 2. GoReleaser configuration

Keep `.goreleaser.yaml` in the tool repository. Its `builds[].main` **must be byte-for-byte the same package path** as `ENTRYPOINT` in the audit commands, and `builds[].binary` must be the executable in archives and `homebrew_casks[].binaries`. Set project/release owner and name from the tool's own public GitHub repository. The following is an illustrative GoReleaser v2 configuration for the four supported targets; adapt paths, archive files, homepage, descriptions, repository and approved public tap to the project. GoReleaser's Go builder `tool: go` selects the executable on `PATH` installed by setup-go from `go.mod`; use readonly module mode and do not run `go mod tidy` during release.
Before the tag-triggered workflow runs, publicly publish and verify durable, exact-version source URLs for every required source offer. Include recipient-facing source-access information in the tool's source documentation before tagging so binary recipients can find it; credentials used only to publish releases or taps must not be needed for source retrieval.
```yaml
version: 2
project_name: example-tool

builds:
  - id: example-tool
    main: ./cmd/example-cmd
    binary: example-tool
    tool: go
    env:
      - CGO_ENABLED=0
      - GOFLAGS=-mod=readonly
    goos: [darwin, linux]
    goarch: [amd64, arm64]
    ldflags:
      - -s -w -X main.version={{ .Version }}

archives:
  - formats: [tar.gz]
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"
    files:
      - README.md
      - LICENSE
      - THIRD-PARTY-NOTICES
      - LICENSES/**
      - PROVENANCE.txt

release:
  github:
    owner: example-org
    name: example-tool

checksum:
  name_template: checksums.txt

homebrew_casks:
  - name: example-tool
    binaries: [example-tool]
    homepage: https://example.com/example-tool
    description: Example command-line application
    repository:
      owner: example-org # replace with TAP_OWNER
      name: homebrew-tools # replace with TAP_REPO
      branch: main
      token: "{{ .Env.HOMEBREW_TAP_GITHUB_TOKEN }}"
    directory: Casks
    url:
      template: "https://github.com/example-org/example-tool/releases/download/{{ .Tag }}/{{ .ArtifactName }}"
    commit_msg_template: "feat: {{ .ProjectName }} {{ .Tag }}"
```

In addition to this archive list, generate and verify a versioned provenance record containing the source commit/tag, actual compiler (`go version`, `go env` evidence), dependency/license/notice inventory, exact notice sources and Go runtime/vendor notices. Ensure `LICENSE`, `LICENSES/`, any complete notices and required source-form offers are present in **each of all four archives**; missing wildcard matches are not evidence that required text was included. If a target requires cgo or its native dependencies, do not claim `CGO_ENABLED=0` builds are viable: first establish the real, audited platform-specific build and reproduce it in the manifest.

To sign/notarize darwin output with the actual owner's authorized Apple Developer ID identity, add GoReleaser v2 `notarize.macos` for this build ID, populated only by the corresponding CI secrets; do not hard-code a holder name. GoReleaser documents the [Go build settings](https://goreleaser.com/customization/builds/builders/go/) and [Homebrew cask fields](https://goreleaser.com/customization/publish/homebrew_casks/). Never strip `com.apple.quarantine` or otherwise bypass Gatekeeper as a fallback for unsigned, unnotarized public artifacts. Provide working signatures/notarization, or make any platform-distribution decision with the project owner explicit; report local macOS assessment separately from CI/API/notary-job status.

Before tagging, verify the config and generate a no-publish snapshot using [GoReleaser's documented commands](https://goreleaser.com/getting-started/quick-start/):

```sh
goreleaser check
goreleaser release --snapshot --clean --skip=publish
```

Snapshot output should include `dist/homebrew/Casks/example-tool.rb` and target archives/checksums. It is evidence of local generation only, not a published release, successful tap write, valid macOS execution, signed identity or notarization.

### 3. Tag and publish

Only after reviewing the full audit, snapshot, source diff and authorization, create a new, immutable semantic-version tag on the exact approved source commit. Confirm no remote tag or public release already uses it. The example variables above define `TAG` as `v${VERSION}`.

```sh
git tag -a "$TAG" -m "Release $TAG"
git push origin "$TAG"
```

The GitHub Actions workflow now publishes the public release and its assets; GoReleaser updates the cask in the public tap. Monitor the workflow and release/tap API result as the publisher. If any asset, signature, checksum, or cask is wrong, do **not** move the tag, overwrite immutable release assets, or patch generated casks manually. Correct the source/configuration, create a new version/tag, and release a fresh set of assets through the same process. A tap commit does not establish that all release-archive checks passed.

### 4. Anonymous download, target and install verification

Verify downloads as an anonymous consumer with no `gh` session, token, private URL strategy, or authorization headers. Run this from a tool-project shell after replacing the placeholder repository, tag, and archive template with the actual generated names; use the filenames published on that release page:

```sh
mkdir -p "$DOWNLOAD_DIR"
for target in darwin_amd64 darwin_arm64 linux_amd64 linux_arm64; do
  archive="${PROJECT}_${VERSION}_${target}.tar.gz" # confirm exact release asset filename
  url="https://github.com/$OWNER/$REPO/releases/download/$TAG/$archive"
  curl --fail --location --silent --show-error "$url" -o "$DOWNLOAD_DIR/$archive"
  shasum -a 256 "$DOWNLOAD_DIR/$archive"
  tar -tzf "$DOWNLOAD_DIR/$archive" | sort
done
curl --fail --location --silent --show-error \
  "https://github.com/$OWNER/$REPO/releases/download/$TAG/checksums.txt" \
  -o "$DOWNLOAD_DIR/checksums.txt"
(cd "$DOWNLOAD_DIR" && shasum -a 256 -c checksums.txt)
```
Compare each computed hash with the release's published `checksums.txt` and the matching generated cask stanza, not with values inferred from filenames. For all four archives inspect every included binary, README, complete applicable tool/dependency/Go/runtime-vendor licenses and notices, source offer, and versioned provenance; examine binary `go version -m` build info and target against the release runner's exact compiler evidence. Inspect actual executable help and runtime prerequisites on a matching native host only: an ARM Mac does not run a Linux archive as a Linux consumer test, and a Linux runner cannot execute an extracted Mac binary. Record all four target archives as artifact evidence separately from native execution evidence.

On macOS, the installing consumer may use the generated public tap and inspect its downloaded native binary:
```sh
brew tap "$TAP_OWNER/$TAP"
brew install --cask "$TAP_OWNER/$TAP/$CASK"
"$TOOL" --help
command -v "$TOOL"
codesign -dv --verbose=2 "$(command -v "$TOOL")" 2>&1
spctl --assess --type execute -vv "$(command -v "$TOOL")"
```

Substitute the actual tap name, cask token, and CLI's supported help option. Installation/help checks do not prove every target, anonymous release-asset access, signing certificate identity, notarization service success, tap API permission, or server-side job status. The publisher separately retains the CI run, release and tap API/status logs, actual signing identity and notary job/log result, checksums, archive text/build-info evidence, and anonymized download result. Report facts separately; do not assert successful signing/notarization on an unsigned or unverified artifact.

### 5. README and release evidence

For a new cask, update the tool-list table in the public tap's `README.md`, including the public project URL, binary/cask token and accurate short description. Retain the source-hash/tag, dependency graph, scanner output/review, compiler environment, archive listing/hash/text review, `go version -m` output per target, native tests actually executed, public download and cask verification, CI/API results, and the authorized signature/notarization evidence with the release record. Future dependency/compiler/build-target/archive/release-identity changes require a new audit and snapshot against the changed source: do not reuse the previous source hash or provenance for a newer version.

## Publisher-specific troubleshooting

- **Invalid private key** when minting the tap token: correct the secret from the owner's real private-key file using the approved secret manager; never paste or put it in the repository.
- **Permission denied** on tap commit: verify the app is installed on the named tap and its token scopes only that repository with `contents: write`.
- **Secret absent**: verify the owning repository was granted access to the named Actions secret; don't make optional signing silently appear successful.
- **Anonymous download fails**: ensure the source repository/release and assets are public and remove private-download/custom-token cask strategies. A GitHub App's ability to write the tap has no effect on consumer access.
- **GoReleaser-generated cask uses deprecated Homebrew syntax**: use the current GoReleaser and its documented Homebrew cask configuration to correct the template in the tool repository, then issue a new release. Do not manually patch the generated cask or use warnings as a reason to erase security attributes.

## Optional deployment profile: this tap

This tap uses organization-managed GitHub Actions secrets:

| Secret | Used for |
| ------ | -------- |
| `HOMEBREW_RELEASER_APP_ID` | GitHub App allowed to write to this tap only |
| `HOMEBREW_RELEASER_APP_PRIVATE_KEY` | private key of that app |
| `MACOS_SIGN_P12` | authorized *Developer ID Application* certificate (base64 `.p12`) |
| `MACOS_SIGN_PASSWORD` | password of that certificate |
| `MACOS_NOTARY_ISSUER_ID`, `MACOS_NOTARY_KEY_ID`, `MACOS_NOTARY_KEY` | App Store Connect API key for this publisher's notarization |

These are deployment-profile details, not playbook requirements: use your own repository scope, secrets manager, application IDs and authorized signing identity. Never copy or expose credential values.
