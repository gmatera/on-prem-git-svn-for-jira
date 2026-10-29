---
title: "Connect self-managed GitLab to Jira Cloud without opening your firewall"
description: "Options to show commits, branches and merge requests from a GitLab instance behind a firewall in the Jira Cloud Development panel, what each requires from your network, and a 5-minute push-based setup."
---

Your company runs Jira Cloud, but GitLab runs on your own servers, behind a firewall. You want each Jira
issue to show its commits, branches and merge requests in the **Development panel**, and your security team
does not want GitLab reachable from the internet. This guide explains the options, what each one requires
from your network, and how to set up a push-based integration in about five minutes.

## Why this is harder than it looks

There are two ways to get development information into Jira Cloud:

- **Pull:** something in the cloud connects to your GitLab to read repositories or to complete the setup.
  GitLab must accept connections from the internet.
- **Push:** GitLab sends events (push, merge request) to Jira over HTTPS. GitLab only needs **outbound**
  HTTPS, the same kind of traffic it already uses, for example, for GitLab's version check.

For a GitLab instance behind a firewall, push is the architecture that does not require a change to your
inbound rules.

## Your options

| Option | Development panel | What your network must allow | Notes |
|---|---|---|---|
| GitLab for Jira Cloud app (official, by GitLab) | Yes | GitLab publicly available, or an internet-facing reverse proxy that accepts connections from GitLab.com | Also requires HTTPS with a publicly trusted certificate ([GitLab docs](https://docs.gitlab.com/administration/settings/jira_cloud_app/)) |
| GitLab's built-in Jira issues integration | No: it adds comments with links to the issue | Outbound from GitLab to Jira | Needs a Jira user's API token (or a service account) stored in GitLab ([GitLab docs](https://docs.gitlab.com/integration/jira/configure/)) |
| Push-based Marketplace apps (such as ours) | Yes | Outbound HTTPS from GitLab to Atlassian only | Configured with one GitLab system hook |

According to GitLab's own documentation, the Development panel for Jira Cloud is filled by the GitLab for
Jira Cloud app ([GitLab docs](https://docs.gitlab.com/integration/jira/development_panel/)); the Jira DVCS
connector is for Jira Data Center and Server only.

## Questions your security team will ask

- **Does anything need to reach our network?** With a push integration, no: GitLab calls Atlassian, never
  the other way round.
- **Where does the data go?** Ask the vendor whether the app has its own servers. Apps in Atlassian's *Runs
  on Atlassian* program use only Atlassian's infrastructure, so the data stays in Atlassian's cloud, with the
  data residency of your Jira site.
- **What data leaves GitLab?** Commit messages, branch names, merge request titles, author names and e-mails,
  repository names and URLs. A well-designed integration never sends source code, diffs or file contents.
- **How are requests authenticated?** With a secret token that GitLab sends with every event; you should be
  able to regenerate it at any time.

## Setup with On-Prem Git & SVN for Jira Cloud (about 5 minutes)

1. Install the app on your Jira Cloud site and open its page (*Settings › Apps*). The **GitLab** tab shows a
   URL and a secret token.
2. In GitLab, go to **Admin area › System hooks › Add new webhook**. Paste the URL and the secret token,
   select **Push events**, **Merge request events** and **Repository update events**, keep **Enable SSL
   verification** selected and save. One system hook covers every project of the instance.
3. In **Admin area › Settings › Network › Performance optimization**, set **Push event hooks limit** to
   `100`. With the default value (3), GitLab sends no details for a push that updates more than three
   branches or tags at once.
4. Click **Test** next to the hook. The app page turns green within a few seconds.
5. Allow outbound HTTPS from GitLab to the host shown on the app page, if your firewall filters outbound
   traffic.

Then write the issue key in your work: in the commit message (`PROJ-123 Fix login`), in the branch name
(`feature/PROJ-123-login`) or in the merge request title.

## Limits to know before you choose

These apply to every integration based on GitLab's system hooks, not only ours:

- **20 commits per push.** GitLab includes only the latest 20 commits in a push event. The branch is
  still updated to its latest commit, but older commits of a very large push do not appear individually.
- **History before the setup.** A push integration sees what happens after it is connected: older commits
  are not imported, and a branch appears with its latest commit when it receives new work.
- **The push event hooks limit** described in step 3: if it is low, pushes that update many branches arrive
  without details. Our app detects this and lists it under *Needs attention*.

## Checklist

- [ ] GitLab 17.11 or later (CE or EE)
- [ ] Outbound HTTPS from GitLab to Atlassian allowed
- [ ] One system hook with push, merge request and repository update events
- [ ] Push event hooks limit set to 100
- [ ] Team convention for issue keys in commits, branches and merge request titles
- [ ] Security review: data sent, data residency, authentication, token rotation

---

*This guide is written by the developers of **On-Prem Git & SVN for Jira Cloud**, a Runs on Atlassian app
that brings self-managed GitLab and Subversion into the Jira Development panel without opening your
firewall. Free for up to 10 users. [Learn more](https://steadyseam.com/on-prem-git-svn-for-jira/) · [User guide](../user-guide.md)*
