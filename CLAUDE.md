# docs — Codebase Guide

## This Project

The Mintlify documentation site for Crovver. MDX pages covering the full product: introduction, quickstart, core concepts, integration guides, and SDK/API references for Node, PHP, and React.

**Framework:** Mintlify · **Format:** MDX · **Config:** `docs.json`

### Run Locally
```bash
# Install Mintlify CLI globally if not already installed
npm i -g mintlify

mintlify dev    # starts on http://localhost:3000
```

### Structure
```
docs.json             ← Mintlify config: nav structure, theme, logo, favicon
introduction.mdx      ← Product overview
quickstart.mdx        ← Getting started guide
development.mdx       ← Local development setup
concepts/             ← Core concepts: orgs, tenants, plans, subscriptions, entitlements
guides/               ← Integration walkthroughs
api-reference/        ← Full REST API reference (Public, Admin, Webhook surfaces)
node-sdk/             ← Node.js SDK reference
react-sdk/            ← React SDK reference
php-sdk/              ← PHP SDK reference
authentication/       ← API key types, auth flows
essentials/           ← Mintlify essentials (markdown features, etc.)
snippets/             ← Reusable MDX snippets
ai-tools/             ← AI/LLM tooling docs
images/               ← Static images
logo/                 ← Logo assets
static/               ← Other static assets
```

---

## Crovver Ecosystem

Crovver is a **subscription management layer** for SaaS products. It sits between a SaaS app and payment providers (Stripe, Khalti, eSewa), handling subscription state, feature entitlements, seat tracking, usage limits, and hosted checkout — so SaaS teams don't build billing themselves. Payment credentials are never stored in the database; they go through Infisical or Vault.

### Sub-Projects
| Folder | What it is | Port / Registry |
|--------|-----------|-----------------|
| `crovver-mvp` | API server + admin dashboard (Next.js 16) | 3000 |
| `crovver-portal` | Customer-facing billing portal (Next.js 15) | 3002 |
| `crovver-node` | Official Node.js/TypeScript SDK | npm: `crovver-node` |
| `crovver-react` | Official React SDK | npm: `crovver-react` |
| `crovver-php` | Official PHP 8.2+ SDK | Packagist: `crovver/crovver-php` |
| `docs` | Mintlify documentation site — **this project** | — |

### Core Data Model
| Entity | Description |
|--------|-------------|
| **Org** | A SaaS company using Crovver. Type `b2b` = workspace-based customers; `d2c` = individual users |
| **Tenant** | The billing unit — a workspace (B2B) or user (D2C). Identified via `external_tenant_id` |
| **Plan** | Pricing tier with `features` (boolean flags) and `limits` (numeric caps). Flat or seat-based |
| **Subscription** | Tenant ↔ Plan binding. Statuses: pending → trialing → active → past_due → canceled |
| **Entitlement** | `canAccess(tenantId, featureKey)` — checks plan features; trial counts as active |

### API Key Types
- `pk_live_` / `pk_test_` — public keys, safe for browser (React SDK)
- `sk_live_` / `sk_test_` — secret keys, backend only (Node SDK, PHP SDK)

### Checkout Flow
1. SaaS frontend calls `redirectToCheckout()` on the React SDK
2. React SDK mints a short-lived JWT via `POST /api/public/auth/checkout-token` on crovver-mvp
3. Browser redirects to `{portalUrl}/pricing?token={jwt}`
4. crovver-portal validates JWT, fetches plans, shows plan picker
5. User picks plan → portal calls `POST /api/public/checkout` → Stripe session created
6. Stripe webhook fires → crovver-mvp activates subscription → `useSubscription().isActive === true`

### Three API Surfaces on crovver-mvp
| Surface | Base path | Auth method | Called by |
|---------|-----------|-------------|-----------|
| Public SDK | `/api/public/*` | Bearer `sk_live_` key | crovver-node, crovver-php |
| Admin dashboard | `/api/admin/*` | Session cookie | Dashboard UI |
| Webhooks | `/api/webhooks/*` | HMAC signature | Stripe, Khalti, eSewa |
