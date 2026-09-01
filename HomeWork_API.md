[Odoo_API_Gym_Membership_Project.md](https://github.com/user-attachments/files/31702490/Odoo_API_Gym_Membership_Project.md)
# Odoo API & REST Integration — Gym Membership Project (End-to-End Guide)

## 1. Odoo's API Landscape

Odoo does **not** ship a native REST API by default. It offers two built-in protocols:

- **XML-RPC** — classic, most-documented integration method. External apps call any model's methods (`search`, `create`, `write`, `unlink`) via `execute_kw`, using XML payloads over HTTP.
- **JSON-RPC 2.0** — same capability as XML-RPC, but with JSON payloads. Easier to work with from most modern languages/frameworks, including C#/.NET.

**For true REST endpoints** (`GET /api/members`, `POST /api/checkin`, etc.), you have two options:
- Use a community/OCA module such as `base_rest`, or a REST API app from the Odoo App Store
- **Build your own custom REST endpoints** inside a custom Odoo module using `http.Controller`

For a portfolio project, **building your own custom module with REST controllers** is the strongest choice — it demonstrates real Odoo development skill, not just consuming existing endpoints.

---

## 2. Authentication Options

| Method | Description | Best for |
|---|---|---|
| **API Key** (Odoo 14+) | Generated per user under Settings → Users → API Keys | Server-to-server calls, simplest setup |
| **Session-based (cookie)** | Login via `/web/session/authenticate`, reuse session cookie | Browser-based/short-lived integrations |
| **OAuth2** | Available via community modules | "Proper" external app auth flows |

For a gym app talking to Odoo from a separate frontend/backend, **API key + JSON-RPC** (or a custom REST controller validating an API key header) is the realistic, production-style approach.

---

## 3. Data Model for a Gym Membership System

### Option A — Use Odoo's built-in Membership module
- Module: `membership`
- Key model: `membership.membership_line`
- Members are represented as `res.partner`
- Handles membership states: paid, free, invoiced, waiting, canceled, expired, non-member

### Option B — Build custom models (recommended for a portfolio project)
- `gym.member` — linked to `res.partner`
- `gym.membership.plan` — monthly/quarterly/annual, price, class credits
- `gym.subscription` — member ↔ plan, start/end date, status
- `gym.checkin` — member, timestamp, location/branch
- `gym.payment` — linked to `account.move` (Odoo's invoicing model)

Custom models show deeper Odoo development skill and give you full control over the API surface.

---

## 4. End-to-End Flow

1. **Member signs up** — external app sends a REST call → creates `res.partner` + `gym.member` in Odoo
2. **Member selects a plan** — external app fetches available plans via `GET`, then `POST`s a new subscription
3. **Payment** — Odoo generates an invoice via `account.move`; optionally integrate a payment gateway module
4. **Check-in** — external app (mobile app or QR scanner) `POST`s a check-in event tied to the member ID
5. **Renewal/expiry automation** — an Odoo scheduled action (cron job) checks subscription end dates and sends reminder emails via Odoo's mail system
6. **Reporting** — Power BI (or Odoo's native dashboards) pulls from these models: active members, churn, revenue by plan, attendance trends

---

## 5. Building the Custom REST Layer — Technical Steps

1. Create a custom Odoo module, e.g. `gym_membership`
2. Define models in `models.py` (as listed in Option B above)
3. Create `controllers.py` using `http.Controller`, defining routes such as:
   - `POST /api/gym/members`
   - `GET /api/gym/plans`
   - `POST /api/gym/checkin`
4. Register routes with:
   ```python
   @http.route('/api/gym/members', type='json', auth='api_key', methods=['POST'], csrf=False)
   ```
5. Return JSON responses manually — Odoo doesn't auto-serialize like typical REST frameworks, so you control the response shape yourself
6. Test endpoints with Postman before connecting any external app

---

## 6. Where a .NET Client Fits

The external-facing piece (member website, mobile backend, or admin tool) can be built in **ASP.NET Core**, calling Odoo's JSON-RPC endpoints or your custom REST controllers via `HttpClient`.

This makes a strong three-part portfolio project:
- **Odoo** — ERP backend, data model, business logic
- **ASP.NET Core** — client-facing app / API consumer
- **Power BI** — reporting and analytics layer on top

---

## 7. Suggested Learning Path

1. Odoo module structure basics (models, views, security)
2. XML-RPC basics — simplest way to prove connectivity first
3. Custom controllers + JSON-RPC/REST routes
4. Authentication (API keys)
5. Build the gym models + one working endpoint end-to-end
6. Connect an external client — Postman first, then a real app
7. Add automation (cron jobs for renewals) and reporting

---

*Next possible step: scaffold the actual module folder structure and a starter `controllers.py` for the check-in endpoint.*
