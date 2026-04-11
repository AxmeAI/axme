# Security Policy

## Reporting a Vulnerability

Please do **not** open a public GitHub issue for security vulnerabilities.

Email: **[contact@axme.ai](mailto:contact@axme.ai)** with subject line `[SECURITY]`.

We will acknowledge receipt within 48 hours and provide a resolution timeline within 5 business days.

## What to Include

A useful report contains:

- **Steps to reproduce** - minimal, clear sequence to trigger the issue
- **Impact** - what can an attacker do? (read data, bypass auth, escalate privileges, etc.)
- **Affected component** - CLI version, SDK version, or cloud.axme.ai endpoint
- **Environment** - OS, language/runtime version if relevant
- **PoC** - proof-of-concept code or curl commands if you have them

The more detail, the faster we can confirm and fix.

## Scope

In scope:

- `axme-control-plane` - gateway, auth, intent routing, delivery
- `axme-sdk-*` - all SDK clients (Python, TypeScript, Go, Java, .NET)
- `axme-cli` - CLI authentication, credential storage
- `cloud.axme.ai` - API, dashboard, MCP server

Out of scope:

- Denial of service attacks (rate limiting, resource exhaustion)
- Social engineering of AXME staff
- Vulnerabilities in third-party dependencies (report to the dependency maintainer)
- Issues already publicly disclosed
- Scanner output without demonstrated exploitability

## Process

After you report:

1. **Acknowledge** - we confirm receipt within 48 hours
2. **Triage** - we assess severity and affected surface within 5 business days
3. **Fix** - we develop and test a patch, coordinating with you on timeline
4. **Release** - patch released, CVE assigned if applicable
5. **Disclosure** - public disclosure after patch is available, typically 90 days from report

We follow coordinated disclosure. Please give us reasonable time before going public.

## Supported Versions

AXME is in Alpha. Only the latest released version of each component receives security fixes.

| Component | Supported |
|---|---|
| axme-cli (latest) | Yes |
| axme-sdk-* (latest) | Yes |
| Older versions | No |

## Recognition

We maintain a list of security researchers who have responsibly disclosed issues to us. If you report a valid vulnerability, we will thank you publicly (with your permission) in the release notes and on our security acknowledgements page.

## Alpha Notice

AXME is currently in Alpha. The security posture is actively hardened. If you find issues that are clearly alpha-stage roughness rather than exploitable vulnerabilities, a regular GitHub issue or email is fine.
