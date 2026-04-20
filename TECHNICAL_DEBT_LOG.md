# Technical Debt Log

## Deferred: LinkedIn OAuth/Sign-In v2 Migration vs Browser-Session Auth

**Issue Status**: Tracked (see GitHub issues)  
**Introduced**: April 2026 (CodeMie remote integration effort)  
**Impact**: Remote MCP deployments depend on browser-session cookies + edge auth, not official LinkedIn OAuth.

### Summary

Current MCP server requires browser session login and cookie-based auth. Integration with LinkedIn Sign-In v2 (OIDC) was evaluated but deferred because:

1. **Architecture Incompatibility**: Current tools perform browser-based web scraping that depends on LinkedIn session cookies (`li_at`, etc.). OIDC bearer tokens don't map to browser session state.

2. **Feature Gap**: Sign-In with LinkedIn v2 scopes (openid/profile/email) are insufficient for current scraping tools (profiles, companies, jobs, messaging). Would require rewrite to official APIs, many of which are unavailable or restricted.

3. **Policy Risk**: LinkedIn ToS prohibit automated tools; this repo's browser-session approach already operates in a constrained model. OIDC doesn't resolve this risk, only shifts it to API-bound mode.

4. **Effort**: Full migration would require:
   - OAuth/OIDC client implementation
   - Token storage and refresh logic
   - API client layer for each tool
   - Compatibility matrix (current scraping tools vs API-available endpoints)
   - Approval/product alignment with LinkedIn

### Decision

**Status**: DEFERRED — continue browser-session auth for existing deployments.

**Recommendation for Future Work**:
1. Document supported auth modes clearly in README.
2. If API expansion becomes priority, produce a capability matrix and cost estimate.
3. Consider dual-mode architecture (browser scraping + OAuth for select tools) only if specific business driver emerges.

### Remote Deployment Implication

CodeMie and other remote clients connecting to this MCP server must:
- Accept that underlying LinkedIn auth is browser-session based (not OAuth).
- Provide HTTPS + edge-level auth (Basic Auth, SSO, mTLS) as the security boundary.
- Not expect Sign-In with LinkedIn v2 as an official auth method for this server.

### Related Files

- [linkedin_mcp_server/setup.py](linkedin_mcp_server/setup.py#L75) — Browser login flow
- [linkedin_mcp_server/authentication.py](linkedin_mcp_server/authentication.py#L50) — Runtime auth state
- [linkedin_mcp_server/core/auth.py](linkedin_mcp_server/core/auth.py#L63) — Auth validation
- [README.md](README.md#L14) — LinkedIn ToS warning

### Followup Issues

1. Feature: Evaluate LinkedIn OAuth migration vs browser-session auth (design phase).
2. Docs: Clarify remote CodeMie auth model and limitation (user-facing clarity).
