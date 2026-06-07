# ADR-003: No authentication

**Status**: accepted

## Context

Beacon is deployed inside a private network (Docker Compose on a single host, or a private Kubernetes namespace). The services it monitors are also internal. The expected callers are other internal services and developers with direct network access — not public clients.

Adding token-based auth would require: a secrets management strategy for API keys, a token issuance endpoint or out-of-band key distribution, and ongoing key rotation. For an internal observability tool with no sensitive data (health check results are not secrets), this operational cost is not justified.

## Decision

Beacon exposes an unauthenticated API. Network-level access control (firewall rules, private subnets, Kubernetes NetworkPolicy) is the responsibility of the deployment operator.

## Consequences

- **Easier**: no secrets to manage, no auth middleware to maintain, simpler integration for internal callers.
- **Harder**: if beacon is ever exposed beyond a private network — even accidentally — any caller can register, delete, or read all monitored services. The deployment documentation must make this constraint explicit.
- **Technical debt**: if the deployment context changes (multi-tenant, public-facing), authentication must be added before the exposure happens, not after. This decision should be revisited when deployment requirements change.
