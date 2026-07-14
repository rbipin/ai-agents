# Deliverable: TIL — git bisect run

## Writing Plan (self-approved — author unavailable for interview)

- **Editor's verdict:** Strong angle. A real incident with hard numbers (3 hours manual vs. ~4 minutes / 8 automatic steps across ~200 commits) attached to a genuinely underused git feature.
- **Archetype:** TIL (with a war-story hook). Target ~300–600 words, quick, humble, generous.
- **Audience / platform:** Any developer / dev.to.
- **Hook:** "Yesterday I spent three hours playing human binary search."
- **Structure:**
  1. The manual ritual and the teammate's one-liner (hook)
  2. What `git bisect run` does — the exit-code contract
  3. The tiny test script (build → start → curl health endpoint)
  4. The payoff numbers and the generalization
- **Core takeaway:** If "is this commit good?" can be expressed as an exit code, you never have to bisect by hand again.
- **Visuals:** None. Two short code blocks carry the post; a diagram would pad a TIL. (Optional: a terminal screenshot of real `git bisect run` output would add authenticity — see editor's notes.)
- **Assumptions made (author unavailable):** exact build command, health-endpoint URL, and last-known-good ref are unknown → marked with `[FILL]`. The script shown is representative, reconstructed from the notes ("curl'd the health endpoint after a local build").

## Three Title Options

1. **From 3 hours to 4 minutes: let `git bisect run` do the bug hunt** ← *recommended: concrete outcome + numbers, the post's strongest asset*
2. **TIL: git bisect can run your test script and find the bad commit for you** — classic dev.to TIL framing, curiosity-driven
3. **Stop checking out commits by hand — git bisect run automates the binary search** — imperative/pain-point framing for readers mid-suffering

---

## The Post

Yesterday I spent three hours playing human binary search.

Our staging deploy was broken, and the breakage was hiding somewhere in a stretch of roughly 200 commits. So I did the ritual every developer knows: check out a commit, build, hit the health endpoint, squint at the result, check out another commit. For three hours.

Then a teammate looked over my shoulder and asked why I wasn't using `git bisect run`. I knew `git bisect`. I did not know about `run`.

## The part of bisect I'd been doing by hand

Plain `git bisect` already does the smart part: you mark one good commit and one bad commit, and it binary-searches through history, halving the suspects each round. But at every step it stops and waits for *you* to test the checkout and type `git bisect good` or `git bisect bad`.

`git bisect run` removes you from the loop. You give it a script, and git runs it on every candidate commit:

- exits `0` → commit is good
- exits `1`–`127` (except 125) → commit is bad
- exits `125` → commit can't be tested, skip it
- exits above `127` → the whole bisect aborts, so a script that dies from a signal won't silently mislabel commits

```bash
git bisect start
git bisect bad HEAD
git bisect good <sha>   # [FILL: your last known-good ref]
git bisect run ./check.sh
```

Then you go get coffee.

## The test script was embarrassingly small

Mine built the project, started it locally, and curled the health endpoint. `curl -f` exits non-zero on an HTTP error status, so the health check *is* the test. No output parsing needed.

```bash
#!/usr/bin/env bash
./build.sh || exit 125                  # [FILL: your build command]
./start-app.sh &                        # [FILL: however you launch the app locally]
sleep 5                                 # let it boot
curl -sf http://localhost:8080/health   # [FILL: your health endpoint]
status=$?
kill $!                                 # tidy up before the next round
exit $status
```

The `exit 125` on a failed build matters more than it looks: it tells bisect "skip this commit" instead of wrongly blaming a commit that merely doesn't compile.

## Three hours by hand, four minutes on autopilot

The math was always in bisect's favor: log₂(200) ≈ 8, so ~200 commits only need about 8 test rounds. `git bisect run` did exactly that. Eight automatic steps, about four minutes, and it printed the exact commit that broke the deploy. My three hours were pure human-in-the-loop overhead: checking out, rebuilding, retesting, losing my place.

If you can express "is this commit good?" as an exit code, you never have to bisect by hand again. That covers more than broken deploys: a script can run one flaky test, time a benchmark and fail past a threshold, or grep build output.

Next time something breaks "somewhere in the last N commits," write the tiny script *first*. Future you gets to drink the coffee while git does the search.

What's the most time you've sunk into manually hunting a bad commit?

---

## Editor's Notes

**Before publishing, replace the `[FILL]` markers:**
- `[FILL: your last known-good ref]` — the actual sha/tag you bisected from
- `[FILL: your build command]` — the real build step
- `[FILL: however you launch the app locally]` — if your build script also starts the app, collapse these two lines and say so
- `[FILL: your health endpoint]` — the real URL/port

**Honesty note:** the script shown is a *representative reconstruction* — the notes only say "curl'd the health endpoint after a local build." Please swap in (or verify against) the actual script; in particular confirm whether the app was started separately or by the build step, and whether cleanup between rounds was needed.

**Fresh-eyes review — applied fixes:**
- Added the exits-above-127-aborts case (the exit-code list previously read as exhaustive and wasn't)
- Added the missing server-start step (a skeptical reader would ask "what is curl talking to?")
- Cut the "The general lesson:" essay-style signpost
- Trimmed the closing question from two questions to one
- Reduced em-dash density and converted one bullet list to prose to break uniform rhythm

**Fresh-eyes review — confirmed good:** hook lands, coffee setup/payoff callback works, tone is properly TIL, all claims trace to the notes, technical facts (exit-code semantics, `curl -f` behavior, log₂(200) ≈ 8) verified correct.

**dev.to specifics:**
- Suggested tags: `git`, `todayilearned`, `productivity`, `devops`
- Cover image optional for a TIL; a terminal screenshot of real `git bisect run` output ending in the culprit-commit line would be the highest-value visual if you want one
- Word count: ~470 (within TIL target of 300–600)
