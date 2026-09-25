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
| [`aws-tunnel`](https://github.com/MDS-BD/aws-tunnel) | SSM tunnels to private AWS services (OpenSearch, ...) without bastions or plugins |

```sh
brew install --cask aws-tunnel
brew upgrade --cask aws-tunnel
```
