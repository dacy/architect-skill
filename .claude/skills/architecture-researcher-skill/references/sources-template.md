# Source Configuration Template

Copy this file to `sources.md` and fill in your organization's details.
The research agent reads `sources.md` at runtime to know how to access each source.

---

## How to configure

For each source, specify:
- **enabled**: true/false
- **access**: `api` (REST endpoint) or `web` (agent browses the UI) or `tool` (built-in)
- **base_url**: root URL for the source
- **auth**: how to authenticate (`api_key`, `oauth`, `sso`, `none`)
- **search_path**: URL pattern for search — use `{query}` as the placeholder
- **notes**: special instructions for navigating or querying this source

---

## sources.md format

```yaml
confluence:
  enabled: true
  access: api           # or: web
  base_url: https://your-confluence.example.com
  auth: api_key         # or: oauth, sso, none
  search_path: /rest/api/content/search?cql=text~"{query}"&limit=10
  space_keys:           # Optional: limit to specific Confluence spaces
    - ARCH
    - PLATFORM
  notes: "Use CQL for precise searches. Space keys narrow results significantly."

api_marketplace:
  enabled: true
  access: web           # or: api
  base_url: https://your-api-marketplace.example.com
  auth: sso
  search_path: /search?q={query}
  notes: "Log in via SSO. Search bar is top-right. Filter by domain in left panel."

application_registry:
  enabled: true
  access: web
  base_url: https://your-cmdb.example.com
  auth: sso
  search_path: /applications?search={query}
  notes: "Filter by 'Active' status to exclude decommissioned apps."

web_search:
  enabled: true
  access: tool          # Uses built-in web_search — no configuration needed
  notes: "Used for public vendor docs and open-source options."
```

---

## Tips

- Start with `access: web` if you don't have API credentials — the agent can browse
- Add `space_keys` for Confluence to avoid noise from unrelated spaces
- Note any VPN / internal network requirements so the agent can flag unreachable sources
- You can add custom sources using the same format with a unique key name

---

## Minimal working config

```yaml
confluence:
  enabled: true
  access: web
  base_url: https://confluence.yourcompany.com
  auth: sso
  search_path: /search?text={query}

web_search:
  enabled: true
  access: tool
```
