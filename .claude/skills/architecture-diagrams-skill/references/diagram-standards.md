# Architecture Diagram Standards

> **TODO:** Replace this placeholder with your organization's diagramming standards.
> This file is read by the `architecture-diagrams-skill` before producing any diagram.

---

## Application ID Format

> **TODO:** Define how Application IDs appear in diagrams.
> Example: `[APP-12345] Payment Service` or `[PAYMENT-SVC] Payment Service`

Format: `[{ID}] {Application Name}`
Registry: [Link to your Application Registry / CMDB]

---

## C1 — System Context Standards

> **TODO:** Define shapes, colors, and conventions for C1 context diagrams.

### Shapes
- Person / User role: [shape — e.g., rounded rectangle, person icon]
- Internal system (in scope): [shape + color]
- External system: [shape + color]
- Connection label format: [e.g., "REST API / HTTPS", "Kafka event"]

### Layout
- [Direction convention — e.g., left-to-right, top-to-bottom]
- [Grouping conventions — e.g., boundary box for internal vs external]

---

## C2 — Container Diagram Standards

> **TODO:** Define shapes, colors, and conventions for C2 container diagrams.

### Container Types and Shapes
- Web application / UI: [shape + color]
- API / microservice: [shape + color]
- Database: [shape + color]
- Message queue / topic: [shape + color]
- File storage: [shape + color]
- External system: [shape + color]

### Connection Labels
- Format: `[Protocol]: [Description]`
- Example: `REST/HTTPS: Submit order` or `Kafka: order.created`

---

## C3 — Component Diagram Standards

> **TODO:** Define company-specific C3 conventions (note: differs from industry-standard C4 C3).

### Component Types
- [Component type 1]: [shape + color]
- [Component type 2]: [shape + color]

### Layout and Grouping
- [Describe company conventions]

---

## C4 / Workload Design Standards

> **TODO:** Define company-specific workload diagram conventions (note: differs from industry-standard C4).

### Workload Elements
- Compute unit: [shape + color]
- Data store: [shape + color]
- Network boundary: [shape + color]
- AWS service: [shape + color]

### Required Information per Element
- Component name + App ID
- Instance type / compute class (for AWS)
- Scaling configuration summary

---

## Sequence Diagram Standards

> **TODO:** Define conventions for sequence diagrams.

### Participant Naming
Format: `[{App-ID}] {Service Name}` or `{Actor Role}`

### Message Label Format
- Synchronous call: `METHOD /path` or `operationName()`
- Async/event: `event: {topic.name}`
- Response: HTTP status + brief description

### Styling
- [Activation box conventions]
- [Return message conventions]
- [Error/exception flow conventions]

---

## Color Palette

> **TODO:** Define the company color scheme for diagrams.

| Element | Fill Color | Border Color | Text Color |
|---------|-----------|-------------|------------|
| In-scope system | [hex] | [hex] | [hex] |
| External system | [hex] | [hex] | [hex] |
| Database | [hex] | [hex] | [hex] |
| Person / actor | [hex] | [hex] | [hex] |
| Boundary box | [hex] | [hex] | [hex] |
