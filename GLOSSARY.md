# Glossary

Every word in this repo, in plain English. Alphabetical.

| Word | Full name | Plain meaning |
|---|---|---|
| ABAC | Attribute-Based Access Control | Permissions decided by rules about facts ("same department, during work hours") |
| Access token | — | The token an app sends to an API. Says what the bearer may do |
| AD | Active Directory | Microsoft's company user directory |
| AES | Advanced Encryption Standard | Encryption with one shared key |
| API gateway | — | A front door that checks requests before they reach services |
| Assertion (SAML) | — | SAML's signed XML statement about who the user is |
| Audience (`aud`) | — | The claim saying which app/API a token is meant for |
| AuthN | Authentication | Proving who you are |
| AuthZ | Authorization | Deciding what you are allowed to do |
| Authorization Code flow | — | The OAuth flow where the browser gets a one-time code and the app swaps it for tokens |
| Authorization server (AS) | — | The server that issues tokens: Keycloak, Auth0, Okta |
| Authentication flow (Keycloak) | — | The steps Keycloak runs at login (password, then OTP, ...) |
| `azp` | Authorized party | Claim: which client requested the token |
| Base64 / Base64URL | — | A way to write bytes as text. **Not encryption** |
| Bearer token | — | Whoever holds it can use it, like cash |
| bcrypt / Argon2 | — | Deliberately slow password-hashing algorithms |
| BOLA | Broken Object Level Authorization | An API lets you access objects that aren't yours (the API version of IDOR) |
| CA | Certificate Authority | A trusted organisation that signs certificates |
| Certificate | X.509 certificate | A signed document binding a public key to a name like `example.com` |
| Claim | — | One fact inside a token, like `"email": "a@b.com"` |
| Client | — | An app registered with the authorization server |
| Client credentials flow | — | An app gets a token as itself, with no user involved |
| Client role (Keycloak) | — | A role that only means something to one client |
| Client scope (Keycloak) | — | A reusable bundle of claims and mappers you attach to clients |
| Client secret | — | A password for an app (confidential clients only) |
| Composite role | — | A role that includes other roles |
| Confidential client | — | An app on a server that can keep a secret |
| Consent | — | The "Allow this app to access ...?" screen |
| Cookie | — | A small value the browser stores and sends back to the same site automatically |
| CORS | Cross-Origin Resource Sharing | Browser rule about whether site A's JavaScript may read site B's response. **Not API security** |
| Credential stuffing | — | Trying username/password pairs leaked from other sites |
| CSP | Content Security Policy | A header that limits which scripts a page can run |
| CSRF | Cross-Site Request Forgery | Another site makes your browser send a request with your cookies |
| CVE | Common Vulnerabilities and Exposures | A public ID number for a known security bug |
| DAST | Dynamic Application Security Testing | Scanning a running app by attacking it |
| Device code flow | — | OAuth flow for TVs/CLIs: "go to this URL and type this code" |
| Digital signature | — | Proof made with a private key, checkable with the public key |
| Discovery document | — | JSON at `/.well-known/openid-configuration` listing all endpoints |
| DN | Distinguished Name | A full LDAP path to an entry, e.g. `cn=alice,ou=people,dc=corp,dc=com` |
| Encryption | — | Hiding data so only a key holder can read it |
| Entra ID | (formerly Azure AD) | Microsoft's cloud identity service |
| `exp` / `iat` / `nbf` | Expiry / Issued at / Not before | Time claims in a token, in seconds since 1970 |
| Grant | — | A way (recipe) to get a token: code, client credentials, refresh, ... |
| Group (Keycloak) | — | A set of users who share roles |
| Hash | — | One-way fingerprint of data |
| HMAC | Hash-based Message Authentication Code | A fingerprint that needs a secret key to create |
| `HttpOnly` / `Secure` / `SameSite` | Cookie flags | No JavaScript access / HTTPS only / limits on cross-site sending |
| HTTPS | HTTP over TLS | Encrypted HTTP |
| ID token | — | OIDC JWT that tells the *app* who logged in. Never send it to APIs |
| Identity brokering | — | Keycloak letting users log in via another provider (Google, Azure AD) |
| IDOR | Insecure Direct Object Reference | Changing an ID in a URL shows someone else's data |
| IdP | Identity Provider | The login server |
| Implicit flow | — | Old OAuth flow that put tokens in the URL. **Deprecated** |
| Introspection | — | Asking the authorization server whether a token is still valid |
| `iss` | Issuer | Claim: who created the token |
| JWE | JSON Web Encryption | An encrypted JWT (rare) |
| JWKS | JSON Web Key Set | The URL where an issuer publishes its public keys |
| JWS | JSON Web Signature | A signed JWT (the normal kind) |
| JWT | JSON Web Token | A signed, self-contained token: `header.payload.signature` |
| Kerberos / SPNEGO | — | The protocol behind Windows desktop single sign-on |
| Key rotation | — | Replacing keys regularly so a leaked key stops working |
| LDAP | Lightweight Directory Access Protocol | The protocol for querying a company directory |
| Least privilege | — | Give only the minimum access needed |
| Master realm | — | Keycloak's admin-only realm. Don't put apps there |
| MFA / 2FA | Multi-Factor / Two-Factor Authentication | Two different kinds of proof (password + phone) |
| mTLS | Mutual TLS | Both sides show certificates |
| Nonce | "number used once" | Random value that stops replay of an ID token |
| OAuth 2.0 | Open Authorization 2.0 | Standard for giving apps limited access without sharing passwords |
| OAuth 2.1 | — | Cleanup of OAuth 2.0: PKCE required, implicit and password grants removed |
| Offline token | — | A refresh token that survives logout, for long-running background access |
| OIDC | OpenID Connect | OAuth 2.0 plus a standard way to learn who the user is |
| Opaque token | — | A random string token; you must ask the issuer what it means |
| OP | OpenID Provider | OIDC's name for the IdP |
| Organizations | — | Keycloak/Auth0 feature for B2B: many customer companies in one realm/tenant |
| OTP / TOTP | One-Time Password / Time-based OTP | The 6-digit codes from an authenticator app |
| OWASP | Open Worldwide Application Security Project | Non-profit publishing free security guides and the Top 10 |
| Passkey / WebAuthn | Web Authentication | Passwordless login with device biometrics and public-key crypto |
| Password grant | Resource Owner Password Credentials | App collects the user's password and sends it for a token. **Deprecated** |
| Pentest | Penetration test | An authorized fake attack |
| PKCE | Proof Key for Code Exchange ("pixy") | Makes a stolen authorization code useless |
| Private key | — | Secret half of a key pair; signs and decrypts |
| Protocol mapper (Keycloak) | — | A rule that puts a specific claim into tokens |
| Public client | — | An app that can't keep secrets (browser, mobile) |
| Public key | — | Shareable half of a key pair; verifies and encrypts |
| RBAC | Role-Based Access Control | Permissions come from roles |
| ReBAC | Relationship-Based Access Control | Permissions come from relationships ("owner of this doc") |
| Realm (Keycloak) | — | An isolated set of users, clients, and settings |
| Realm role (Keycloak) | — | A role valid across the whole realm |
| Redirect URI | — | Where the authorization server sends the browser back |
| Refresh token | — | Used to get new access tokens without logging in again |
| Required action (Keycloak) | — | Something the user must do at next login (change password, set up OTP) |
| Resource server (RS) | — | The API that holds data and checks tokens |
| RP | Relying Party | An app that trusts an IdP for login |
| RSA / ECDSA | — | Algorithms for public/private key crypto |
| RS256 / ES256 / HS256 | — | JWT signing algorithms: RSA / elliptic curve / HMAC shared secret |
| Salt | — | Random data added to a password before hashing |
| SAML | Security Assertion Markup Language | Older XML-based single sign-on standard |
| SAST | Static Application Security Testing | Scanning source code for security bugs |
| Scope | — | Permission label on a token, e.g. `read:photos`, `openid`, `email` |
| SCIM | System for Cross-domain Identity Management | Standard API for creating/removing users in other apps |
| Service account (Keycloak) | — | The "robot user" behind a client using client credentials |
| Service mesh | — | Infrastructure that secures service-to-service traffic automatically |
| Session | — | The server's memory of a logged-in visitor |
| SIEM | Security Information and Event Management | Central place for security logs and alerts |
| SLO | Single Logout | Logging out of all apps at once |
| SP | Service Provider | SAML's name for an app |
| SPI | Service Provider Interface | Keycloak's plug-in points for custom Java code |
| SQLi | SQL Injection | User input becomes part of a database command |
| SSO | Single Sign-On | Log in once, use many apps |
| SSRF | Server-Side Request Forgery | Your server is tricked into calling internal URLs |
| `state` | — | Random value in the OAuth redirect that blocks CSRF on the callback |
| STRIDE | Spoofing, Tampering, Repudiation, Info disclosure, DoS, Elevation | A checklist of threat types |
| `sub` | Subject | Claim: the permanent ID of the user (or service account) |
| Tenant (Auth0) | — | Auth0's version of a realm |
| TLS | Transport Layer Security | Encryption for data in transit; the "S" in HTTPS |
| Token exchange | RFC 8693 | Swapping one token for another (different audience/scope) |
| UMA | User-Managed Access | OAuth extension for sharing your resources with others; used by Keycloak Authorization Services |
| Userinfo endpoint | — | OIDC endpoint returning the logged-in user's profile |
| XSS | Cross-Site Scripting | Attacker's JavaScript runs in your site |
| Zero trust | — | Never trust a request just because it's "inside the network" |
