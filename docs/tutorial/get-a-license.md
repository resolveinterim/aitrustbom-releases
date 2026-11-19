# Get a license

**Goal:** activate a free trial (or paid) license so that you can run the paid
commands on [page 4](4-answer.md) and [page 5](5-publish.md).

**You will need:** the repository from [page 3](3-approve.md) and an email
address that can receive the key. Allow a few minutes for the email to arrive.

Approved `export`, `answer` and `publish` are paid commands. `init`, `scan`,
`gate`, `request-approval`, `readiness`, `verify`, `export --draft` and every
`license` subcommand are [free](../licensing.md#free-and-paid-commands), so
pages 1 to 3 need no license. The trial is 30 days, with no payment details
required, one per GitHub owner.

## Try it

1. **Check the owner.** The license is bound to the GitHub owner in the fetch
   URL of `origin` ([owner binding](../licensing.md#owner-binding)). If you
   forked the starter on page 1, that is your account. If you cloned the
   original repository instead, fork it, then point `origin` at your fork and
   keep the local push URL from page 1:

   **PowerShell / bash:**

   ```sh
   git remote set-url origin https://github.com/<your-owner>/abingdon-taxi-solutions.git
   git remote get-url --push origin
   ```

   The second command should still print the local bare repository path.
   If it prints the GitHub URL, run
   `git remote set-url --push origin ../abingdon-origin.git`.

2. **Request a trial** ([reference](../cli/license.md)):

   **PowerShell / bash:**

   ```sh
   trustbom license request-trial --email <you@example.com>
   ```

   The command prints a privacy notice, sends your email address, the GitHub
   owner and the client version to the license service, and replies generically
   (if eligible, a key has been emailed). The service emails the key and never
   prints it.

3. **Activate the key** when it arrives. The key is read from a hidden prompt
   (or standard input):

   **PowerShell / bash:**

   ```sh
   trustbom license activate
   trustbom license status
   ```

   For a trial, `status` reports `License: trial` and ends with
   `Trial ends: YYYY-MM-DD (UTC). A trial does not renew.`

> **In production.** For a paid subscription, buy through the link the
> commands print, then run `trustbom license activate` again with the paid key.
> Support contact, renewal and outage rules are in
> [licensing](../licensing.md#trial-purchase-and-keys). CI jobs read the key
> from `TRUSTBOM_LICENSE_KEY`; see [licensing](../licensing.md#continuous-integration).

## If a paid command refuses

Without a license a paid command writes nothing and exits `4`:

```text
license-missing: No license key found, and this run cannot prompt for one. Set TRUSTBOM_LICENSE_KEY (for example from a CI secret) or run 'trustbom license activate'. To request a free 30-day trial key, run 'trustbom license request-trial --email <email>'.
```

An interactive paid command with no license also offers the trial at a key
prompt. Other `license-*` diagnostics are explained in
[licensing](../licensing.md#diagnostics-and-exit-codes).

## Checkpoint

- `trustbom license status` reports an active trial or paid license for your
  GitHub owner.

Next: [4. Answer](4-answer.md).
