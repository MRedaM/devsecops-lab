# Day 1 Findings

Draft entries based on a manual read of the code. Confirm each one against the real Semgrep / Gitleaks output after the first pipeline run, and rewrite the "Why" in your own words.

## Finding 1
- Tool / rule: Semgrep (expected: a sequelize/SQL injection rule from p/owasp-top-ten)
- File and line: app/routes/login.ts:34
- What it claims: User-controlled `req.body.email` is concatenated into a raw SQL query.
- Verdict: intentional (Juice Shop), and a genuine vulnerability in any real app
- Why: Template string interpolates the email straight into `sequelize.query`, so `' OR 1=1--` bypasses login. Juice Shop ships this on purpose (loginAdminChallenge).
- Fix or exception: Use bind parameters (`replacements` / `bind`) instead of string interpolation. Fix would be a real change in a real app; here, record as intentional.
- OWASP category: A05 Injection

## Finding 2
- Tool / rule: Semgrep (expected: same SQL injection rule)
- File and line: app/routes/search.ts:23
- What it claims: `criteria` from the request is interpolated into a LIKE query.
- Verdict: intentional (Juice Shop)
- Why: Same pattern as Finding 1 (unionSqlInjectionChallenge).
- Fix or exception: Parameterize the query with `replacements`. Good candidate for the Task 5 fix-on-a-branch if you want a small, safe change.
- OWASP category: A05 Injection

## Finding 3
- Tool / rule: Gitleaks (expected: `private-key`)
- File and line: app/lib/insecurity.ts:21 (also app/infrastructure/terraform/networking.tf:171)
- What it claims: A hardcoded RSA private key is committed.
- Verdict: intentional (Juice Shop)
- Why: The key signs the app's JWTs and is public in the upstream repo; it protects nothing and exists so players can forge tokens. In a real repo this would be a true positive: rotate the key and move it to a vault.
- Fix or exception: Allowlist by path in `.gitleaks.toml` with a comment (see below), not a blanket ignore.
- OWASP category: A04 Cryptographic Failures

## Proposed `.gitleaks.toml` addition

Add only after you have seen the real Gitleaks output and confirmed these are the hits:

```toml
[[allowlist.regexes]]
description = "Juice Shop ships a deliberately public demo RSA key (JWT signing); not a real secret"
regex = '''-----BEGIN RSA PRIVATE KEY-----\\r?\\nMIICXAIBAAKBgQDNwqLEe9'''
```

## Self-check answers

Write your own answers here, out loud first, then in a sentence each (questions are in `day1-notes.md` section 6).
