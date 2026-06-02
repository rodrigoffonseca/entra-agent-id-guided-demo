# Entra Agent ID — Interactive Guided Demo

An interactive, self-contained HTML webapp that walks you through creating, configuring, and governing [Microsoft Entra Agent ID](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/agent-identity-overview) artifacts step-by-step — directly in your own tenant.

> **Based on** the [entra-agent-id-preview-guide](https://github.com/astaykov/entra-agent-id-preview-guide) by Anton Staykov.

## What It Does

This demo provides a guided, point-and-click experience for setting up and governing Entra Agent ID, covering all key scenarios:

| Scenario | Description | Steps |
|----------|-------------|-------|
| **Autonomous Agent** | Agent operates under its own service principal identity (no user) | 01 → 02 → 03 → 05 |
| **Digital Colleague** | Agent has its own M365 identity (email, Teams, calendar) | 01 → 02 → 03 → 04 |
| **On-Behalf-Of** | Agent acts on behalf of a human user | 01 → 06 |
| **Third-Party Sidecar** | Containerized auth sidecar for non-Microsoft agents (AWS Bedrock, Ollama, n8n) | 01 → 07 |
| **Governance Audit** | Audit blueprints & identities for missing sponsors, bulk remediate, set up continuous alerts | 08 |
| **Access Package Governance** | Governed API permission assignment to agent identities via Entitlement Management | 01 → 02 → 09 |

Each step includes:
- 📝 **Plain-language explanation** of what the step does and why
- 🔗 **Exact Graph API call** with method, URL, and request body
- 📋 **Parameter breakdowns** explaining every field
- ▶️ **One-click execution** against your real tenant
- 📊 **Full response viewer** showing status codes and JSON output
- 📚 **Links to Microsoft documentation** for deeper reading

## Guided Demo Sections

### 01 — Agent Blueprint
Create the foundational application registration that serves as the template for all agent identities.

| Step | Description | API |
|------|-------------|-----|
| 01.01 | Create Agent Identity Blueprint | `POST /beta/applications/` with `@odata.type: AgentIdentityBlueprint` |
| 01.02 | Create Blueprint Service Principal | `POST /beta/serviceprincipals/graph.agentIdentityBlueprintPrincipal` |
| 01.03 | Add Blueprint Credential | Client Secret or Managed Identity FIC |
| 01.04 | Expose API Scope | `PATCH /beta/applications/{id}` to add `access_agent` scope |

### 02 — Agent Identity
Create a service principal linked to the blueprint for autonomous agent scenarios.

| Step | Description | API |
|------|-------------|-----|
| 02.01 | Create Agent Identity | `POST /beta/serviceprincipals/graph.agentIdentity` |

### 03 — Agentic User (Digital Colleague)
Create a user object that the AI agent will operate as — with email, Teams, and calendar.

| Step | Description | API |
|------|-------------|-----|
| 03.01 | Create Agentic User | `POST /beta/users` with `@odata.type: AgenticUser` |
| 03.02 | Grant Permissions | `POST /beta/oauth2PermissionGrants` for Graph scopes |

### 04 — Digital Colleague Authentication
Three-hop FIC token chain: Blueprint → Agent Identity → Agentic User.

| Step | Description | Flow |
|------|-------------|------|
| 04.01 | Blueprint FIC Token | `client_credentials` + `fmi_path` → FIC token |
| 04.02 | Agent Identity FIC Token | `client_assertion` with Blueprint FIC → Identity FIC token |
| 04.03 | Agent User Token | `user_fic` grant type → Agentic User token |
| 04.04 | Test /me Endpoint | `GET /v1.0/me` with agent user token |
| 04.05 | Get Member Groups | `POST /v1.0/me/getMemberGroups` |

### 05 — Autonomous Agent Authentication
Two-hop FIC chain: Blueprint → Agent Identity with client credentials for Graph.

| Step | Description | Flow |
|------|-------------|------|
| 05.01 | Blueprint FIC Token | Same as 04.01 |
| 05.02 | Agent Graph Token | `client_credentials` with Blueprint FIC → Graph token |

### 06 — On-Behalf-Of User
Standard OAuth 2.0 Authorization Code Flow with PKCE using the Blueprint's exposed scope.

| Step | Description | Flow |
|------|-------------|------|
| 06.01 | Auth Code Flow | PKCE-based authorization code → token redemption |

### 07 — Third-Party Sidecar
Deploy a containerized Microsoft Entra Auth SDK sidecar alongside third-party AI agents.

| Step | Description | Details |
|------|-------------|---------|
| 07.01 | Sidecar Architecture | Architecture overview and code examples (Python, Node.js, cURL) |
| 07.02 | Docker Compose Setup | Generate `docker-compose.yml` with sidecar + agent containers |
| 07.03 | Request Token via Sidecar | Test token acquisition through the sidecar endpoint |

### 08 — Governance Audit
Audit, remediate, and continuously monitor agent identity governance in your tenant.

| Step | Description | Details |
|------|-------------|---------|
| 08.01 | Blueprints Audit | List all Agent Blueprints, check for missing sponsors, download CSV report |
| 08.02 | Agent Identities Audit | List all Agent Identities, check for missing sponsors, download CSV report |
| 08.03 | KQL Alert Rule | KQL queries for Azure Monitor to alert on ungoverned agent objects |

### 09 — Access Package Governance
Create an Entitlement Management access package that assigns Graph API permissions to an autonomous agent identity, with the sponsor requesting on behalf of the agent.

| Step | Description | API |
|------|-------------|-----|
| 09.01 | Create Catalog | `POST /beta/identityGovernance/entitlementManagement/catalogs` |
| 09.02 | Add Graph Resource | `POST /beta/identityGovernance/entitlementManagement/resourceRequests` |
| 09.03 | Create Access Package | `POST /beta/identityGovernance/entitlementManagement/accessPackages` |
| 09.04 | Add API Permissions | `POST /beta/.../accessPackages/{id}/accessPackageResourceRoleScopes` |
| 09.05 | Assignment Policy | `POST /beta/identityGovernance/entitlementManagement/assignmentPolicies` |
| 09.06 | Assign to Agent | `POST /beta/identityGovernance/entitlementManagement/assignmentRequests` |

**Access Package features include:**
- **Sponsor-driven requests** — the agent's sponsor requests API permissions on behalf of the agent identity
- **Time-limited access** — 90-day default duration with renewal capability
- **Lifecycle governance** — automatic revocation when assignments expire
- **Audit trail** — full visibility into who requested what permissions and when
- **Separation of concerns** — API permissions for agents kept in a dedicated catalog, separate from human access

**Governance features include:**
- **Tenant-wide audit** — scans all blueprints and agent identities with automatic pagination
- **Sponsor verification** — checks each object individually for sponsor assignments
- **Visual dashboard** — summary cards showing total, missing, OK, and failed counts
- **CSV export** — download audit results with Excel-compatible encoding and formula injection protection
- **Bulk remediation** — assign a sponsor (user UPN or security group) to all flagged items in one click
- **API preview** — inspect the exact Graph API call before executing bulk updates
- **Continuous monitoring** — ready-to-use KQL queries for Azure Monitor alert rules with step-by-step setup instructions

## Identity Primitives Covered

- **Agent Identity Blueprint** — A special application registration (`@odata.type: Microsoft.Graph.AgentIdentityBlueprint`) that serves as the template for agent identities
- **Agent Identity** — A service principal linked to a blueprint, used for autonomous agent scenarios
- **Agentic User (Digital Colleague)** — A special User object with email, Teams, OneDrive, and calendar — driven by AI
- **Federated Identity Credentials (FIC)** — Trust chain that links Blueprint → Agent Identity → Agentic User without shared secrets

## Prerequisites

### Sections 01–08 (No additional licenses required)

1. **An Entra ID tenant** with the Agent ID preview enabled
2. **A user account** with the **Agent ID Administrator** directory role (or Global Administrator)
3. **A SPA app registration** in your tenant:
   - Platform: **Single-page application (SPA)**
   - Redirect URI: `http://localhost:8080/index.html`
   - Delegated permissions: `Application.ReadWrite.All`, `User.ReadWrite.All`, `DelegatedPermissionGrant.ReadWrite.All`
4. **For governance alerts (step 08.03):** A Log Analytics workspace with Entra ID Diagnostic Settings streaming Audit Logs

> **Note:** Sections 01 through 08 work with any Entra ID tenant that has the Agent ID preview enabled — no additional licenses beyond the base Entra ID Free/P1 are required.

### Section 09 — Access Package Governance (Additional license required)

5. **License:** One of the following is required for Entitlement Management:
   - **Microsoft Entra ID Governance** (standalone add-on)
   - **Microsoft Entra Suite**
   - **Microsoft 365 E7**
6. **Role:** The **Identity Governance Administrator** directory role (or Global Administrator)
7. **SPA permission:** Add the delegated permission `EntitlementManagement.ReadWrite.All` to your SPA app registration

## Quick Start

1. **Clone this repo:**
   ```bash
   git clone https://github.com/rfonseca_microsoft/entra-agent-id-guided-demo.git
   cd entra-agent-id-guided-demo
   ```

2. **Start a local HTTP server** (MSAL.js requires HTTP, not `file://`):
   ```bash
   # Python
   python -m http.server 8080

   # Node.js
   npx http-server -p 8080

   # Any other static file server on port 8080
   ```

3. **Open in your browser:**
   ```
   http://localhost:8080/index.html
   ```

4. **Enter your SPA Client ID** and click **Sign In**

5. **Follow the steps** — the app auto-detects your tenant ID, MS Graph SP, and verifies you have the required roles.

> **Note:** Steps 04–05 (FIC token chain) use `client_credentials` flows against the Entra token endpoint, which blocks browser CORS by design. The app generates **PowerShell** commands for you to run in a terminal — paste the JSON response back into the UI to continue.

## Features

- **Interactive sign-in** via MSAL.js v3 — authenticates against your own Entra tenant
- **Auto-detection** of Tenant ID and MS Graph Service Principal Object ID
- **Role verification** on sign-in — checks for Agent ID Administrator and required API permissions
- **Sponsor flexibility** — supports both individual users (UPN) and security groups as sponsors
- **Configurable display names** for Blueprint, Agent Identity, and Agent User
- **FIC token chain visualization** — explains each hop in the trust chain with clear diagrams
- **Blueprint authentication options** — both Client Secret (demo) and Managed Identity FIC (production)
- **Third-party sidecar** — Docker Compose generation for containerized auth alongside non-Microsoft agents
- **Governance audit** — tenant-wide sponsor audit with CSV export and bulk remediation
- **KQL alert queries** — ready-to-deploy Azure Monitor alerts for continuous governance monitoring
- **Progress tracking** — completed steps are marked with checkmarks
- **Config persistence** — all settings saved in localStorage across sessions

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Agent Blueprint │ ──→ │ Agent Identity  │ ──→ │  Agentic User   │
│  (Application)   │     │ (Svc Principal) │     │(Digital Colleague)│
└─────────────────┘     └─────────────────┘     └─────────────────┘
         ↑                                               │
┌─────────────────┐                              ┌───────▼───────┐
│ AI Application  │                              │   MS Graph    │
│  (Your AI App)  │                              │  (Resource)   │
└─────────────────┘                              └───────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Governance Layer                                               │
│  ┌──────────┐  ┌──────────────┐  ┌───────────────────────────┐  │
│  │  Audit   │→ │ Bulk Fix     │→ │ Azure Monitor KQL Alerts  │  │
│  │ (08.01/2)│  │ (Sponsors)   │  │ (Continuous Monitoring)   │  │
│  └──────────┘  └──────────────┘  └───────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Authentication Flows

The demo covers four distinct authentication patterns:

1. **Digital Colleague (Section 04):** 3-hop FIC chain → Blueprint FIC → Agent Identity FIC → Agent User token (`user_fic` grant type)
2. **Autonomous Agent (Section 05):** 2-hop FIC chain → Blueprint FIC → Agent Identity Graph token (`client_credentials`)
3. **On-Behalf-Of (Section 06):** Standard OAuth Authorization Code Flow with PKCE using the Blueprint's `access_agent` scope
4. **Third-Party Sidecar (Section 07):** Containerized auth sidecar handles all credential management — agents request tokens via local HTTP

## Tech Stack

- **Single HTML file** — no build step, no dependencies to install
- **MSAL.js v3** (loaded from CDN) for interactive Entra authentication
- **Microsoft Graph Beta API** for all agent identity operations
- **Pure CSS** — no frameworks, dark-friendly design

## Microsoft Documentation References

- [Agent Identity Overview](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/agent-identity-overview)
- [Federated Identity Credentials](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/federated-identity-credentials)
- [Workload Identity Federation](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation)
- [Microsoft Graph API — Applications](https://learn.microsoft.com/en-us/graph/api/application-post-applications)
- [Client Credentials Flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-client-creds-grant-flow)
- [On-Behalf-Of Flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow)
- [Integrate Entra Logs with Log Analytics](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-integrate-activity-logs-with-azure-monitor-logs)
- [Create Log Alert Rules](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-create-log-alert-rule)
- [Entitlement Management Access Packages](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create)
- [Add API Permissions to Access Package](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-resources#add-an-api-permission)
- [Tutorial: Manage Access Packages via Graph API](https://learn.microsoft.com/en-us/graph/tutorial-access-package-api)

## Disclaimer

This tool is intended for **demo and proof-of-concept purposes**. It executes real Graph API calls against your tenant. Use in a test/dev tenant. Client secrets created during the demo should be rotated or deleted after use.

## License

MIT

## Credits

- Based on the guide by [Anton Staykov](https://github.com/astaykov/entra-agent-id-preview-guide)
- Built with [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js) by Microsoft
