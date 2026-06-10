# Constitution

Architectural decisions that apply to every module. Do not revisit these without a deliberate reason.

## Storage
- SQLite is the database. One file: `data/jobs.db`.
- Every job Apify returns is written to SQLite immediately. Duplicates are skipped via unique constraint on Apify job ID.
- Scores and rankings are columns on the same `jobs` table — updated in place when a module runs.
- At the end of each run, export a ranked Excel file to `output/job_rankings_{date}.xlsx` for human review.
- `data/jobs.db` is committed to git. `output/` is gitignored.

## Module Design
- Each module does exactly one thing. If it grows beyond ~150 lines, split it.
- Modules do not import each other. They read from and write to the database independently.
- This means any module can be re-run in isolation without re-running the whole pipeline.

## API Strategy
- Use `claude-haiku-4-5-20251001` for volume tasks (scoring many jobs).
- Use `claude-sonnet-4-6` for quality tasks (resume optimization, one job at a time).
- All external API calls (Anthropic, Apify) are async using `asyncio`.

## Resume
- The optimizer never rewrites the full resume.
- It only modifies experience bullet points and the summary section of `resume/resume.tex`.
- Each tailored version is saved as `output/{company-slug}-{role-slug}/resume.tex` and compiled to PDF via `pdflatex`.
- The original `resume/resume.tex` is never overwritten.

## Testing
- Tests never call real APIs. Use saved fixture responses (sample JSON files in `tests/fixtures/`).
- A module is not complete until it has at least one passing test.

## Git
- One branch per module: `feature/job-fetcher`, `feature/job-scorer`, etc.
- Merge to `main` only when tests pass.
