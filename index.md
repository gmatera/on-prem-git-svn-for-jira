# On-Prem Git & SVN for Jira Cloud

![On-Prem Git & SVN logo](assets/logo-144.png)

Commits, branches and merge requests from your **self-managed GitLab** and revisions and branches from your
**Subversion (SVN)** servers, in the Development panel of Jira Cloud issues.

- **Nothing to open in your firewall.** Your servers send the events to Jira; Jira never connects to your
  network.
- **Runs on Atlassian.** No servers of ours, no calls outside Atlassian: the data stays in Atlassian's cloud,
  with the data residency of your Jira site.
- **Nothing lost silently.** Events are kept until Jira confirms them; anything that could not be applied,
  and any push or revision that never arrived, is shown on the app page with what to do.
- **Five minutes per server.** One system hook for the whole GitLab instance; one command installs the SVN
  hook on every repository of a server (Linux or Windows, VisualSVN Server included).

## Documentation

- [User guide](user-guide.md): setup, how work is linked to issues, troubleshooting, data and privacy.
- [Support](support.md)
- [Privacy policy](privacy-policy.md), [Data Processing Addendum](dpa.md), [Terms](terms.md),
  [Security](security.md)
