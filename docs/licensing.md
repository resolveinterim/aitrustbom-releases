# Licensing and entitlement

AI TrustBOM has a free tier and a paid tier. Five commands are paid; every
other command is free and never checks a license. The offer is a 30-day free
trial with no card, then a paid subscription per GitHub owner (an organization
or user account), sold through Paddle (the merchant of record: the
seller on your invoice, which handles tax).

Release availability is published on the GitHub Releases page and the npm registry.

## Free and paid commands

| Free: never checks a license, never prompts | Paid: active trial or subscription required |
|---|---|
| `init` in all modes; `scan` and `gate`, single-system or portfolio, any number of systems | Approved `export` (without `--draft`) |
| `readiness`, `request-approval` | `publish`, including `--draft` and `--from` |
| `export --draft` (watermarked, never approved) | `pack`, including `--from` |
| `license` and every `license` subcommand | `answer` in every mode, including workbook reuse and `--suggest-only`; `drift` |

A fresh clone with no license, key or network can run the whole free journey:
`init`, `scan`, `gate`, `readiness` and `export --draft`. Free commands keep
their ordinary `0`/`2`/`3` exit codes and never print a `license-*`
diagnostic, even with an expired trial, a revoked key or a corrupt cache. A
free approval never grants a license or turns draft export into
approved export.

A paid command checks the license **before doing any work**, so a failed check
writes and renders nothing. Argument errors (exit `2`) are reported first and
never prompt.

## Trial, purchase and keys

- **Trial:** 30 days from when the trial key is issued; no card. One trial
  per GitHub owner. The service emails the key to the requester (that is the
  email verification) and never returns it in a response.
- **Paid:** access runs to Paddle's paid-through date plus 7 days. A renewal
  payment moves it forward; cancelling or a failed payment needs no action.
  On the first payment the service emails a **new paid key**; the trial key
  simply expires. Replace the key once, locally (`license activate`) and in the `TRUSTBOM_LICENSE_KEY` CI secret.
- **Lease:** a signed, owner-bound lease valid until the earlier of the
  entitlement end and 30 days. The CLI asks for a new one only when fewer than
  7 days remain, so a paid command with a valid cached lease runs fully
  offline.
- **Lost or leaked key, organization rename, corrections:** contact support;
  support reissues, revokes, rebinds or extends the license. There is no
  self-service portal.

### First paid command: the key prompt

The first time you run a paid command with no key or lease, an interactive
run shows:

```text
Enter your license key, or press Enter to request a free 30-day trial:
```

Paste a key to activate it, or press Enter to request a trial. A trial request
shows a short privacy notice (it sends your email address, the GitHub owner
and the client version to the license service, which emails you a key), asks
for confirmation, the email address and the GitHub owner (default taken from
the repository origin), and then asks you to paste the emailed key. The
command then continues. The CLI keeps no first-use timestamp or local trial
state, so deleting local files does not extend a trial.

The prompt appears only when standard input is not redirected and the run is
not CI. Otherwise the paid command fails with `license-missing` and
instructions; it never prompts.

### Management commands

```powershell
trustbom license request-trial --email <email> [--github-owner <owner>]
trustbom license activate [--github-owner <owner>]
trustbom license refresh [--github-owner <owner>]
trustbom license status [--json] [--github-owner <owner>]
```

- `request-trial` asks the service to email a trial key (a scripted
  alternative to the prompt). It prints the privacy notice, replies
  generically ("if eligible, a key has been emailed") and exits `0` when the
  request was made.
- `activate` reads the key from standard input rather than an argument, so it
  stays out of shell history; avoid key literals in scripts. It exchanges the
  key for a signed lease and stores both for the current user.
- `refresh` forces a lease refresh immediately.
- `status` reports the cached lease (owner, trial or subscription, expiry and whether a refresh
  is due) entirely offline.
- `trustbom license` (no subcommand) prints the embedded license terms and the
  online link, works offline and is never restricted.

Management commands are never paywalled and report failures without
`license-*` identifiers (also in JSON). Operational failures exit `2`. `status`
exits `4` (the paid-command entitlement code) when there is no lease, the lease
is for another owner, did not verify, is revoked or has ended, so a script can
tell those from an operational failure; a usable lease, including one due for
renewal, exits `0`.

```powershell
$licenseKey | trustbom license activate
trustbom license status --json
```

`$licenseKey` stands for a securely obtained key, not a literal to put in a
script. The lease and key are stored at:

- Windows: `%USERPROFILE%\.trustbom\license.json`
- Elsewhere: `${XDG_CONFIG_HOME:-~/.config}/trustbom/license.json`

Only the license key and the latest lease are stored. Repository contents are
never sent: a lease request sends the key and the client version; a trial
request sends the email, GitHub owner and client version.

### Owner binding

Every lease is bound to a GitHub owner, compared in lower case. The owner is
resolved, in order: `GITHUB_REPOSITORY_OWNER` (GitHub Actions), the `origin`
remote's GitHub owner segment, then the `--github-owner` option of the
management commands. Paid commands have no owner option: they use the Actions
owner or the origin remote, and fail with `license-owner-mismatch` when the
owner cannot be resolved or differs from the lease. The message then says to
run from a repository owned by the lease owner, get a license for the
repository's owner, or contact support. A `--github-owner` value that differs
from the owner resolved from Actions or `origin` is ignored, with a warning.

### Refresh and outage behavior

A cached lease is used as is until fewer than 7 days remain; then one refresh
is attempted. An expired cached lease triggers one lease request before
`license-expired` is reported. The five-minute clock-skew allowance applies.

If the service is unreachable once a refresh is due, a developer machine keeps
using the cached lease and the paid command prints a warning on standard
error, but never past the lease's own signed expiry. A definitive refusal
(revoked or invalid) is not an outage and blocks even while the cached lease
is unexpired. Offline revocation is therefore not instantaneous: a cached
lease may remain usable until its signed expiry, at most 30 days.

### Continuous integration

Set `TRUSTBOM_LICENSE_KEY` instead of running `activate`. When set it takes
priority over any stored key, the lease is not written to the user-scoped
store, and a fresh lease is requested on every run, so a paid step fails with
`license-service-unreachable` while the service is unavailable. CI never
prompts: with no key a paid command fails with `license-missing` and
instructions. Free commands need no secret, and fork pull requests never
receive the vendor's key.

#### Create the `license-key` secret

1. In the vendor repository (or organization) open **Settings → Secrets and
   variables → Actions** and create a secret named `TRUSTBOM_LICENSE_KEY`
   holding your license key. Never put the key in a workflow file, an input
   default or a log; only secrets are passed to the CLI, always as the
   `TRUSTBOM_LICENSE_KEY` environment variable and never as an argument.
2. `trustbom init --growth` generates a caller workflow that already passes it
   to the Growth reusable workflow (`secrets: license-key: ${{
   secrets.TRUSTBOM_LICENSE_KEY }}`). Only the Growth job that runs `pack`,
   `answer` and `publish` receives the key, and that job runs on pushes to the
   default branch only. The standard workflow and the free scan/gate Action
   run no paid command and need no secret.
3. For your own paid step (`export`, `drift` and so on), set the variable on
   that step after installing the CLI:

   ```yaml
   - run: trustbom export --fail-on high
     env:
       TRUSTBOM_LICENSE_KEY: ${{ secrets.TRUSTBOM_LICENSE_KEY }}
   ```

If the secret is missing or empty the step fails with the usual exit `4`
`license-missing` diagnostic. Pull requests from forks never receive secrets
and never run the paid steps, so the free `scan`/`gate` review is unaffected.
The owner is read from `GITHUB_REPOSITORY_OWNER` on a GitHub runner, so the
key must belong to the account that owns the repository.

### Diagnostics and exit codes

A paid command whose check fails writes `license-<id>: <message>` to standard
error, writes nothing to standard output and exits `4`. The six diagnostics:

| Diagnostic | Meaning |
|---|---|
| `license-missing` | No key or lease (a non-interactive run prints how to get one) |
| `license-owner-mismatch` | The lease is for another owner, or the owner cannot be resolved |
| `license-invalid` | Tampered or wrongly signed lease, unknown key, or a build with no embedded licensing configuration |
| `license-expired` | The trial or subscription has ended (the message says how to buy) |
| `license-revoked` | Support revoked the license |
| `license-service-unreachable` | A lease was needed and the service could not be reached |

A valid result with a warning (service outage with a usable cached lease) runs
the command and prints the warning on standard error. Exit `4` is reserved for
entitlement failures (paid commands, and `license status` with no usable lease);
other exit codes are unchanged (see the
[CLI reference](cli-reference.md)).

## Multiple engineers

One GitHub-owner license covers all the vendor's engineers and CI, with no
seat or machine cap. A vendor administrator receives the key (by email, for a
trial or on first purchase), keeps it in the company's password manager or
secrets system and shares it securely. Each engineer activates the same key
with `license activate` and stores a separate lease in their own
profile. CI uses the same key as an organization or repository secret exposed
only to paid steps. The GitHub owner has a single trial; later engineers do
not reset it.

Offboarding removes secret access. A shared key cannot be revoked for one
engineer; contact support to reissue it, then distribute the replacement.
Issued leases may remain usable until their signed expiry (at most 30 days on
a developer machine); CI is affected immediately. The key is a bearer
credential and the owner is a client assertion (a locally editable Git origin
is not proof of membership), so secret custody and rotation matter. Contact
support if a trial was used by someone else under your GitHub owner name or needs a reset.
