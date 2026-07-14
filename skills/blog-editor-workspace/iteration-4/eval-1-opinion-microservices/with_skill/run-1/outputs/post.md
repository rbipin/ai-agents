# Blog Editor Deliverable — Opinion Piece: "Most Teams Don't Need Microservices"

This file contains the complete output of the blog-editor skill run: the plan,
three candidate titles, the finished post, the SEO packet, and editor's notes.
Interview phase was run autonomously (user unavailable) — assumptions are
called out below and gaps are marked `[FILL: ...]` inline in the post.

---

## Phase 1 — Editor's Gate (verdict)

**✅ Strong angle.** The "2 of 12 companies actually needed it" data point,
paired with the specific 40%-time-on-plumbing failure pattern, gives this
real teeth over the generic monolith-vs-microservices takes that flood this
topic — it's hard-won consulting experience, not a vibes-based opinion.

**Archetype:** Opinion / hot take — confident but fair, bold claim up front,
steelman of the other side, concrete criteria instead of vibes.

---

## Phase 2 — Interview

Skipped by user request (unavailable to answer questions). Proceeding
autonomously per Phase 3 self-approval, with the following assumptions
made explicit and gaps marked `[FILL]` in the draft itself:

- Assumed "mid-size companies" in the notes means roughly 50-500 employee
  engineering orgs, consistent with an "engineering leads and staff
  engineers at mid-size companies" audience.
- Assumed the specific breakdown of where the 40% of engineering time goes
  (mesh upkeep, tracing, contract-test maintenance) is illustrative of the
  named categories in the notes, not a precise measured audit — flagged as
  `[FILL]` in the post rather than presenting invented sub-numbers as fact.
- No specific company names, quotes, or benchmarks were invented — the
  fresh-eyes review (Phase 5) caught and removed several examples that had
  crept in with invented specifics (see "What the review caught," below).

---

## Phase 3 — The Writing Plan (self-approved)

- **Archetype:** Opinion / hot take.
- **Working title:** "Most Teams Don't Need Microservices (Here's the Math)"
- **Search phrase:** "when should you use microservices" — anchors the
  title, main heading, and opening paragraphs.
- **Hook:** Bold claim up front — "In five years consulting for a dozen
  mid-size companies, I've seen exactly two that needed microservices."
- **Sections:**
  1. The pattern I keep seeing (8-person team, 15 services, 40% time on
     plumbing) — establishes the concrete failure mode.
  2. Why teams actually do this (resume-driven development, conference
     envy) — the honest, slightly uncomfortable truth.
  3. Steelman: when microservices genuinely make sense (independent
     scaling, 20+ engineer separate teams, compliance domains) — fairness
     required by the opinion archetype.
  4. The monolith gets you 90% of the benefit — modular monolith as the
     practical alternative, with a comparison table.
  5. A concrete test/checklist readers can apply this week.
- **Takeaway:** a three-question test for whether a specific split is
  justified, plus a concrete next action (draw module boundaries this
  week) and an engagement question for dev.to's comment-driven reach.
- **Visuals:** one comparison table (modular monolith vs. microservices
  criteria) — earns its place by replacing what would otherwise be a
  paragraph of hedged criteria; no Mermaid diagram needed since there's no
  process/flow to show, and zero screenshots since there's no artifact to
  display.

No split recommended — this is one core idea (a decision framework), not
two posts.

---

## Phase 4-5 — Draft & Fresh-Eyes Review

Drafted the full post, then sent it to an independent subagent reviewer
with only the draft and the raw notes (not this plan) to review as a
first-time reader, per the skill's Phase 5 instructions.

**What the review caught and I fixed silently:**
- An inline `[FILL: ...]` bracket note had leaked mid-paragraph in-line —
  confirmed this was intentional (a genuine gap) and kept it, but verified
  it reads cleanly as a marker rather than broken prose.
- "Six months later they have 15 services" — the six-month timeframe
  wasn't in the notes; softened to "not long after" rather than inventing
  a specific timeline.
- An invented example contrasting "Kubernetes, gRPC, event-driven
  architecture" against a Rails app serving "ten times the traffic" and
  conference talks about scaling to "a billion requests" with "200
  services" — none of these specifics were in the notes. Replaced with
  general, non-fabricated language ("a well-organized monolith... the one
  actually carrying the business," "companies operating at a completely
  different scale").
- The video-transcoding example under "independent scaling needs" wasn't
  in the notes either — reframed explicitly as an illustrative "say,"
  rather than presenting it as an observed case.

**Reviewer flags left for the user's judgment (not auto-applied):**
- The title promises "here's the math," but the only hard number
  substantiated in the post is the 40% figure, which is itself marked
  `[FILL]`. If you can supply a real breakdown or drop the `[FILL]`, the
  title's promise lands harder. Alternatively, soften the title if you'd
  rather not lean on "math."
- No spelling/grammar/AI-tell issues were found — the reviewer confirmed
  no banned phrases, no keyword stuffing, and natural paragraph rhythm.

---

## Phase 6 — Three Titles

1. **"Most Teams Don't Need Microservices (Here's the Math)"** — bold
   promise, and the one I'd run with: it front-loads the contrarian claim
   the audience will click for, and "the math" sets up the 2-of-12 and 40%
   numbers without overpromising a spreadsheet.
2. **"2 Out of 12: How Rarely Microservices Actually Pay Off"** — number +
   concrete takeaway strategy, leans hardest into the consulting stat.
3. **"Why Your 8-Person Team Doesn't Need 15 Microservices"** — why +
   specific promise strategy, most concrete about the exact failure
   pattern described in the post.

**Recommended: #1.** It's the boldest without overreaching, and it's the
only one of the three that also carries the search phrase's intent
("when should you use microservices") implicitly in its framing.

---

## The Post

# Most Teams Don't Need Microservices (Here's the Math)

In five years consulting for a dozen mid-size companies, I've seen exactly two that actually needed microservices. The other ten split up anyway — and spent months paying for it.

If you're an engineering lead weighing "should we finally break up the monolith," here's the pattern I keep seeing, and the criteria I now use instead of vibes.

## The pattern: 8 people, 15 services, no time for features

The setup is almost always the same. An 8-person team owns one product. Someone reads a blog post (maybe this one, ironically) or watches a conference talk, and the team decides it's time to "do microservices properly." Not long after, they have 15 services.

Now every feature that used to be a function call is a network call. Every deploy needs a service mesh to route it. Every bug report needs distributed tracing to even locate the failure. Every contract between two services needs a contract test to keep it from silently breaking. None of this is optional once you've split — it's the tax you pay for the split, and it doesn't scale down.

Across the teams I've watched go through this, the number that sticks with me is 40%: that's roughly how much of the engineering week goes to infrastructure plumbing instead of the product — keeping the mesh healthy, chasing a trace across six services to find one slow query, updating contract tests because a teammate renamed a field. `[FILL: cite the specific consulting engagements or a rough breakdown of where the 40% goes, if you want to name names or show a rough time-audit]`

Eight people can build a lot of product. Split into 15 services, and most of those eight are now also part-time platform engineers, whether they wanted the job or not.

## Why teams actually do this

Ask a team why they split into microservices and you'll hear "scalability" or "team autonomy." Watch what actually happens and it's usually simpler: resume-driven development and conference-talk envy.

Microservices look good on a resume. "Designed and operated a distributed system" reads better than "maintained a well-organized monolith," even when the monolith was the one actually carrying the business. And after watching enough conference talks from companies operating at a completely different scale, it starts to feel like the thing serious engineering orgs do — regardless of whether your 8-person team is anywhere near that problem.

I don't think this is usually cynical. It's more that "we should modernize our architecture" is an easier sentence to say out loud than "we're bored of maintaining this codebase" or "I want microservices experience on my resume." But the effect on the team is the same either way: complexity taken on for reasons that have nothing to do with the product's actual needs.

## When microservices genuinely earn their keep

To be fair to the other side: I have seen it work, and it's worth being precise about when.

- **Independent scaling needs.** One part of the system genuinely needs to scale differently from the rest — say, a background processing pipeline that needs to burst hard while the rest of the app sits at steady load. Splitting it out means you scale the expensive part without paying to scale everything else.
- **Genuinely separate teams.** Once you're past roughly 20 engineers organized into teams that ship independently, a shared monolith becomes its own coordination tax — merge conflicts, deploy queues, one team's bug freezing everyone's release. At that size, service boundaries can mirror team boundaries and reduce friction instead of adding it.
- **Different compliance domains.** Payment data under PCI DSS and marketing analytics data have no business sharing a deploy pipeline or a data store. Regulatory isolation is one of the few reasons to split that has nothing to do with scale or team size at all.

Notice what's common to all three: they're properties of your organization or your regulatory environment, not aspirations for what your architecture "should" look like. If none of the three apply to you, that's your answer.

## The monolith gets you 90% of the benefit

Here's what almost nobody pitches at conferences: a monolith with well-drawn module boundaries gets you most of what people actually want from microservices, without the network hop.

Want independent ownership of a domain? Enforce module boundaries with a linter and clear package structure — no team touches another team's package without a reviewed interface. Want to reason about one part of the system without loading the whole thing in your head? A well-factored module gives you that; you don't need a wire protocol to get isolation.

What you don't get, and don't need yet, is independent deployability and independent scaling — the two things that actually require a network boundary. Most 8-person teams aren't limited by either of those. They're limited by how much product they can ship, and the monolith doesn't take away from that budget the way 15 services do.

| | Modular monolith | Microservices |
|---|---|---|
| Team size it fits | Under ~20 engineers, one product | 20+ engineers, multiple independent teams |
| Deploy story | One deploy, one rollback | Coordinated or independent deploys, more moving parts |
| Debugging a request | Stack trace, one process | Distributed tracing across services |
| Infra tax | Low — CI/CD for one app | Service mesh, contract tests, service discovery |
| Scaling | Scale the whole app together | Scale each service independently |
| Where it breaks down | Org outgrows one deployable unit | Org is too small to staff the plumbing |

## A test you can run this week

You don't need a six-month re-architecture debate to answer this. Pick one service you're considering splitting off, and ask three questions: Does it need to scale independently from the rest of the app under real load you've actually measured? Is it owned by a team that ships on its own schedule, separate from everyone else? Does it sit in a different compliance boundary that legally can't share infrastructure with the rest?

If you answered no to all three, you don't have a microservices problem — you have a module-boundary problem, and a linter rule is cheaper than a service mesh.

If your team is smaller than 20 engineers and none of the three apply, the highest-leverage thing you can do this week isn't drawing service boundaries — it's drawing module boundaries inside the app you already have.

What's the split you're most tempted to make right now, and which of the three questions does it actually answer yes to?

---

## SEO Packet

- **Meta description:** "After 5 years consulting for a dozen mid-size
  companies, I've seen 2 that needed microservices. Here's the concrete
  test to know if you're one of them."
  (152 characters — honest pitch, no keyword pile-up.)
- **URL slug:** `most-teams-dont-need-microservices`
- **Canonical note:** If this is cross-posted anywhere beyond dev.to
  (personal blog, Medium, etc.), set the canonical URL to wherever it's
  published first so the copies don't compete with each other in search.

---

## Editor's Notes

**`[FILL]` items only the user can supply:**
- The 40%-of-engineering-time breakdown in section 1 — currently
  presented as an approximate, observed pattern across engagements. If
  you have a real time-audit, named case, or more precise number, add it;
  otherwise consider softening "40%" to "a third to half" if you're not
  confident defending the specific figure under scrutiny.

**Reviewer flags needing your judgment:**
- The title's "here's the math" promise is only as strong as the 40%
  figure above — resolve the `[FILL]` or consider a softer title (see
  title #3) if you'd rather not lean on precision you can't back up.

**Visuals to create:** none required — the piece uses one markdown
comparison table (already in the post) instead of a diagram or
screenshot, since there's no process flow or UI artifact to show.

**One-line summary for social:** "I've consulted for a dozen mid-size
companies over 5 years — only 2 actually needed microservices. Here's the
math on why most teams should stick with a modular monolith."
