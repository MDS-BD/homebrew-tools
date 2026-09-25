# homebrew-tools

Homebrew tap for MDS-BD internal tools. Casks are published automatically by each project's GoReleaser release:
do not edit them by hand.

## Setup

This repository and the release assets are private, so Homebrew needs a GitHub token able to read them
(fine-grained, *Contents: read* on `MDS-BD/homebrew-tools` and each tool repo, or a classic token with `repo`):

```sh
export HOMEBREW_GITHUB_API_TOKEN=<token>          # add it to your shell profile
brew tap MDS-BD/tools git@github.com:MDS-BD/homebrew-tools.git
```

## Tools

| Cask | Description |
| ---- | ----------- |
| [`opensearch-proxy`](https://github.com/MDS-BD/opensearch-proxy) | SSM-tunnelled, SigV4-signing proxy to a private AWS OpenSearch domain |

```sh
brew install --cask opensearch-proxy
brew upgrade --cask opensearch-proxy
```
