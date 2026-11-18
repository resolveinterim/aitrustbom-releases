# Install the CLI

For CI/PR gating, use the Docker-free GitHub Action that `trustbom init`
writes into your repository; it installs the pinned `trustbom` npm package
at run time and needs Node.js, npm and Git, not Docker or the .NET SDK.

The primary route is the installer script for your platform: it installs a
self-contained binary and needs neither Node.js nor the .NET SDK. npm is an
alternative (see below).

The installers take an **explicit** source:

```sh
# Linux/macOS: offline directory containing the matching RID archive and SHA256SUMS
sh scripts/install.sh --release-dir ./artifacts/release --install-dir "$HOME/.local/bin"
# Once a release is published (v<version> is the release tag, for example v1.0.0):
sh scripts/install.sh --base-url "https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>"
```

```powershell
# Windows: offline directory containing the win-x64 archive and SHA256SUMS
.\scripts\install.ps1 -ReleaseDir .\artifacts\release -InstallDir "$HOME/.local/bin"
# Once a release is published:
.\scripts\install.ps1 -BaseUrl "https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>"
```

`--base-url`/`-BaseUrl` downloads both the matching RID archive and
`SHA256SUMS`; `--release-dir`/`-ReleaseDir` reads both from disk. There is no
default hosted URL: always pass the release URL (or an internal mirror of it)
or a directory. Release archives are hosted as GitHub Release assets in the
public `resolveinterim/aitrustbom-releases` repository, which holds only the
assets, a README and the license text.

Once a release exists, download the installers from that same release page:
`https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>/install.sh`
and `.../install.ps1`. The release's `SHA256SUMS` covers the archives only, not
the installer scripts: a checksum served from the same release would add no
trust. Download the script over HTTPS and read it before running it. The
supported RIDs are `win-x64`, `linux-x64`,
`linux-arm64`, `osx-arm64`, and `osx-x64`; the shell installer supports
Linux/macOS and the PowerShell installer supports Windows. Both verify the
archive checksum **before** installing `trustbom` and its `tbom` shim; a
matching checksum protects against accidental corruption, not a malicious
source that replaces both the archive and checksum file. The default
installation directory is `$HOME/.local/bin`; add it to `PATH` if needed.
Git is still required at run time. Linux requires ICU (for example `libicu74` on
Ubuntu 24.04, `libicu` on Fedora/RHEL); the installer warns if it is missing, and
without it the binary aborts at start-up. Alpine/musl is not
supported. To update a script install, rerun the installer with a new
matching archive and `SHA256SUMS` from a trusted release source. To uninstall
that install, remove only `trustbom` and `tbom` on Linux/macOS, or
`trustbom.exe` and `tbom.cmd` on Windows, from the chosen install directory;
do not remove other files there. For npm
use `npm update --global trustbom` / `npm uninstall --global trustbom`.
Homebrew, winget, and other package-manager feeds are not currently available.

## Alternative: npm

To run `trustbom` without installing it, or to install it
globally, use npm (this needs Node.js and npm, not the .NET SDK):

```
npx trustbom scan
npm install --global trustbom
```

The global install provides `trustbom` and `tbom`. To use an internal npm mirror for either command, set `npm_config_registry` to the
mirror's registry URL in the invoking environment; mirror the exact
`trustbom` version **and its matching platform package**
(`@resolveinterim/trustbom-<rid>`, for example `@resolveinterim/trustbom-linux-x64`). The generated and
root composite Actions run `npm install` with `--ignore-scripts` and honor
the same `npm_config_registry` environment variable on the install step.
Configure that variable in the caller workflow/job environment and provision
registry credentials there as needed; never commit credentials. The Action's
pinned version is not automatically advanced when the CLI is updated.

## Offline and mirrored installs

Copy the matching archive and `SHA256SUMS`
from the same release to the target machine and use `--release-dir` /
`-ReleaseDir`; no network access is needed. To serve archives internally, copy
the release's files to an HTTPS location and pass that location as the base
URL. npm and the Actions use an internal registry through
`npm_config_registry`, as described above.

## Unsigned binaries (first-run warnings)

The Windows and macOS binaries are
**not code-signed or notarized** at launch. A binary downloaded in a browser
carries a quarantine marker, so:

- **Windows SmartScreen** may show "Windows protected your PC". Choose
  **More info**, then **Run anyway**, or unblock the file first with
  `Unblock-File .\trustbom.exe` in PowerShell.
- **macOS Gatekeeper** may say the binary "cannot be opened because the
  developer cannot be verified". Remove the marker with
  `xattr -d com.apple.quarantine ./trustbom`, or use **System Settings >
  Privacy & Security > Open Anyway**.

The npm, script-installer and Action routes download without a browser, so they
do not trigger these warnings. Always verify the archive against `SHA256SUMS`
from the same release (the installers do this for you).

## License terms, version and banner

Run `trustbom license` at any time to read the license terms for the installed
version. It works offline, is never restricted, and also points to the
terms at <https://aitrustbom.com/legal/terms/> and the privacy policy at <https://aitrustbom.com/legal/privacy/>.

Running `trustbom` without arguments or requesting help (including command
help) prints one UTF-8 identity line before the normal output, for example:

```text
AI TrustBOM v1.0.0+build-0042 (x86_64-unknown-linux-gnu)
```

The product tagline, "AI disclosure evidence for your repo", appears in the
top-level help description rather than in the banner.

The version is the build's informational version and the platform describes
the running process. CI builds of `main` reserve increasing identifiers such
as `1.0.0+build-0042`. Local and other unreserved builds use a distinct
`1.0.0-dev.0` identity by default; for an identifiable local build, resolve a
numeric identity from the full Git commit history with
`./scripts/Resolve-Version.ps1` and pass it to `dotnet build` as
`-p:TrustBomBuildVersion=<resolved-version>`. The development number advances
with commits, not with repeated local builds of the same commit; incomplete
Git history is rejected rather than returning a misleading number.
`trustbom --version` prints that same build identity without a banner. Use
`trustbom --no-logo --help` or `trustbom scan --no-logo --help` to suppress
the line in automation. `--no-logo` is accepted before or after the command;
ordinary command output, generated artifacts, and `--version` never include
the line. Without arguments, the existing missing-command exit status
remains unchanged.

The npm packages and self-contained binaries carry the build identity. Standalone builds
without that argument use the unreserved identity; they never take a main CI
number. `trust/card.json` retains the stable three-part `cli_version` and no
build suffix enters approval hashes.

## Next

Run [`trustbom init`](cli/init.md) in your repository, or follow the
[tutorial](tutorial/README.md). The [CLI reference](cli/README.md) lists every
command.
