# homebrew-tools

Homebrew tap for MDS-BD internal tools. Casks are published automatically by each project's GoReleaser release:
do not edit them by hand. To publish a new tool, see [PUBLISHING.md](PUBLISHING.md).

## Setup

This repository and the release assets are private, so Homebrew needs a GitHub token able to read them. If you use
the GitHub CLI, its token works:

```sh
export HOMEBREW_GITHUB_API_TOKEN=$(gh auth token)   # add it to your shell profile
brew tap MDS-BD/tools git@github.com:MDS-BD/homebrew-tools.git
```

Otherwise create a fine-grained token with *Contents: read* on `MDS-BD/homebrew-tools` and each tool repo, or a
classic token with the `repo` scope.

## Tools

| Cask | Description |
| ---- | ----------- |
| [`aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) | SSM tunnels to private AWS services (OpenSearch, ...) without bastions or plugins — temporarily unavailable until the sanitized `v0.0.4` release republishes its cask |

Install with the full name, `mds-bd/tools/<cask>`, so a public cask with the same name is never picked. Add the tap
first: without it Homebrew clones this private repo over HTTPS, which fails unless git has GitHub credentials.

```sh
# Available again after the v0.0.4 release republishes the cask:
brew install --cask mds-bd/tools/aws-tunnel
brew upgrade --cask mds-bd/tools/aws-tunnel
```

Homebrew 7 prints ``Calling `postflight` is deprecated!`` on install: GoReleaser still generates the old stanza
([goreleaser#6870](https://github.com/goreleaser/goreleaser/issues/6870)). It is harmless, and fixed on the next
release once GoReleaser ships the change; do not edit the cask here.
