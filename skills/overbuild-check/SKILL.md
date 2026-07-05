---
name: overbuild-check
description: >
  Precheck before building. Use BEFORE writing a new helper/util/module, before
  adding a dependency (pip install / adding to pyproject/requirements), or before
  implementing anything that "feels like it might already exist". Checks the local
  codebase and installed packages first, then live package registries, and recommends
  reuse over rebuild.
---

# Overbuild check

Your job: stop the user from rebuilding something that already exists. Run this the
moment a request implies writing new code for a general capability (rate limiting,
retries, caching, file watching, parsing, HTTP, dates, CLI args, JSON logging, etc.).

Do NOT run it for genuinely bespoke domain logic (their business rules, their schema).

## Procedure

Climb this ladder and stop at the first rung that answers the request.

1. **Already in this codebase?** Grep for the capability by name and by concept
   (`Grep` for likely function/class names and related keywords). If a helper, util,
   or module already does it → reuse that. Strongest possible result.

2. **Already an installed/declared dependency?** Read `pyproject.toml` /
   `requirements.txt` and, if a venv is active, run `pip list` (or `uv pip list`).
   If a dependency already in the project solves it → use it. No install, no new dep.

3. **Standard library?** If the language's stdlib covers it (`functools.lru_cache`,
   `itertools`, `pathlib`, `sqlite3`, `datetime`, `argparse`…), use it. No dependency.

4. **A well-known package?** Only now, and only if 1–3 found nothing. Find candidates,
   then verify they're real and healthy (see **Registry lookup** below).

5. **Genuinely nothing fits?** Then building is justified. Recommend the *smallest*
   implementation — ideally a one-liner or a few lines — not a framework.

## Registry lookup

You don't have a special tool for this — use your own fetch/shell tools. Two moves:

**Discover candidates.** You already know most major libraries; name the likely ones
from memory first. To find more, search GitHub (relevance is good, no key needed):

```
https://api.github.com/search/repositories?q=<terms>+language:<lang>&sort=stars&order=desc&per_page=5
```

Prefer `WebFetch` (it summarizes the response) over raw `curl` so you don't pull a
huge JSON blob into context. With `curl`, trim it: pipe through
`jq '.items[] | {full_name, stargazers_count, description, html_url, pushed_at}'`.

**Verify a named package is real and maintained.** ecosyste.ms gives cross-ecosystem
stats via a package URL (`purl`). Ecosystems: `pypi`, `npm`, `cargo`, `go`,
`rubygems`, `packagist`, `maven`, `nuget`.

```
https://packages.ecosyste.ms/api/v1/packages/lookup?purl=pkg:pypi/<name>
```

Returns a list; take the first. Useful fields: `latest_release_published_at` (is it
maintained?), `repo_metadata.stargazers_count`, `dependent_repos_count`, `licenses`.
Do NOT use its keyword search — it's noisy; use GitHub for discovery instead.

Notes:
- Both endpoints are keyless. Unauthenticated GitHub search rate-limits fast; if the
  user has a `GITHUB_TOKEN` in their environment, add `Authorization: Bearer $TOKEN`.
- A hit on a registry outranks writing new code. A hit already in the codebase or an
  installed dep (rungs 1–2) outranks any registry package.

## Output to the user

Report what you found and your recommendation, briefly:

- **Reuse (codebase/dep/stdlib):** name it, show how to call it, don't write new code.
- **Install existing:** name the package + install command + a one-line usage sketch,
  and cite its stars / last release so they can trust it.
- **Build:** state that nothing fit and why, then propose the minimal version.

Keep it short. The point is to save the user from overbuilding, not to lecture.
