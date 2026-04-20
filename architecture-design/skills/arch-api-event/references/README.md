# API Standards Reference

This folder contains organization-specific API standards that the `api-patterns` skill enforces.

## How to populate

Drop your organization's API standards documents here. The skill will load them at runtime and
use them as the authoritative source for pattern enforcement and deviation flagging.

## Expected files (add as available)

| File | Contents |
|------|----------|
| `api-standards.md` | Preferred API styles, versioning policy, auth requirements |
| `error-codes.md` | enterprise error code taxonomy by domain |
| `gateway-policy.md` | API Gateway routing, rate limit tiers, approval process |
| `openapi-template.yaml` | Standard OpenAPI spec template for new APIs |

## How the skill uses these files

When standards files are present, the skill reads them before generating output and:
- Uses them as the authoritative enforcement source (overrides built-in defaults)
- Cites the specific standard when flagging a deviation (e.g. "per api-standards.md §3.2")
- Applies any templates found here as starting points for generated specs

When files are absent, the skill falls back to built-in enterprise defaults.
