# QCo Research Pack: Grok, Grok Bot, xAI API (and Claude Cowork) with Remote MCP and Microsoft Entra ID

**Type:** Vendor-documentation evidence pack. **No recommendations.** This pack describes what the sources say. It does not tell QCo what to choose.
**Prepared:** 2026-09-29 (retrieved 07:23–07:45 ET). **Status:** Build-time verification blocker for the Company KB pilot (v0.4 architecture).
**Scope:** Can each target agent connect to QCo's read-only remote MCP server on Azure, so that every call runs as the person asking, signed in through Entra ID?

**Confidence scale:** **High** = first-party doc read today · **Medium** = first-party marketing, help or staff forum answer, or consistent third parties · **Low** = single third party or community report · **Unverified** = no source found.

**On doc dates:** docs.x.ai and cursor.com/docs pages show no "last updated" date and return no `Last-Modified` header (checked 2026-09-29 11:26 UTC). Where no date is shown, this pack says "no date shown; retrieved 2026-09-29". Model names in xAI code samples (`grok-4.7`) show the page is current as of the retrieval date.

---

## 0. Architecture context (as briefed, not verified)

- QCo's MCP server is read-only and is built in-house or with a partner. It runs on Azure and exposes fixed tools: search the certified KB, look up who owns something, fetch a cited doc, and pass product questions to a graph. It also offers a small REST fallback.
- Every call must run as the asking person (Entra sign-in plus on-behalf-of), so an agent never sees more than that person can.
- The server wraps **Strety's public REST API** using per-person read-only tokens.
  - *Evidence note (Low):* No official Strety API reference was found today. Third-party code says Strety uses **its own OAuth 2.0** (authorize at `https://2.strety.com/oauth/authorize`, token at `https://2.strety.com/api/v1/oauth/token`, base URL `https://2.strety.com/api/v1`, scopes `read`/`write`, access tokens expire after about 2 hours, rate limit 10 requests per 10 seconds). Sources: [brentwpeterson/mcp-strety](https://github.com/brentwpeterson/mcp-strety), [n8n-nodes-strety](https://github.com/ajoshuasmith/n8n-nodes-strety), [Pipedream Strety](https://pipedream.com/apps/strety).
  - *Descriptive implication:* If Strety is not an Entra-protected resource, Entra OBO cannot mint Strety tokens (OBO only issues tokens for Entra-registered APIs). The server would have to map the Entra user (for example the `oid` claim) to that person's stored Strety token. This point is **Unverified** against Strety's own docs.
- Target agents: **Claude Cowork**, **Grok** (grok.com and apps, including Business and Enterprise), and **Grok Bot** (the brief calls it "Grok Bots").

---

## 1. BLUF (evidence only)

**Question: Can the agent connect to a remote MCP server that authenticates users through Entra ID sign-in?**

| Agent | Answer | Why (short) | Confidence |
|---|---|---|---|
| **Claude Cowork** | **YES (documented mechanism).** Entra is not named. | Custom remote MCP connectors are documented for Cowork. OAuth works through DCR or CIMD out of the box, and an admin can enter a **static OAuth Client ID and Secret** in "Advanced settings". Entra has no DCR or CIMD, so the pre-registered client path is the one that fits Entra. | High for the mechanism; Medium for Entra (inference) |
| **Grok** (grok.com/apps, Business/Enterprise) | **UNVERIFIED** | Custom MCP connectors are documented, and so is "complete any required authentication (OAuth or API keys)". xAI documents **no** static client-ID field, no redirect URI, no DCR/CIMD detail and nothing on Entra. Community reports conflict: some say Grok needs DCR and has no client-ID or header field, others describe an "OAuth Credentials Required" screen that takes a Client ID. Needs a hands-on test against Entra. | High that custom MCP exists; Unverified for Entra |
| **Grok Bot** | **UNVERIFIED; the evidence leans NO today** | Official docs cover plugins/connectors and an admin MCP allowlist but say nothing about Entra for custom MCP. Cursor staff (forum, Sep 2026) say Grok Bot connectors authenticate **only via OAuth**, register through **DCR**, and have **no safe way to set custom headers**. Entra does not support DCR, and no static client-ID entry is documented for the Bot. | Medium (staff forum), Unverified by test |
| **xAI API** (Responses API / SDK remote MCP tool) | **YES, conditionally (app-mediated)** | The remote MCP tool takes `server_url` plus an `authorization` token and extra `headers` **per request**. xAI runs no OAuth sign-in itself. The calling app has to sign the user in with Entra and pass a token issued for the MCP server's audience. | High |

**BLUF bullets:**
1. **Grok custom MCP is real and documented** (grok.com/connectors → New Connector → Custom, public HTTPS URL only). On Business/Enterprise a team admin must first add it in console.x.ai (Grok Business → Connectors → Add Connector → Other). xAI says nothing about how auth works for custom MCP beyond "OAuth or API keys". (High)
2. **Grok's Microsoft 365 connectors** (OneDrive, SharePoint, Outlook, Teams) are xAI-built. They use per-user **delegated** Entra OAuth after a tenant admin grants consent. This is evidence Grok can do per-user Entra for *its own* apps, **not** for a custom MCP server. (High)
3. **Grok Bot** is the official name. It is a joint xAI/Cursor product: docs live at docs.x.ai/grok-bot and cursor.com/docs/grok-bot, and it runs on Cursor (Anysphere) infrastructure. Custom MCP is added through chat. Tokens stay on Cursor's backend and are per member. There is a known DCR bug where it sends a `cursor://` redirect URI, and no secure header entry exists. (Medium–High)
4. **Grok Bot admin controls exist** (Enterprise): an MCP URL allowlist, connector policy through the Team Marketplace, SAML SSO with Entra, SCIM, audit logs and Action Recording. There is **no way to push connectors** to members. (High)
5. **Neither Grok nor Grok Bot documents passing the user's Entra sign-in token to an MCP server.** Grok Enterprise SSO (Azure AD/Entra) is for signing in to Grok itself. Under the MCP spec, passing that token through would be forbidden anyway, because tokens must be issued for the MCP server's audience. (High for the spec; Unverified for vendor behavior)
6. **Entra has no DCR or CIMD** (Microsoft Learn, High). The MCP spec (2025-11-25 and 2026-07-28) makes DCR a MAY and CIMD a SHOULD, and says clients SHOULD support pre-registered static clients. Whether an agent accepts a pre-registered client ID is therefore the deciding feature. (High)
7. **The xAI API remote MCP tool** supports Streamable HTTP and SSE, a per-request bearer `authorization`, custom `headers` and `allowed_tools`. `require_approval` and `connector_id` are not supported. (High)
8. **Community reports are Low confidence.** For Grok: hangs after OAuth, no prompt for API keys or headers, DCR callback failures. For Grok Bot: hung custom remotes blocked all connectors (fixed 7 Sep 2026), "fetch failed" when the server isn't publicly reachable, and DCR rejected by Cloudflare over the `cursor://` redirect (open as of 25 Sep 2026). No relevant X posts were found. (Low)

---

## 2. Capability matrix

| Row | Remote/custom MCP | Config path | Transport | Auth methods documented | Entra specifically | Per-user identity | REST / non-MCP fallback | Admin controls | Doc date | Confidence |
|---|---|---|---|---|---|---|---|---|---|---|
| **Grok consumer** (grok.com, apps) | Yes ("Custom MCP connectors") | grok.com/connectors → New Connector → Custom → URL → auth | Streamable HTTP and SSE both implied by the tunneling page (Cloudflare quick tunnels don't support SSE; use ngrok) | "Complete any required authentication"; "If your MCP server requires OAuth or API keys, you will still complete that flow in Grok." No detail on DCR, static client, headers or redirect URI | Not documented for custom MCP | Connector is added per user account (inference). Token handling not documented | No OpenAPI/"actions" tool found | None (consumer) | No date shown; retrieved 2026-09-29 | High (exists); Unverified (auth detail) |
| **Grok Business / Enterprise** | Yes, admin-provisioned | console.x.ai → team → Grok Business → Connectors → + Add Connector → **Other** → MCP URL → "Complete any required authentication"; members then connect on grok.com/connectors | As above | As above. Needs Team Read-Write permission | Entra SSO for **Grok sign-in** (Enterprise: "Azure AD" listed). Built-in M365 connectors use tenant ID plus admin consent plus per-user delegated scopes | Built-in M365 connectors: per-user delegated (High). Custom MCP: whether the admin's auth at provisioning is shared or each member signs in is **not documented** | None found | Admin must provision each connector (effectively an allowlist); Configure/Remove; Enterprise-only Organizations, SSO, SCIM with group→role→license mapping | No date shown; retrieved 2026-09-29. Business launch post undated | High (provisioning); Unverified (custom auth) |
| **Grok Bot** | Yes (plugins/connectors; custom remote MCP added **through chat**) | Tell the Bot "Add this MCP server: https://…/mcp"; OAuth through the Plugins "Authenticate" card; tokens held on Cursor's connector backend | Streamable HTTP or SSE, public HTTPS only (staff) | OAuth only (staff, 17 Sep 2026). DCR `redirect_uris` = `cursor://anysphere.cursor-mcp/oauth/callback`, `https://www.cursor.com/agents/mcp/oauth/callback`, `http://localhost:8787/callback`. `AddMcpServer.headers` exists but there is "no safe way" to enter secrets | Entra is documented only for **SAML SSO to Cursor/Grok Bot** and for Conditional Access on the Bot computer's browser. Nothing for MCP | Per member: tokens on Cursor backend, and "Bots act as the signed-in member". Exceptions: Team Bots in Slack or group chats use one computer, and "team-managed connectors may use team or service-account credentials" | The Bot has a shell and browser (computer use). REST with a per-user Entra token is not documented as a feature | Enterprise: org enable switch, MCP URL allowlist, connector policy (Team Marketplace), Network Controls, SCIM, audit logs, Action Recording, OTel. No connector push | cursor.com docs: no date shown; forum posts 12 Aug–25 Sep 2026 | High (admin); Medium (auth, from staff forum) |
| **xAI API** | Yes ("Remote MCP Tools") | `tools:[{type:"mcp", server_url, server_label, allowed_tools, authorization, headers}]` in Responses API / xAI SDK / Speech-to-Speech | "Only Streaming HTTP and SSE transports are supported" | `authorization` (token placed in the Authorization header); arbitrary `headers`. xAI performs no OAuth flow | Not mentioned. The app can supply any bearer token, including a per-user Entra access token for the MCP API | Possible per request if the calling app gets a per-user token (inference from the per-request parameter) | Function calling: the model returns a `function_call`, **your code** runs it (for example calling QCo REST with the user's bearer token) | xAI Console teams, API keys, audit log, ZDR | No date shown; retrieved 2026-09-29 | High |
| *(Reference)* **Claude Cowork** | Yes (custom connectors) | Org settings → Connectors → Add → Custom → Web → URL → Advanced settings (OAuth Client ID/Secret) | Remote MCP from Anthropic cloud | `oauth_dcr`, `oauth_cimd` out of box; admin-entered static OAuth client; `static_headers` (beta, admin-entered) | Not named; the pre-registered client path fits Entra's lack of DCR (inference) | Per user OAuth | n/a here | Org admin adds connectors | Help article undated; retrieved 2026-09-29 | High |

---

## 3. Q1: Grok (consumer, Business, Enterprise)

### 3.1 Custom remote MCP support: documented (High)
- "If you need to connect Grok to a service not available in the catalog, you can bring your own Model Context Protocol (MCP) server… 1. Go to grok.com/connectors. 2. Click New Connector, then select Custom. 3. Enter the MCP server URL and complete any required authentication. Grok will discover the tools your MCP server exposes…" [docs.x.ai/grok/connectors](https://docs.x.ai/grok/connectors)
- "Connectors are available to all Grok users… For Grok Business and Enterprise users, a team admin must first provision a connector in the cloud console." (same page)
- The server "must be reachable over the public internet". Grok rejects localhost and private ranges (127.0.0.1, 10.x, 172.16.x, 192.168.x). [Custom MCP Server Tunneling](https://docs.x.ai/grok/connectors/custom-mcp-tunneling)

### 3.2 Transport: Medium (implied)
- The tunneling page says "Cloudflare quick tunnels do not support Server-Sent Events (SSE). If your MCP server uses the SSE transport, use ngrok instead. Servers using the newer Streamable HTTP transport work fine with Cloudflare." This implies Grok accepts both SSE and Streamable HTTP. There is no explicit transport table for grok.com connectors.

### 3.3 Auth: what is documented vs observed
| Item | Documented by xAI | Observed / community | Confidence |
|---|---|---|---|
| OAuth | "complete any required authentication"; "If your MCP server requires OAuth or API keys, you will still complete that flow in Grok" | Many vendor guides describe browser OAuth sign-in after pasting the URL (Nimply, PortEden, Zephex) | High (exists); Medium (flow) |
| OAuth 2.1 / PKCE | Not documented | Zephex and doctorSIM describe PKCE with no client secret | Low–Medium |
| DCR (RFC 7591) | Not documented | david.coffee: "many MCP servers fail due to missing dynamic client registration". usecarly (Lofty, Jobber) says the Grok dialog has "no field for a client ID" and cites a GitHub issue (Unraid MCP, reportedly opened 6 Sep 2026) where DCR failed on Grok's callback. The issue itself was **not located** | Low |
| Static client ID ("pre-registration") | Not documented | **Conflicting.** Zephex and doctorSIM describe an optional second screen, **"OAuth Credentials Required"**, taking a Client ID (secret blank, PKCE on, scopes). usecarly says there's no such field. **This matters for Entra** | Low (conflicting) |
| CIMD | Not documented | None found | Unverified |
| Static bearer / API-key header | "API keys" mentioned, but no UI described | Plane issue #9055 (May 2026): "No prompt appeared for entering the API key or headers". Fat Heron: "If the connector form accepts a header…" (hedged) | Low |
| Redirect URI Grok uses | **Not documented** | A forum report mentions "the existing Grok web callback" without giving the value | Unverified |
| Entra ID for custom MCP | Not documented | None found | Unverified |
| Per-user vs shared token | Consumer: each user adds their own connector. Business/Enterprise: the admin adds the custom MCP with "complete any required authentication"; for catalog connectors "team members can then connect their own accounts". **Not stated for custom MCP** | usecarly/tempreon repeat the admin-provisioning gate | Unverified (custom) |

### 3.4 Built-in Microsoft 365 connectors, Entra-relevant (High)
These are xAI's own connectors, not custom MCP. They show how Grok uses Entra:
- **OneDrive** (Business/Enterprise only). The admin supplies the Entra tenant ID and grants admin consent, then "individual team members can connect their own Microsoft accounts". Scopes `Files.ReadWrite`, `User.Read`, `offline_access`, "delegated and scoped to the signed-in user". [OneDrive](https://docs.x.ai/grok/connectors/onedrive)
- **SharePoint** (Business/Enterprise only). Two access modes: delegated (`Sites.Read.All`) or application (`Sites.Selected` plus a site picker). Background sync indexing runs as a dedicated user or as the app. "Indexed content is access-checked against the querying user on every request." Write access needs a separate Entra app (`Files.ReadWrite.All`). "SpaceXAI does not use your SharePoint data for model training." [SharePoint](https://docs.x.ai/grok/connectors/sharepoint)
- The built-in list also includes Outlook Mail & Calendar and Microsoft Teams. [Connectors](https://docs.x.ai/grok/connectors)

### 3.5 Grok REST/OpenAPI fallback
- No grok.com feature for OpenAPI "actions" or REST tools with per-user OAuth was found in xAI docs. **Unverified / not found.**

---

## 4. Q2: Grok Bot (the brief's "Grok Bots")

### 4.1 What the product is (High)
- Official name: **Grok Bot**. "Bots are AI teammates… Each Bot works on a persistent cloud computer with a browser, filesystem, and terminal." [docs.x.ai/grok-bot/overview](https://docs.x.ai/grok-bot/overview)
- Enterprise launch: "Grok Bot is now available for enterprises. Grok and Cursor Enterprise customers have free usage…" (post undated). [x.ai/news/grok-bot-for-enterprise](https://x.ai/news/grok-bot-for-enterprise)
- It runs in Cursor's cloud with one Firecracker microVM per user, and is administered from the **Cursor dashboard**. Access comes with paid Cursor plans, Cursor Teams/Enterprise, or a linked SuperGrok subscription. [cursor.com/docs/grok-bot/teams](https://cursor.com/docs/grok-bot/teams)
- Grok Bot is **not administered from console.x.ai**. Its identity, SSO and SCIM go through the Cursor app in Entra. [identity](https://cursor.com/docs/grok-bot/identity)

### 4.2 Custom remote MCP: Medium
- Docs: Bots "use connectors where available". Plugins are opened from the sidebar or an in-chat Connect card, and "OAuth tokens are held on Cursor's connector backend, and the Bot invokes tools without receiving them." [Work with Grok Bot](https://cursor.com/docs/grok-bot/work) (High)
- Custom server (Cursor staff, 30 Aug 2026): "There isn't a settings form for this… just tell your bot to add your MCP server, e.g. 'Add this MCP server: https://your-app.example.com/mcp'… your custom application's MCP endpoint needs to be reachable over the internet (a public HTTPS URL, using streamable HTTP or SSE)." [forum 169965](https://forum.cursor.com/t/grokbot-custom-connectors/169965) (Medium)
- Public-only is by design (staff, 12 Aug 2026): "Grok Bot is a cloud agent… the connection to the remote MCP and the OAuth discovery run from Cursor's infrastructure… treat public endpoint only as the rule for the cloud path." [forum 168188](https://forum.cursor.com/t/grok-bot-custom-remote-mcp-oauth-never-starts-fetch-failed-same-url-works-in-cursor-ide/168188) (Medium)

### 4.3 Auth: documented vs staff statements vs community
| Item | Evidence | Confidence |
|---|---|---|
| OAuth only | Staff (17 Sep 2026): "Connectors in Grok Bot authenticate only via OAuth. Tokens are stored on the connector backend… There are no owner-only settings, no separate secrets form, and no secret reference for arbitrary headers." [forum 171877](https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877) | Medium |
| Custom headers / API keys | Staff: "`AddMcpServer.headers` does exist at the API level, but there's currently nowhere to enter a secret there safely, not via chat and not via model-visible tool args." | Medium |
| DCR | Staff lists the DCR `redirect_uris`: `cursor://anysphere.cursor-mcp/oauth/callback`, `https://www.cursor.com/agents/mcp/oauth/callback`, `http://localhost:8787/callback`. **Bug:** authorization servers that allow only https/localhost reject the whole registration. "We're tracking it." No target version as of 25 Sep 2026 | Medium |
| Static OAuth client ID | Documented for the **Cursor IDE `mcp.json`** (`auth.CLIENT_ID`, `CLIENT_SECRET`, `scopes`; redirects `https://www.cursor.com/agents/mcp/oauth/callback`, `http://localhost:8787/callback`) [cursor.com/docs/mcp](https://cursor.com/docs/mcp). **Not documented for Grok Bot.** The Bot's connector UI "only offers OAuth authentication and account renaming" (user, forum 171877) | High (IDE); Unverified (Bot) |
| Entra for custom MCP | Not documented. Entra appears only for SAML SSO to Cursor and for Conditional Access on the Bot computer's browser ("doesn't apply to plugin sign-in") | Unverified |
| Per-user token | "Bots act as the signed-in member… Connector tokens stay on Cursor's backend… Team-managed connectors are the one exception: they may use team or service-account credentials." [security](https://cursor.com/docs/grok-bot/security). Team Bots in Slack/group chats "use one computer of its own"; "Secrets, plugins, and files the owner adds to a Team Bot are available in every teammate's chat." [teams](https://cursor.com/docs/grok-bot/teams) | High |
| Passing the user's Entra token to MCP | Not documented. Grok Bot sign-in uses Cursor SAML SSO, which is separate from plugin OAuth | Unverified |

### 4.4 Grok Bot REST fallback
- A Bot can use shell, browser and computer use. The documented credential path is "a secure secret request" for "supported connections", plus the browser signed in through the org IdP. A per-user Entra-token REST tool is **not documented**. Staff note (forum 168188): "Grok Bot cannot use the stdio mcp-remote bridge pattern… (remote HTTP only)" (user statement that staff did not dispute). A host-local bridge is not confirmed as planned.

---

## 5. Q3: Paths for agents that can't do remote MCP with Entra, plus Entra and MCP background

### 5.1 xAI API remote MCP tool (High)
- "Remote MCP tools are supported in the xAI native SDK, the OpenAI compatible Responses API, and the Speech to Speech API." "`require_approval` and `connector_id`… are not currently supported." [remote-mcp](https://docs.x.ai/developers/tools/remote-mcp)
- Parameters: `server_url` (required; "Only Streaming HTTP and SSE transports are supported"), `server_label` (required), `server_description`, `allowed_tools` (SDK: `allowed_tool_names`), `authorization` ("A token that will be set in the Authorization header on requests to the MCP server"), `headers` (SDK: `extra_headers`).
- "xAI manages the MCP server connection and interaction on your behalf." The call therefore comes from xAI infrastructure, so the server needs a publicly reachable URL (inference, consistent with the Grok connector docs).
- *Descriptive:* per-user identity depends on the calling app. The app would sign the user in with Entra, get an access token whose audience is the QCo MCP API, and send it as `authorization` on each request.

### 5.2 xAI function calling (High)
- "The model requests the call, you execute it locally, and return the result." Custom tools "pause execution and return to you for handling." [function-calling](https://docs.x.ai/developers/tools/function-calling). With function calling, the REST call to QCo (with the user's bearer token) happens in the app's own code, and the token never passes through xAI.

### 5.3 Data handling for the xAI API (relevant to tool payloads)
- API requests and responses are kept for 30 days (encrypted), and xAI does not train on them. Team-level ZDR is available but disables stateful Responses, Files, Collections and Batch. A US regional endpoint exists, but "server-side tools… are outside the guarantee". [security FAQ](https://docs.x.ai/developers/faq/security) (High)
- Grok Business: "no training on it, ever". Enterprise Vault offers a dedicated data plane and CMEK. [x.ai/news/grok-business](https://x.ai/news/grok-business) (Medium, marketing)

### 5.4 How Entra issues per-user tokens for a custom API (Microsoft docs, High)
1. **Register the API and expose scopes.** Set an Application ID URI (`api://…`), add delegated scopes (for example `KB.Read`) with "Admins and users" or "Admins only" consent, and optionally pre-authorize client apps. Tokens carry `scp`. [Expose a web API](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-configure-app-expose-web-apis) (ms.date 2025-05-14; updated_at 2026-06-15)
2. **Client gets a user token** for that API using auth code plus PKCE (the client must be registered in Entra, with its redirect URI).
3. **OBO:** the middle tier POSTs to `https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token` with `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer`, `assertion=<incoming token>`, `requested_token_use=on_behalf_of`, and a client secret or certificate. OBO "only uses delegated scopes"; "Applications can't redeem a token for a different app"; Conditional Access failures return `interaction_required` plus a claims challenge that has to be surfaced to the client. Consent comes through `knownClientApplications` with `.default`, pre-authorization, or admin consent. [OBO flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow) (ms.date 2025-01-04; updated_at 2026-06-15)
4. **Entra and MCP:** "Some providers support Dynamic Client Registration (DCR), but many don't, including Microsoft Entra ID. When DCR isn't available, the client needs to be preconfigured with a client ID." "Pass-through… create[s] security vulnerabilities… obtain a new token through the on-behalf-of flow." App Service can serve Protected Resource Metadata through `WEBSITE_AUTH_PRM_DEFAULT_WITH_SCOPES` (preview). [App Service MCP auth](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-mcp) (ms.date 2025-11-04)
5. Microsoft blog (Pamela Fox, 19 Jan 2026, updated 2 Feb 2026): "Microsoft Entra does support authorization server metadata, but it does not support either DCR or CIMD." FastMCP's DCR proxy over Entra is "intended only for development and testing"; "Microsoft recommends using pre-registered client applications." [Tech Community](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/using-on-behalf-of-flow-for-entra-based-mcp-servers/4486760) (Medium, first-party blog)
6. Community (Low): the mcp-remote README says "Microsoft Entra ID v2 answers `AADSTS9010010`" to the RFC 8707 `resource` parameter and offers `--disable-resource-parameter`. [mcp-remote README](https://github.com/geelen/mcp-remote). This conflicts with the MCP spec's "clients MUST send" resource. Not verified in Microsoft docs.

### 5.5 MCP authorization model (spec, High)
- Revisions: 2025-06-18, 2025-11-25, and the current **2026-07-28** (`/specification/latest` redirects there; the versioning page also lists 2026-09-28, not reviewed).
- 2025-06-18: servers MUST implement PRM (RFC 9728) and a `WWW-Authenticate` 401; AS MUST provide RFC 8414 metadata; **DCR SHOULD**; clients MUST send the RFC 8707 `resource`; PKCE MUST; audience validation MUST; **token passthrough forbidden**. [2025-06-18](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
- 2025-11-25: adds **CIMD (SHOULD)**, drops **DCR to MAY** ("backwards compatibility"), adds OIDC Discovery as an alternative to RFC 8414, requires the client to verify `code_challenge_methods_supported`, and sets the registration priority: pre-registered → CIMD → DCR → prompt the user. "MCP clients SHOULD support an option for static client credentials." Scope step-up via 403 `insufficient_scope`. [2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- 2026-07-28: keeps CIMD SHOULD, DCR MAY, PRM MUST and RFC 8707 MUST; adds client recording and validation of the AS `issuer`. [2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)

### 5.6 Alternatives if native remote MCP with Entra isn't available (descriptive only)
| Pattern | How it works | Per-user Entra? | Evidence |
|---|---|---|---|
| **Agent calls REST with a bearer token** | The agent or app calls QCo REST with `Authorization: Bearer <user token>`. With the xAI API this is function calling (app-executed). | Yes, if the app gets a per-user token; not documented for grok.com or Grok Bot | xAI function calling (High) |
| **Local stdio-to-remote proxy (mcp-remote)** | A local process speaks stdio to the client and HTTP to the remote server, and runs OAuth in a local browser. Supports `--header`/`--header-file`, `--static-oauth-client-info` (pre-registered client), `--resource`, `--disable-resource-parameter`, and `--transport http-first|sse-only`. | Yes (runs as the local user) | mcp-remote README (Low–Medium, third party). **Doesn't apply to grok.com or Grok Bot:** both are cloud-hosted and URL-only (forum 168188) |
| **API gateway (Azure API Management)** | APIM exposes REST APIs as MCP servers or fronts existing ones. It validates inbound Entra tokens (`validate-azure-ad-token`) and can inject backend tokens through credential manager (`get-authorization-context` + `set-header`). Credential-manager connections are permitted by access policies on Entra users, groups or service principals. | Inbound per-user validation yes; backend token per user depends on the connection design | [APIM secure MCP](https://learn.microsoft.com/en-us/azure/api-management/secure-mcp-servers), [credential manager](https://learn.microsoft.com/en-us/azure/api-management/credentials-overview), [process flow](https://learn.microsoft.com/en-us/azure/api-management/credentials-process-flow) (High) |
| **OAuth proxy / DCR shim in front of Entra** | The server or proxy acts as the AS, implements DCR, and forwards to Entra (FastMCP `AzureProvider`). Microsoft labels it dev/test only. | Yes | Microsoft Tech Community blog (Medium) |
| **xAI API remote MCP with per-request headers** | The app passes `authorization`/`headers` on every Responses call. | Yes, if the app provides a per-user token | xAI remote MCP doc (High) |
| **Cursor IDE `mcp.json` (not Grok Bot)** | The IDE supports static OAuth client plus headers. Staff pointed users there as the workaround. | Yes | cursor.com/docs/mcp (High); forum (Medium) |

---

## 6. Q4: Admin controls

### Grok Business / Enterprise (xAI console) (High)
- Connector provisioning: "a team admin must add a connector in the cloud console before team members can connect and use it… console.x.ai → Grok Business → Connectors"; custom MCP through "+ Add Connector → Other → MCP server URL → Complete any required authentication"; Configure ("access controls, allowed sites"); Remove (deletes indexed data). Requires **Team Read-Write**. [connector-management](https://docs.x.ai/grok/connector-management)
- Organizations (Enterprise only): domain association, SSO ("Okta, Azure AD, Google Workspace"), enforced org-wide; SCIM through sso.x.ai with IdP group → role → teams/ACLs/licenses; a preview step before activation. [organization](https://docs.x.ai/grok/organization)
- Business vs Enterprise (marketing): Enterprise adds custom SSO, SCIM, "advanced audit and security controls", and Vault (dedicated data plane, CMEK). [x.ai/news/grok-business](https://x.ai/news/grok-business) (Medium)
- **Gap:** no documented per-tool allowlist for grok.com custom connectors (unlike `allowed_tools` in the API). A community blog claims Grok has "no approval system… you can only disable tools completely" (david.coffee, Low).

### Grok Bot (Cursor dashboard) (High)
- Enterprise only: org-wide enable plus Manage Group Access, **MCP allowlist** (URL entry patterns plus per-server tool allowlists: "URL entries approve remote HTTP/SSE MCP servers by URL entry pattern"), Network Controls, Team Setup/Secrets, Enforce Auto-review plus rules, Action Recording (90-day), audit logs (including "MCP authentication" events), OTel export, SCIM. [security](https://cursor.com/docs/grok-bot/security), [teams](https://cursor.com/docs/grok-bot/teams), [cursor.com/docs/mcp](https://cursor.com/docs/mcp)
- Connector policy: "Grok Bot inherits your team's Cursor connector policy… Set which servers members can use from your Team Marketplace… Pushing connectors to members, whether mandatory or default-on, is not available."
- Identity: SAML 2.0 SSO with Entra, done through the **existing Cursor enterprise app**, not a separate Grok Bot app. Entra group assignment needs P1/P2 and doesn't include nested groups. A Conditional Access carve-out for the Linux Bot computer applies to in-browser app sign-in only, not plugins. [identity](https://cursor.com/docs/grok-bot/identity)
- Data: with Privacy Mode on, data isn't used for training; Bot computers run in the US; ZDR follows Cursor's provider agreements. [security](https://cursor.com/docs/grok-bot/security)

### Claude Cowork (brief confirmation, High)
- "Custom connectors using remote MCP are available on Claude, Cowork, and Claude Desktop… Free, Pro, Max, Team, and Enterprise." Org admins add them under Organization settings → Connectors → Add → Custom → Web, with optional "Advanced settings" for an OAuth Client ID and Secret. Connections come "from Anthropic's cloud infrastructure". [support.claude.com 11175166](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
- Auth types: `oauth_dcr`, `oauth_cimd` (out of box), `oauth_anthropic_creds`, `custom_connection`, `static_headers` (beta, admin-entered), `none`. "Supplying your own pre-registered client ID… avoids dynamic client registration entirely." [claude.com connectors authentication](https://claude.com/docs/connectors/building/authentication)

---

## 7. Community reports (all Low confidence)

| Date | Product | Source | Report |
|---|---|---|---|
| 2026-05-12 (open) | Grok | [GitHub makeplane/plane #9055](https://github.com/makeplane/plane/issues/9055) | Custom connector stuck on "Connecting…" after OAuth consent. With the PAT endpoint, "No prompt appeared for entering the API key or headers". The Plane maintainer suspects the Grok side. |
| ~2026-09-06 (secondhand) | Grok | Cited by [usecarly Jobber](https://www.usecarly.com/blog/grok-jobber-integration/) and [usecarly Lofty](https://www.usecarly.com/blog/grok-lofty-integration/) | Unraid MCP GitHub issue: DCR failed on Grok's callback address; "Grok's dialog only does OAuth with PKCE and dynamic registration, with no way to hand it a token". **The original issue was not located.** |
| undated | Grok | [david.coffee](https://david.coffee/grok-for-everything/) | "many MCP servers fail due to missing dynamic client registration"; "No approval system". |
| undated | Grok | [Zephex](https://zephex.dev/docs/editors/grok), [doctorSIM](https://www.doctorsim.com/agents/references/grok-bot.md) | Describe an "OAuth Credentials Required" screen that takes a static Client ID (PKCE, blank secret). doctorSIM: "Grok and Grok Bot decide auth once, while adding." |
| 2026-08-12 | Grok Bot | [forum 168188](https://forum.cursor.com/t/grok-bot-custom-remote-mcp-oauth-never-starts-fetch-failed-same-url-works-in-cursor-ide/168188) | OAuth "never started… fetch failed". Staff: the endpoint must be public; this is by design. |
| 2026-08-14 → fixed 2026-09-07 | Grok Bot | [forum 168350](https://forum.cursor.com/t/grok-bot-hung-custom-mcp-remotes-are-invisible-in-plugins-yours-and-uninstall-also-times-out-discovery-catch-22/168350) | One hung custom remote caused `DeadlineExceeded` for all connectors and couldn't be deleted. Staff gave a delete path through cursor.com/agents → + → MCP Servers → Delete user config. |
| 2026-08-30 | Grok Bot | [forum 169965](https://forum.cursor.com/t/grokbot-custom-connectors/169965) | No settings form for custom MCP; add it through chat (staff). |
| 2026-09-16 → 25 (open) | Grok Bot | [forum 171877](https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877) | DCR rejected by Cloudflare Access because of the `cursor://` redirect. Staff: OAuth only, no safe headers, fix tracked, no ETA. The same server worked with **Grok web** OAuth and read-only calls. |
| — | Both | X (Twitter) | Searched; **no relevant X posts found** (the search index may not cover X). |

---

## 8. Open verification items (gaps)

1. **Grok custom MCP + Entra (blocker).** Does the grok.com custom connector (consumer and Business "Other") accept a **pre-registered Entra client ID** (and secret)? What redirect URI does it use? Does it send RFC 8707 `resource`, and does Entra v2 accept it? Does it support CIMD? *Not documented.*
2. **Grok Business custom MCP token scope.** When an admin provisions a custom MCP in console.x.ai and "completes authentication", is that token shared org-wide, or does each member sign in on grok.com? *Not documented.*
3. **Grok header/API-key entry.** xAI mentions "API keys", but no UI is documented and one community report saw no prompt. *Unverified.*
4. **Grok Bot + Entra.** No documented static client-ID entry for Bot custom MCP. DCR is the only registration path described by staff, and Entra lacks DCR. The `cursor://` redirect bug is open. *Unverified by test.*
5. **Grok Bot team-managed connectors.** Docs mention "team-managed connectors… may use team or service-account credentials", but no page explains how to configure one for a custom MCP. *Gap.*
6. **Token pass-through.** Neither vendor documents forwarding the user's Entra SSO token to an MCP server. The MCP spec forbids passthrough, and Microsoft calls it a vulnerability. *Documented as not a spec pattern; vendor behavior unverified.*
7. **Entra `resource` parameter.** A community claim (mcp-remote) says Entra v2 rejects RFC 8707 `resource` (`AADSTS9010010`). *Not verified in Microsoft docs.*
8. **Strety auth model.** No official Strety API docs were found. Third parties show Strety's own OAuth (not Entra), so the Entra → Strety per-person token mapping is outside OBO. *Unverified.*
9. **Doc dates.** docs.x.ai and cursor.com/docs show no update dates. Re-check before build. The Grok Business and Grok Bot Enterprise launch posts are undated on the page.
10. **Per-tool controls on grok.com.** No documented per-tool allowlist for custom connectors. *Gap.*

---

## 9. Sources (retrieved 2026-09-29 unless noted)

**xAI (first-party)**
- Connectors: https://docs.x.ai/grok/connectors (no date shown)
- Connector Management (Business/Enterprise): https://docs.x.ai/grok/connector-management (no date shown)
- Custom MCP Server Tunneling: https://docs.x.ai/grok/connectors/custom-mcp-tunneling (no date shown)
- OneDrive connector: https://docs.x.ai/grok/connectors/onedrive (no date shown)
- SharePoint connector: https://docs.x.ai/grok/connectors/sharepoint (no date shown)
- Organization Management (SSO/SCIM): https://docs.x.ai/grok/organization (no date shown)
- Remote MCP Tools (API): https://docs.x.ai/developers/tools/remote-mcp (no date shown; samples use grok-4.7)
- Function Calling: https://docs.x.ai/developers/tools/function-calling (no date shown)
- Security FAQ (retention/ZDR): https://docs.x.ai/developers/faq/security (no date shown)
- xAI Docs MCP server (used for doc search; custom-connector OAuth queries returned "No results"): https://docs.x.ai/api/mcp
- Grok Bot overview: https://docs.x.ai/grok-bot/overview (no date shown)
- Grok Business/Enterprise announcement: https://x.ai/news/grok-business (undated on page)
- Grok Bot for Enterprise: https://x.ai/news/grok-bot-for-enterprise (undated on page)

**Cursor / Grok Bot (first-party)**
- Grok Bot for Teams and Enterprise: https://cursor.com/docs/grok-bot/teams
- Grok Bot security: https://cursor.com/docs/grok-bot/security
- Configure identity and access: https://cursor.com/docs/grok-bot/identity
- Work with Grok Bot: https://cursor.com/docs/grok-bot/work
- Cursor MCP (static OAuth, allowlist): https://cursor.com/docs/mcp
- Staff forum answers: https://forum.cursor.com/t/grokbot-custom-connectors/169965 (2026-08-30); https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877 (2026-09-16 to 09-25); https://forum.cursor.com/t/grok-bot-custom-remote-mcp-oauth-never-starts-fetch-failed-same-url-works-in-cursor-ide/168188 (2026-08-12); https://forum.cursor.com/t/grok-bot-hung-custom-mcp-remotes-are-invisible-in-plugins-yours-and-uninstall-also-times-out-discovery-catch-22/168350 (2026-08-14 to 09-11)

**Anthropic (first-party)**
- Custom connectors using remote MCP: https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
- Authentication for connectors: https://claude.com/docs/connectors/building/authentication

**Microsoft (first-party)**
- OBO flow: https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow (ms.date 2025-01-04; updated 2026-06-15)
- Expose a web API: https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-configure-app-expose-web-apis (ms.date 2025-05-14; updated 2026-06-15)
- App Service MCP server authorization: https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-mcp (ms.date 2025-11-04)
- APIM secure MCP servers: https://learn.microsoft.com/en-us/azure/api-management/secure-mcp-servers
- APIM credential manager: https://learn.microsoft.com/en-us/azure/api-management/credentials-overview ; https://learn.microsoft.com/en-us/azure/api-management/credentials-process-flow
- APIM MCP overview: https://learn.microsoft.com/en-us/azure/api-management/mcp-server-overview
- Tech Community, OBO for Entra-based MCP servers (2026-01-19, updated 2026-02-02): https://techcommunity.microsoft.com/blog/azuredevcommunityblog/using-on-behalf-of-flow-for-entra-based-mcp-servers/4486760

**MCP specification**
- 2025-06-18: https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization
- 2025-11-25: https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization
- 2026-07-28 (current "latest"): https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization

**Third party / community (Low unless noted)**
- mcp-remote: https://github.com/geelen/mcp-remote
- Plane issue #9055: https://github.com/makeplane/plane/issues/9055
- usecarly (Grok MCP, M365, Jobber, Lofty): https://www.usecarly.com/blog/grok-mcp/ ; https://www.usecarly.com/blog/grok-microsoft-365/ ; https://www.usecarly.com/blog/grok-jobber-integration/ ; https://www.usecarly.com/blog/grok-lofty-integration/
- Tempreon guide: https://tempreon.com/guides/add-mcp-server-to-grok
- PortEden: https://porteden.com/blog/grok-connectors/
- Zephex: https://zephex.dev/docs/editors/grok
- doctorSIM: https://www.doctorsim.com/agents/references/grok-bot.md
- Fat Heron: https://fatheron.dev/docs/agents/grok
- Nimply: https://developer.nimply.io/docs/integrations/grok
- david.coffee: https://david.coffee/grok-for-everything/
- Strety (third-party code): https://github.com/brentwpeterson/mcp-strety ; https://github.com/ajoshuasmith/n8n-nodes-strety ; https://pipedream.com/apps/strety
