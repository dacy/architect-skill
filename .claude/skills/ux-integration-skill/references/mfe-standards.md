# MFE & Operational Desktop Standards

> **TODO:** Replace this placeholder with your organization's MFE and operational desktop standards.
> This file is read by the `ux-integration-skill` before producing any UI/UX design.

---

## Operational Desktop Overview

> **TODO:** Describe the company's operational desktop shell application.

- **Shell framework:** [e.g., Single-SPA, Module Federation host, custom]
- **Shell version:** [current version]
- **Documentation:** [Link to internal docs]
- **Owner / team:** [Team name + contact]

---

## MFE Framework

> **TODO:** Define the MFE framework and module federation configuration.

- **Module federation:** [Webpack 5 / Vite / other] — version [x.x]
- **MFE entry point contract:** [how the shell loads MFEs — remote URL, manifest]
- **Approved UI frameworks:** [e.g., React 18+, Angular 15+] — no other frameworks without exception

---

## MFE Registration

> **TODO:** Document how MFEs are registered with the shell.

```json
{
  "name": "my-mfe",
  "route": "/feature-path",
  "remoteEntry": "https://cdn.company.com/my-mfe/remoteEntry.js",
  "exposedModule": "./App",
  "permissions": ["ROLE_NAME"],
  "version": "1.0.0"
}
```

Registry location: [Link to MFE registry]

---

## Shell Shared Services API

> **TODO:** Document the services the shell exposes to MFEs.

### User Context
```typescript
// How MFEs access the current user
shell.getUser(): { id, name, email, roles, tenantId }
```

### Navigation
```typescript
// How MFEs trigger navigation
shell.navigate(route: string): void
```

### Notifications
```typescript
// How MFEs push notifications
shell.notify({ type: 'success'|'error'|'info', message: string }): void
```

### Theming / Styling
- Design token system: [CSS variables / design token library]
- Shared component library: [Name + link]
- Theme: [how MFEs consume the shell theme]

---

## Cross-MFE Event Bus

> **TODO:** Document the event bus API for cross-MFE communication.

```typescript
// Subscribe to events
shell.eventBus.on('eventName', handler: (payload) => void): unsubscribe

// Publish events
shell.eventBus.emit('eventName', payload): void
```

**Registered cross-MFE events:**

| Event Name | Publisher | Consumers | Payload Schema |
|-----------|-----------|-----------|---------------|
| [event] | [MFE] | [MFE(s)] | [Schema] |

---

## Lifecycle Hooks

> **TODO:** Document MFE lifecycle hooks.

```typescript
export const { bootstrap, mount, unmount } = singleSpaReact({
  // company-standard configuration
})
```

---

## Deployment

> **TODO:** Document how MFEs are built and deployed.

- **Build output:** static bundle (JS + CSS)
- **CDN:** [Company CDN — URL pattern]
- **CI/CD pipeline:** [Approved pipeline template / link]
- **Version strategy:** [How breaking changes are managed]
