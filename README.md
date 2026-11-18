# AI TrustBOM releases

**AI TrustBOM** is a git-native tool that keeps a cited, human-approved record of the AI in your software. It scans your repository for supported evidence of AI providers, models, frameworks and tools, adds your team's explicit declarations, and lets a designated reviewer approve the exact record through a normal Git pull request. Pull-request checks then flag relevant changes against the approved record.

The approved record is a single source of evidence you can reuse for:

- **Buyer reviews and questionnaires:** send a cited evidence pack and answer AI and security questions.
- **Attestation and assurance:** give auditors, compliance officers and internal Security or Procurement teams a versioned, reviewable snapshot.

It runs on your machine or in your own GitHub workflow; no hosted repository scanning is required. AI TrustBOM is a product of [Resolve Interim Ltd](https://resolveinterim.com).

## What this repository is for

This repository hosts:

1. **Downloadable releases** of the `trustbom` CLI (installers and checksum-verified archives for Windows, Linux and macOS).
2. **Developer documentation, tutorials and guides** in [`docs/`](docs/README.md).

It does not contain source code.

## Download and install

Get the [latest release](https://github.com/resolveinterim/aitrustbom-releases/releases/latest), or browse [all releases](https://github.com/resolveinterim/aitrustbom-releases/releases).

If you already have Node.js and npm:

```sh
npm install --global trustbom
```

Otherwise use the standalone installer for your platform. See the [installation guide](docs/installing.md) for offline and mirrored installs, checksums, updating and first-run warnings, or the [quick start](https://aitrustbom.com/install/).

## Documentation

- [Guides index](docs/README.md): everything below, in reading order.
- [How you use AI TrustBOM](docs/workflow.md): the end-to-end workflow.
- [Tutorial](docs/tutorial/README.md): a hands-on first journey with a starter repository.
- [CLI reference](docs/cli/README.md): one page per command.
- [GitHub Action](docs/github-action.md): the generated pull-request workflow.
- [Licensing](docs/licensing.md): free tier, 30-day trial and subscription.

## More information

- Product site and pricing: <https://aitrustbom.com>
- Support: <support@aitrustbom.com>
- License: see [LICENSE.txt](LICENSE.txt). AI TrustBOM is proprietary software, not open source.
