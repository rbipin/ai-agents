# envdiff — Blog Editor Deliverable

*Autonomous run: interview phase skipped per user request. Plan self-approved. Gaps marked `[FILL: ...]`.*

---

## Phase 1 — Editor's Gate

- **Concrete takeaway:** how to catch environment-config drift (missing vars, type mismatches, prod-unsafe values) before it causes an incident, using a small open-source Go CLI.
- **Reader:** DevOps engineers and backend devs running multi-stage deployments who've been burned — or worry they will be — by silent config gaps between environments.
- **Angle:** a real prod incident (missing `REDIS_URL`) as the origin story, not a feature-list dump. Hard-won, specific, credible.
- **Verdict:** ✅ **Strong angle** — the incident supplies a genuine "why," and the tool's modest scope (800 lines, single binary, 340 stars in two months) reads as credible rather than hyped.
- **Archetype:** **Showcase / plug**, blended with **I built something cool** for the origin story. Tone: honest promoter — lead with the reader's problem, admit limitations openly, credibility is the marketing.

## Phase 2 — Interview

Skipped by user request. Gaps that would normally be resolved by questions are marked `[FILL: ...]` in the draft below:
- Exact GitHub repo URL for envdiff.
- Actual distribution/install method (assumed `go install` as a placeholder — confirm or replace with release-binary instructions).

## Phase 3 — The Writing Plan (self-approved)

- **Archetype:** Showcase / plug, blended with I-built-this.
- **Working title:** "How a Missing REDIS_URL Took Down Prod (and the CLI Tool I Built So It Wouldn't Happen Again)"
- **Search phrase:** "environment config drift detection tool" (naturally covers "diff env variables across environments" too)
- **Hook:** Prod checkout breaks five minutes after a green deploy because `REDIS_URL` existed in staging but never made it to prod.
- **Sections (5):**
  1. The incident — cold open, drop the reader into the outage.
  2. The invisible gap — why config drift never shows up in code review.
  3. What envdiff actually checks — the three drift categories, with real sample output.
  4. Why it's a single binary and 800 lines — the design philosophy, kept tight.
  5. What it doesn't do yet — honest limitations (YAML, regex-only secrets scanning) + call for contributors.
- **Takeaway:** Run envdiff against your staging/prod configs before your next deploy; open an issue if you hit the YAML gap.
- **Visuals:** one terminal screenshot of the diff output — justified because it replaces a paragraph of description with the actual tool behavior, and doubles as social proof. No Mermaid diagram needed; this isn't a process/architecture story.
- **Platform:** dev.to — standard markdown, code blocks, closes with an engagement question (comments drive reach there).

---

## Phase 4–6 — Final Deliverable

### Three titles

1. **"How a Missing REDIS_URL Took Down Prod (and the CLI Tool I Built So It Wouldn't Happen Again)"** — story-first, keeps the incident promise.
2. **"Why Your Staging and Prod Configs Are Lying to Each Other"** — bold claim, provocative framing for skimmers.
3. **"envdiff: A 800-Line Go Tool That Catches Config Drift Before Prod Does"** — number + concrete promise, most literal/searchable.

**Running with #1** — it's the one with genuine narrative pull, keeps the search phrase's intent ("config drift," "prod incident") implicit in a way that reads human, and the parenthetical promises the payoff (the tool) without burying the hook.

---

### The Post

# How a Missing REDIS_URL Took Down Prod (and the CLI Tool I Built So It Wouldn't Happen Again)

It was a Tuesday deploy, nothing special about it. We pushed to prod, the health checks went green, and then five minutes later the checkout service started throwing connection errors. Redis was unreachable — not because Redis was down, but because `REDIS_URL` simply didn't exist in the prod environment. It was sitting right there in staging, doing its job, and nobody had copied it over when we cut the prod config.

That's the kind of bug that doesn't show up in code review. Your staging environment works. Your tests pass. The diff on your pull request is clean. The failure lives entirely in a place nobody diffs: the environment configs themselves.

After the incident retro, I built [envdiff](https://github.com/example/envdiff) `[FILL: confirm exact repo URL]` — a small CLI that compares environment configs across deployment stages and tells you what's actually different, not just in values, but in shape: missing keys, mismatched types, and suspicious values that shouldn't be in production.

## The gap between staging and prod is invisible until it isn't

Config drift is one of those problems every team has and almost nobody tracks. You add a var in staging to test something, mean to add it to prod, and forget. Or a var exists everywhere but someone fat-fingers a boolean as a string. Or — the classic — `localhost` ends up as a database host in a prod `.env` file because someone copy-pasted a template and never finished it.

None of this shows up until the moment it matters, usually in production, usually at the worst time. envdiff exists because I wanted a five-second answer to "did we actually finish configuring prod?" before finding out the hard way.

## What envdiff actually checks

Point it at two or more environment files and it flags three categories of drift:

- **Missing variables** — present in one stage, absent in another (this is the `REDIS_URL` case)
- **Type mismatches** — a var that's a number in staging and a string in prod, or a boolean that isn't formatted consistently
- **Suspicious values** — things like `localhost`, `127.0.0.1`, or obviously-test API keys showing up in a prod config

```
$ envdiff diff staging.env prod.env
[MISSING]  REDIS_URL        present in staging.env, missing in prod.env
[SUSPECT]  DB_HOST=localhost in prod.env (looks like a dev value)
[TYPE]     RATE_LIMIT        staging.env="100" (string) prod.env=100 (int)
```

[IMAGE: terminal screenshot of envdiff running against sample staging.env and prod.env files, showing the three drift categories in colored output | alt: envdiff CLI output showing a missing REDIS_URL, a suspicious localhost value, and a type mismatch between staging and prod environment files | file: envdiff-cli-output-drift-example.png]

That's the whole pitch. It's not trying to be a secrets manager or a config management platform — it's trying to answer one question fast, in CI or on your laptop, before a deploy goes out.

## Why it's a single binary and 800 lines

I built this over a few weekends, and I kept it deliberately small: about 800 lines of Go, one binary, no runtime dependencies. Download it, run it — that's the whole install story. During an incident, the last thing I want is a dependency negotiation before I can even run the check.

It's picked up 340 GitHub stars in about two months, which tells me "did prod actually get configured right" is a more universal Tuesday than just mine.

## What it doesn't do yet

Being straight about the edges matters more than the pitch. Right now:

- **Only `.env` files and JSON configs are supported.** If your configs live in YAML (Kubernetes ConfigMaps, Helm values), envdiff can't read them yet. That's the most requested feature, and it's next.
- **Secrets detection is basic regex**, not entropy analysis or a real secrets-scanning engine. It'll catch obvious patterns (`API_KEY=sk_test_...`) but don't rely on it as your only secrets safety net — pair it with a dedicated secrets scanner for anything that matters.
- It compares config shape and values you give it — it has no idea what a var is *supposed* to be beyond what it infers from the other stages, so a var that's wrong in *every* stage will slip through.

If you hit one of these limits, that's exactly the kind of issue I want on the [GitHub issue tracker](https://github.com/example/envdiff/issues) `[FILL: confirm exact repo URL]` — YAML support is the next big piece of work, and I'd rather build it with people who actually need it than guess at the shape alone.

## Try it before your next prod deploy

If your team has ever shipped a config gap to prod — and if you're running more than one environment, you probably have — point envdiff at your staging and prod files once and see what it says. It takes less time than writing the incident retro.

```
[FILL: exact install command — e.g. `go install github.com/<you>/envdiff@latest` if you publish a Go module, or a curl/GitHub Releases command if you ship prebuilt binaries]
envdiff diff staging.env prod.env
```

What's the closest your team has come to a `REDIS_URL`-shaped incident — a config that quietly existed in one environment and not another? I'd like to hear it, and if you try envdiff and it's missing something you need, [open an issue](https://github.com/example/envdiff/issues) `[FILL: confirm exact repo URL]` — contributions and feature requests are what will get YAML support done faster.

---

### SEO Packet

- **Meta description:** "A missing REDIS_URL took down our prod checkout service. Here's envdiff, the open-source Go CLI I built to catch environment config drift before it ships."  *(153 chars)*
- **URL slug:** `envdiff-environment-config-drift-cli`
- **Canonical note:** If this post is cross-posted anywhere besides dev.to (personal blog, Medium, Hashnode), set the canonical URL to wherever it lives first so the copies don't compete with each other in search.

---

### Editor's Notes

**`[FILL]` items — only you can supply these:**
1. Exact GitHub repo URL for envdiff (used 3 places in the post — currently placeholder `github.com/example/envdiff`).
2. Actual install/distribution method — the draft assumes `go install`, but confirm whether you publish a Go module or ship prebuilt binaries via GitHub Releases, and swap the code block accordingly.

**Reviewer flags (judgment calls — your call):**
- The "Why it's a single binary and 800 lines" section restated the tool's plainness three ways in the original draft ("small," "deliberately small," "no dependency negotiation," "no Python virtualenv, no npm install"). I tightened it to one pass — confirm the shorter version still carries the point you want, or restore more of the original flavor if you liked the rhythm.
- Reviewer confirmed the hook, tone, and structure all land well for the showcase archetype — no other flags.

**Visuals to create:**
- One terminal screenshot: run `envdiff diff` against a sample `staging.env` / `prod.env` pair that reproduces the three drift types (missing var, suspicious `localhost`, type mismatch) shown in the code block above. Crop to just the relevant output lines; colored/highlighted diff output if your CLI supports it will read better than plain monochrome text. File: `envdiff-cli-output-drift-example.png`.

**One-line summary for sharing on social:**
"A missing REDIS_URL took down our prod checkout — so I built envdiff, an open-source Go CLI that catches environment config drift before it ships. 340 stars in 2 months, looking for contributors."
