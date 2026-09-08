# Security policy

## Reporting a vulnerability

Do not open a public issue for a suspected vulnerability. Use
[GitHub private vulnerability reporting](https://github.com/IggyGG/cargoless/security/advisories/new)
so maintainers can investigate before details are disclosed. If GitHub private
reporting is unavailable, email `security@triform.ai` with a concise impact
summary and reproduction steps. Never include live credentials, customer data,
or other secrets in a report.

We will acknowledge a complete report within three business days and coordinate
remediation and disclosure with the reporter.

## Scope

Reports about command execution boundaries, repository or CI credentials,
artifact provenance, release signing, merge-lane integrity, and filesystem
isolation are especially valuable. Ordinary product bugs and feature requests
belong in the public issue tracker.

## Supported versions

Cargoless is pre-1.0. Security fixes are provided on the latest release and on
`main`; older snapshots may be asked to upgrade before receiving a backport.
