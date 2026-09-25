# Publishing a tool to this tap

Tools are released by [GoReleaser](https://goreleaser.com) from their own repository's GitHub Actions: a `v*` tag
builds the binaries, publishes a GitHub release, signs and notarizes the macOS builds, and pushes a cask to
`Casks/` here. Nobody edits this repository by hand.

[`MDS-BD/aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) is the reference setup: copy its `.goreleaser.yaml` and
`.github/workflows/release.yml`, then change the names as below. The examples here are for Go; GoReleaser also
builds Rust, Zig, Bun and Deno projects, and prebuilt binaries.

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

`.github/workflows/release.yml` in the tool repository; nothing to change except the Go setup for other languages:

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
- **`notarize.macos`**: signs and notarizes the darwin binaries with the org Apple credentials, from the Linux
  runner. Keep it: unsigned binaries are blocked by Gatekeeper.
- **`release.github`**: `owner: MDS-BD`, `name: <your repo>`.
- **`homebrew_casks`**:
  - `name`, `binaries`, `homepage`, `description` for your tool;
  - `repository: {owner: MDS-BD, name: homebrew-tools, branch: main, token: "{{ .Env.HOMEBREW_TAP_GITHUB_TOKEN }}"}`
    and `directory: Casks`;
  - `url.template` pointing at **your** repository's releases, with
    `using: GitHubPrivateRepositoryReleaseDownloadStrategy`;
  - the `custom_block` defining that download strategy, copied unchanged. Private repositories' release assets are
    not publicly downloadable: the strategy fetches them through the GitHub API with the user's
    `HOMEBREW_GITHUB_API_TOKEN`. Public tools can drop both `url.using` and `custom_block`.

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
