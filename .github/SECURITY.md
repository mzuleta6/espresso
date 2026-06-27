# Security Policy

## Supported versions

| Version | Supported |
|---|---|
| 1.1.0 | ✅ |
| < 1.1.0 | ❌ |

Only the latest release receives security fixes.

## Scope

The primary security concern for this plugin is **SKILL.md content integrity**. A malicious modification to `SKILL.md` would silently affect the behavior of Claude for all users who have it installed with `always: true`.

In scope:
- Malicious content in `SKILL.md` (prompt injection, safety bypass)
- Broken install paths that cause users to load unexpected skill files
- Supply chain attacks via CI/CD workflow modifications

Out of scope:
- GitHub account security of individual users
- Claude Code application vulnerabilities (report to Anthropic)

## Reporting a vulnerability

**Do not open a public issue for security vulnerabilities.** Public issues expose the vulnerability before it is fixed.

Use GitHub's private vulnerability reporting:
1. Go to the Security tab of this repository
2. Click "Report a vulnerability"
3. Describe the issue with steps to reproduce

Expected response time: within 72 hours.

## Disclosure policy

Once a fix is released, vulnerabilities will be disclosed publicly via a GitHub Security Advisory with full details.