# Authorization and Multi-Tenancy

## Authorization models

| Model | Mechanism | Breaks when |
|---|---|---|
| **RBAC** | Users -> roles -> permissions | Combinatorial explosion (50 functions x 20 locations = 10K roles) |
| **ABAC** | Policies evaluate attributes on subject/resource/environment | Hard to audit, PIP is SPOF, can't model "manager sees team's files" |
| **ReBAC (Zanzibar)** | Permissions from relationship graph. Tuple: `doc:X#editor@user:Bob` | Cold-start complexity, consistency tokens needed |
| **PBAC** | Externalized policies in dedicated language (Rego, Cedar, CEL) | Learning curve for policy language |

**Recommended hybrid**: RBAC for defaults, ReBAC for resources, ABAC for guardrails.

## Authorization tools (2026)

| Tool | Model | Language | Best for |
|---|---|---|---|
| **OPA** | ABAC/PBAC | Rego | K8s admission, infra policies, general-purpose |
| **Cerbos** | RBAC+ABAC | YAML + CEL | App/API authz, low learning curve |
| **SpiceDB** | ReBAC (Zanzibar) | Schema + tuples | Full Zanzibar consistency, hierarchical sharing |
| **OpenFGA** | ReBAC (Zanzibar) | DSL + tuples | CNCF Incubating, VS Code extension |
| **Cedar** | RBAC+ABAC | Cedar language | Formal verification, AWS Verified Permissions |
| **CASL** | RBAC+ABAC (JS) | TypeScript | Isomorphic (server + client), 6KB, Prisma integration |

### When to pick which

| Scenario | Tool |
|---|---|
| Kubernetes admission control | OPA |
| API/app authz, low learning curve | Cerbos |
| Google Drive-like hierarchical sharing | SpiceDB or OpenFGA |
| AWS-native with formal verification | Cedar |
| TypeScript monorepo, client+server shared | CASL |

## PostgreSQL Row-Level Security (RLS)

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON orders
  USING ((select auth.uid()) = user_id);
```

**Critical pattern**: Use `(select auth.uid())` (subquery) not bare `auth.uid()` — subquery evaluates once and caches vs re-evaluating per row.

**`SET LOCAL` not `SET`**: `SET LOCAL` is transaction-scoped (safe with connection pooling). `SET` contaminates pooled connections across tenants.

RLS as security floor: use BOTH RLS AND application-level checks. RLS provides defense-in-depth.

## Multi-tenancy isolation models

| Model | Isolation | Cost | Complexity |
|---|---|---|---|
| **Shared schema, tenant_id column** | Low (app-enforced) | Lowest | Lowest |
| **Schema-per-tenant** | Medium (DB-enforced) | Medium | Medium |
| **Database-per-tenant** | Highest | Highest | Highest |
| **Shared + RLS** | Medium-High (DB-enforced) | Low | Medium |

**2026 default**: Shared schema + RLS + tenant_id column. Graduate to schema-per-tenant for compliance.

## Cross-tenant attack prevention

- Return 404 (not 403) for resources outside tenant — prevents enumeration
- Use UUIDs, not sequential IDs
- RLS policies on ALL tables with tenant data
- Audit logging for cross-tenant access attempts
- Separate encryption keys per tenant for sensitive data

## Data residency

GDPR, PIPL, and similar regulations may require data to reside in specific regions. Options:
- Region-specific database instances with routing
- Database sharding by region
- Encryption at rest with tenant-scoped keys
