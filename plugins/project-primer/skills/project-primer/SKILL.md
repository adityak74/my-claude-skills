---
name: project-primer
description: Use when starting a new repository, publishing an existing one, or when a repo is missing its open source basics - no README worth reading, no LICENSE, no CONTRIBUTING or pull request template, an empty GitHub description or homepage, default labels, or a project website that is not linked from the repo.
---

# Project Primer

## Core principle

A repo has one job in its first 15 seconds: let a stranger decide whether they
want it, then get them running in one copy-paste. Everything below serves that.

Work on the repo in the current directory. **Never silently overwrite an
existing file** — show the diff, or write `<file>.new` and ask.

## 0. Gather facts first — one round, not seven

Infer everything already on disk. Do not ask for what you can read:

| Fact | Source |
|---|---|
| owner/repo, description, homepage, visibility | `gh repo view --json nameWithOwner,description,homepageUrl,isPrivate` |
| language, build, test, package name | `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Makefile` |
| CI status | `.github/workflows/` |
| author, year | `git config user.name`, `date +%Y` |
| what exists already | `git ls-files` |

Then ask **once**, in a single batch, only for what is genuinely not on disk:

1. The one-line pitch — *what it is* and *who it's for*.
2. Website / docs URL, if any.
3. Whether a demo GIF, screenshot, or asciinema cast exists.
4. Anything non-MIT about licensing.

If a question goes unanswered, write the file anyway with a `<!-- TODO: ... -->`
marker and list every marker in your final summary. Never block on question 3.

## 1. LICENSE

MIT unless the user says otherwise. Pull the canonical text — do not type it
from memory:

```bash
gh api licenses/mit --jq .body \
  | sed "s/\[year\]/$(date +%Y)/; s/\[fullname\]/$(git config user.name)/" \
  > LICENSE
```

The file must be named `LICENSE` at the repo root or GitHub will not detect it.

## 2. README — the part that actually matters

Sections in this order. **Drop what doesn't apply; never pad.** A short README
that answers the question beats a long one that buries it.

```markdown
# project-name

> One line: what it is and who it's for. No adjectives, no "blazingly fast".

[badges: license · CI · version · downloads — four max]

[HERO: demo GIF, screenshot, or a 5-line code sample showing the payoff]

## Install
[one copy-pasteable block, the real command]

## Quickstart
[the smallest complete example that produces visible output]

## Usage / Features
[what it does, in the order a user meets it]

## Configuration
[table: option · type · default · what it does]

## Why not X?
[only in a crowded space — honest one-liners, no strawmen]

## Contributing
[link CONTRIBUTING.md, one sentence on how to run tests]

## Star History
[the block from §3]

## License
MIT © Author
```

Non-negotiable rules:

- **Above the fold** — pitch, badge row, and hero must land before the first
  scroll. What it is, why care, how to install, in that order.
- **Every code block runs as written.** No pseudo-commands, no `<your-key-here>`
  where a real default exists, no unexplained placeholders.
- **Show, don't list.** One real example beats ten bullet points of features.
- **No table of contents** under ~300 lines — GitHub renders its own outline.
- **No badge wall.** License, CI, version, downloads. Not "made with love",
  not a visitor counter, not PRs-welcome.
- **No TODO/lorem left behind.** Placeholders are flagged to the user, not shipped quietly.

## 3. Star history chart

Append near the bottom, above `## License`. The `<picture>` form is required —
a bare `<img>` is unreadable in dark mode:

```markdown
## Star History

<a href="https://star-history.com/#OWNER/REPO&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=OWNER/REPO&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=OWNER/REPO&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=OWNER/REPO&type=Date" />
  </picture>
</a>
```

Substitute the real `OWNER/REPO` everywhere — the chart renders empty otherwise.

## 4. Community health files

GitHub's community profile expects these. Write them at the repo root, except
the templates which live under `.github/`.

**CONTRIBUTING.md** — the only file people actually read before opening a PR.
Keep it to: how to set up (real commands), how to run tests, branch/commit
conventions, and what a good PR looks like. Skip the essay.

**CODE_OF_CONDUCT.md** — fetch the standard, don't invent one:

```bash
gh api codes_of_conduct/contributor_covenant --jq .body > CODE_OF_CONDUCT.md
```

Then replace `[INSERT CONTACT METHOD]` with a real address, or flag it as a TODO.

**SECURITY.md** — where to report a vulnerability privately, and which versions
are supported. Three lines is enough. Point it at GitHub private reporting and
tell the user to turn that on (Settings → Security → Private vulnerability
reporting); there is no stable CLI flag for it.

## 5. Pull request template

`.github/pull_request_template.md` — short enough that people fill it in:

```markdown
## What

<!-- One or two sentences. What changes and why. -->

## Why

<!-- Link the issue: Closes #123 -->

## How to test

<!-- Exact commands a reviewer runs to see this work. -->

## Checklist

- [ ] Tests pass locally
- [ ] Docs/README updated if behavior changed
- [ ] No unrelated changes in this diff
```

Optionally add `.github/ISSUE_TEMPLATE/bug_report.yml` and `feature_request.yml`
— only if the user wants them; an unfilled template is worse than none.

## 6. Repo metadata via `gh api`

Description, homepage, and topics are what GitHub search and the sidebar use.
An empty description is the single most common reason a good repo gets skipped.

```bash
gh api -X PATCH repos/{owner}/{repo} \
  -f description="One-line pitch, under 350 chars, no emoji spam" \
  -f homepage="https://your-site.dev" \
  -F has_issues=true \
  -F has_wiki=false \
  -F has_projects=false \
  -F delete_branch_on_merge=true
```

Topics are **replaced wholesale** by this call — always send the full list:

```bash
gh api -X PUT repos/{owner}/{repo}/topics \
  -f 'names[]=cli' -f 'names[]=rust' -f 'names[]=developer-tools'
```

Rules: lowercase, hyphenated, 5–10 topics. Include the language, the domain,
and the category — those are what people browse by.

> `gh repo edit --description ... --homepage ... --add-topic a,b` does the same
> in one line. Use `gh api` when you need fields `gh repo edit` doesn't expose.

**Website:** if the project has a site or docs URL, it goes in three places —
the `homepage` field above, the README pitch area, and the repo's About panel
(which reads `homepage`). Missing any one of them loses traffic.

## 7. Labels

Default GitHub labels are noise. Replace them with a set that triages:

```bash
while IFS='|' read -r name color desc; do
  gh label create "$name" --color "$color" --description "$desc" --force
done <<'LABELS'
bug|d73a4a|Something is broken
enhancement|a2eeef|New feature or request
documentation|0075ca|Docs, README, or examples
good first issue|7057ff|Small, well-scoped, newcomer-friendly
help wanted|008672|Maintainer is looking for help here
question|d876e3|Usage question or clarification
performance|fbca04|Speed, memory, or footprint
security|b60205|Vulnerability or hardening
dependencies|0366d6|Dependency bumps and updates
breaking change|e11d21|Requires a major version bump
LABELS
```

`--force` updates a label that already exists instead of erroring. The `gh api`
equivalent, if `gh label` is unavailable:

```bash
gh api -X POST repos/{owner}/{repo}/labels \
  -f name=bug -f color=d73a4a -f description="Something is broken"
```

Then delete the defaults nobody uses:

```bash
for l in duplicate invalid wontfix; do gh label delete "$l" --yes 2>/dev/null; done
```

## 8. Verify — don't claim done, check

```bash
gh api repos/{owner}/{repo}/community/profile \
  --jq '"health: \(.health_percentage)%", (.files | to_entries[] | select(.value == null) | "missing: \(.key)")'
gh repo view --json description,homepageUrl,repositoryTopics
gh label list
```

Report the health percentage, anything still missing, and every `TODO` marker
you left. If the repo is private, note that `community/profile` and the star
chart only work once it's public.

## Quick reference

| Step | Command / file |
|---|---|
| License | `gh api licenses/mit --jq .body > LICENSE` |
| README | title · pitch · badges · hero · install · quickstart · usage · star history · license |
| Star chart | `<picture>` block with dark + light `star-history.com` sources |
| Guidelines | `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md` |
| PR template | `.github/pull_request_template.md` |
| Metadata | `gh api -X PATCH repos/{owner}/{repo} -f description= -f homepage=` |
| Topics | `gh api -X PUT repos/{owner}/{repo}/topics -f 'names[]=...'` |
| Labels | `gh label create ... --force` |
| Verify | `gh api repos/{owner}/{repo}/community/profile` |

## Common mistakes

| Mistake | Fix |
|---|---|
| README opens with a logo and a badge wall | Pitch first. Badges after, four max. |
| "Installation" section with no real command | Copy-pasteable block or delete the section. |
| Star chart added as a bare `<img>` | Use `<picture>` — the light chart is invisible in dark mode. |
| `OWNER/REPO` left in the star-history URLs | Substitute the real slug; the chart is blank otherwise. |
| Topics call wipes existing topics | The endpoint is `PUT` — always send the full list. |
| Repo description left empty | It is the one line that shows in search results. |
| Website only in the README | Set `homepage` too, or the About panel stays bare. |
| Overwrote a README the user had written | Show the diff first. Always. |
