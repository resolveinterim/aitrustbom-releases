# Install options

The quick install is on the [install page](https://aitrustbom.com/install/):
use npm if you already have Node.js, or a standalone installer for Windows,
Linux or macOS. This page covers what it does not: checking an install,
offline and mirrored installs, checksums, updating, uninstalling and the
first-run warnings for unsigned binaries.

You need Git. Node.js and npm are needed only
for the npm route and for the GitHub Action that `trustbom init` writes, which
installs the pinned `trustbom` npm package on the runner.

## Check the install

```sh
trustbom --version
```

This prints the build identity without a banner, for example
`1.0.0+build-0042`. Running `trustbom` with no arguments, or asking for help,
prints one identity line before the normal output; add `--no-logo` (before or
after the command) to suppress it in automation. Generated artifacts never
include the line. The `tbom` command is a short alias that runs `trustbom`.

`trustbom license` prints the license terms for the installed version. It works
offline and needs no licence.

## Offline and mirrored installs

Download the matching archive, `SHA256SUMS` and the installer script from the
same release, copy them to the target machine and install from the directory.
No network access is needed:

```sh
sh install.sh --release-dir ./release --install-dir "$HOME/.local/bin"
```

```powershell
.\install.ps1 -ReleaseDir .\release -InstallDir "$HOME/.local/bin"
```

To serve a release internally, copy its files to an HTTPS location you control
and pass that location as the base URL:

```sh
sh install.sh --base-url https://mirror.example.com/trustbom/v1.0.0
```

`--base-url` (`-BaseUrl` on Windows) downloads the matching archive and
`SHA256SUMS`; `--release-dir` (`-ReleaseDir`) reads both from disk. There is
no default URL, so always pass one of them.

For npm, set `npm_config_registry` to your mirror's registry URL in the
invoking environment, and mirror the exact `trustbom` version **and its
matching platform package** (`@resolveinterim/trustbom-<rid>`, for example
`@resolveinterim/trustbom-linux-x64`). The GitHub Action runs `npm install`
with `--ignore-scripts` and honours the same variable. Set it in the workflow
or job environment, provide registry credentials there, and never commit
credentials. The Action's pinned version does not advance when you update the
CLI.

## Checksums and the installer scripts

Both installers verify the archive against `SHA256SUMS` **before** installing
`trustbom` and `tbom`. A matching checksum protects against accidental
corruption, not against a malicious source that replaces both the archive and
the checksum file, so take releases from a source you trust.

`SHA256SUMS` covers the archives only, not the installer scripts: a checksum
served from the same release would add no trust. Download each script over
HTTPS and read it before running it.

## Platforms

The supported platforms are `win-x64`, `linux-x64`, `linux-arm64`, `osx-arm64`
and `osx-x64`. The shell installer supports Linux and macOS; the PowerShell
installer supports Windows. Both install into `$HOME/.local/bin` by default;
add it to your `PATH` if needed.

Linux needs the ICU runtime library (for example `libicu74` on Ubuntu 24.04,
`libicu` on Fedora or RHEL). The installer warns if it is missing, and without
it the binary stops at start-up. Alpine and musl are not supported.

## Update and uninstall

- **Script install:** rerun the installer with the new release's archive and
  `SHA256SUMS`. To uninstall, remove only `trustbom` and `tbom` (Linux and
  macOS) or `trustbom.exe` and `tbom.cmd` (Windows) from the install
  directory, and leave the other files there.
- **npm:** `npm update --global trustbom` and
  `npm uninstall --global trustbom`.

Homebrew, winget and other package-manager feeds are not currently available.

## Unsigned binaries (first-run warnings)

The Windows and macOS binaries are **not code-signed or notarized**. A binary
downloaded in a browser carries a quarantine marker, so:

- **Windows SmartScreen** may show "Windows protected your PC". Choose
  **More info**, then **Run anyway**, or unblock the file first with
  `Unblock-File .\trustbom.exe` in PowerShell.
- **macOS Gatekeeper** may say the binary "cannot be opened because the
  developer cannot be verified". Remove the marker with
  `xattr -d com.apple.quarantine ./trustbom`, or use **System Settings >
  Privacy & Security > Open Anyway**.

The npm, script-installer and Action routes download without a browser, so they
do not trigger these warnings.

## Next

Run [`trustbom init`](cli/init.md) in your repository, or follow the
[tutorial](tutorial/README.md). The [CLI reference](cli/README.md) lists every
command.
