# Security Policy

Security is a first-class concern in Eidolon, not an afterthought — the product accumulates an
intimate model of its users (personality, episodic memory, reality data: steps/sleep/HRV). That
"you-graph" is both the moat and the single most sensitive asset, so protecting it is a product
requirement, not just an operational one. See the full
[Security & Threat Model](./docs/SECURITY_THREAT_MODEL.md).

## Reporting a vulnerability

**Please do not open a public issue for security vulnerabilities.**

- Report privately via [GitHub Security Advisories](https://github.com/UTA0619/eidolon-portfolio/security/advisories/new),
  or email the maintainer (see the GitHub profile).
- Include: a description, reproduction steps, affected component, and potential impact.
- You'll get an acknowledgement within **72 hours** and a remediation timeline after triage.

Please act in good faith: give us reasonable time to fix issues before disclosure, and avoid
accessing or modifying other users' data.

## Scope

| In scope | Out of scope |
|---|---|
| Auth / session handling | Findings requiring a rooted/jailbroken device with physical access |
| The AI gateway (auth, rate-limit, cost guard, prompt-injection) | Denial-of-wallet *theory* without a working PoC |
| Data access boundaries (RLS / you-graph isolation) | Social engineering of maintainers |
| Client secret handling | Third-party provider outages |

## Security posture (summary)

- **No AI provider keys or billing on the client.** All model calls route through server-side
  Supabase Edge Functions behind auth, rate-limiting, and a hard cost guard
  ([ADR-003](./docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md)).
- **Data isolation** enforced at the database layer (RLS), with `user_id` included in queries as a
  second line of defense.
- **Governance as code** — user-ownership and deletion guarantees are encoded as CI-enforced
  invariants ([ADR-004](./docs/DECISIONS/ADR-004-governance-as-code.md)), so "real deletion means
  real deletion" is testable, not just promised.
- **Secrets** are never committed; configuration is injected at build/deploy time.
