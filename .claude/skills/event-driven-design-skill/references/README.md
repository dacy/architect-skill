# Event-Driven Architecture Standards Reference

This folder contains organization-specific event and messaging standards that the
`event-driven-design` skill enforces.

## How to populate

Drop your organization's event architecture standards documents here. The skill loads them at runtime.

## Expected files (add as available)

| File | Contents |
|------|----------|
| `event-standards.md` | Kafka cluster policy, topic conventions, retention rules |
| `cloudevents-schema.md` | enterprise CloudEvents envelope spec and required fields |
| `asyncapi-template.yaml` | Standard AsyncAPI 2.x template for new event contracts |
| `schema-registry-guide.md` | Schema Registry location, Avro/JSON Schema conventions |
| `kafka-naming.md` | Topic naming convention, consumer group naming rules |

## How the skill uses these files

When standards files are present, the skill reads them before generating output and:
- Uses them as the authoritative source for all naming conventions and patterns
- Cites the specific standard when flagging a deviation
- Applies any templates found here as starting points

When files are absent, the skill falls back to built-in Kafka + CloudEvents defaults.
