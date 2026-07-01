---
topic: book-summaries
status: wip
tags: [security, threat-modeling]
---

# fundamentals

The vocabulary and design principles the rest of the book builds on. Izar Tarandach and Matthew J. Coles, *Threat Modeling* (O'Reilly).

## the risk vocabulary

The terms form a chain, not a pile:

- **weakness**: a defect in design or implementation. May or may not be reachable.
- **vulnerability**: a weakness an attacker can actually reach and abuse.
- **exploitability**: how easily that abuse can happen, and with how much access.
- **zero-day**: a vulnerability attackers know about before a fix exists.
- **actor**: whoever acts on the system, benign or hostile.
- **threat**: the potential for a vulnerability to be exploited and cause harm.
- **threat event**: an actor's attempt to realize that potential.
- **impact / loss**: what the event costs you: money, data, reputation, function.
- **severity**: how bad the defect itself is, independent of context.
- **risk**: impact × likelihood the threat event happens. The only one of these you manage.

So: a weakness becomes a vulnerability when it's reachable, a vulnerability plus an actor with intent is a threat, and a threat weighted by likelihood and loss is risk.

## CVSS scores severity, not risk

CVSS tells you two things: the likelihood an attacker succeeds *once they try*, and how much damage they can do. It cannot tell you whether anyone will try, what the impacted system is worth, or what the fix costs. Those three (likelihood an attack gets initiated, value of the system or function, cost to mitigate) are what drive the risk calculation. Raw severity communicates a defect well and manages risk badly.

Two methods go further:

- **DREAD**: score Damage, Reproducibility, Exploitability, Affected users, Discoverability. Quick, but the scores are subjective.
- **FAIR** (Factor Analysis of Information Risk): models financial impact properly. Accurate but heavy. The calculations and simulations need computational and financial-modeling skill, which is not what security SMEs have: their expertise is finding weaknesses and threats, not valuing losses. Don't run FAIR live in a review session. Buy a tool or hire the skill if you adopt it.

## the properties you're protecting

**CIA**: confidentiality, integrity, availability. Plus two distinctions worth keeping:

- **confidentiality**: controlled access to private info that's been shared.
- **privacy**: the right to *not have that info exposed* to unauthorized third parties. Not the same thing, though people use the terms interchangeably and usually mean privacy when they say confidentiality. Confidentiality is a prerequisite for privacy.
- **integrity**: data and transactions haven't been tampered with.
- **availability**: the system is there when needed.
- **safety**: "freedom from unacceptable risk of physical injury or of damage to the health of people", directly or indirectly through damage to property or environment. Enters the model once software touches the physical world.

## identity and access

Three steps, in order: **identification** (who you claim to be), **authentication** (proving the claim), **authorization** (what the proven identity may do).

Four access-control models:

- **MAC** (mandatory): a central policy decides; owners can't override.
- **DAC** (discretionary): the resource owner grants access.
- **RBAC** (role-based): permissions attach to roles, users get roles.
- **capability-based**: holding an unforgeable token *is* the permission.

**Logging** and **auditing** back all four: access has to be reviewable after the fact.

## design principles

- **zero trust**: verify every request; nothing earns trust from where it sits.
- **least privilege**: each component gets the minimum access it needs.
- **defense in depth**: layered controls, so one failure isn't a breach.
- **separation of privilege**: no single condition should grant access on its own.
- **keep it simple**: complexity hides flaws.
- **no secret sauce**: no security through obscurity. The design must survive every implementation detail being known and published.
- **psychological acceptability**: humans are the weakest link. Users frustrated by strong security will route around it, so usability is a design constraint, not polish.
- **fail secure**: on failure, deny by default.
- **build in, not bolt on**: security designed in from the start, not patched onto a finished system.

## logging: what to capture, what to never log

A security analyst needs three answers from your logs:

- **who** performed the action that produced the event?
- **when** did it happen?
- **what** functionality or data did the process or user touch?

**Nonrepudiation** (closely tied to integrity): a record of who did what, where each entry keeps its own integrity, so no actor can deny having performed an action.

Never log:

- **PII** in plain text.
- **sensitive content** passed in API or function calls.
- **clear-text versions** of encrypted content.
- **cryptographic secrets**: passwords, decryption keys.
