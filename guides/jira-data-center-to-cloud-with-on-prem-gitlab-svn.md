---
title: "Moving Jira Data Center to Cloud while keeping GitLab and SVN on-premises"
description: "Jira Data Center reaches end of life in March 2029. A checklist to keep commits, branches and SVN revisions linked to issues when Jira moves to the cloud and the code stays on your servers."
---

Atlassian is retiring Data Center. New customers can no longer buy it since **30 March 2026**, existing
customers can no longer buy new licenses or expansions from **30 March 2028**, and Data Center reaches end
of life on **28 March 2029** ([Atlassian](https://www.atlassian.com/migration)). Many companies are moving
Jira to the cloud while their source code stays where it is: self-managed GitLab, Subversion, or both. This
guide lists what changes for the link between issues and code, and how to plan it so that developers lose
nothing on cutover day.

## What changes when Jira moves to the cloud

On Data Center, Jira often reached your code servers directly, on the same network: through the Jira DVCS
connector, application links to FishEye, or apps that index repositories. In the cloud, Jira is outside your
network. Three things follow:

1. **Integrations that need Jira to reach your servers stop working** unless you open your network. The Jira
   DVCS connector is for Data Center and Server; for Jira Cloud, GitLab's documentation points to the GitLab
   for Jira Cloud app ([GitLab docs](https://docs.gitlab.com/integration/jira/development_panel/)), which
   requires GitLab to be publicly available or behind an internet-facing reverse proxy
   ([GitLab docs](https://docs.gitlab.com/administration/settings/jira_cloud_app/)).
2. **FishEye is going away anyway:** support ends on 15 May 2028
   ([Atlassian](https://www.atlassian.com/licensing/crucible)).
3. **The Development panel is filled by integrations.** Plan to connect the new integration as part of the
   migration, and decide how much history you want to see in the cloud.

## Plan

### Before the migration

- **Inventory.** List the GitLab instances and SVN servers, the projects that use them, and how developers
  reference issues today (commit messages, branch names, merge request titles).
- **Keep the project keys.** Commit messages and branch names contain issue keys such as `PROJ-123`. If a
  project changes key in the cloud, existing references no longer match.
- **Choose an integration that fits your network.** If GitLab and SVN must not be reachable from the
  internet, choose a push-based integration: your servers send events to Jira over outbound HTTPS.
- **Security review.** Prepare the answers your security team will ask for: which data leaves your servers,
  where it is stored (data residency), whether the vendor runs its own servers, how requests are
  authenticated. Apps in Atlassian's *Runs on Atlassian* program store data only in Atlassian's cloud.
- **Check app access rules.** In *Atlassian Administration › Security › Data security policies*, app access
  rules can block apps from reading or writing data in some spaces or projects.
- **Try it on the cloud site** with one test project and one repository before the cutover.

### On cutover day

- Connect GitLab (one system hook for the whole instance) and the SVN servers (one hook per repository,
  installed with one command per server).
- For SVN, send the history you want to see in the cloud, for example the revisions of the last six months.
- Ask a few developers to push a commit that references an issue and check the Development panel.

### After the migration

- Remove the old integrations (DVCS connector accounts, application links to FishEye) when you
  decommission the Data Center instance.
- Agree on a short convention for issue keys and add it to the team's onboarding notes.
- Watch the integration's status page for a few days for anything that could not be applied.

## How On-Prem Git & SVN for Jira Cloud fits

- **Push-based:** GitLab (system hook) and SVN servers (commit hook) send events over outbound HTTPS. Nothing
  in your network has to be opened.
- **Runs on Atlassian:** no servers of ours; the data stays in Atlassian's cloud with your site's data
  residency. Privacy policy, data processing addendum and security page are public.
- **One app for GitLab and Subversion,** with the same issue-key conventions for both.
- **SVN history import** with one command; **missing pushes and revisions are detected** and listed with the
  command to send them again.
- **About five minutes per server.**
- **GitLab history:** events are received from the moment the system hook is connected; older commits are
  not imported.

## Checklist

- [ ] Inventory of GitLab instances, SVN servers and projects
- [ ] Same project keys in the cloud
- [ ] Integration chosen and security review completed
- [ ] App access rules checked
- [ ] Test on the cloud site with one project
- [ ] Outbound HTTPS from GitLab and SVN servers to Atlassian allowed
- [ ] Cutover steps: connect GitLab, connect SVN, import SVN history, verify
- [ ] Old integrations removed from Data Center

---

*This guide is written by the developers of **On-Prem Git & SVN for Jira Cloud**, a Runs on Atlassian app
that brings self-managed GitLab and Subversion into the Jira Development panel without opening your
firewall. Free for up to 10 users. [Learn more](https://steadyseam.com/on-prem-git-svn-for-jira/) · [User guide](../user-guide.md)*
