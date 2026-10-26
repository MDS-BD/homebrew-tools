# homebrew-tools

Homebrew tap for MDS-BD internal tools. Casks are published automatically by each project's GoReleaser release:
do not edit them by hand. To publish a new tool, see [PUBLISHING.md](PUBLISHING.md).

## Licensing

This tap's own material is licensed under the MIT License in [LICENSE](LICENSE). The copied GoReleaser private-release download strategy in the generated cask has its scoped attribution and MIT notice in [THIRD_PARTY_NOTICES](THIRD_PARTY_NOTICES). Application binaries are downloaded from separate project releases; each application project must include its own complete third-party license and notice bundle in every release archive. See [PUBLISHING.md](PUBLISHING.md#rerunning-the-license-audit) for the canonical audit rerun and [PUBLISHING.md](PUBLISHING.md#license-and-release-archive-requirements) for release requirements.

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
| [`aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) | SSM tunnels to private AWS services (OpenSearch, ...) without bastions or plugins |

Install with the full name, `mds-bd/tools/<cask>`, so a public cask with the same name is never picked. Add the tap
first: without it Homebrew clones this private repo over HTTPS, which fails unless git has GitHub credentials.

```sh
brew install --cask mds-bd/tools/aws-tunnel
brew upgrade --cask mds-bd/tools/aws-tunnel
```

Homebrew 7 prints ``Calling `postflight` is deprecated!`` on install: GoReleaser still generates the old stanza
([goreleaser#6870](https://github.com/goreleaser/goreleaser/issues/6870)). It is harmless, and fixed on the next
release once GoReleaser ships the change; do not edit the cask here.
