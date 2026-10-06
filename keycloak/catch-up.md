# Catch-up: rebuild the Level 1 baseline in one paste

**What this is:** one copy-paste block that recreates everything Modules 03–05 build, so you can jump into any later module (Level 2, 3 or 4) without redoing Level 1 by hand.

**Use it on an EMPTY Keycloak only** (a brand-new container with a new volume). It is not idempotent: run it twice, or on a realm that already has some of these things, and you'll get "Conflict" / "already exists" errors and a half-finished setup. Start clean first:

```bash
docker rm -f keycloak && docker volume rm keycloak-data
docker run -d --name keycloak -p 8180:8080 \
  -v keycloak-data:/opt/keycloak/data \
  -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.8.0 start-dev
```

Wait for the `Keycloak 26.8.0 ... started` line (`docker logs -f keycloak`, about 10–20 seconds), then paste the block below into your terminal (zsh or bash).

---

## What you get

| Thing | Details | Built in |
|---|---|---|
| Realm `learning` | display name `Learning Realm`, forgot-password and remember-me on | [Module 03](level-1-basics/03-realms-and-users.md) |
| User `alice` / `alice123` | Alice Smith, alice@example.com, email verified, group `admins` | Modules 03, 05 |
| User `bob` / `bob123` | Bob Jones, bob@example.com, email verified, group `staff` | Modules 03, 05 |
| Client `learning-cli` | public, Direct access grants ON, Standard flow OFF (labs only; password grant is deprecated) | [Module 04](level-1-basics/04-clients.md) |
| Client `web-app` | confidential, secret `web-app-secret`, Standard flow ON, Direct access grants OFF, redirect `http://localhost:8000/auth/callback`, post-logout redirect `http://localhost:8000/`, web origin `http://localhost:8000`, PKCE S256 required | Module 04 |
| Client `notes-api` | confidential, all flows OFF (only receives tokens), client roles `notes:read`, `notes:write` | Modules 04, 05 |
| Realm roles | `admin`, `user` | [Module 05](level-1-basics/05-roles-and-groups.md) |
| Group `admins` | realm roles `admin` + `user`; `notes-api` roles `notes:read` + `notes:write` | Module 05 |
| Group `staff` | realm role `user`; `notes-api` role `notes:read` | Module 05 |

Not included (they were side experiments in Module 03, not part of the baseline): the test user `carol` and the "unmanaged attributes" user-profile setting.

---

## The block

```bash
unalias kcadm 2>/dev/null
# 0. Helpers: kcadm runs inside the container; log it in as the master admin
kcadm() { docker exec keycloak /opt/keycloak/bin/kcadm.sh "$@"; }
kcadm config credentials --server http://localhost:8080 --realm master --user admin --password admin

# 1. Realm (Module 03)
kcadm create realms -s realm=learning -s enabled=true -s displayName='Learning Realm' \
  -s resetPasswordAllowed=true -s rememberMe=true

# 2. Users (Module 03)
kcadm create users -r learning -s username=alice -s enabled=true \
  -s firstName=Alice -s lastName=Smith -s email=alice@example.com -s emailVerified=true
kcadm set-password -r learning --username alice --new-password alice123
kcadm create users -r learning -s username=bob -s enabled=true \
  -s firstName=Bob -s lastName=Jones -s email=bob@example.com -s emailVerified=true
kcadm set-password -r learning --username bob --new-password bob123

# 3. Clients (Module 04)
kcadm create clients -r learning -s clientId=learning-cli -s publicClient=true \
  -s directAccessGrantsEnabled=true -s standardFlowEnabled=false
kcadm create clients -r learning -s clientId=web-app -s publicClient=false \
  -s secret=web-app-secret -s standardFlowEnabled=true -s directAccessGrantsEnabled=false \
  -s 'redirectUris=["http://localhost:8000/auth/callback"]' \
  -s 'webOrigins=["http://localhost:8000"]' \
  -s 'attributes."post.logout.redirect.uris"=http://localhost:8000/' \
  -s 'attributes."pkce.code.challenge.method"=S256'
kcadm create clients -r learning -s clientId=notes-api -s publicClient=false \
  -s standardFlowEnabled=false -s directAccessGrantsEnabled=false \
  -s serviceAccountsEnabled=false -s implicitFlowEnabled=false

# 4. Roles (Module 05)
kcadm create roles -r learning -s name=admin -s 'description=Can manage everything'
kcadm create roles -r learning -s name=user -s 'description=A normal signed-in user'
NOTES_ID=$(kcadm get clients -r learning -q clientId=notes-api --fields id --format csv --noquotes)
kcadm create clients/$NOTES_ID/roles -r learning -s name=notes:read -s 'description=Read notes'
kcadm create clients/$NOTES_ID/roles -r learning -s name=notes:write -s 'description=Create and edit notes'

# 5. Groups and their roles (Module 05)
kcadm create groups -r learning -s name=admins
kcadm create groups -r learning -s name=staff
kcadm add-roles -r learning --gname admins --rolename admin --rolename user
kcadm add-roles -r learning --gname staff --rolename user
kcadm add-roles -r learning --gname admins --cclientid notes-api --rolename notes:read --rolename notes:write
kcadm add-roles -r learning --gname staff --cclientid notes-api --rolename notes:read

# 6. Put users in groups (Module 05)
ALICE_ID=$(kcadm get users -r learning -q username=alice -q exact=true --fields id --format csv --noquotes)
BOB_ID=$(kcadm get users -r learning -q username=bob -q exact=true --fields id --format csv --noquotes)
ADMINS_ID=$(kcadm get groups -r learning -q search=admins --fields id --format csv --noquotes)
STAFF_ID=$(kcadm get groups -r learning -q search=staff --fields id --format csv --noquotes)
kcadm update users/$ALICE_ID/groups/$ADMINS_ID -r learning -n
kcadm update users/$BOB_ID/groups/$STAFF_ID -r learning -n
echo "Level 1 baseline created."
```

It prints one `Created new ...` line per realm, user, client, role and group, then `Level 1 baseline created.` The IDs will differ on your machine. The `kcadm` here is a shell **function**, not an alias, so it also works inside scripts; the first line removes the Module 02 alias if you have it, because an alias with the same name would break the function definition.

---

## Check it worked

Paste this. It logs in alice and bob with `learning-cli`, prints their roles from the token, and shows the key `web-app` settings:

```bash
jwt_payload() {
  echo "$1" | jq -R 'split(".")[1] | gsub("-";"+") | gsub("_";"/")
    | . + ("=" * ((4 - length % 4) % 4)) | @base64d | fromjson'
}
for U in alice:alice123 bob:bob123; do
  TOKEN=$(curl -s -X POST http://localhost:8180/realms/learning/protocol/openid-connect/token \
    -d grant_type=password -d client_id=learning-cli \
    -d username=${U%%:*} -d password=${U#*:} | jq -r .access_token)
  jwt_payload "$TOKEN" | jq -c '{user: .preferred_username, realm_roles: .realm_access.roles, notes_api: .resource_access["notes-api"].roles}'
done
kcadm get clients -r learning -q clientId=web-app | jq -c '.[0] | {clientId, publicClient, secret, redirectUris, pkce: .attributes["pkce.code.challenge.method"]}'
```

Real output from a fresh Keycloak 26.8.0:

```
{"user":"alice","realm_roles":["default-roles-learning","offline_access","admin","uma_authorization","user"],"notes_api":["notes:read","notes:write"]}
{"user":"bob","realm_roles":["default-roles-learning","offline_access","uma_authorization","user"],"notes_api":["notes:read"]}
{"clientId":"web-app","publicClient":false,"secret":"web-app-secret","redirectUris":["http://localhost:8000/auth/callback"],"pkce":"S256"}
```

If you see exactly these roles, you're at the end of Level 1. Continue with Level 2 from the [course map](README.md).

**Something failed?** The usual causes: Keycloak wasn't finished starting (wait and retry on a fresh volume), the realm already existed (reset first), or `jq` isn't installed (`brew install jq`).
