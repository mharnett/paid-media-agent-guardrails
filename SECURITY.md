# Security policy

This repo is text: rules, patterns and examples for agents that write to ad accounts. It has no server, no write gate and no credentials. I maintain it on my own. If a page here would lead someone into one of the problems below, please tell me privately. If the problem is in one of my servers, report it in that server's repo.

## Reporting a problem

Please keep the details out of public issues, pull requests and discussions.

- If this repo's **Security** tab shows a **Report a vulnerability** button, use it. That report is private.
- If there's no button, open an issue that asks for a private contact and says nothing else about the problem.

In the private report, the useful details are the page, the rule or example, the smallest steps that show the problem and what someone could do with it. Please use made-up account ids, and leave out real tokens and account data.

## In scope

- **Write-gate bypass.** A rule or example that, if followed, gets an agent around a write gate.
- **A write that skips PAUSED staging.** A rule or example that, if followed, creates something live that should start PAUSED.
- **Credential leakage.** A rule or example that, if followed, exposes a token, secret or key, or a real one that's in these pages or their history.

## What I assume

- **Your own machine and credentials.** Nothing here runs by itself. You apply these pages on a computer you control, with your own platform credentials, to accounts you control.
