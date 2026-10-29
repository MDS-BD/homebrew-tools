# homebrew-tools

Homebrew tap for MDS-BD internal tools. Casks are published automatically by each project's GoReleaser release:
do not edit them by hand. To publish a new tool, see [PUBLISHING.md](PUBLISHING.md).

## Licensing

This tap's own material is licensed under the MIT License in [LICENSE](LICENSE). Each application project must include its own complete third-party license and notice bundle in every release archive. See [PUBLISHING.md](PUBLISHING.md#rerunning-the-license-audit) for the canonical audit rerun and [PUBLISHING.md](PUBLISHING.md#license-and-release-archive-requirements) for release requirements.

## Setup

Both this tap and the tools' GitHub release assets are public, so Homebrew can install them without a GitHub token or git credentials. Add the tap with its public URL:

```sh
brew tap MDS-BD/tools
```

## Tools

| Cask | Description |
| ---- | ----------- |
| [`aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) | SSM tunnels to private AWS services (OpenSearch, ...) without bastions or plugins |
| [`ecr-mirror`](https://github.com/MDS-BD/ecr-mirror) | Mirror container images between ECR registries reachable through different AWS profiles |

Install and upgrade with each cask's full name, `mds-bd/tools/<cask>`:

```sh
brew install --cask mds-bd/tools/aws-tunnel
brew upgrade --cask mds-bd/tools/aws-tunnel
brew install --cask mds-bd/tools/ecr-mirror
brew upgrade --cask mds-bd/tools/ecr-mirror
```

Homebrew 7 prints ``Calling `postflight` is deprecated!`` on install: GoReleaser still generates the old stanza
([goreleaser#6870](https://github.com/goreleaser/goreleaser/issues/6870)). It is harmless, and fixed on the next
release once GoReleaser ships the change; do not edit the cask here.
