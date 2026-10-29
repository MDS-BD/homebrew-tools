# homebrew-tools

Public Homebrew tap for command-line tools released from their own GitHub Actions. Release assets are anonymous downloads; GoReleaser publishes the casks, so do not edit them by hand. Follow the [company-agnostic public package publication playbook](PUBLISHING.md) before adding or releasing a tool.

## Licensing

This tap's own material is licensed under the MIT License in [LICENSE](LICENSE). Each application project must include its own complete third-party license and notice bundle in every release archive. See [PUBLISHING.md](PUBLISHING.md#rerunning-the-license-audit) for the canonical audit rerun and [PUBLISHING.md](PUBLISHING.md#license-and-release-archive-requirements) for release requirements.

## Setup
The tap and every application's release archive must be publicly readable. Homebrew consumers install without GitHub credentials; only publishers need separately scoped credentials for tag release and tap writes. Add the public tap:

```sh
brew tap MDS-BD/tools  # Homebrew tap name; the GitHub repository is homebrew-tools
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
Homebrew 7 currently reports this tap's GoReleaser-generated `postflight` deprecation ([goreleaser#6870](https://github.com/goreleaser/goreleaser/issues/6870)). It is tap-specific behavior, not a general release step. Do not edit generated casks by hand; update the source GoReleaser configuration using current documentation instead.
