# Ready-to-File GitHub Issues

Copy-paste these into GitHub issue templates when filing.

## Issue 1: Feature Request — OAuth Migration Debt

**Title**: [FEATURE] Evaluate LinkedIn OAuth migration vs browser-session auth

**Feature Description**:
Evaluate whether to add an optional OAuth/OIDC-based authentication mode for official LinkedIn APIs while keeping the current browser-session scraping mode.

Current remote setup relies on browser-session cookies and edge-level auth. This debt item captures the technical and business tradeoffs of a potential OAuth migration.

**Use Case**:
- Reduce reliance on browser-session authentication for some use cases.
- Clarify product boundaries and supported auth models in documentation.
- Improve long-term maintainability and security posture for externally exposed deployments.
- Enable potential future integration with official LinkedIn APIs beyond web scraping.

**Suggested Approach**:
1. Document a capability matrix: current scraping tools vs what's available through official LinkedIn APIs.
2. Identify policy and scope constraints (LinkedIn ToS, product access, regional limits).
3. Estimate effort for full dual-mode architecture or API-only migration.
4. Publish a decision memo recommending either:
   - Dual-mode (keep scraping + add OAuth for select tools).
   - API-only mode (rewrite all tools to official APIs).
   - Explicit non-go (defer indefinitely, stick with browser-session).
5. Create follow-up implementation issues if migration is approved.

**Label**: `enhancement`, `debt`, `authentication`

---

## Issue 2: Documentation — CodeMie Remote Auth Clarity

**Title**: [DOCS] Clarify remote CodeMie auth model and known limitations

**Location**:
- README.md: Remote HTTP usage section (around [README.md#L95](README.md#L95))
- README.md: Deployment security notes section
- Relevant: TECHNICAL_DEBT_LOG.md (newly created)

**Problem**:
Current documentation does not explicitly state that:
1. Remote MCP deployments (e.g., CodeMie) depend on browser-session cookies, not OAuth/Sign-In with LinkedIn v2.
2. Edge-level auth (HTTPS + credentials) is mandatory for safe remote exposure.
3. MCP client must support standard session header semantics (Mcp-Session-Id).

This causes confusion when evaluating LinkedIn Sign-In v2 integration or deploying to remote hosts.

**Suggested Fix**:
Add a "Known Limitations" callout box and a "Secure Remote Deployment Checklist" to README:

```markdown
### Known Limitations

**Remote Deployments**: This MCP server uses LinkedIn browser-session authentication (cookies), not OAuth/Sign-In with LinkedIn v2.
When exposing the server to remote clients (e.g., CodeMie on another machine):
- Underlying LinkedIn auth remains browser-session based.
- You must secure the endpoint at the edge (HTTPS + credentials).
- See [Secure Remote Deployment Checklist](#secure-remote-deployment-checklist) below.

### Secure Remote Deployment Checklist

- [ ] Run MCP server bound to localhost only (do not use 0.0.0.0).
- [ ] Front with an HTTPS-terminating proxy or tunnel (Cloudflare Tunnel, ngrok, VPS reverse proxy).
- [ ] Enable edge authentication (Basic Auth, SSO, mTLS, or API Gateway).
- [ ] Confirm remote client supports MCP streamable-http session flow (initialize → tools/call with Mcp-Session-Id header).
- [ ] Test with MCP Inspector or a trusted MCP client before connecting external services.
```

Include a reference to TECHNICAL_DEBT_LOG.md for detailed tradeoffs.

**Label**: `documentation`

---

## Debt Tracking Summary

Both issues should reference [TECHNICAL_DEBT_LOG.md](TECHNICAL_DEBT_LOG.md) in the repository. This file documents the decision to defer OAuth migration and the design context for remote CodeMie integration.

**Key Evidence**:
- Browser login flow: [linkedin_mcp_server/setup.py#L75](linkedin_mcp_server/setup.py#L75), [setup.py#L93](linkedin_mcp_server/setup.py#L93)
- Runtime auth: [linkedin_mcp_server/authentication.py#L50](linkedin_mcp_server/authentication.py#L50), [core/auth.py#L63](linkedin_mcp_server/core/auth.py#L63)
- MCP session behavior: [AGENTS.md#L43-L56](AGENTS.md#L43)
- Policy note: [README.md#L14](README.md#L14)
