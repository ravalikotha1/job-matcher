# job-matcher

AI-powered job search assistant. Fetches job postings via Apify, scores them against a resume, ranks them by fit, and optimizes the resume for top matches.

## What This Project Does

1. **Job Fetcher** — pulls job postings from Apify (Google Jobs, LinkedIn, Indeed)
2. **Resume Parser** — reads resume from `resume/resume.tex` and extracts structured data
3. **Job Scorer** — calls Claude API to compare each job JD against the resume, returns match % and skill gaps
4. **Ranker** — sorts jobs by score + personal preferences into must-apply → skip
5. **Resume Optimizer** — for top-ranked jobs, suggests targeted changes to summary and skills section only

## Tech Stack

- Python 3.9+ (system Python at `/usr/bin/python3`)
- Claude API (Anthropic) — `claude-haiku-4-5-20251001` for scoring, `claude-sonnet-4-6` for resume optimization
- Apify API — job scraping
- `python-dotenv` for environment variables
- `sqlite3` (built-in) for job tracking database — `data/jobs.db`
- `pandas` + `openpyxl` for Excel export — `output/job_rankings_{date}.xlsx`
- Resume source of truth is LaTeX (`.tex`) — compiled locally via `pdflatex` (BasicTeX at `/Library/TeX/texbin/pdflatex`)
- Compiled PDFs go to `output/{company-slug}-{role-slug}/resume.pdf`

## Project Structure

```
job-matcher/
├── CLAUDE.md                    # This file
├── .env                         # Secrets (never commit)
├── .env.example                 # Template for secrets
├── resume/
│   └── resume.tex               # Resume in LaTeX (source of truth, never overwritten)
├── data/
│   └── jobs.db                  # SQLite database (committed to git)
├── src/
│   ├── job_fetcher.py
│   ├── resume_parser.py
│   ├── job_scorer.py
│   ├── ranker.py
│   └── resume_optimizer.py
├── tests/
│   └── fixtures/                # Saved API responses for tests (no real API calls)
├── output/                      # Generated files (gitignored)
└── .specify/
    ├── memory/
    │   └── constitution.md      # Architectural rules
    └── specs/
        ├── 001-job-fetcher/spec.md
        ├── 002-resume-parser/spec.md
        ├── 003-job-scorer/spec.md
        ├── 004-ranker/spec.md
        └── 005-resume-optimizer/spec.md
```

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run tests
pytest tests/ -v

# Run the pipeline (when built)
python src/main.py
```

## How to Work on This Project

- Read the spec for the feature before writing any code: `.specify/specs/00X-feature-name/spec.md`
- Build one module at a time. Do not start the next until the current one has passing tests.
- Never hardcode API keys. Always read from environment variables.
- Keep each module independently runnable and testable.

## Secrets

- `ANTHROPIC_API_KEY` — Claude API key
- `APIFY_API_TOKEN` — Apify API token
- Both live in `.env` (gitignored). See `.env.example` for the template.

## Boundaries

- **Never** commit `.env` or any file containing API keys
- **Never** rewrite the entire resume — only suggest targeted changes (summary + skills section)
- **Ask before** changing the scoring logic or ranking weights
- **Always** write a test before marking a module complete
