# Blog Post Deliverable

## Plan

### Phase 1: Editor's Gate

✅ **Strong angle** — this works because it turns a common developer pain (manual regression hunting) into one concrete, immediately useful trick, backed by a real before/after: 3 hours manually checking commits versus 4 minutes with `git bisect run`.

**Archetype:** TIL, with a light incident/war-story opening.

### Phase 2: Interview

No interview questions asked per instruction. I treated the supplied notes as the source of truth and used `[FILL: ...]` markers where specific details were missing.

### Phase 3: Writing Plan

**Archetype:** TIL, borrowing a little from incident storytelling.

**Working title:** TIL: `git bisect run` Can Find a Bad Commit for You

**Hook:** I spent 3 hours manually checking commits before learning Git could have automated the search.

**Sections:**

1. **The slow way I started** — establish the relatable pain: checking commits by hand across a ~200-commit range.
2. **The tiny command I wish I knew earlier** — introduce `git bisect run` and the good/bad exit-code contract.
3. **The script was boring, which was the point** — show the minimal shape of a health-check script using placeholders for missing project-specific details.
4. **What changed** — contrast 3 hours of manual work with 4 minutes and 8 automatic steps.
5. **The next time a regression appears** — give the reader a concrete next action.

**Takeaway:** If you can express “good” versus “bad” as a script exit code, `git bisect run` can automate the regression search.

**Visuals:** None. For dev.to, the code snippet explains the workflow more clearly than a diagram would.

**Self-approval:** Approved to write because the plan keeps one TIL-sized idea, uses only the supplied facts, and leaves placeholders where project-specific details are missing.

## Title Options

1. **Recommended: TIL: `git bisect run` Can Find a Bad Commit for You**  
   *Strategy: clear TIL promise.* This is the best fit for dev.to because it is direct, searchable, and honest about the scope.

2. **How I Cut a 3-Hour Git Hunt to 4 Minutes**  
   *Strategy: how-title with concrete before/after.* Stronger story hook, but slightly less explicit about the command.

3. **8 Automatic Steps Beat 200 Manual Commit Checks**  
   *Strategy: numbers + contrast.* Punchy, but it needs the subtitle or opening to reveal `git bisect run` quickly.

## Post

# TIL: `git bisect run` Can Find a Bad Commit for You

Yesterday I spent about three hours doing the kind of debugging that makes you question your life choices: checking out commits one by one, trying to find which change in a range of roughly 200 had broken our staging deploy.

I had a local build. I had a health endpoint. I had determination.

What I did not have, apparently, was awareness of `git bisect run`.

## I was doing the binary search by hand

I knew about `git bisect` in the general sense: mark one commit as good, mark another as bad, and Git helps you narrow down the commit that introduced the problem.

But I was still in the loop manually:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Then Git would check out a candidate commit, and I would build, run the app, hit the health endpoint, decide whether it was good or bad, type the next command, and repeat.

That is already better than walking 200 commits linearly. It is also still a lot of human babysitting.

A teammate saw what I was doing and pointed out the part I had missed: if the check can be scripted, Git can run the whole bisect for you.

## The trick is just an exit code

`git bisect run` takes a command or script and runs it at each candidate commit.

The contract is wonderfully simple:

- exit `0` means the commit is good
- non-zero means the commit is bad

So instead of manually building and checking the deploy health each time, I could wrap that test in a small script:

```bash
#!/usr/bin/env bash
set -e

[FILL: build command]
curl -fsS [FILL: local health endpoint]
```

Then run:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
git bisect run ./check-staging-health.sh
```

That was it. Git checked out each candidate commit, ran the script, read the exit code, and moved to the next point in the search.

## The result was annoyingly good

The manual version had already eaten about three hours.

The automated version found the culprit commit in about four minutes and eight steps.

That number makes sense once you remember what bisect is doing. With around 200 commits, Git does not need to test anywhere close to 200 of them. It keeps cutting the search space in half, and the script removes me from the repetitive part.

The nice part is that the script did not need to be clever. It only needed to answer one question reliably: “Does this commit pass the health check?”

If yes, exit `0`. If no, exit non-zero.

## The part I am taking with me

The useful lesson was not just “use `git bisect`.” I sort of knew that already.

The lesson was: when you can turn a regression check into a script, you can stop being the test runner.

A health endpoint works. A unit test works. A CLI smoke test works. A curl command works. The only thing Git needs is a dependable signal.

Next time I hit a regression across a wide commit range, I am going to ask one question before doing anything manually:

> Can I write a tiny script that says good or bad?

If the answer is yes, `git bisect run` gets the job.

What is the most useful tiny Git command you learned embarrassingly late?

## Editor's Notes

### `[FILL: ...]` items only the user can supply

- `[FILL: build command]` — the exact local build command used before the health check.
- `[FILL: local health endpoint]` — the exact URL or endpoint that was curled.
- `<known-good-commit>` — optional: replace with the real known-good commit hash if you want the example to feel more concrete.

### Fresh-eyes review summary

Performed as a separate self-review pass because nested delegation was not available under the current execution constraints.

- **Spelling & grammar:** Cleaned for grammar, markdown formatting, and code fences.
- **Emotion:** Hook is direct and relatable; the light humor comes from the real debugging pain and is not overused.
- **Content:** All concrete claims trace back to the notes: ~200 commits, 3 hours manual work, 4 minutes, 8 automatic steps, health endpoint after local build.
- **AI-tells:** Removed generic setup language and avoided banned phrases.

### Reviewer flags

- The script is intentionally generic because the build command and endpoint were not provided. Filling those in would make the post feel more grounded.
- If the staging failure involved a memorable symptom, error message, or Friday-afternoon timing, one sentence could strengthen the opening without changing the TIL format.

### Visuals to create

None recommended. A diagram would decorate rather than clarify; the short command sequence and script snippet are enough.

### One-line social summary

I spent 3 hours manually hunting a bad commit, then learned `git bisect run` could do the same search in 4 minutes with one tiny health-check script.
