# Security Reference

Deep-dive checklist for anything that touches user input, auth, money, or data storage. Linked from `robust-coding-practices/SKILL.md` §4 and `code-review/SKILL.md` §2.

## Injection

- **SQL**: always use parameterized queries / prepared statements / an ORM's query builder. Never string-concatenate or f-string user input into a query, including for "just an internal admin tool."
- **Shell**: never pass untrusted input to `shell=True` (Python), `exec()`/`system()` (Node), or backticks. Use argument-array APIs (`subprocess.run([...])`) so the shell never re-interprets the string.
- **Path traversal**: when accepting a filename/path from user input, resolve it and verify it's still inside the intended base directory before using it (reject `../` sequences, symlink escapes).
- **Template injection**: never pass user input directly as a template string to be rendered/evaluated (e.g. Jinja2 `Template(user_input)`, or building HTML via string concat) — treat it as data, not template source.
- **LDAP/XML/NoSQL injection**: same principle as SQL — untrusted input needs the query-builder/parameterized path for whatever store you're using, not string concatenation.

## Output encoding

- Escape all untrusted data before rendering into HTML — rely on the templating engine's auto-escaping (Jinja2, ERB, JSX) rather than manual escaping, which is easy to get wrong or forget.
- Never use `innerHTML`, `dangerouslySetInnerHTML`, or `v-html` with unsanitized user content. If rich text is genuinely needed, sanitize with a vetted library (DOMPurify) first.
- Set correct `Content-Type` headers so browsers don't sniff and misinterpret uploaded content as HTML/JS.

## Secrets & credentials

- No hardcoded API keys, passwords, tokens, or connection strings in source — load from environment variables or a secrets manager.
- Add credential files (`.env`, `*.pem`, `credentials.json`) to `.gitignore` *before* they're ever committed, not after.
- Rotate anything that was accidentally committed — removing it from the latest commit doesn't remove it from git history.
- Don't log secrets, even at debug level — grep logging statements for anything that might include a token/password/full card number.

## Authentication & authorization

- Every new endpoint/resource access needs an explicit authz check — "the user is logged in" is not the same as "the user is allowed to access *this specific resource*." Check ownership/permission on the actual object ID (guards against IDOR — insecure direct object reference).
- Don't trust client-side checks (disabled buttons, hidden fields, client-side role checks) as the actual enforcement — always re-check server-side.
- Use constant-time comparison for secrets/tokens (`hmac.compare_digest`, not `==`) to avoid timing attacks.
- Session tokens: use secure, httpOnly, sameSite cookies where applicable; invalidate sessions on logout and password change.

## Cryptography

- Never write custom crypto — use vetted libraries.
- Passwords: `bcrypt`, `argon2`, or `scrypt` — never plain hashing (MD5/SHA1/SHA256 alone) and never reversible encryption.
- Use TLS for anything crossing a network boundary; don't disable certificate verification "for now" (it tends to stay disabled).
- JWTs: verify signature and expiry on every use; don't accept `alg: none`; don't put sensitive data in the payload (it's base64, not encrypted).

## Deserialization

- Never `pickle.loads()`, `eval()`, or `exec()` on untrusted data — these can execute arbitrary code.
- `yaml.load()` → use `yaml.safe_load()`.
- Validate the *shape* of deserialized data (schema validation) before trusting it, even for "safe" formats like JSON.

## Dependencies

- Before adding a new dependency for anything security-sensitive (auth, crypto, parsing untrusted input), do a quick sanity check: is it actively maintained, does it have known open CVEs, does it have reasonable adoption?
- Run the ecosystem's audit tool when practical (`npm audit`, `pip-audit`, `cargo audit`) and address high/critical findings before shipping.

## Production hygiene

- Debug mode, verbose stack traces, and admin/debug endpoints should be off/removed before anything is exposed to the internet — a stack trace can leak file paths, library versions, and internal logic.
- Default-open CORS (`Access-Control-Allow-Origin: *`) should be deliberate, not a leftover from local dev.
- Rate-limit anything that's expensive or security-sensitive (login attempts, password reset requests, expensive queries) to blunt brute-force and DoS.

## The one question to always ask

For any code path that touches user input, auth, money, or storage: *"What happens if this input is malicious, and what's the worst this code path could be tricked into doing?"* If you can't answer confidently, that's a sign more validation or a second look is needed before shipping.
