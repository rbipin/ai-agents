# Blog Editor Deliverable: TIL — git bisect run

> Produced autonomously per author's instruction (no interview available). Plan was self-approved. Missing specifics are marked `[FILL: ...]`.

---

## 1. Plan (self-approved)

- **Editor's Gate verdict:** ✅ Worth writing. Real incident, hard numbers (3 hours manual vs. ~4 minutes / 8 automatic steps across ~200 commits), and a widely useful, underused git feature.
- **Archetype:** TIL (quick, humble, generous) with a light war-story opening. Target length ~300–600 words.
- **Audience:** Any developer. **Platform:** dev.to.
- **Search phrase the post targets:** "automate git bisect" / "git bisect run script".
- **Hook:** Three hours of manual binary search vs. one flag the author didn't know existed.
- **Structure:**
  1. Hook — the 3-hour manual hunt through ~200 commits.
  2. "The part of git bisect nobody told me about" — the `git bisect run` contract (exit 0 = good, non-zero = bad) + commands.
  3. "The whole test script was a build plus a curl" — the tiny script, plus two practical footnotes (exit code 125 = skip; keep the script outside the repo).
  4. "200 commits, 8 steps, 4 minutes" — the payoff, the math (log₂ 200 ≈ 8), the takeaway, one action for the reader.
- **Visuals:** None. Two code blocks carry the post; images would add nothing to a TIL this size.
- **Engagement:** Closing question (dev.to convention).
- **Interview:** Skipped per author instruction; gaps carry `[FILL]` markers (last-known-good ref, exact build/run commands, real health endpoint).

---

## 2. Three Title Options

1. **TIL: git bisect Can Run the Whole Hunt for You** *(recommended — honest TIL framing, names the feature)*
2. **How I Went From 3 Hours of Manual git Checkouts to a 4-Minute `git bisect run`** *(outcome-forward, strong for search/skimmers)*
3. **200 Suspect Commits, 8 Automatic Steps: Meet `git bisect run`** *(numbers-led, curiosity hook)*

---

## 3. The Post

# TIL: git bisect Can Run the Whole Hunt for You

Yesterday I spent three hours doing binary search by hand. Our staging deploy had broken somewhere in a stretch of roughly 200 commits, and I was checking out commits one at a time, building, poking the server, and taking notes like it was 2009.

Then a teammate glanced at my terminal and introduced me to `git bisect run`.

## The part of git bisect nobody told me about

I knew `git bisect` — you mark a good commit and a bad one, and git walks you through a binary search, asking "good or bad?" at each step. What I didn't know is that you can hand it a script and it will answer that question itself:

```bash
git bisect start
git bisect bad HEAD
git bisect good <last-known-good>   # [FILL: the ref you knew was healthy]
git bisect run ./check.sh
```

The contract is simple: the script exits `0` if the commit is good, non-zero if it's bad. Git checks out a commit, runs your script, reads the exit code, and jumps to the next commit. No human in the loop.

## The whole test script was a build plus a curl

Ours was barely a script. Build locally, start the app, hit the health endpoint:

```bash
#!/usr/bin/env bash
set -e
# [FILL: your build + run commands, e.g. npm ci && npm run build,
#  then start the server in the background and wait for it to come up]
curl -sf http://localhost:3000/health   # [FILL: your real health endpoint]
```

With `set -e`, any failing step makes the script exit non-zero, which tells git "bad commit." Two footnotes worth knowing. First, exit code `125` has a special meaning — "can't test this commit, skip it" — useful when a commit doesn't build for unrelated reasons. Second, keep the script outside the repo (or copy it out first), since git will be checking out old commits underneath it.

## 200 commits, 8 steps, 4 minutes

Binary search is why this works so well: log₂(200) ≈ 7.6, so ~200 suspects collapse into 8 automated checks. `git bisect run` found the culprit commit in about 4 minutes. I had spent three hours and still hadn't found it.

The lesson I'm keeping: when git asks you the same yes/no question over and over, there's probably a flag that lets a script answer it instead. Next time something breaks "somewhere in the last N commits," write the five-line check script first — or better, write a tiny `check.sh` for your project this week and stash it somewhere you'll find it mid-incident.

What's the longest you've spent manually hunting a bad commit before someone showed you a better way?

---

## 4. SEO / Publishing Packet (dev.to)

- **Slug:** `til-git-bisect-run-automate-binary-search`
- **Meta / post description (≤160 chars):** Spent 3 hours manually bisecting ~200 commits. `git bisect run` + a 5-line script found the bad commit in 4 minutes. Here's the whole setup.
- **Tags:** `#git`, `#devops`, `#productivity`, `#todayilearned`
- **Cover image:** Optional; a plain terminal screenshot of `git bisect run` output would fit. Not required for a TIL.
- **Canonical URL:** If cross-posting from a personal blog later, set `canonical_url` in the dev.to front matter.
- **Suggested dev.to front matter:**
  ```yaml
  ---
  title: "TIL: git bisect Can Run the Whole Hunt for You"
  published: true
  tags: git, devops, productivity, todayilearned
  ---
  ```

---

## 5. Editor's Notes

- **Fresh-eyes review:** Done by an independent reviewer with only the raw notes + draft. Verdict: "publishable with light touch-ups." All touch-ups applied: deduped repeated "handy," split a 60-word sentence, replaced an invented teammate quote with narration matching the notes ("a teammate showed me"), removed the unbacked "maybe halfway there" claim, added a backgrounding hint to the sample script so it doesn't read as if `curl` would race the server.
- **Author-added facts to own before publishing (both are correct, documented git behavior, but they're not in your notes):**
  - Exit code `125` = "skip this commit" in `git bisect run`.
  - Keeping the check script outside the repo (git rewrites the working tree during bisect).
- **`[FILL]` markers to resolve (3):** last-known-good ref, your actual build/run commands, your real health endpoint URL.
- **Unicode check:** `log₂(200)` uses a subscript character — renders fine on dev.to, but preview it; swap to `log2(200)` if your pipeline mangles it.
- **Numbers verified:** ⌈log₂ 200⌉ = 8, matching your "8 automatic steps." Nice detail — leave it in.
- **Word count:** ~430 words for the post body — inside the 300–600 TIL band.
- **Assumptions made (no interview):** TIL archetype over full war story (notes are thin on incident drama, rich on the tip); zero images; engagement question kept because platform is dev.to.
