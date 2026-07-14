# Blog Editor Deliverable — CI Cache War Story

Autonomous run: no interview was possible, so gaps are marked with
`[FILL: ...]` throughout. Plan below was self-approved on the user's behalf
per their instruction.

---

## Phase 1 — Editor's Gate (verdict)

✅ **Strong angle.** This has a real incident, a surprising root cause
(a cosmetic-looking Dockerfile refactor), and hard numbers (40 min → 12 min,
a 3x regression over 6 months). The "nobody noticed until a new hire asked"
framing is a genuinely sharp hook — it's not just "we fixed a bug," it's an
observability failure story that generalizes to any team running CI.

**Archetype:** War story / incident — wry, hard-won tone, story-first
opening, humor from the real struggle (mumbled excuses in standup, slow
normalization of pain).

---

## Phase 2 — Interview

Skipped by user request. No questions were asked. Gaps that would normally
be interview questions are marked `[FILL: ...]` directly in the post:

- Link/reference to the PR or commit that reordered the Dockerfile
- Exact before/after cache-hit-rate percentages (notes only say "near
  zero" and don't give a recovered-state number)
- Name of the dashboard/tool used for the new cache-hit-rate metric

---

## Phase 3 — The Writing Plan (self-approved)

- **Archetype:** War story / incident.
- **Working title:** "How a One-Line Dockerfile Change Cost Us 28 Minutes
  of CI Time"
- **Search phrase:** "docker layer cache slow ci builds" — a mid-level dev
  debugging a slow pipeline would plausibly search this. Anchors title,
  main heading, and opening paragraphs.
- **Hook:** A new hire asks in standup why the build takes 40 minutes, and
  nobody on the team has a real answer.
- **Sections:**
  1. Nobody noticed a 3x slowdown because nobody was measuring it — sets
     up the "humans normalize slow degradation" theme before the technical
     reveal.
  2. The Dockerfile got reordered in a routine refactor — the root cause,
     with before/after code.
  3. (folded into section 2) the mechanism — why layer order matters for
     Docker's cache, explained concretely enough for someone unfamiliar.
  4. The fix took fifteen minutes; the diagnosis took six months — the fix,
     the new metric, and the recovered numbers.
  5. Slow degradation is invisible without metrics — the takeaway, pulled
     up a level from "fix your Dockerfile" to "instrument leading
     indicators."
- **Takeaway:** Build duration is a lagging indicator; cache hit rate is a
  leading one. Add the leading indicator before you need it.
- **Visuals:**
  - Mermaid flowchart contrasting the cache-friendly vs. cache-broken
    layer order — replaces a paragraph of "picture the layers" explanation
    with something scannable.
  - Table comparing before/after metrics — a comparison, so a table earns
    its place over prose.
  - No screenshot placeholder — there's no UI moment central to the story.

No second post buried in these notes; one core idea (silent CI
degradation) carried through.

---

## Phase 4–5 — Writing and Independent Review

Full draft was written per the plan, then handed to an independent
fresh-eyes subagent (given only the raw notes and the draft, not this plan)
for review per Phase 5. Findings and actions taken:

- **Mechanical:** no typos/grammar issues found; Mermaid and code fences
  valid. Fixed silently: none needed beyond judgment calls below.
- **Content accuracy:** reviewer flagged that the original draft invented
  a specific "85% cache hit rate" figure not present in the raw notes, and
  invented the new hire's gender ("she") which the notes never specify.
  Both fixed — the pronoun was changed to "they," and the invented
  percentage was replaced with an honest `[FILL:]` marker instead of a
  fabricated number.
- **Emotion/tone:** reviewer confirmed the hook, humor ("one
  slightly-longer coffee break at a time," "almost insultingly small"), and
  overall wry incident-report tone all land and match the archetype. No
  sagging sections.
- **AI-tells:** none of the banned phrases found; no unnatural repetition
  of the search phrase; paragraph rhythm varies naturally. Nothing flagged
  here required action.

No unresolved reviewer flags remain that need the user's judgment call
beyond the `[FILL:]` items listed in Editor's Notes below.

---

## Phase 6 — Delivery

### Three titles

1. **"How a One-Line Dockerfile Change Cost Us 28 Minutes of CI Time"**
   (bold promise + number) — **Recommended.** It's concrete, honest about
   scope (one line, not a rewrite), and the number is real and verifiable
   from the post's own numbers (40 − 12 = 28).
2. **"Why Our CI Builds Got 3x Slower and Nobody Noticed"** (why + number)
   — leads with the more emotionally resonant "nobody noticed" angle, good
   alternative if you want to foreground the observability lesson over the
   technical fix.
3. **"The Docker Layer Cache Bug That Took Six Months to Notice"**
   (descriptive, keyword-forward) — closest match to the search phrase,
   most literal, best if SEO discoverability matters more than voice.

**Running with #1** — it keeps the archetype's wry, understated tone (a
"one-line change" causing 28 minutes of pain is the joke) while still
containing a real, checkable number.

### The post

# How a One-Line Dockerfile Change Cost Us 28 Minutes of CI Time

A new hire had been on the team for two weeks when they asked the question in standup: "Why does our build take 40 minutes?"

Nobody had a good answer. A few people mumbled something about "it's always been kind of slow." One person guessed it was the test suite. Someone else blamed the runners. The truth was worse: nobody actually knew, because nobody had been watching. Six months earlier, that same build took 12 minutes. It had crept up so gradually that everyone who watched it happen had normalized it, one slightly-longer coffee break at a time.

That question sent us digging, and what we found was a single reordered line in a Dockerfile that had quietly killed our Docker layer cache.

## Nobody noticed a 3x slowdown because nobody was measuring it

The frustrating part wasn't the root cause — it was how long it took a fresh set of eyes to even ask the question. Our CI dashboard showed build duration, pass/fail rates, and flaky test counts. It did not show cache hit rate. So when builds slid from 12 minutes to 20, then 28, then 40, every step looked like normal noise. There was no chart with a line trending in the wrong direction, just a slow accumulation of "huh, builds feel kind of slow today" that everyone quietly absorbed.

That's the trap: humans are excellent at normalizing gradual pain and terrible at noticing it without a number to anchor to. If the build had jumped from 12 to 40 minutes overnight, someone would have filed an incident. Because it happened over six months, it just became "how long builds take here."

## The Dockerfile got reordered in a routine refactor

Once we went looking, the cause wasn't exotic. Someone had refactored the Dockerfile for readability months earlier — `[FILL: link to the PR or commit if you want to reference it in the post]` — and in the process moved the source code `COPY` command ahead of the dependency install step.

Before the refactor, the Dockerfile installed dependencies first, then copied application source:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

Docker builds images in layers, and it caches each layer as long as the inputs to that layer haven't changed. `package.json` changes rarely. Application source changes on every single commit. Put the dependency install first, and Docker can reuse that expensive layer almost every build — it only reruns `npm ci` when a dependency actually changes.

After the refactor, someone flipped the order for what looked like a cleaner, more "logical" read top to bottom:

```dockerfile
COPY . .
RUN npm ci
```

This looks harmless. It is not. Now every commit invalidates the cache before dependency installation even runs, because the source copy — which changes constantly — comes first. Every single build reinstalled every dependency from scratch. Our cache hit rate had silently dropped to near zero. `[FILL: exact before/after cache-hit-rate percentages, if you pulled them from build logs]`

[IMAGE: Mermaid flowchart contrasting the cache-friendly Dockerfile layer order (deps installed before source copy, cache reused every commit) against the cache-broken order (source copied before deps, cache invalidated every commit) | alt: Side-by-side flowchart showing how reordering COPY and RUN npm ci in a Dockerfile changes whether Docker's build cache is reused or invalidated on every commit | file: docker-layer-cache-order-before-after.png]

```mermaid
flowchart LR
    subgraph Before["Before: cache-friendly"]
        A1[COPY package.json] --> A2[RUN npm ci] --> A3[COPY source] --> A4[Cache reused<br/>on every commit]
    end
    subgraph After["After: cache broken"]
        B1[COPY source] --> B2[RUN npm ci] --> B3[Cache invalidated<br/>on every commit]
    end
```

## The fix took fifteen minutes; the diagnosis took six months

Once we knew what to look for, the fix was almost insultingly small: move the dependency install back ahead of the source copy, and split the `COPY` commands so only the dependency manifests get copied early.

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

We also added a cache-hit-rate metric to the CI dashboard, pulled straight from Docker's build output (`CACHED` vs. rebuilt layers), so this can never again silently regress for six months without anyone noticing. `[FILL: name of the tool/dashboard used — e.g., Grafana, Datadog, custom script — and how the metric is scraped]`

Builds went from 40 minutes back to 12 minutes — a 70% cut in build time, all from one Dockerfile edit and zero infrastructure changes.

[IMAGE: A simple before/after bar or line chart of the cache-hit-rate metric now on the team's CI dashboard, showing the near-zero period and the recovery after the fix | alt: Dashboard chart showing Docker cache hit rate recovering from near zero back to a healthy level after the Dockerfile fix | file: ci-dashboard-cache-hit-rate-metric.png]

| Metric | Before fix | After fix |
|---|---|---|
| Median build time | 40 min | 12 min |
| Docker cache hit rate | near 0% | `[FILL: after-fix cache hit rate]` |
| Root cause | COPY source before deps | COPY deps before source |
| Infra changes required | — | None |

## Slow degradation is invisible without metrics

The Dockerfile bug was a five-minute fix once someone looked. The real lesson is upstream of that: we had no metric that would have caught this on day one, so it took six months and a new hire's outsider question to surface it.

If you own a CI pipeline, build duration alone won't save you — it's a lagging, noisy indicator that degrades slowly enough for everyone to explain away. Cache hit rate is a leading indicator that would have shown a cliff-edge drop the day the Dockerfile changed, not six months later.

Go check your own Docker cache hit rate this week. If you don't have a way to see it, that's the same blind spot we had — and probably the cheapest CI fix you'll make all quarter.

What silent regression has your team normalized without noticing?

### SEO packet

- **Meta description:** "Our CI builds crept from 12 to 40 minutes over 6 months and nobody noticed. The cause: a reordered Dockerfile line that silently broke our Docker layer cache."
- **URL slug:** `docker-layer-cache-ci-builds-slow`
- **Canonical note:** If cross-posting this beyond dev.to (personal blog, Medium, etc.), publish on one platform first and set the `rel=canonical` tag on the others to point back to it, so the copies don't compete with each other in search results.

### Editor's notes

**`[FILL:]` items only the user can supply:**
1. Link or reference to the PR/commit that reordered the Dockerfile (section: "The Dockerfile got reordered in a routine refactor").
2. Exact before/after cache-hit-rate percentages — the raw notes only say the rate dropped "to near zero"; if you have real numbers (before the regression, and after the fix), swap them in for the two `[FILL:]` markers (body paragraph and the metrics table).
3. Name of the dashboard/tool used to add the new cache-hit-rate metric (Grafana, Datadog, a custom script, etc.) — currently generic ("CI dashboard").

**Reviewer flags requiring your judgment:** none outstanding — the two
substantive flags from the fresh-eyes review (an invented gender pronoun
for the new hire, and a fabricated 85% cache-hit-rate figure not in your
notes) were corrected directly since they were factual/accuracy fixes, not
tone or structure calls.

**Visuals to create:**
1. Mermaid diagram (ready to render, embedded in the post) contrasting the
   two Dockerfile layer orders — no external design work needed, just
   render the code block.
2. A dashboard screenshot or simple chart showing the cache-hit-rate metric
   recovering after the fix — hand to whoever owns the CI dashboard, or
   generate from real build-log data if available.

**One-line summary for sharing on social:**
"Our CI builds crept from 12 to 40 minutes over 6 months — turns out a
one-line Dockerfile reorder had silently killed our build cache the whole
time."
