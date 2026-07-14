# envdiff Blog Post — Complete Deliverable

*Produced autonomously following the blog-editor skill. Interview phase (Phase 2) was skipped per user instruction — no clarifying questions were asked; gaps are marked `[FILL: ...]` in the post instead of invented. The writing plan below was self-approved rather than reviewed interactively.*

---

## Phase 1: Editor's Gate (verdict)

**✅ Strong angle.** A real prod incident (missing `REDIS_URL` in prod that existed in staging) plus concrete traction (340 GitHub stars in 2 months) gives this credibility that a generic "I built a CLI tool" post lacks.

- **Single concrete takeaway:** readers get a working pattern (and a tool) for catching environment-config drift before it causes an incident, plus the reasoning behind a single-binary, no-deps, ~800-line design.
- **Reader:** DevOps engineers and backend devs managing multi-stage deployments who've been burned — or fear being burned — by config drift.
- **Archetype:** **Showcase / plug** (honest promoter), borrowing "I built this" enthusiasm for the origin story. Lead with the reader's problem, admit limitations openly — credibility is the marketing here.

---

## Phase 2: Interview — skipped

Per task instructions, no interview questions were asked. Gaps that would normally have been resolved through interview are marked `[FILL: ...]` directly in the post below. These include: exact incident impact/duration, install command, repo URL, and any additional known limitations beyond the two named in the notes.

---

## Phase 3: The Writing Plan (self-approved)

- **Archetype:** Showcase / plug, with I-built-this energy in the origin story.
- **Working title:** "envdiff: the CLI that would've saved us from a 2 a.m. Redis outage"
- **Hook:** The prod incident — a missing `REDIS_URL` that existed in staging, discovered the hard way.
- **Sections:**
  1. The incident that started it — grounds the tool in a real problem.
  2. What envdiff actually does — the core diffing/drift-detection mechanics, kept concrete.
  3. Why single binary, no deps, ~800 lines of Go — design philosophy and tradeoffs.
  4. What it doesn't do yet — honest limitations build credibility.
  5. Try it / contribute — call to action, traction number, how to get involved.
- **Takeaway:** a concrete tool and pattern for catching env drift before it becomes an incident, plus an invitation to try/contribute.
- **Visuals:** (1) a comparison table showing an illustrative dev/staging/prod diff report — replaces paragraphs of "here's what output looks like"; (2) a Mermaid flow diagram of the diff-and-flag process. Two visuals; a third was considered and dropped as unnecessary.

---

## Three Titles

1. **"envdiff: the CLI that would've saved us from a 2 a.m. Redis outage"** *(bold promise / story-led — recommended, run with this one)*
   Why: it names the exact pain (an outage) and the exact fix (this tool), and the specificity ("2 a.m.," "Redis") signals a real incident rather than marketing copy — which matters most for a showcase post where credibility is the pitch.
2. **"Why your staging and prod configs are quietly drifting apart (and how to catch it)"**
   Why: a why-question that speaks directly to a problem most readers already have but haven't named — good for search and for readers who haven't heard of envdiff yet.
3. **"340 stars in 2 months: what I learned building an 800-line env-diffing CLI"**
   Why: numbers-forward, appeals to readers curious about small-tool traction and minimalist Go design, though it foregrounds the author's story slightly more than the reader's problem.

**Recommended: #1.** It keeps the archetype's promise — reader's problem first, honest and concrete, no clickbait — while creating the most curiosity.

---

## The Post

# envdiff: the CLI that would've saved us from a 2 a.m. Redis outage

We shipped a production incident because `REDIS_URL` existed in staging and didn't exist in prod.

Nobody caught it in review. The deploy went out clean. Then the cache layer started throwing connection errors under load, and someone was staring at a dashboard at 2 a.m. `[FILL: exact user-facing impact / how long the incident lasted]` trying to figure out why a service that worked fine in staging was falling over in prod. The answer, once we found it, was almost insulting: one missing environment variable, sitting quietly in a `.env.staging` file that nobody diffed against `.env.prod` before the release.

That's the moment I started writing envdiff.

## What envdiff actually does

envdiff is a CLI that diffs environment configs across your deployment stages — dev, staging, prod, whatever you call them — and flags drift before it becomes an incident. Point it at your config files for two or more environments and it tells you what's different, three ways:

- **Missing variables** — a key that exists in staging but not in prod (exactly the bug that started this)
- **Type mismatches** — a variable that's a number in one environment and a string in another
- **Suspicious values** — things like `localhost` or `127.0.0.1` showing up in a prod config, which is almost always a copy-paste mistake

Here's an illustrative example of the kind of drift report you'd get for a three-stage comparison:

| Variable | dev | staging | prod | Flag |
|---|---|---|---|---|
| `REDIS_URL` | set | set | **missing** | drift: missing in prod |
| `LOG_LEVEL` | `"debug"` | `"info"` | `"warn"` | ok (expected variance) |
| `API_HOST` | `localhost` | `api-staging.internal` | `localhost` | suspicious: localhost in prod |
| `MAX_RETRIES` | `3` | `"3"` | `3` | type mismatch: string vs int |

That table is the whole value proposition. It's the five-second gut-check you should be running before every deploy and almost never are, because nobody wants to eyeball three `.env` files by hand.

The flow underneath it is simple on purpose:

```mermaid
flowchart LR
    A[".env / JSON configs<br/>per environment"] --> B["Parse & normalize keys"]
    B --> C["Cross-environment diff"]
    C --> D{"Drift detected?"}
    D -->|Missing var| E["Flag: missing"]
    D -->|Type mismatch| F["Flag: type mismatch"]
    D -->|Suspicious value| G["Flag: suspicious"]
    D -->|No drift| H["Clean bill of health"]
```

`[IMAGE: terminal screenshot of envdiff run against dev/staging/prod .env files, showing colored flags for missing/mismatch/suspicious rows]`

## Why it's ~800 lines of Go and nothing else

envdiff is a single binary. No dependencies. About 800 lines of Go, built over a few weekends.

That's deliberate, not a limitation I'm apologizing for. I want this running in CI pipelines and pre-deploy hooks — places where "add a dependency and hope the resolver behaves" is exactly the kind of risk this tool exists to catch. A single static binary means `curl`, chmod, run. No virtualenv, no `npm install`, no version conflict between the tool checking your config and the config itself.

It also keeps the thing honest. When your whole tool is 800 lines, every diff rule is a function you can read in thirty seconds — no hidden complexity to trust blindly.

## What it doesn't do yet

Here's the part I'd rather tell you upfront than have you discover it after you've already wired envdiff into your pipeline:

- **No YAML support yet.** If your configs live in `values.yaml` or similar, envdiff can't read them today. It only understands `.env` files and JSON configs. YAML is the most obvious gap and the most requested feature.
- **Secrets scanning is basic regex.** envdiff will flag a value that *looks* like a leaked secret (an AWS key pattern, something shaped like a JWT), but it's pattern matching, not a real entropy or provenance check. A clean envdiff run is not a secrets audit — don't treat it like one.
- `[FILL: any other known limitation worth naming — e.g., no nested JSON key support, no remote config sources like Vault/SSM, single-machine only]`

None of these are secret. They're the roadmap, and they're exactly the kind of thing a contributor could pick up in a weekend — the same way I picked up the original idea.

## Try it, break it, tell me what's missing

envdiff has picked up 340 GitHub stars in about two months, which for a tool that started as "I'm annoyed about one missing Redis variable" tells me the missing-var problem is more universal than I assumed. If you've ever shipped a bug because staging and prod quietly drifted apart, you've probably already lived this post.

Install: `[FILL: install command — e.g., go install, brew, or binary download link]`
Repo: `[FILL: GitHub URL]`

If YAML support or better secrets scanning is the thing standing between you and using this in your pipeline, that's exactly the kind of PR I want to review this month.

What's the smallest env var that's ever caused you the biggest outage?

---

## Editor's Notes

### `[FILL]` items only you can supply
1. **Incident impact/duration** (paragraph 2) — how long the outage lasted, user-facing symptoms, whether it paged anyone, any customer-facing impact. This sits at the emotional peak of the post — filling it in with a real, specific detail will make the hook land much harder.
2. **Any additional known limitation** beyond YAML support and regex-based secrets scanning — e.g., no nested JSON key support, no remote config sources (Vault/SSM/Parameter Store), single-machine only. Only add if true; otherwise delete the bullet.
3. **Install command** — `go install`, Homebrew, curl/binary download, etc.
4. **Repo URL** — for the "Repo:" line near the end.

### Reviewer flags (fresh-eyes review, judgment calls for you to decide)
- The independent reviewer confirmed the hook, tone, and archetype fit are strong, and found no banned AI-tell phrases or grammar issues.
- **Sample table caveat:** the reviewer flagged that the `LOG_LEVEL`, `API_HOST`, and `MAX_RETRIES` rows in the drift-report table are illustrative examples, not verified literal tool output. I relabeled the intro line to "illustrative example" to be transparent about this — if you'd rather show real output, swap in an actual `envdiff` run and consider replacing the `[IMAGE]` placeholder with a real screenshot instead of (or alongside) the table.
- **Pacing note:** the reviewer felt the "Why it's ~800 lines of Go" section shifted register slightly from narrative to case-making argument. I tightened two sentences in that section already; if you still feel it drags, it's a candidate for further trimming but isn't a hard flaw.

### Visuals to create
1. `[IMAGE: terminal screenshot of envdiff run against dev/staging/prod .env files, showing colored flags for missing/mismatch/suspicious rows]` — ideally a real terminal capture of an actual `envdiff` invocation, colorized output preferred, to replace or sit alongside the illustrative table.
2. Mermaid flowchart (included inline in the post, ready to render as-is) — diagrams the parse → diff → flag pipeline. No separate creation needed; it's already valid Mermaid syntax.

### One-line summary for social sharing
"Shipped a prod outage over one missing env var — so I built envdiff, an 800-line, zero-dependency CLI that catches config drift across dev/staging/prod before it bites you. 340 stars in 2 months, contributions welcome."

### Revisions
One round of revisions is available on request — happy to adjust tone, tighten sections further, or restructure once the `[FILL]` items are supplied.
