# Learning progress

Update this after every session: tick the phase, add the date, and write one line about what clicked (or didn't).

**Current phase:** 0, How the web works
**Backend language chosen:** _(Python/FastAPI or Java/Spring Boot, decide before P1)_

| Phase | Topic | Started | Finished | Exit artifact |
|---|---|---|---|---|
| 0 | How the web works | | | P1 |
| 1 | Cryptography basics | | | P2, P3 |
| 2 | Authentication | | | P4, P5 |
| 3 | Authorization | | | P6 |
| 4 | Tokens & JWT | | | P7 |
| 5 | OAuth 2.0 | | | P8, P9 |
| 6 | OpenID Connect & SSO | | | P10 |
| 7 | Keycloak | | | K1–K12, P11–P16 |
| 8 | Auth0 & other providers | | | P17 |
| 9 | Web application security | | | P18, P19 |
| 10 | API & service-to-service | | | P20 |
| 11 | Enterprise identity | | | P21 |
| 12 | Security operations | | | P22 |
| 13 | Capstones | | | P23 / P24 |

## Session log

| Date | What I did | What clicked | What's still confusing |
|---|---|---|---|
| | | | |

## Self-check questions

You're ready to move on from a phase when you can answer its questions out loud, without notes.

- **Phase 0:** What's the difference between `401` and `403`? How does the browser know to send a cookie?
- **Phase 1:** Why is Base64 not encryption? Why hash passwords with bcrypt instead of SHA-256?
- **Phase 2:** What does `HttpOnly` protect against? What are the three kinds of authentication factor?
- **Phase 3:** Why isn't hiding a button enough? What is IDOR?
- **Phase 4:** Which four things must you check when validating a JWT? Can anyone read a JWT's payload?
- **Phase 5:** Why does PKCE exist? Why is the password grant deprecated? What does `state` protect?
- **Phase 6:** ID token vs access token: who reads which? How does SSO skip the password prompt?
- **Phase 7:** Realm vs client? Public vs confidential client? Realm role vs client role? Why is `aud` important?
- **Phase 8:** What's the Auth0 word for a Keycloak realm? What stays your responsibility with a hosted IdP?
- **Phase 9:** Why doesn't CORS protect an API? How do you stop SQL injection?
- **Phase 10:** How does a service prove who it is with no user? What is token exchange for?
- **Phase 11:** What is LDAP? When would you still see SAML?
- **Phase 12:** Why must tokens never be logged? What does STRIDE stand for?
