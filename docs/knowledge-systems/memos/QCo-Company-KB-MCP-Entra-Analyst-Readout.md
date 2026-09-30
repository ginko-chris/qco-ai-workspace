# Company KB MCP + Entra — Analyst Readout
**From:** QCo Analyst · **For:** Chris · **Date:** 2026-09-29
**Evidence:** `research/QCo-Grok-MCP-Entra-Research-Pack.md` (Knowledge Systems Researcher, evidence only). The recommendations below are Analyst interpretation.

## BLUF
1. **This is a build-time item. It doesn't block the conceptual design.** Company KB design stays green-lit as is.
2. **Build the MCP server to the MCP spec with Entra and nothing else.** The server is an Entra-protected API and each agent is a pre-registered client, so there's no dynamic-registration shim. Microsoft calls those dev/test only. This keeps the server independent of any one agent. Whichever agent passes the test plugs in, and one that fails can't force a redesign.
3. **Run a short hands-on test before choosing the first client.** Stand up a throwaway MCP server with one `whoami` tool behind a test Entra app registration, and try to connect it from Grok Business and from Claude Cowork. It answers open items 1, 2 and 7: whether a fixed client ID is accepted, what redirect URI is used, whether each user signs in separately or the admin's token is shared, and whether Entra accepts the `resource` parameter. It takes about half a day with the MSP or partner, and uses no production data.
4. **Current read by agent:**
   - **Claude Cowork** has a documented path through an admin-entered fixed client ID. It's the fallback if Grok fails the test.
   - **Grok (Business/Enterprise)** is unknown until tested. The community reports conflict.
   - **Grok Bot** isn't planned as a Company KB client for now. It only supports dynamic registration, which Entra doesn't offer, there's no safe way to set headers, and the redirect bug has no fix date. On top of that, Team Bots share one computer per user. Revisit when a fixed client ID is supported.
   - **xAI API** works with the app handling Entra sign-in. Hold it as the route for an in-house app, and don't build one just for the pilot.
5. **Strety is the harder part.** Strety appears to use its own OAuth, not Entra. If that's right, Entra's on-behalf-of exchange can't issue Strety tokens. The honest pattern is a one-time Strety consent per person to the QCo server, with refresh tokens encrypted in Key Vault and keyed to the Entra user ID, read scope only. That means QCo holds a small credential vault, so it needs an owner, token rotation, and a revoke-on-offboarding step in the SCIM/offboarding checklist.
   - Confirm Strety's auth model with Strety directly before build. It's Low confidence today.
   - Reject a single shared Strety service token with QCo-side filtering. It breaks "each user sees only what they can see" and turns any filter bug into a leak.
6. **Front door.** Azure API Management checking Entra tokens in front of the server is a sensible production choice for thin IT, because the MSP can operate it. Keep the live Strety cite filter (MCP stub rule 6) in the server, not the gateway.

## Reject
- Passing a user's Entra sign-in token straight through to the MCP server. The MCP spec forbids it and Microsoft calls it a vulnerability.
- A DCR/OAuth proxy shim in production.
- Any Strety write scope.

## Ask (one, reversible)
Approve the half-day test with the MSP or partner: a test Entra app registration, a public `whoami` MCP endpoint, and a connection test from Grok Business and Claude Cowork. The result chooses the first pilot client.
