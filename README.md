# DevSecOps Lab

A one-week hands-on lab: a deliberately vulnerable app ([OWASP Juice Shop](https://github.com/juice-shop/juice-shop)) run through a progressively more secure CI/CD pipeline.

| Day | Adds |
|---|---|
| 1 | Secrets scanning (Gitleaks) + SAST (Semgrep) in GitHub Actions |
| 2 | Docker, SCA + image scanning (Trivy), SBOM, quality gates |
| 3 | DAST (OWASP ZAP), port to Azure DevOps |
| 4 | Terraform + Checkov, secrets management |
| 5 | Kubernetes security (kind, RBAC, NetworkPolicy, Kyverno) |
| 6 | Detection, incident runbook, compliance mapping |
| 7 | Reusable templates + playbook |

## Layout

```
.
├── app/                         # Juice Shop source (the vulnerable target)
├── .github/workflows/
│   └── devsecops.yml            # the pipeline
├── .gitleaks.toml               # secrets scanning config + documented exceptions
├── .semgrepignore
├── docs/
│   ├── day1-notes.md            # study notes + triage log
│   └── findings.md              # your triage decisions (you create this)
└── README.md
```

## Day 1 setup (about 20 minutes)

1. Create a new **public** GitHub repo named `devsecops-lab` (public keeps Actions and code scanning free).
2. On your computer, unzip this scaffold into a folder and `cd` into it.
3. Pull Juice Shop in as the target app:
   ```bash
   git clone --depth 1 https://github.com/juice-shop/juice-shop.git app
   rm -rf app/.git
   rm -rf app/.github        # avoid running Juice Shop's own workflows in your repo
   ```
4. Initialise and push:
   ```bash
   git init -b main
   git add .
   git commit -m "Initial lab scaffold with Juice Shop and Day 1 pipeline"
   git remote add origin https://github.com/<your-username>/devsecops-lab.git
   git push -u origin main
   ```
5. Open the **Actions** tab and watch both jobs run. Then open **Security > Code scanning** for Semgrep results.

If the Gitleaks job fails on its first run, that is expected and is the point of the exercise (see the triage task in `docs/day1-notes.md`).
If the Semgrep SARIF upload step fails, check that the repo is public or that code scanning is enabled in **Settings > Advanced Security**. The raw report is still saved as a build artifact either way.

## Rules for this repo

- No real secrets, ever. Juice Shop contains fake ones on purpose.
- Every suppression or allowlist entry gets a comment saying why.
- Pin or review third-party actions before trusting them (Day 7 hardens this).
