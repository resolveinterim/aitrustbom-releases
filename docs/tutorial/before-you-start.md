# Before you start

**Why:** the tutorial runs on your own copy of the starter repository, with
the real GitHub Action and real pull requests, so you need a GitHub account,
Git and the `trustbom` command. You do not need a license key until page 7.

## Try it

1. **Check what you need.**
   - A GitHub account. The fork you make is public, which the enforcement step
     on page 9 needs.
   - Git, with your name and email set (the tutorial makes commits):
     `git config --global user.name` and `git config --global user.email`
     should each print a value.
   - The [GitHub CLI](https://cli.github.com/) (`gh`), signed in with
     `gh auth login`. Pages 3, 4 and 9 use it to open pull requests and apply
     the enforcement rule. You can open pull requests in the browser instead,
     but page 9 needs `gh`.
   - A shell: PowerShell, or bash on Linux, macOS, Git Bash or WSL.

2. **Install `trustbom`.** With Node.js 18 or later, use npm. Without
   Node.js, use the
   installer script on the [install page](https://aitrustbom.com/install/);
   [install options](../installing.md) covers checksums and offline installs.

   **PowerShell / bash:**

   ```sh
   npm install --global trustbom
   trustbom --version
   ```

   The second command prints the installed version, for example `1.0.3`.

3. **Fork the starter.** Open
   [`resolveinterim/abingdon-taxi-solutions`](https://github.com/resolveinterim/abingdon-taxi-solutions)
   and select **Fork**. Keep the name `abingdon-taxi-solutions` and your
   account as the owner. The trial on page 7 is tied to the account that owns
   the repository, so the fork must be yours.

4. **Check that Actions can run in the fork.** GitHub can turn off workflows
   in a new fork. Open the fork's **Actions** tab. If it says workflows are not
   being run on this fork, select
   **I understand my workflows, go ahead and enable them**. If it shows no
   such message, Actions are already on. The starter has no workflow yet;
   page 2 adds it.

5. **Clone the fork** (replace `<your-owner>` with your GitHub account):

   **PowerShell / bash:**

   ```sh
   git clone https://github.com/<your-owner>/abingdon-taxi-solutions.git
   cd abingdon-taxi-solutions
   gh repo set-default <your-owner>/abingdon-taxi-solutions
   ```

   The last line tells `gh` to open pull requests in your fork, not in the
   original repository.

## What you now have

A clone of your own fork, with `trustbom` installed and nothing set up yet.
Read `service/experience.md`
for the company's story and
`rfi/request.csv`
for the questions a customer has sent.

Next: [1. See your AI](1-see-your-ai.md).
