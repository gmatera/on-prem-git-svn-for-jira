---
title: "Subversion (SVN) in Jira Cloud: replacing FishEye"
description: "FishEye support ends in May 2028. How to keep Subversion revisions and branches linked to Jira Cloud issues with a commit hook, without opening your firewall."
---

For years, many teams saw their Subversion revisions in Jira through **FishEye**. Atlassian is retiring
FishEye and Crucible: existing licenses can be renewed, and are supported, until **15 May 2028**
([Atlassian](https://www.atlassian.com/licensing/crucible)). At the same time more and more Jira sites are
moving to the cloud. If your code is still in SVN, you need a new way to keep revisions linked to issues.
This guide compares the options and shows how to connect SVN to Jira Cloud with a commit hook.

## What you are actually replacing

FishEye did two different jobs. It is worth separating them, because the replacements are different:

| FishEye feature | Replacement |
|---|---|
| Revisions and branches shown in Jira issues | A Jira integration that fills the issue's **Development panel** |
| Browsing and searching code, revision history, diffs | The web interface of your SVN server (VisualSVN Server, ViewVC, WebSVN) or a code search tool |
| Code reviews (Crucible) | A review tool that supports SVN, or a move to Git and merge requests |

Atlassian's recommended path is aimed at Git users (Bitbucket). For SVN you have two realistic choices.

## Option 1: migrate from SVN to Git

The long-term option, if your teams are ready for it. It is a project of its own: converting history,
branches and tags, retraining people, rewriting build scripts. It usually takes months for large
repositories, and some teams keep SVN for good reasons (large binary files, path-based permissions, simple
linear workflows).

## Option 2: keep SVN and connect it to Jira Cloud

Subversion has no webhooks, but it has **hooks**: scripts that the server runs after every commit. A
`post-commit` hook can send each revision to Jira over HTTPS. This has three advantages:

- **Nothing to open in your firewall.** The SVN server calls out; Jira never connects to it.
- **Works with any SVN server:** VisualSVN Server on Windows, Apache httpd with mod_dav_svn, svnserve.
- **No server to run.** No indexing service like FishEye: the hook sends each revision as it happens.

What to check in any hook-based solution:

- **Commits must never wait for Jira.** The hook should send in the background, so a slow network never
  slows down a commit.
- **Nothing should be lost.** If Jira or the network is unavailable, revisions must be sent later, in order.
  SVN revision numbers are consecutive, so a good integration can detect a missing one and tell you.
- **Existing hooks must be preserved.** Many servers already use `post-commit` for e-mails or CI triggers.
- **History import.** You probably want the revisions of the last months in Jira on day one.

## Setup with On-Prem Git & SVN for Jira Cloud (about 5 minutes per server)

1. Install the app on your Jira Cloud site and open its page (*Settings › Apps*), tab **Subversion (SVN)**.
2. Save the configuration shown there as `connector.conf` on the SVN server, and set `REPOSITORY_URL` to the
   web address of your repositories, for example `https://svn.example.com/svn/{repo}`. With VisualSVN
   Server you can also set `REVISION_URL` so that links in Jira open the revision in VisualSVN's web
   interface: this covers FishEye's "click through to the revision".
3. Save the script in the same folder (`connector-hook.sh` on Linux, `connector-hook.ps1` on Windows) and
   install it on every repository with one command:
   - Linux: `sh connector-hook.sh install /path/to/repositories/*`
   - Windows (PowerShell as administrator):
     `powershell -ExecutionPolicy Bypass -File connector-hook.ps1 install C:\Repositories\*`

   The script adds itself to the `post-commit` and `post-revprop-change` hooks. If a repository already
   has one of these hooks, it does not change it and prints the line to add.
4. Import recent history, for example revisions 1500 to 2000:
   `sh connector-hook.sh resend /path/to/repository 1500 2000`.

## How revisions are linked to issues

| Where you write the issue key | Example | What appears in the issue |
|---|---|---|
| Log message | `PROJ-123 Fix rounding in invoices` | revision `r1234` with its message and author |
| Branch name (standard layout) | `branches/proj-123-invoices` or `project/branches/proj-123-invoices` | the branch and its latest revision |

Corrected log messages (`svn propset --revprop svn:log`) are updated in Jira, and deleting a branch removes
it from the issue.

## Checklist

- [ ] List your SVN servers and repositories, and who administers each server
- [ ] Outbound HTTPS from the SVN servers to Atlassian allowed (or a proxy configured)
- [ ] Standard layout (`trunk`, `branches`, `tags`), also inside project folders
- [ ] Existing `post-commit` hooks identified
- [ ] Web interface for code browsing chosen, to replace FishEye's browsing
- [ ] Range of history to import decided
- [ ] Team convention for issue keys in log messages and branch names

---

*This guide is written by the developers of **On-Prem Git & SVN for Jira Cloud**, a Runs on Atlassian app
that brings Subversion and self-managed GitLab into the Jira Development panel without opening your
firewall. Free for up to 10 users. [Learn more](https://steadyseam.com/on-prem-git-svn-for-jira/) · [User guide](../user-guide.md)*
