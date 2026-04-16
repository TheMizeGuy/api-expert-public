# Testing and Contract Verification

## Test pyramid for APIs

```
         /\
        /E2\      few
       / E  \
      /------\
     /CONTRACT\
    /----------\
   / INTEGRATION\
  /--------------\
 /     UNIT       \  many
/__________________\
```

Contract testing reshapes the pyramid — replaces slow integration tests with fast consumer-driven contracts.

## Consumer-driven contract testing (Pact)

Consumer defines expectations -> generates `.pact` JSON -> provider verifies it can meet those expectations.

| Property | Detail |
|---|---|
| Scope | Only parts of communication actually used by consumer(s) |
| Format | Example-based (concrete request/response pairs) |
| Independence | Services deploy independently without synchronized releases |

**Provider states**: `given "user exists"` clause names a state; provider maps to setup handler.

### Pactflow BDCT (Bi-Directional Contract Testing)

Both consumer and provider publish contracts independently. Pactflow cross-verifies compatibility without running provider tests. Lower barrier to entry than CDC.

## Spec-based testing

| Tool | Approach | Best for |
|---|---|---|
| **Schemathesis** | Property-based fuzzing from OpenAPI spec | Finding edge cases, mutation testing of API contracts |
| **Prism** | Mock server + validation proxy from OpenAPI | Dev: mock server; CI: validation proxy |
| **Dredd** | Execute spec examples against running server | Simple pass/fail contract checks |

## Load testing tools

| Tool | Language | Strength |
|---|---|---|
| **k6** | JavaScript (Go engine) | Developer-friendly, CI-native, cloud option |
| **Artillery** | YAML + JS | Simple config, good for quick profiles |
| **Locust** | Python | Distributed, real user simulation |
| **Gatling** | Scala/Java | Enterprise, detailed reports |

### k6 SLO-aligned thresholds

```javascript
export const options = {
  thresholds: {
    http_req_duration: ['p(95)<200', 'p(99)<500'],
    http_req_failed: ['rate<0.001'],
  },
};
```

## Security testing

| Tool | Type | Use |
|---|---|---|
| **OWASP ZAP** | DAST (Dynamic) | Automated scan against running API |
| **Nuclei** | DAST | Template-based vulnerability scanner |
| **Semgrep** | SAST (Static) | Pattern-based code analysis |
| **CodeQL** | SAST | Deep semantic analysis (GitHub-native) |

Run ZAP baseline scan in CI. Semgrep rules for common API vulnerabilities.

## Mutation testing

Line coverage is not quality — mutation testing is. Tools introduce small code changes (mutants) and verify tests catch them.

| Tool | Language |
|---|---|
| **Stryker** | JavaScript/TypeScript |
| **mutmut** | Python |
| **PIT** | Java/JVM |

A 93% line coverage can have 34% mutation score — 60-point gap of fake coverage.

## Mocking tools

| Tool | Use |
|---|---|
| **MSW (Mock Service Worker)** | Browser + Node.js HTTP mocking |
| **nock** | Node.js HTTP interceptor |
| **WireMock** | JVM HTTP mock server |
| **testcontainers** | Real dependencies in Docker for integration tests |

## Integration testing

Use testcontainers for real databases, Redis, message brokers. Tests run against actual services, not mocks. Compose with `docker compose` for multi-service integration.

## Contract test workflow

1. Consumer writes test defining expected request/response
2. Pact file generated from consumer test
3. Provider verifies Pact file against running service
4. Both publish results to Pact Broker
5. `can-i-deploy` check gates CI/CD pipeline
