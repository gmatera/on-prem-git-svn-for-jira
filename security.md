# Security

**Reporting a vulnerability.** Write to **sassimaterasas@gmail.com** with the details; please do not disclose it publicly
before we have answered. We acknowledge reports within two business days and keep you informed until
the issue is fixed.

**How the app is built**

- It runs entirely on Atlassian Forge (Runs on Atlassian): no servers of ours, and no calls to any host
  outside Atlassian. Data is kept in Forge hosted storage and in your Jira site, with your site's data
  residency.
- Your GitLab and Subversion servers call one HTTPS web trigger. Every request must carry the secret
  token of your installation, compared in constant time; requests without it are rejected and nothing is
  stored. Administrators can regenerate the token at any time.
- The app page is available only to Jira administrators, and every operation of the page is checked on
  the server.
- The app never receives source code, diffs or file contents. See the [privacy policy](privacy-policy.md)
  for the data it stores and for how long.
- The Subversion hook script sends the token to curl on standard input (never on the command line), keeps
  its configuration readable only by the server account on Linux, and never changes existing hooks.
- Every change is covered by automated tests (unit, contract tests with real GitLab and Subversion
  payloads, chaos and mutation testing) before release.
