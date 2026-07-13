# Blog Editor Deliverable

## Phase 1: Editor's Gate

✅ **Strong angle** — the piece has a sharp, unpopular takeaway backed by field experience: after a dozen mid-size consulting engagements, only a small minority actually needed microservices.

## Phase 2: Interview Notes

No interview questions asked because the author is unavailable. The supplied notes already answer the core angle, audience, platform, examples, and conditions where microservices make sense. Missing specifics are marked with `[FILL: ...]`.

## Phase 3: Writing Plan — Self-Approved

**Working title:** Most Teams Should Not Adopt Microservices

**Hook:** If an eight-person team with one product turns itself into fifteen services, it usually has not bought architectural freedom; it has bought a distributed systems department.

**Sections:**

1. **Microservices are not the default prize for getting serious** — establish the thesis and credibility from consulting across mid-size companies.
2. **The pattern I keep seeing** — show the common eight-person/one-product/fifteen-service failure mode and the 40% infrastructure tax.
3. **The monolith most teams actually need** — explain how a modular monolith captures most of the benefit without distributed coordination overhead.
4. **The uncomfortable reasons teams choose microservices anyway** — name resume-driven development and conference-talk envy without dunking on engineers.
5. **When microservices are the right answer** — provide a concrete decision filter: independent scaling, separate teams, compliance boundaries.

**Takeaway:** Engineering leads should make microservices earn their place with concrete organizational and operational needs, not use them as the default marker of maturity.

**Visuals:**

1. A comparison table contrasting “modular monolith” and “microservices” by team cost, operational burden, and when each fits. This replaces several paragraphs of explanation.
2. No diagrams or screenshots. The argument is conceptual and decision-oriented; extra visuals would decorate rather than clarify.

Plan self-approved per task instruction.

## Phase 5: Titles

1. **Recommended: Why Most Teams Should Not Adopt Microservices** — direct, opinionated, and honest about the promise the post keeps.
2. **The 40% Infrastructure Tax Hidden Inside Microservices** — uses the strongest concrete number from the notes and signals a cost-focused argument.
3. **Before You Split the Monolith, Make Microservices Earn It** — a bold practical frame aimed at engineering leads making architecture decisions.

## Phase 4: Post

# Why Most Teams Should Not Adopt Microservices

Most teams should not adopt microservices.

That sounds strange because microservices have become a kind of architectural graduation ceremony. You start with a monolith, you get serious, you split things up, and suddenly your diagrams look like conference slides. The boxes multiply. The arrows get more impressive. Everyone says “bounded context” more often.

Then the eight-person team with one product wakes up owning fifteen services and spends two days arguing about whether the problem is in service A, service B, the message broker, the gateway, or the tracing setup that was supposed to make this easier.

I have consulted for roughly a dozen mid-size companies over the last five years. Out of those, maybe two actually needed microservices. The rest needed clearer module boundaries, better deployment discipline, and fewer ways for a request to fail before lunch.

## The pattern is painfully repeatable

The version I keep seeing looks like this:

A team has one main product. It has maybe eight engineers. The monolith is annoying, but not catastrophic. Builds are slower than people want. Some files are too large. A few features are tangled together. Engineers complain that every change feels risky.

Those are real problems. They deserve attention.

But instead of fixing the boundaries inside the application, the team decides the boundary must be a network boundary. The product becomes ten, twelve, fifteen services. Each service needs a repository, pipeline, deployment config, ownership rules, API contracts, dashboards, alerts, secrets, versioning, and a plan for what happens when its friends are unavailable.

The team has not removed complexity. It has promoted complexity to infrastructure.

In one common pattern from my consulting work, a small team like this ends up spending around 40% of engineering time on plumbing: service mesh work, distributed tracing, contract tests, deployment coordination, environment drift, and debugging failures that only exist because the system is now distributed.

Some of that work is valuable if you need the architecture. If you do not, it is a very expensive way to avoid renaming a few modules.

## A modular monolith gets you most of the benefit

The uncomfortable truth is that a monolith with good module boundaries gives many teams about 90% of what they hoped microservices would provide.

You can still separate domains. You can still enforce ownership. You can still keep billing from reaching directly into notifications like a raccoon in a pantry. You can still create clean interfaces, test modules independently, and make it obvious where a change belongs.

The difference is that function calls stay function calls. Transactions can remain transactions. Local debugging still works. A developer can run the system without needing a miniature cloud platform on their laptop, three YAML files blessed by an infrastructure priest, and a lucky mug.

A modular monolith is not “just keep the mess.” That is the false choice that sends teams toward microservices too early. The real choice is between boundaries in code and boundaries over the network.

Network boundaries are more expensive. They are harder to change. They require observability, retries, timeouts, versioning, backward compatibility, and a serious understanding of failure modes. Those costs may be worth paying. But they are not free, and they are rarely the first bill a mid-size product team should volunteer to pay.

| Question | Modular monolith | Microservices |
|---|---|---|
| Best fit | One product, one team, shared release goals | Multiple teams, distinct services, independent release goals |
| Main benefit | Clear boundaries without distributed systems overhead | Independent deployment, scaling, and ownership |
| Hidden cost | Requires discipline to keep modules clean | Requires platform maturity and operational expertise |
| Failure mode | Big ball of mud if boundaries are ignored | Distributed big ball of mud, now with latency |
| Start here when | The pain is code organization | The pain is organizational or operational independence |

That last row matters. If your pain is code organization, fix code organization. If your pain is that separate teams cannot move independently, or one part of the system has radically different scaling or compliance needs, then microservices may be a serious answer.

## The real reasons are often not technical

Teams rarely say, “We chose microservices because the architecture looked good on resumes.”

But resume-driven development is real. So is conference-talk envy. Engineers are ambitious. Engineering leaders want to attract strong people. Nobody wants their stack to sound boring. A single deployable application with excellent internal boundaries does not make a flashy architecture diagram.

It does, however, ship features.

This is where leaders need to be honest about incentives. If the team is choosing microservices because the current system is hard to understand, that is not enough. If the team is choosing them because “that is what serious companies do,” that is worse. Serious companies also have teams dedicated to platform engineering, incident response, observability, reliability, and developer experience. Copying the service count without copying the support structure is not architecture. It is cosplay with pager alerts.

Microservices can make a strong team faster when the organization is shaped for them. They can also make a small team slower while everyone feels very sophisticated about the slowdown.

The question is not whether microservices are good or bad. The question is whether they solve your bottleneck.

For many mid-size companies, the bottleneck is not independent scaling. It is not separate compliance domains. It is not twenty engineers stepping on each other across multiple product areas. The bottleneck is unclear ownership, weak internal boundaries, inconsistent testing, and fear of changing old code.

A service boundary will not fix those habits. It will amplify them.

## When microservices do make sense

There are teams that should adopt microservices. I have seen a couple where the move was justified.

The strongest signal is independent scaling. If one part of the system has wildly different load, resource needs, or availability requirements, separating it may be reasonable. A reporting workload, real-time ingestion pipeline, or latency-sensitive API may deserve its own deployment and operational model. [FILL: specific anonymized example from your consulting work, if you can share one.]

Another signal is genuinely separate teams. Not “we hope to hire more people someday,” but multiple teams with clear ownership and enough engineering capacity to operate what they own. As a rough threshold, I start taking the argument more seriously around 20+ engineers, especially when different groups need to release independently.

Different compliance domains can also justify the split. If one part of the product handles data with stricter controls, audit requirements, or isolation needs, service boundaries may map to real risk boundaries. In that case, the architecture is serving governance, not fashion.

Those are concrete reasons. They are very different from “the monolith feels old.”

## Make the architecture prove its value

Before splitting a monolith, ask a few blunt questions:

- What exact problem becomes easier after the split?
- Which team owns each service, including incidents?
- What percentage of engineering time are we willing to spend on platform and coordination work?
- Can we get most of the value by enforcing module boundaries inside the current codebase?
- What new failure modes are we accepting?

If the answers are vague, pause.

The boring answer may be the right one: keep one deployable application, define stronger modules, clean up ownership, improve tests around the seams, and make deployment safer. That work does not look as impressive on a slide. It does not give every noun in your domain its own repository. It probably will not become a keynote.

But for most mid-size teams, it is the work that returns engineering time to the product.

This week, pick one area of your monolith that people are afraid to touch. Draw its real dependencies. Choose one boundary to enforce in code before you create a network boundary you will have to operate for years.

Where have you seen microservices help a team move faster — or quietly become the product?

## Editor's Notes

**Items only the author can supply:**

- `[FILL: specific anonymized example from your consulting work, if you can share one.]`
- Optional: one concrete “before and after” outcome from one of the two companies that genuinely needed microservices.
- Optional: a dev.to cover image, such as a simple split-screen: “8 engineers / 1 product” on one side and “15 services / 40% plumbing” on the other.

**Visuals to create:**

- The included markdown comparison table is the primary visual. No screenshot or diagram is necessary.

**One-line social summary:**

After consulting for a dozen mid-size companies, I have seen microservices help far fewer teams than they slow down; most teams need better module boundaries, not more services.

**Revision offer:**

I can do one revision pass for sharper tone, stronger dev.to hook, or a more diplomatic version for leadership audiences.
