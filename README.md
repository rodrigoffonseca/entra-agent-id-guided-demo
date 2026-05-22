# Entra Agent ID — Interactive Guided Demo

An interactive, self-contained HTML webapp that walks you through creating and configuring [Microsoft Entra Agent ID](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/agent-identity-overview) artifacts step-by-step — directly in your own tenant.

> **Based on** the [entra-agent-id-preview-guide](https://github.com/astaykov/entra-agent-id-preview-guide) by Anton Staykov.

## What It Does

This demo provides a guided, point-and-click experience for setting up Entra Agent ID, covering all key scenarios:

| Scenario | Description | Steps |
|----------|-------------|-------|
| **Autonomous Agent** | Agent operates under its own service principal identity (no user) | 01 → 02 → 03 → 05 |
| **Digital Colleague** | Agent has its own M365 identity (email, Teams, calendar) | 01 → 02 → 03 → 04 |
| **On-Behalf-Of** | Agent acts on behalf of a human user | 01 → 06 |

Each step includes:
- 📝 **Plain-language explanation** of what the step does and why
- 🔗 **Exact Graph API call** with method, URL, and request body
- 📋 **Parameter breakdowns** explaining every field
- ▶️ **One-click execution** against your real tenant
- 📊 **Full response viewer** showing status codes and JSON output
- 📚 **Links to Microsoft documentation** for deeper reading

## Identity Primitives Covered

- **Agent Identity Blueprint** — A special application registration (`@odata.type: Microsoft.Graph.AgentIdentityBlueprint`) that serves as the template for agent identities
- **Agent Identity** — A service principal linked to a blueprint, used for autonomous agent scenarios
- **Agentic User (Digital Colleague)** — A special User object with email, Teams, OneDrive, and calendar — driven by AI
- **Federated Identity Credentials (FIC)** — Trust chain that links Blueprint → Agent Identity → Agentic User without shared secrets

## Prerequisites

1. **An Entra ID tenant** with the Agent ID preview enabled
2. **A user account** with the **Agent ID Administrator** directory role (or Global Administrator)
3. **A SPA app registration** in your tenant:
   - Platform: **Single-page application (SPA)**
   - Redirect URI: `http://localhost:8080/index.html`
   - Delegated permissions: `Application.ReadWrite.All`, `User.ReadWrite.All`, `DelegatedPermissionGrant.ReadWrite.All`

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

## Features

- **Interactive sign-in** via MSAL.js v3 — authenticates against your own Entra tenant
- **Auto-detection** of Tenant ID and MS Graph Service Principal Object ID
- **Role verification** on sign-in — checks for Agent ID Administrator and required API permissions
- **Sponsor flexibility** — supports both individual users (UPN) and security groups as sponsors
- **Configurable display names** for Blueprint, Agent Identity, and Agent User
- **FIC token chain visualization** — explains each hop in the trust chain with clear diagrams
- **Blueprint authentication options** — both Client Secret (demo) and Managed Identity FIC (production)
- **Progress tracking** — completed steps are marked with checkmarks
- **Config persistence** — all settings saved in localStorage across sessions

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Agent Blueprint │ ──→ │ Agent Identity  │ ──→ │  Agentic User   │
│  (Application)   │     │ (Svc Principal) │     │(Digital Colleague)│
└─────────────────┘     └─────────────────┘     └─────────────────┘
         ↑
┌─────────────────┐
│ AI Application  │  acts on-behalf-of user via Blueprint scope
│  (Your AI App)  │
└─────────────────┘
```

## Authentication Flows

The demo covers three distinct authentication patterns:

1. **Digital Colleague (Section 04):** 3-hop FIC chain → Blueprint FIC → Agent Identity FIC → Agent User token (`user_fic` grant type)
2. **Autonomous Agent (Section 05):** 2-hop FIC chain → Blueprint FIC → Agent Identity Graph token (`client_credentials`)
3. **On-Behalf-Of (Section 06):** Standard OAuth Authorization Code Flow with the Blueprint's `access_agent` scope

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

## Disclaimer

This tool is intended for **demo and proof-of-concept purposes**. It executes real Graph API calls against your tenant. Use in a test/dev tenant. Client secrets created during the demo should be rotated or deleted after use.

## License

MIT

## Credits

- Based on the guide by [Anton Staykov](https://github.com/astaykov/entra-agent-id-preview-guide)
- Built with [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js) by Microsoft
