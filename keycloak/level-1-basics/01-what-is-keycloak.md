# Module 01: What is Keycloak, and why does it exist?

**You'll learn:** the problem Keycloak solves, the few web and security words you need (HTTP, headers, `401`/`403`, tokens, JWT, OAuth 2.0, OpenID Connect, SSO), and how one Keycloak serves apps written in many languages.

**Before this:** nothing. This is the first module.

**Starting state:** nothing needs to exist yet. You will install Keycloak in [Module 02](02-run-and-tour.md).

---

## The idea in simple words

Imagine a company with ten office buildings. In a bad company, **every building has its own security guard with its own list of names**. You need a different badge for each building. When you leave the company, someone has to remember to cross your name off ten lists. Usually they forget one.

In a good company, there is **one front desk**. You show your passport there once. The desk gives you a **badge**. Every building's door just looks at the badge: "Is this a real company badge? Is it still valid? Does it say this person may enter?" When you leave, the front desk stops giving you badges. Done.

Software has the same problem:

- **Every app builds its own login.** Ten apps, ten login pages, ten password tables, ten chances to get it wrong.
- **Passwords everywhere.** Each app stores passwords. One leak hurts everyone.
- **No single sign-on.** You log in again and again, once per app.
- **Leavers keep access.** Someone leaves the company, but their account in app number seven is still active.

**Keycloak is the front desk.** It is one server that:

1. shows the login page,
2. checks passwords (and extra checks like a phone code),
3. hands out **tokens** (the badges) to apps,
4. keeps the one list of users, their groups and their roles.

Your apps stop handling passwords. They only check badges.

The fancy name for "front desk" is **identity provider** (IdP).

---

## The big picture

```mermaid
flowchart LR
  U[Person with a browser]
  APP[Web app<br/>Python]
  KC[Keycloak<br/>the front desk]
  API1[Notes API<br/>Python]
  API2[Billing API<br/>Java]
  API3[Search API<br/>Go]

  U -- 1. opens app --> APP
  APP -- 2. sends person to log in --> KC
  U -- 3. types password at Keycloak only --> KC
  KC -- 4. gives a token --> APP
  APP -- 5. calls API with token --> API1
  APP -- 5. --> API2
  APP -- 5. --> API3
  API1 -. 6. checks token with Keycloak's public key .-> KC
```

Read it as a story:

1. Alice opens the web app.
2. The app does **not** show its own login form. It sends Alice's browser to Keycloak.
3. Alice types her password **into Keycloak**, never into the app.
4. Keycloak sends the app a **token**: a signed note saying "this is Alice, she has role `admin`, valid for 5 minutes".
5. The app calls APIs and attaches the token to each call.
6. Each API checks the token's signature with Keycloak's public key, then decides: allowed or not.

---

## Just-enough background

You don't need to know any of this yet. Here is the minimum, in plain words.

### HTTP: letters between computers

When your browser (or a Python program) talks to a server, it sends a **request** and gets back a **response**. Think of letters:

- The **request** says what you want: `GET /notes` ("give me the notes").
- Each letter has **headers**: labels on the envelope, like `Content-Type: application/json`.
- The **response** has a **status code**, a number that says how it went: `200` OK, `302` "go to this other address", `404` not found.

Tokens travel in one special header:

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

"Bearer" means: "whoever carries this token gets in", like cash.

### `401` vs `403`

These two cause endless confusion:

| Code | Official name | What it really means | Front-desk version |
|---|---|---|---|
| `401` | Unauthorized | "I don't know who you are." No token, or a bad/expired one. | You have no badge. |
| `403` | Forbidden | "I know who you are, and you're not allowed." | Your badge is real, but it doesn't open this door. |

You can see a real `401` from Keycloak itself (after Module 02 is running). Ask its admin API for the list of realms without a token:

```bash
curl -s -w "\nHTTP status: %{http_code}\n" http://localhost:8180/admin/realms
```

```
{"error":"HTTP 401 Unauthorized"}
HTTP status: 401
```

No badge, so: "I don't know who you are."

### What is a token?

A **token** is a piece of text that proves something. Keycloak's tokens are like **a signed ID card**: everything is written on the card itself, and it carries a stamp (a **signature**) that only Keycloak can make. Any app can check the stamp without calling Keycloak every time.

### What does a JWT look like?

Keycloak's tokens are **JWTs** (JSON Web Tokens, said "jot"). A JWT is three chunks of text joined by dots:

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhbGljZSJ9.Sflk...signature
      header               payload           signature
```

| Part | What's in it |
|---|---|
| Header | Which algorithm signed it, and which key (`kid`) |
| Payload | The facts ("claims"): who the user is, their roles, when it expires |
| Signature | The stamp. Change one letter in the payload and the stamp no longer matches |

The first two parts are only **Base64-encoded**, which is a way of writing data with safe letters. It is **not secret**. Anyone can read a JWT. The signature only stops people from **changing** it. You'll decode real ones in [Module 06](06-tokens.md).

### OAuth 2.0: the hotel key card

You check into a hotel. You show your passport **once**, at the front desk. The desk gives you a **key card** that opens **only your room**, **only until checkout**. You never hand your passport to the room door.

**OAuth 2.0** is the rulebook for this. The app (the room door) never sees your password (your passport). It gets a limited **access token** (the key card) from the front desk (Keycloak).

### OpenID Connect: OAuth + "who you are"

OAuth alone only says "this card opens room 412". It doesn't say **who** you are. **OpenID Connect** (OIDC) is a small layer on top of OAuth that adds an **ID token**: a card that says "this is Alice Smith, alice@example.com". When people say "log in with Keycloak", they mean OpenID Connect.

### SSO: log in once

**Single sign-on (SSO)** means: you log in at Keycloak once, and every app that uses the same Keycloak lets you in without asking again. Keycloak remembers you with its own cookie, so the second app gets a token straight away.

---

## One Keycloak, many languages

Your company has services in Python, Java, Go, Node and more. They can all use the same Keycloak because Keycloak only speaks **open standards**:

- **HTTP** to log people in and hand out tokens,
- **JWT** for the tokens,
- **JWKS**, a public web page listing Keycloak's public keys, to check signatures.

Every language has a library that can read a JWT and check its signature. A Python service and a Java service check the very same token in the same way. Keycloak doesn't care what language you wrote your service in.

Every Keycloak realm publishes a **discovery document**: a public JSON page listing all its addresses. Every OIDC provider has one. Here is Google's (a real, public one you can fetch right now):

```bash
curl -s https://accounts.google.com/.well-known/openid-configuration \
  | jq '{issuer, authorization_endpoint, token_endpoint, jwks_uri}'
```

```
{
  "issuer": "https://accounts.google.com",
  "authorization_endpoint": "https://accounts.google.com/o/oauth2/v2/auth",
  "token_endpoint": "https://oauth2.googleapis.com/token",
  "jwks_uri": "https://www.googleapis.com/oauth2/v3/certs"
}
```

Keycloak has exactly the same kind of page. Libraries read this page and configure themselves. That's why "switch from Google login to Keycloak" is often a one-line change.

---

## What Keycloak does, and what it does NOT do

| Keycloak DOES | Keycloak does NOT |
|---|---|
| Show login, registration and password-reset pages | Protect your API by itself |
| Store users, groups, roles | Know what "can edit invoice 42" means in your app |
| Check passwords, phone codes, passkeys | Stop a service that forgets to check tokens |
| Hand out signed tokens | Run your business rules |
| Let users log in with Google/GitHub/company directory | |
| Log people out everywhere | |

The most important line: **Keycloak does not protect your API.** It only hands out badges. If your Python service doesn't check the token, anybody can call it. You will write that check in [Module 06](06-tokens.md) and use it for real in Level 2.

---

## Keycloak vs Auth0 (and friends)

- **Keycloak** is free, open source, and you run it yourself (your servers, your data, your upgrades).
- **Auth0** (and Okta, Azure Entra ID, AWS Cognito) are paid cloud services: someone else runs the server; you pay per user.
- Both speak the same standards (OAuth 2.0, OpenID Connect, JWT), so what you learn here carries over.

---

## Jargon table

| Word | Plain meaning | Example |
|---|---|---|
| Identity provider (IdP) | The front desk that knows who people are and gives out badges | Keycloak, Google, Auth0 |
| HTTP request / response | The letter you send / the reply | `GET /notes` → `200 OK` |
| Header | A label on the letter's envelope | `Authorization: Bearer abc...` |
| Status code | A number saying how the request went | `200`, `401`, `403` |
| Authentication (AuthN) | Proving who you are | Typing your password |
| Authorization (AuthZ) | Deciding what you may do | "Only admins can delete" |
| Token | A piece of text that proves something | A JWT from Keycloak |
| JWT | A signed token in three dot-separated parts | `xxxxx.yyyyy.zzzzz` |
| Claim | One fact inside a JWT | `"email": "alice@example.com"` |
| Signature | The stamp that proves the token wasn't changed | Third part of a JWT |
| Bearer token | Whoever holds it gets in | `Authorization: Bearer ...` |
| OAuth 2.0 | Rules for getting a limited key card instead of sharing a password | Access tokens |
| OpenID Connect (OIDC) | OAuth plus "who the user is" | ID token, "Log in with..." |
| Access token | The key card an app shows to an API | Sent to the notes API |
| ID token | The card that says who logged in, for the app only | Shows "Hi Alice" |
| SSO | Log in once, get into all apps | Log into app A, app B lets you straight in |
| Discovery document | A public JSON page listing a provider's addresses | `/.well-known/openid-configuration` |
| JWKS | A public page with the provider's public keys | `jwks_uri` above |

---

## Hands-on

Only one small thing, since Keycloak is installed in the next module:

1. **Why:** see that "OIDC provider" is not magic; it's just public web pages.
   Run the Google discovery-document command above. Look at `jwks_uri`, then open that address in your browser. You'll see Google's public keys. Keycloak's look the same.

---

## The one trap

**Thinking "we use Keycloak, so our API is secure."** Keycloak gives out badges. It does not stand at your API's door. Each service must check every token it receives: the signature, the expiry, who issued it, and who it's for. A service that skips this check is wide open, no matter how good your Keycloak setup is.

---

## Check yourself

1. Name two problems you get when every app builds its own login.
2. A request comes in with no token. `401` or `403`?
3. Alice's token is valid but she calls an admin-only endpoint without the admin role. `401` or `403`?
4. Can anyone read the claims inside a JWT? What does the signature protect?
5. What does OpenID Connect add on top of OAuth 2.0?

<details>
<summary>Answers</summary>

1. Any two of: passwords stored in many places; many login pages to build and secure; no single sign-on; leavers keep access in apps someone forgot to update.
2. `401`: "I don't know who you are."
3. `403`: "I know who you are, and you're not allowed."
4. Yes, anyone can read them; they're only Base64. The signature protects against **changes**: if anyone edits a claim, verification fails.
5. Identity: the ID token and a standard way to learn who logged in (name, email, user ID).

</details>

---

## Try this

1. Fetch the discovery document of another provider, for example `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Find the same four fields.
2. Write down every app at your company that has its own login. That list is what Keycloak replaces.
3. In your browser, open DevTools → Network on any site you're logged into, and look for an `Authorization` header or a `Cookie` header on requests.

Next: [Module 02: Run Keycloak and take the tour](02-run-and-tour.md).
