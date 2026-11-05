---
name: nestjs
description: >-
  General NestJS backend engineering skill. Use when building a new API feature,
  implementing a domain module, or making architectural backend decisions. Covers
  approach, layer responsibilities, and which sub-skills to reach for.
---

# NestJS Engineering

## Before writing code

1. **Understand the domain** — what entity/resource is this? What are its lifecycle events?
2. **Map the data flow** — HTTP input → DTO → service → repository → response
3. **Identify side effects** — emails, events, external API calls, cache invalidation
4. **Pick the right sub-skill** — see table below

## Sub-skills

| Task | Skill | Path |
|------|-------|------|
| Scaffolding a new domain module | `nestjs-module` | `.agents/skills/backend/nestjs-module/SKILL.md` |

## Approach to a feature

1. Define the entity and its fields
2. Write the input DTOs (create, update, query) with class-validator
3. Define the response DTO — map only what the client needs
4. Implement the repository (queries, persistence)
5. Implement the service (business logic, error throwing)
6. Implement the controller (routes, guards, DTO binding)
7. Register everything in the module
8. Write unit tests for the service, e2e tests for the controller

## Design defaults

- One module per business domain
- Throw HTTP exceptions from services — never from controllers
- Never expose entity internals in responses — always map to a response DTO
- Env variables validated with Zod at startup — fail fast on misconfiguration
- Global ValidationPipe with `whitelist: true` and `transform: true`

## Checklist

```
- [ ] Entity defined with correct types and decorators
- [ ] Input DTOs validated with class-validator
- [ ] Response DTO maps only public fields
- [ ] Service throws typed HTTP exceptions on failure
- [ ] Module registered in AppModule (or parent module)
- [ ] Unit test for service business logic
- [ ] E2E test for happy path and error cases
```
