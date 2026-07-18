---
name: overbuild-check
description: >
  Precheck before building. Use BEFORE writing a new helper/util/module, before
  adding a dependency, or before implementing any capability with a name you could
  search for on its own (rate limiting, retries, caching, parsing, etc.) rather
  than bespoke business logic. Checks the local codebase and installed packages
  first, then live package registries when unsure, and recommends reuse over rebuild.
---

# Overbuild check

Your job: stop the user from rebuilding something that already exists. Run this the
moment a request implies writing new code for a general capability (rate limiting,
retries, caching, file watching, parsing, HTTP, dates, CLI args, JSON logging, etc.).

Do NOT run it for genuinely bespoke domain logic. Test: does the capability have a
name you could search for on its own — a noun, not tied to this app's schema or
business rules? If not, it's bespoke; skip this skill.

## Procedure

Climb this ladder and stop at the first rung that answers the request.

1. **Already in this codebase?** Grep for the capability by name and by concept
   (`Grep` for likely function/class names and related keywords). If a helper, util,
   or module already does it → reuse that. Strongest possible result.

2. **Already an installed/declared dependency?** Read this language's manifest/
   lockfile (`pyproject.toml`/`requirements.txt`, `package.json`, `go.mod`,
   `Cargo.toml`, `Gemfile`, `composer.json`, `pom.xml`…) and list installed packages
   (`pip list`, `npm ls`, `go list -m all`…). If a dependency already in the project
   solves it → use it. No install, no new dep.

3. **Standard library?** If the language's stdlib covers it (Python:
   `functools.lru_cache`, `pathlib`; Node: `node:fs`, `node:crypto`; Go: `net/http`,
   `encoding/json`; …), use it. No dependency.

4. **A well-known package?** Only now, and only if 1–3 found nothing. Name likely
   candidates from memory. If you're confident one is current and maintained,
   recommend it directly — no fetch needed. Only verify first (see **Registry
   lookup** below) if you're unsure — unfamiliar name, or it might be
   renamed/abandoned.

5. **Genuinely nothing fits?** Then building is justified. Recommend the *smallest*
   implementation — ideally a one-liner or a few lines — not a framework.

## Registry lookup

Use only when you're not already confident about a candidate. Two moves, both via
your own fetch tools (prefer `WebFetch` over raw `curl` so a huge JSON blob doesn't
land in context):

**Don't know a candidate?** Search GitHub (keyless, but rate-limits fast — use a
`GITHUB_TOKEN` if the user has one set):

```
https://api.github.com/search/repositories?q=<terms>+language:<lang>&sort=stars&order=desc&per_page=5
```

**Not sure a named package is maintained?** Look it up on ecosyste.ms by package URL
(`purl`; ecosystems: `pypi`, `npm`, `cargo`, `go`, `rubygems`, `packagist`, `maven`,
`nuget`). Skip its keyword search — it's noisy; use GitHub for that instead.

```
https://packages.ecosyste.ms/api/v1/packages/lookup?purl=pkg:pypi/<name>
```

Take the first result; check `latest_release_published_at` (is it maintained?),
`repo_metadata.stargazers_count`, `dependent_repos_count`, `licenses`.

A hit here outranks writing new code — but rungs 1–2 (codebase/installed dep) always
outrank a registry package.

## Output to the user

Report what you found and your recommendation, briefly:

- **Reuse (codebase/dep/stdlib):** name it, show how to call it, don't write new code.
- **Install existing:** name the package + install command + a one-line usage sketch,
  and cite its stars / last release so they can trust it.
- **Build:** state that nothing fit and why, then propose the minimal version.

Keep it short. The point is to save the user from overbuilding, not to lecture.
