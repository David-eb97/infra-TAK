# Federation Hub + Authentik — login runbook

This documents what actually mattered when FedHub “worked” vs when it looked broken. Use it after fresh deploy or when login loops.

## 0) Architecture (current)

- **Caddy** terminates TLS at `https://fedhub.<fqdn>` and **reverse_proxies** to the Fed Hub host (default HTTP **8080** when OAuth is on).
- **Auth** is **one path**: Fed Hub’s built-in OIDC → **Authentik** (OAuth2 provider slug usually `fedhub`). Caddy does **not** use `forward_auth` on the Fed Hub vhost (that used to create a second Authentik app and fight OAuth cookies).

## 1) What you are looking at

- FedHub’s web UI may show a **“Keycloak”** login. That is **FedHub’s OIDC UI** (vendor naming). It does **not** mean Keycloak is installed.
- Your IdP is **Authentik**. The flow should end up at an Authentik **authorize** URL after you use that UI.

## 2) One command to verify OAuth wiring (run on console VPS)

```bash
curl -skD - "https://fedhub.test8.taktical.net/api/oauth/login/auth?force=true" -o /dev/null | grep -iE '^HTTP/|^location:'
```

**Good:**

- `HTTP/2 302` (or `HTTP/1.1 302`)
- `location:` contains `application/o/authorize`
- `redirect_uri=` matches what FedHub has in `federation-hub-ui.yml` (usually `https://fedhub.<fqdn>/api/oauth/login/redirect`)

**Bad:**

- `4xx` / no `Location`
- `redirect_uri` in the Location header does **not** match FedHub config or Authentik provider redirect URIs → fix Authentik OAuth2 provider **redirect_uris** and FedHub `keycloak*RedirectUri` lines together.

## 3) Authentik OAuth2 provider redirect URIs (FedHub app)

For public HTTPS on 443, **strict** Authentik matching should include (code now sets these on create/patch):

- `https://fedhub.<fqdn>/api/oauth/login/redirect`
- `https://fedhub.<fqdn>:443/api/oauth/login/redirect`
- Legacy: `https://fedhub.<fqdn>/login/redirect` and `:443` variant (older Fed Hub builds)

(Older wrong value was `/login/redirect` only without `/api/oauth/` — that breaks the callback.)

## 4) FedHub `federation-hub-ui.yml` essentials (remote host)

Typical required keys when using Authentik as OIDC:

- `allowOauth: true`
- `keycloakServerName: https://tak.<fqdn>` (or your Authentik public host — must match cert you pin)
- `keycloakTlsCertFile: /opt/tak/certs/keycloak.der` (DER of **that** host’s TLS cert)
- `keycloakClientId` / `keycloakSecret` from Authentik OAuth2 provider
- `keycloakRedirectUri` and `keycloakrRedirectUri` → `https://fedhub.<fqdn>/api/oauth/login/redirect`
- `keycloakConfigurationEndpoint` → `https://tak.<fqdn>/application/o/fedhub/.well-known/openid-configuration` (slug must match provider slug, usually `fedhub`)

**YAML gotcha:** Before appending lines with `tee -a`, ensure the file ends with a newline or keys can concatenate on one line and Spring will fail to parse YAML.

**Directory gotcha:** `/opt/tak/certs` must exist before writing `keycloak.der` (fresh VPS often has no `/opt/tak/certs`).

## 5) Caddy upstream port

- `web_ui_port` in console settings must match a port FedHub is **actually listening on** (`ss -tlnp` on FedHub host).
- Changing upstream without confirming listeners causes **502**.

## 6) FedHub UI `:0` / `ERR_UNSAFE_PORT` (browser)

If the SPA calls `https://fedhub.<fqdn>:0/api/...`, set localStorage once on `https://fedhub.<fqdn>/login` (DevTools console), then reload:

```js
localStorage.clear();
localStorage.setItem("api_port", "443");
localStorage.setItem("configuration", JSON.stringify({
  roger_federation: { server: { protocol: "https:", name: "fedhub.<fqdn>", port: 443, basePath: "/api/" } }
}));
location.reload();
```

Replace `fedhub.<fqdn>` with your real hostname.

## 7) Claim test mode vs admin group mode

**Stable test (narrow):**

- `keycloakClaimName: preferred_username`
- `keycloakAdminClaimValue: webadmin`

**Production (group-based):**

- `keycloakClaimName: groups`
- `keycloakAdminClaimValue: authentik Admins`

Switch only after OIDC redirect and session are proven with the test mode.

## 8) Authentik apps

**Current:** only the **OAuth2/OIDC** provider + application for Fed Hub (slug `fedhub`, name “Federation Hub”) is required for UI login.

If you still see an old **Proxy** app “Federation Hub” / slug `federation-hub` from earlier installs, it is **unused** by Caddy now — you may remove it in Authentik to avoid confusion (optional).

## 9) Reliable user habit (reduces double prompts)

Open `https://tak.<fqdn>` (or your Authentik session primer) **before** `https://fedhub.<fqdn>` so the IdP session already exists.

---

If this runbook drifts from code, update **both** — the goal is one place operators can follow without guessing.
