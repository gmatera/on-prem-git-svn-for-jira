# On-Prem Git & SVN for Jira Cloud: user guide

See commits, branches and merge requests from your **self-managed GitLab**, and revisions and branches from
your **Subversion (SVN)** servers, in the Development panel of Jira issues. Your servers send the events to
Jira, so **nothing in your network has to be opened to the internet**, and the data stays in Atlassian's
cloud.

## Requirements

- Jira Cloud (Software or Service Management), with permission to install apps.
- **GitLab:** self-managed GitLab 17.11 or later, CE or EE (tested with 17.11, 18.11 and 19.5), and a GitLab
  administrator account.
- **Subversion:** any SVN server, such as VisualSVN Server, Apache httpd with mod_dav_svn, or svnserve
  (tested with Subversion 1.14 and 1.15). You need administrator access to the server.
  - Linux: `sh`, `curl`, `base64` and the Subversion command-line tools (`svn`, `svnlook`), which Linux SVN
    servers already have. Other Unix systems work if they have the same programs (tested on Linux).
  - Windows: Windows PowerShell 5.1, which every supported Windows Server has. VisualSVN Server includes
    the command-line tools.
- Your GitLab or SVN servers must be able to make outbound HTTPS calls to Atlassian's cloud. The exact
  host is shown on the connector page, for example `xxxx.webtrigger.atlassian.app`.

## Connect GitLab (about 5 minutes)

1. **Install** the app from the Atlassian Marketplace.
2. In Jira, open **Settings › Apps › the connector page**, or click **Configure** in *Manage apps*. Open the
   **GitLab** tab. It shows a **URL** and a **secret token**.
3. In GitLab go to **Admin area › System hooks › Add new webhook**:
   - paste the **URL** and the **Secret token**;
   - under **Trigger** select **Push events**, **Merge request events** and **Repository update events**;
   - keep **Enable SSL verification** selected and click **Add system hook**.
4. In GitLab go to **Admin area › Settings › Network › Performance optimization** and set
   **Push event hooks limit** to `100`. With the default value (3), GitLab sends no details for a push
   that updates more than 3 branches or tags at once.
5. Click **Test** next to the hook in GitLab. The connector page turns green within a few seconds.

## Connect Subversion (about 5 minutes per server)

Subversion has no webhooks, so a small script on the SVN server sends every revision to Jira. It never
slows down a commit: revisions are sent in the background, and revisions that cannot be sent (network,
Jira unavailable) are sent automatically after the next commit.

1. On the connector page open the **Subversion (SVN)** tab.
2. **Save the configuration.** Copy the text shown under step 1 into a file named `connector.conf` in a
   folder of your choice on the SVN server. It already contains the URL and the secret token. Replace the
   example value of `REPOSITORY_URL` with the web address of your repositories (`{repo}` is replaced by the
   repository folder name): it is used for the links shown in Jira, and the installation does not start
   until it is set. With VisualSVN Server you can also set `REVISION_URL` to
   `https://your-server/!/#{repo}/commit/r{rev}` so links open the revision in VisualSVN's web interface.
3. **Save the script in the same folder:** `connector-hook.sh` on Linux and Unix, `connector-hook.ps1` on
   Windows. Both are shown on the connector page.
4. **Install it on your repositories with one command.**
   - Linux (as root or as the owner of the repositories):
     `sh connector-hook.sh install /path/to/repositories/*`
   - Windows (PowerShell opened as administrator):
     `powershell -ExecutionPolicy Bypass -File connector-hook.ps1 install C:\Repositories\*`

   For each repository the script copies itself and `connector.conf` into the repository's `hooks`
   folder, creates the `post-commit` and `post-revprop-change` hooks, and sends the latest revision as a
   test. The connector page turns green. If a repository already has one of these hooks (for example
   to send e-mails), the script **does not change it** and prints the line to add to it. On Windows this
   includes hooks written as `.exe`, `.bat` or `.wsf` files, which Subversion runs instead of a `.cmd`.
   If the Subversion tools are outside the usual folders, the script writes their folder into `SVN_BIN`
   in the copied `connector.conf`, because hooks run without your PATH.

To import recent history, send a range of revisions again, for example the last 500:
`sh connector-hook.sh resend /path/to/repository 1500 2000`. The revisions are added to their issues;
branches that the connector has not seen yet appear in Jira at their next commit (so that a branch deleted
long ago is never shown again). New commits keep flowing during a long import.

## How work is linked to Jira issues

| Where you write the issue key | Example | What appears in the issue |
|---|---|---|
| Commit message (uppercase key) | `PROJ-123 Fix login` | the commit (GitLab) or the revision, shown as `r1234` (SVN) |
| Branch name (any case) | GitLab `feature/PROJ-123-login`; SVN `branches/proj-123-login` | the branch and its latest commit |
| Merge request title or source branch (GitLab) | `PROJ-123 Login page` | the merge request with its status (open, merged, declined) |

Deleting a branch (in GitLab, or with `svn delete` on `branches/<name>` in SVN) removes it from the
Development panel. In SVN, a corrected log message (`svn propset --revprop svn:log`) updates the message
in Jira.

**Subversion layout.** Branches are recognised with the standard layout: `trunk`, `branches/<name>` and
`tags/<name>`, also inside project folders (`project/trunk`, `project/branches/<name>`). Tags are not shown
(Jira does not display them).

## Good to know

- **More than 20 commits in one GitLab push.** GitLab includes only the latest 20 commits in the event, and
  the connector cannot fetch the others without reaching into your network. The branch is still updated to
  its latest commit. The connector page records the push as a warning.
- **GitLab pushes that update many branches.** If the *Push event hooks limit* is lower than the number of
  branches in a push, GitLab sends no details for that push. The connector detects this and shows it under
  **Needs attention**. The next push to each branch updates Jira.
- **SVN revisions that were never received.** Revision numbers are consecutive, so the connector notices a
  missing one (for example if the hook was removed for a while) and shows it under **Needs attention** with
  the command to send it again.
- **Very large SVN revisions** (tens of thousands of changed files) are sent without the list of files: the
  revision appears in Jira, but the branches it touched are not updated by that revision.
- **Merge request author.** GitLab events only say who performed each action, so the author is shown only
  when it is certain. SVN shows the user name of the revision's author.
- **Keys of issues that do not exist** (for example `UTF-8` written in a commit message) are ignored by
  Jira.
- **Renaming an SVN repository folder** makes it a new repository in Jira (the old one keeps its history).
  Changing http to https or the port in `REPOSITORY_URL` does not.
- **Correcting an SVN log message** updates the message in Jira and adds the new issue keys. Removing a key
  from the message does not unlink the revision from that issue.

## The connector page

| Status | Meaning | What to do |
|---|---|---|
| Waiting for the first event | Nothing received yet | Complete the setup: click **Test** in GitLab, or run the install command on the SVN server |
| Connected | Everything works | — |
| Delayed | Jira asked to slow down, or events are waiting longer than usual | Nothing: events are kept and applied automatically |
| No recent events | Nothing received for 7 days | GitLab: open the hook (Admin area › System hooks › Edit) and check **Recent events**. SVN: run the `status` command below |
| Blocked by Jira | Jira denies the app access | An organization admin should check *Atlassian Administration › Security › Data security policies* (app access rules) |

**Needs attention** lists anything that could not be applied, with the reason. Use **Retry** after
fixing the cause, or **Dismiss**.

## Troubleshooting GitLab deliveries

In GitLab, the hook's **Recent events** list shows every delivery and the response from Jira:

| Response | Meaning | What to do |
|---|---|---|
| 200 | Accepted (or already received) | — |
| 401 | Wrong or missing secret token | Copy the token again from the connector page |
| 400 | The event could not be read | The connector page shows the reason under *Last rejected request* |
| 402 | No active license for the app on this Jira site | Check the subscription in *Manage apps* |
| 503 | Temporarily unavailable | Click **Resend request** in *Recent events* |
| Connection error | GitLab cannot reach Atlassian | Allow outbound HTTPS to the host shown on the connector page |

## Troubleshooting Subversion deliveries

If the hook cannot work because of its configuration (for example `connector.conf` is incomplete or the
server account cannot write to the repository's `hooks/connector` folder), whoever commits sees a warning
that starts with `Jira connector:`. The commit itself always completes, and the waiting revisions are sent
once the problem is fixed.

On the SVN server, the `status` command shows the configuration check, the last delivered revision and the
most recent log lines:

- Linux: `sh /path/to/repository/hooks/connector-hook.sh status /path/to/repository`
- Windows: `powershell -ExecutionPolicy Bypass -File C:\Repositories\name\hooks\connector-hook.ps1 status C:\Repositories\name`

The full log is in `hooks/connector/connector.log` inside the repository.

| Log message | Meaning | What to do |
|---|---|---|
| `not delivered (HTTP 401: ...)` | Wrong secret token | Put the token from the connector page in `connector.conf` and run `install` again |
| `not delivered (HTTP 402: ...)` | No active license for the app on this Jira site | Check the subscription in *Manage apps* |
| `not delivered (Could not resolve host ...)`, `(Failed to connect ...)`, `(The remote name could not be resolved ...)` or another connection error | The server cannot reach Atlassian | Allow outbound HTTPS to the host shown on the connector page; behind a proxy set `PROXY` in `connector.conf`. On Red Hat-based systems with SELinux: `setsebool -P httpd_can_network_connect 1` |
| `rejected by the connector (HTTP 400)` | The revision could not be read, for example `REPOSITORY_URL` is not an http(s) address | The connector page shows the reason under *Last rejected request*; fix it, then use `resend` |
| `curl not found`, `svn not found` | A required program is missing | Install it; for the Subversion tools you can set `SVN_BIN` in `connector.conf` |
| `svnlook cannot read the repository (...)` | The `svnlook` found is missing or older than the server (for example `/usr/bin` instead of `/opt/...`) | Set `SVN_BIN` in the repository's `hooks/connector.conf` to the folder of the server's Subversion tools |
| `resend: rX-rY do not exist, skipped` | A resend asked for revisions above the latest one (for example after restoring the repository from a backup) | Nothing: the other revisions were sent |

Nothing is lost while a problem lasts: as soon as it is fixed, the next commit sends every revision that
was waiting, in order. If Windows policies only allow signed PowerShell scripts, ask your administrators
to allow this script, because Windows will not run it otherwise.

## Secret token

Generate a new token on the connector page if you think it has leaked. The old token stops working within a
minute. Right after, paste the new one into the system hook in GitLab and into `connector.conf` on your SVN
servers, then run the `install` command again so every repository gets the new configuration.

## Data, privacy and security

- **Where data is stored.** Only in Atlassian's cloud (Forge hosted storage and Jira), with the same data
  residency as your Jira site. The app has no servers of its own and makes no calls outside Atlassian.
- **What is stored.**
  - Repository paths, names and URLs, branch names, merge request titles and states.
  - Commit and revision messages, author names; for GitLab also author e-mails (Jira uses the e-mail to show
    the author).
  - The GitLab user name of whoever pushed.
- **What is never stored.** Source code, diffs, file contents and file names. Only the number of changed
  files is kept.
- **Retention.**
  - Events are kept only until they are applied to Jira.
  - Activity history is kept for 14 days.
  - Problem records are kept until you dismiss them or retry them.
  - The last applied state of each branch and merge request is kept for 90 days.
  - To detect missing pushes and revisions, the connector keeps for 2 days a marker for each one it
    expects, and for 365 days the number of the last revision received from each SVN repository.
  - On uninstall, Jira deletes the development data the app sent, and Forge deletes the app storage after
    28 days.
- **Access.** Only Jira administrators can use the connector page, and every request is checked on the
  server. GitLab and the SVN script are authenticated with the secret token, compared in constant time. On
  the SVN server the token is kept in `connector.conf`, readable only by the server account on Linux.

## Uninstalling

Uninstalling removes the development data from Jira issues. If you install the app again, GitLab and the SVN
servers have to be connected again, because the URL may change. To remove the connector from an SVN
repository, delete the connector lines from `hooks/post-commit` and `hooks/post-revprop-change` (on Windows
`post-commit.cmd` and `post-revprop-change.cmd`; or the files, if the installation created them) and the
`hooks/connector*` files. To stop it only for a while, remove the hook lines but keep `hooks/connector`:
after installing again, the revisions committed in the meantime are sent automatically.
