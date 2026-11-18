# `trustbom license`

## Synopsis

```text
trustbom license
trustbom license activate [--github-owner <owner>]
trustbom license refresh [--github-owner <owner>]
trustbom license request-trial --email <email> [--github-owner <owner>]
trustbom license status [--github-owner <owner>] [--json]
```

## Purpose

`trustbom license` prints the license terms for the installed version. It works
offline, is never restricted, and points to the terms at
<https://aitrustbom.com/legal/terms/> and the privacy policy at
<https://aitrustbom.com/legal/privacy/>. The subcommands manage the license on
this computer. For the trial, purchase, CI and outage rules see the
[licensing guide](../licensing.md).

## Options

| Subcommand and option | Meaning |
| --- | --- |
| `activate [--github-owner <owner>]` | Exchange a license key (prompted, hidden, or piped on standard input; never pass it as an argument) for a signed, verified lease bound to the resolved GitHub owner. Stores both for the current user at `%USERPROFILE%\.trustbom\license.json` (Windows) or `${XDG_CONFIG_HOME:-~/.config}/trustbom/license.json` (elsewhere). |
| `refresh [--github-owner <owner>]` | Renew the license on this computer now, regardless of remaining lease time. |
| `request-trial --email <email> [--github-owner <owner>]` | Request a free 30-day trial key by email. `--email` is required. The service emails the key (never returned); the 30-day trial starts when the key is issued. |
| `status [--github-owner <owner>] [--json]` | Report the cached license state offline, without contacting the license service. `--json` prints machine-readable JSON. |

`--github-owner` names the GitHub owner to bind or evaluate when it cannot be
resolved from the `origin` remote or `GITHUB_REPOSITORY_OWNER`.

Management commands:

- `request-trial`: passing `--email` is the consent: the command prints the
  privacy notice, then sends the email, owner and client version to the service,
  which emails the key and never returns it. A key is emailed only if the owner
  is eligible for a trial, so if no email arrives check your spam folder, then
  contact support or buy a subscription (the command prints both). Activate the
  key when it arrives. If `--github-owner` differs from the owner resolved from
  `GITHUB_REPOSITORY_OWNER` or the `origin` remote, the command warns that the
  option was ignored.
- `status` reports the cached license offline: owner, whether it is a trial or a
  subscription, the lease end (the signed copy stored on this computer, renewed
  automatically; not the subscription end), the trial or subscription end when
  it is known, and whether a renewal is due. Text and `--json` report the same
  facts; the text prints `License: trial` or `License: subscription`, while
  `--json` keeps `kind` with the values `trial` and `paid`. Dates are shown as
  `YYYY-MM-DD (UTC)`. A subscription that outlasts the lease has no end date in
  the lease (`entitlementExpiresAtUtc` is `null`); status says it renews
  automatically. A trial lease cannot be refreshed past the trial, so for a
  trial status prints `Trial ends: YYYY-MM-DD (UTC). A trial does not renew.`
  (the lease end when the lease carries no entitlement end) and never mentions
  renewal or a purchase receipt. When a trial or subscription has ended, status
  and paid commands use the same wording and point to the buy page and
  `trustbom license activate`.

Management commands report failures as a plain message, without `license-*`
identifiers (including `license status --json`). Only `activate`, `refresh`,
`request-trial`, the paid-command key prompt and a paid command's lease request
use the network.

Release builds embed the license service URL and the verification keys and never
trust a runtime service URL, key or clock override. A build without them reports
`license-invalid` from a paid command and "Licensing is not configured" from the
management commands.

## License enforcement

Five commands are paid and check the license before doing any work, so a
failed check writes and renders nothing: approved [`export`](export.md) (not
`export --draft`), [`publish`](publish.md) (including `--draft` and `--from`),
[`pack`](pack.md) (including `--from`), [`answer`](answer.md) (every mode,
including workbook reuse and `--suggest-only`) and [`drift`](drift.md). Every
other command is free and never checks a license, prompts or contacts the
license service: `init`, `scan`, `gate` (single-system or portfolio),
`readiness`, `request-approval`, `export --draft` and every `license`
subcommand. A fresh clone with no license can run that whole free journey.
Argument errors (exit `2`) are reported before the check and never prompt.

**Exit code `4` (`CliExitCodes.EntitlementFailed`)** means a paid command's
entitlement check failed. The command writes `license-<id>: <message>` to
standard error (standard output stays empty) and one of these diagnostics:
`license-missing`, `license-owner-mismatch`, `license-invalid`,
`license-expired`, `license-revoked` or `license-service-unreachable`. Paid
commands have no `--json` output. A valid result with a warning (the service
is unreachable but a cached lease is still valid) runs the command and prints
the warning on standard error.

**No key yet:** an interactive paid command (standard input not redirected,
not CI) prompts "Enter your license key, or press Enter to request a free
30-day trial". For a trial it shows a privacy notice, asks for confirmation,
your email and GitHub owner (default from the repository origin), and the
emailed key, then continues. A non-interactive run never prompts and fails
with `license-missing` and instructions. In CI set `TRUSTBOM_LICENSE_KEY`: it
takes priority over any stored key, is not persisted and requests a fresh
lease on every run. A valid cached lease with at least 7 days left runs
offline; a lease with fewer days left is refreshed first.

The GitHub owner is resolved from `GITHUB_REPOSITORY_OWNER` in GitHub
Actions, then the `origin` remote; paid commands have no owner option, and an
unresolved owner reports `license-owner-mismatch`.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. `license status` also exits `0` when the lease is valid, including when a renewal is due. |
| `1` | Parser or usage error, including a missing required option such as `license request-trial --email`. |
| `2` | Operational failure of a management command. |
| `4` | `license status` found the cached state not usable (no lease, a lease for another GitHub owner, a lease that did not verify, a suspended license, or a trial, subscription or lease that has ended). Paid commands exit `4` on any entitlement failure. |

## Licence

Free. `license` and every `license` subcommand are never paywalled and never
check a license.

## Example

```powershell
trustbom license status
```

From a development build, which embeds no license service (real output):

```text
License status could not be determined: Licensing is not configured for this build: the license service URL and lease signing keys are embedded at build time and cannot be supplied at runtime.
```

Exit code `2`. In a release build with an activated trial, the text report
starts `License: trial` and ends with `Trial ends: YYYY-MM-DD (UTC). A trial does not renew.`;
exit code `0`.

## Common errors

- **`license-missing`** from a paid command in a non-interactive run: activate a
  key with `trustbom license activate`, or set `TRUSTBOM_LICENSE_KEY` in CI.
- **`license-owner-mismatch`**: the lease is bound to a different GitHub owner
  from the one resolved from `GITHUB_REPOSITORY_OWNER` or `origin`.
- **No trial email arrives**: check your spam folder, then contact support or
  buy a subscription.
- **`license status` exits `4`**: the cached state is not usable; activate a
  key.

## See also

- [Licensing and entitlement](../licensing.md), including the diagnostics table
  and CI setup
- [`export`](export.md), [`publish`](publish.md), [`pack`](pack.md),
  [`answer`](answer.md), [`drift`](drift.md)
