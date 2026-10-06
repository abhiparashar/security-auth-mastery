# Projects

24 projects, smallest to biggest. Nothing is pre-built: **you build every one.**

**Rules:**
- Pick **one** backend language and stick with it. Good choices: **Python + FastAPI** (fast to write) or **Java + Spring Boot** (most common with Keycloak at companies).
- Each project lives in its own folder here (`projects/p01-http-explorer/`, ...) with a short `NOTES.md`: what you built, what broke, what you learned.
- **"Done when"** is the finish line. If you can't show it, the project isn't done.
- After building, always **attack your own project** (wrong token, expired token, other user's ID, no token) and write down what happened.

| Size | Meaning |
|---|---|
| S | 1–3 hours |
| M | 1–2 days |
| L | 3–7 days |
| XL | 2–4 weeks |

---

## Phase 0–1: Foundations

### P1: HTTP explorer (S, phase 0)
Build a tiny server with 3 routes: one returns HTML, one returns JSON, one sets a cookie and redirects.
- **Done when:** you have DevTools screenshots of each request and can explain every request and response header in `NOTES.md`. You can reproduce each request with `curl -v`.

### P2: Password hasher (S, phase 1)
A script that stores users' passwords with bcrypt or Argon2, and checks logins.
- **Done when:** two users with the same password have different stored hashes (show it); you've timed SHA-256 vs bcrypt for 1,000 guesses and written down why slow is good here.

### P3: Sign and verify (S, phase 1)
Generate an RSA or EC key pair with `openssl`. Sign a file with the private key, verify it with the public key, change one byte, verify again.
- **Done when:** you can show the verification failing after tampering, and explain why the public key can't be used to create a signature.

---

## Phase 2–4: Login, permissions, tokens (no libraries for the security parts)

### P4: Hand-made login (M, phase 2)
Register, login, logout, and a protected `/profile` page, using sessions and cookies. No auth library.
- **Done when:** cookies have `HttpOnly`, `Secure`, `SameSite`; logout really destroys the session (an old cookie stops working); 5 failed logins trigger a delay.

### P5: Add TOTP (S, phase 2)
Add authenticator-app 2FA to P4 (a QR code to scan, then a 6-digit code at login).
- **Done when:** you can log in with Google Authenticator or similar, and a wrong or old code is rejected.

### P6: Roles and IDOR hunt (M, phase 3)
Add `user` and `admin` roles plus "my orders" to P4. Then attack it with `curl`: call admin routes as a user, and read another user's order by changing the ID.
- **Done when:** every attack you tried returns `403` or `404`, and `NOTES.md` lists each attack, the bug you found (there will be at least one), and the fix.

### P7: JWT by hand (M, phase 4)
Build and verify JWTs *without a JWT library*: Base64URL-encode the header and payload, sign with HMAC-SHA256, verify. Then do RS256 with a JWT library and a JWKS file.
- **Done when:** you show (1) a valid token passing, (2) a modified payload failing, (3) an expired token failing, (4) a token with the wrong `aud` failing, (5) an `alg: none` token being rejected.

---

## Phase 5–6: OAuth 2.0 and OIDC

### P8: "Login with GitHub", by hand (M, phase 5)
Implement Authorization Code + PKCE against GitHub (or Google) **without an OAuth library**: build the redirect URL, handle the callback, check `state`, swap the code, call the API.
- **Done when:** login works end to end, and a tampered `state` is rejected. `NOTES.md` has a sequence diagram of your real requests.

### P9: Two services, client credentials (S, phase 5)
Service A gets a token with client credentials (from Keycloak, or any provider) and calls Service B, which validates it.
- **Done when:** B rejects calls with no token, an expired token, and a token from a different client.

### P10: SSO across two apps (M, phase 6)
Two separate apps (different ports) that both log in through one OIDC provider.
- **Done when:** after logging into app 1, opening app 2 logs you in with no password prompt; logging out is handled deliberately (write down what happens in each app).

---

## Phase 7: Keycloak (the core of this repo)

Do [`KEYCLOAK.md`](../KEYCLOAK.md) lessons K1–K12 first.

### P11: Keycloak in Docker Compose (S)
Keycloak + PostgreSQL in `docker-compose.yml`, with your realm auto-imported from JSON at startup.
- **Done when:** `docker compose down -v && docker compose up` gives you the same realm, clients, and roles every time, with zero clicking.

### P12: Protect a web app (M)
A web app that logs users in via Keycloak (Authorization Code + PKCE) using a standard OIDC library. Show the user's name and roles. Add logout.
- **Done when:** login, logout, and an expired session all behave correctly; a user without the `app-user` role sees a "not allowed" page.

### P13: Protect an API (M)
An `orders-api` that accepts **only** access tokens from Keycloak. Validate signature (via JWKS), `exp`, `iss`, and `aud`. `GET /orders` needs role `orders:read`; `POST /orders` needs `orders:write` (client roles).
- **Done when:** a test script proves 401 for no/expired/forged tokens, 401 for a token whose `aud` isn't `orders-api`, 403 for a missing role, and 200 otherwise. The web app from P12 calls this API.

### P14: Groups, MFA, and the login experience (M)
Create groups (`finance`, `engineering`) that carry roles. Require OTP for the `finance` group only (conditional authentication flow). Set a password policy and turn on brute-force detection. Apply a simple custom login theme.
- **Done when:** a finance user is forced to set up OTP and an engineering user isn't; 5 wrong passwords temporarily lock the account; the login page shows your logo.

### P15: Identity brokering and federation (M)
Add "Log in with GitHub/Google" to the realm. Run an OpenLDAP container and federate its users into Keycloak.
- **Done when:** an LDAP user and a GitHub user can both log into P12, and you can explain where each user's password actually lives.

### P16: Keycloak as code (M)
Automate the whole realm (clients, roles, groups, mappers, flows) with `kcadm.sh` scripts, a realm JSON import, or Terraform's Keycloak provider. Add a CI step that spins it up and runs P13's test script.
- **Done when:** a fresh environment is rebuilt from git in one command, and the API tests pass against it.

---

## Phase 8: Auth0

### P17: Same app, different provider (S–M)
Point P12 + P13 at an Auth0 free tenant instead of Keycloak.
- **Done when:** only configuration changed (show the diff), and `NOTES.md` has a Keycloak ↔ Auth0 comparison: concepts, cost, effort, and where each would fit at your company.

---

## Phase 9: Attacks

### P18: Hack OWASP Juice Shop (L)
Run Juice Shop in Docker and solve 15+ challenges across injection, XSS, broken access control, and broken authentication. Finish at least 20 PortSwigger Web Security Academy labs (do the JWT and OAuth topics).
- **Done when:** each challenge has a short write-up: what you did, why it worked, and how to fix it.

### P19: Security review of your own work (M)
Review P4–P13 with the OWASP ASVS (Level 1) checklist. Add security headers, CSRF protection, input validation, and dependency scanning.
- **Done when:** a findings table (issue → severity → fix → commit link), and every high-severity issue is fixed.

---

## Phase 10–12: Services, enterprise, operations

### P20: Secure service chain (L, phase 10)
`gateway → orders → payments`. A user calls the gateway; `orders` calls `payments` on the user's behalf using **token exchange** (or a service account where no user context is needed). Each service checks audience and roles. Put secrets in Vault (dev mode is fine).
- **Done when:** `payments` rejects a token meant for `orders`; no secret appears in git or logs; a diagram shows which token travels on which hop.

### P21: Enterprise bridge (M, phase 11)
Keycloak federating users from LDAP, with one OIDC app and one SAML app (e.g. a SAML test SP) both using SSO.
- **Done when:** one LDAP login gets you into both apps; disabling the user in LDAP blocks the next login; you can read and explain a SAML assertion.

### P22: Threat model and alerting (M, phase 12)
Write a STRIDE threat model for P20. Turn on Keycloak login and admin events and ship them somewhere searchable (e.g. Loki/Grafana or OpenSearch). Alert on brute-force attempts and on admin role changes.
- **Done when:** a scripted brute-force attempt triggers your alert, and the threat model lists 10+ threats with their mitigations.

---

## Phase 13: Capstones

### P23: Company identity platform (XL)
Everything together, run like real production:
- Keycloak in **production mode** (`start`), PostgreSQL, TLS, behind a reverse proxy, two nodes
- LDAP federation, Google/GitHub brokering, MFA for privileged groups, branded theme
- Three clients: web app (P12), API (P13), batch job (service account)
- Realm fully in git (P16), CI pipeline, health and metrics dashboards, login-event alerts (P22)
- A **runbook**: how to add a new app, rotate realm keys, rotate client secrets, upgrade Keycloak, and restore from backup
- **Done when:** a colleague can onboard a new app using only your runbook, and you can explain every claim in every token your system issues.

### P24: Break it, then fix it (L)
Build a deliberately vulnerable OAuth/OIDC app with 8+ real-world mistakes: no `state`, no PKCE, wildcard redirect URI, ID token sent to the API, no `aud` check, tokens in `localStorage` + an XSS bug, logged tokens, long-lived tokens with no revocation. Write an exploit for each, then fix each one.
- **Done when:** for every bug there's an exploit script that works on the `vulnerable` branch and fails on the `fixed` branch.

---

## Progress

| # | Project | Phase | Status |
|---|---|---|---|
| P1 | HTTP explorer | 0 | ⬜ |
| P2 | Password hasher | 1 | ⬜ |
| P3 | Sign and verify | 1 | ⬜ |
| P4 | Hand-made login | 2 | ⬜ |
| P5 | Add TOTP | 2 | ⬜ |
| P6 | Roles and IDOR hunt | 3 | ⬜ |
| P7 | JWT by hand | 4 | ⬜ |
| P8 | Login with GitHub, by hand | 5 | ⬜ |
| P9 | Two services, client credentials | 5 | ⬜ |
| P10 | SSO across two apps | 6 | ⬜ |
| P11 | Keycloak in Docker Compose | 7 | ⬜ |
| P12 | Protect a web app | 7 | ⬜ |
| P13 | Protect an API | 7 | ⬜ |
| P14 | Groups, MFA, login experience | 7 | ⬜ |
| P15 | Identity brokering and federation | 7 | ⬜ |
| P16 | Keycloak as code | 7 | ⬜ |
| P17 | Same app, Auth0 | 8 | ⬜ |
| P18 | Hack OWASP Juice Shop | 9 | ⬜ |
| P19 | Security review of your own work | 9 | ⬜ |
| P20 | Secure service chain | 10 | ⬜ |
| P21 | Enterprise bridge | 11 | ⬜ |
| P22 | Threat model and alerting | 12 | ⬜ |
| P23 | Company identity platform | 13 | ⬜ |
| P24 | Break it, then fix it | 13 | ⬜ |
