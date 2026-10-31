# Day 1 Findings

Scan result (first run): 85 Semgrep findings: 40 error, 42 warning, 3 note.
GitHub shows Semgrep's own levels (error/warning/note) because the SARIF carries no security score, so Critical/High/Medium/Low all read 0.

## Finding 1
- Tool / rule: Semgrep `dockerfile.security.missing-user.missing-user` (error)
- File and line: app/Dockerfile (first build stage, lines 1-20; confirm the exact line on the alert)
- What it claims: The container runs as root because no `USER` instruction is set.
- Verdict: false positive in practice (low risk)
- Why: The Dockerfile is multi-stage. The `installer` stage has no `USER`, but it is discarded after the build. The shipped final stage sets `USER 65532` (line 39) on a distroless base.
- Fix or exception: Optional hardening: run the build stage as non-root too, which limits what a malicious npm install script can do (A03). Needs a Docker build to verify, so not done blind. No suppression added.
- OWASP category: A02 Security Misconfiguration (hardening angle: A03 Supply Chain Failures)

## Finding 2
- Tool / rule: Semgrep `javascript.express.security.audit.express-check-directory-listing.express-check-directory-listing` (error)
- File and line: app/server.ts:290 (`/ftp`); similar at :269 (`/infrastructure`) and :294 (`/.well-known`)
- What it claims: `serveIndex` exposes directory listings.
- Verdict: intentional (Juice Shop)
- Why: The `/ftp` listing is the deliberate directoryListingChallenge.
- Fix or exception: In a real app, remove `serveIndex` and serve only explicitly named files.
- OWASP category: A02 Security Misconfiguration

## Finding 3 (to complete)
- Tool / rule: Semgrep `javascript.express.security.audit.xss.pug.explicit-unescape.template-explicit-unescape`
- File and line: TODO: copy from the alert. My grep of app/views/*.pug found no `!{` or `!=` output syntax, so check the file and line before deciding.
- What it claims: A Pug template outputs data unescaped, which risks XSS.
- Verdict: TODO (true positive vs. false positive depends on whether the flagged line really outputs user data)
- Why: TODO
- Fix or exception: TODO
- OWASP category: A05 Injection (XSS)

## Correction to my earlier drafts
I predicted Semgrep would flag the SQL injection in app/routes/login.ts:34 and app/routes/search.ts:23. It did not, so those are not findings. The code is still vulnerable (string-interpolated queries); this is a SAST miss and a good example of why DAST (Day 3) exists.

## Gitleaks triage
Rules hit: generic-api-key, jwt, private-key. All were fake demo data and are allowlisted by path in `.gitleaks.toml`, each with a written reason:
- app/data/static/users.yml: seeded demo users with fake passwords
- app/frontend/src/app/app.guard.spec.ts: jwt.io example token in a unit test
- app/lib/insecurity.ts and the terraform/networking.tf copies: Juice Shop's public demo RSA key (rotate and vault a key like this in a real project)

## Self-check answers

Write your own answers here, out loud first, then a sentence each (questions are in `day1-notes.md` section 6).
