# Data Processing Addendum: On-Prem Git & SVN for Jira Cloud

*Last updated: 29 October 2026.*

This Data Processing Addendum ("DPA") forms part of the agreement under which the customer ("Customer")
uses the Atlassian Marketplace app **On-Prem Git & SVN for Jira Cloud** (the "App") provided by **Sasso di
Matera Giuseppe S.a.s.** (Via Clefi 2, 20146 Milano (MI), Italy, VAT IT13043470965) ("Provider"). It applies to personal data
processed by the App on behalf of the Customer, as required by Article 28 of Regulation (EU) 2016/679
("GDPR") and by similar laws.

## 1. Subject matter, nature and purpose

The App receives events from the Customer's GitLab and Subversion servers and sends development information
(commits, branches, merge requests) to the Customer's Jira Cloud site. Processing consists of receiving,
storing for a limited time and transmitting that information to Jira, only to provide the App.

## 2. Personal data and data subjects

- **Data subjects:** people who commit, push or open merge requests in the Customer's repositories.
- **Personal data:** names and, for GitLab, e-mail addresses of commit authors; SVN user names; GitLab
  user names; any personal data the Customer's users write in commit messages, branch names or merge
  request titles.
- No special categories of personal data are intended to be processed.

## 3. Duration

For as long as the App is installed, plus the retention periods in the [Privacy Policy](privacy-policy.md).
On uninstallation, Jira deletes the development information sent by the App and the Forge platform deletes
the App's storage after 28 days.

## 4. Provider obligations

The Provider:
1. processes personal data only on the documented instructions of the Customer, given by installing,
   configuring and using the App;
2. ensures that anyone authorized to process the data is bound by confidentiality;
3. implements appropriate technical and organizational measures (Article 32 GDPR). The App runs only on
   Atlassian's Forge platform ("Runs on Atlassian"), makes no calls outside Atlassian, limits its settings
   to Jira administrators and authenticates the Customer's servers with a secret token;
4. engages Atlassian (Forge hosting) as the only sub-processor, which the Customer authorizes, and will
   inform the Customer of any intended change, giving it the opportunity to object;
5. assists the Customer, as far as possible, in responding to requests from data subjects and in meeting
   its obligations under Articles 32 to 36 GDPR;
6. notifies the Customer without undue delay after becoming aware of a personal data breach affecting the
   App;
7. deletes the personal data at the end of the service as described in section 3;
8. makes available the information necessary to demonstrate compliance with this DPA and allows for audits
   in the form of written questionnaires answered within 30 days.

## 5. International transfers

Personal data stays in Atlassian's cloud, in the location given by the data residency of the Customer's
Jira site. The Provider does not transfer it elsewhere.

## 6. Contact

support@steadyseam.com
