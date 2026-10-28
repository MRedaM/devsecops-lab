# Day 1 Notes: Shift-left, OWASP Top 10, Secrets and SAST

## 1. Shift-left in one paragraph

Fixing a vulnerability costs far less the earlier it is found. A secret caught in a pull request is a one-line change. The same secret found in production means rotating credentials, auditing logs and possibly an incident. DevSecOps puts automated security checks at every stage of the pipeline so developers get fast feedback, instead of a pen-test report weeks later.

| Stage | Security check | Tool (this lab) |
|---|---|---|
| Code / commit | Secrets scanning, pre-commit hooks | Gitleaks |
| Build | SAST (static analysis of source) | Semgrep |
| Build | SCA (vulnerable dependencies), SBOM | Trivy (Day 2) |
| Package | Container image scanning | Trivy (Day 2) |
| Test | DAST (attack the running app) | OWASP ZAP (Day 3) |
| Deploy | IaC scanning, policy enforcement | Checkov, Kyverno (Days 4-5) |
| Run | Monitoring, detection, response | Sentinel / SIEM (Day 6) |

**Interview line:** "Shift-left means moving security checks earlier into the developer workflow, automated and fast, so findings are cheap to fix. It does not replace later controls; it adds earlier ones."

## 2. OWASP Top 10 (2025 edition)

The 2025 list changed from 2021: SSRF merged into Broken Access Control, "Vulnerable and Outdated Components" became the broader Software Supply Chain Failures, and Mishandling of Exceptional Conditions is new. Know both editions exist, since many teams and tools still reference 2021.

| # | Category | What it means | Example | Pipeline control |
|---|---|---|---|---|
| A01 | Broken Access Control | Users can act outside their permissions (includes SSRF) | Changing `/orders/123` to `/orders/124` shows another user's order (IDOR) | Authorization tests, SAST rules, DAST |
| A02 | Security Misconfiguration | Insecure defaults or settings | Debug mode on, open storage bucket, default admin password | IaC scanning, CIS benchmarks, DAST |
| A03 | Software Supply Chain Failures | Compromised or vulnerable dependencies, build systems, registries | Malicious npm package; unpinned CI action | SCA, SBOM, pinned dependencies, signing |
| A04 | Cryptographic Failures | Weak or missing encryption | Passwords hashed with MD5; HTTP instead of HTTPS | SAST, TLS checks |
| A05 | Injection | Untrusted input interpreted as code or commands | `' OR 1=1--` in a login form (SQL injection); XSS | SAST (Semgrep), DAST (ZAP) |
| A06 | Insecure Design | Flaws in the design itself, not the code | Password reset with no rate limit or lockout | Threat modeling, design review |
| A07 | Authentication Failures | Weak login, session or credential handling | Credential stuffing works; session ID never expires | Config review, DAST, MFA |
| A08 | Software or Data Integrity Failures | Code or data trusted without verification | Unsigned updates; insecure deserialization | Signed artifacts, SAST |
| A09 | Security Logging and Alerting Failures | Attacks cannot be detected or investigated | Failed logins not logged; no alert on admin actions | Logging standards, SIEM (Day 6) |
| A10 | Mishandling of Exceptional Conditions | Errors and edge cases handled unsafely | Stack trace leaked to user; failing open after an exception | Code review, SAST, fuzzing |

**Exercise:** for each row, write one sentence in your own words in a personal notes file. You should be able to explain any category in 30 seconds in an interview.

## 3. Secrets scanning (Gitleaks)

Why it matters: secrets in Git history stay there even after deletion. Anyone with repo access, or a leaked clone, can read them.

Key points to know:
- **Detection** works from regex rules plus entropy checks on known key formats (AWS, GitHub, Slack, private keys).
- **Scan history, not only the latest commit** (`fetch-depth: 0` in the workflow).
- **A leaked secret must be rotated**, not just removed from the code.
- **Prevention** beats detection: a pre-commit hook, plus secrets stored in a vault and injected at runtime (Day 4).
- **False positives** go in the allowlist with a written reason, never silently ignored.

## 4. SAST (Semgrep)

SAST analyses source code without running it, matching patterns (for example "user input flows into a SQL query").

- Fast, runs on every pull request, points to the exact line.
- Weaknesses: false positives, cannot see runtime behaviour or configuration, language-specific.
- Semgrep rules are readable YAML, so teams can write custom rules for their own patterns (a strong "reusable component" story for the Carrier role).
- We run it in **report-only mode** today. Failing builds on findings comes after you know what the noise looks like.

## 5. Today's hands-on tasks

1. [ ] Complete the setup in `README.md` and get both jobs to run.
2. [ ] Open the Semgrep results (Security tab or the `semgrep-report` artifact). Count findings by severity.
3. [ ] Pick **three** Semgrep findings. For each, decide: true positive, false positive, or intentional (Juice Shop is vulnerable on purpose). Record it in `docs/findings.md` using the template below.
4. [ ] Read the Gitleaks output. For each finding, decide if it is a real secret. If it is a documented false positive, add an allowlist entry with a comment, and push.
5. [ ] Fix one true positive in a small way on a branch, open a pull request, and watch the pipeline re-run. This is the full shift-left loop.
6. [ ] Write down answers to the self-check questions below.

### Findings template (`docs/findings.md`)

```
## Finding 1
- Tool / rule:
- File and line:
- What it claims:
- Verdict: true positive | false positive | intentional (Juice Shop)
- Why:
- Fix or exception:
- OWASP category:
```

## 6. Self-check (answer out loud)

1. What does shift-left mean, and why is it cheaper?
2. Which OWASP Top 10 category does a SQL injection fall under, and which pipeline tools would catch it?
3. A developer commits an AWS key and deletes it in the next commit. Is the problem solved? What do you do?
4. What can SAST not find that DAST can?
5. Why did we scan with `fetch-depth: 0`?
6. Why is the pipeline set to `permissions: contents: read` by default?
7. What is the difference between an allowlist entry and ignoring a finding?
