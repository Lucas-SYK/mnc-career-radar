# Codex Agent Guide — Campus Radar

## Mission
Build a trustworthy China-mainland **foreign multinational campus recruitment** aggregator targeting automation, control science and engineering, industrial software, machine vision and robotics. Preserve individual applicant control over submissions.

## Invariants
- Never claim a company is currently recruiting solely because its careers homepage is reachable.
- Do not fabricate positions, deadline dates, locations, grad-year eligibility, Greenhouse tokens or application URLs.
- Public data feeds may be retrieved only from verified official careers URLs and explicit public APIs, obeying robots and reasonable rate limits.
- Never bypass CAPTCHAs, login barriers, anti-bot mechanisms, or ATS submission authorization.
- Do not auto-submit applications. Ask for user approval on every real submission, especially when third-party account sessions and PII are involved.
- Never put resume PDFs, phone numbers, email addresses, access tokens or cookies into `docs/`, source control or client-side javascript.
- Keep `.github/workflows/daily-update.yml` scheduled for UTC 16:00 (Asia/Shanghai midnight). Be explicit that Actions can be delayed.
- Generated site remains on GitHub Pages and uses no mandatory backend or external JS CDN.

## Quick checks
```bash
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python scripts/update_jobs.py --offline
python -m http.server 8000 -d docs
```

## Next development tasks (priority order)
1. Implement real official job-level adapters for companies from `config/companies.json`, each documented with source URL, company ownership verification and a parser test using scrubbed fixtures. Distinguish newly discovered jobs, still-open jobs, unverified posts and withdrawn jobs.
2. Build source health dashboard plus job freshness evidence. Do not treat failed requests as closed jobs.
3. Add filters for 2026/2027/2028 grad years, entry requirements, German/English, technical field and Qingdao / Shandong vs national cities; state requirements are sourced, not inferred.
4. Add private applicant-side profile and skill/resume gap analysis without storing PII in public GitHub Pages files.
5. Add optional **local** Playwright assisted form filling only on verified supported domains, human confirmation before submit, explicit permission-based data handling.
6. Improve UI accessibility, keyboard navigation, mobile viewport, test coverage and resilience of source retrieval.

## Acceptance
- `python -m unittest discover -s tests -v` passes.
- `python scripts/update_jobs.py --offline` creates valid JSON and does not require external network.
- GitHub Pages deploys from `docs/` using the same workflow that fetches public data.
- The interface clearly distinguishes "specific verified job" vs "official portal", and "last fetched" vs "revalidated open".
