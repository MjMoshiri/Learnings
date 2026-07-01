---
topic: book-summaries
status: wip
tags: [security, threat-modeling]
---

# fundamentals

The vocabulary and design principles the rest of the book builds on. Izar Tarandach and Matthew J. Coles, *Threat Modeling* (O'Reilly).

## the risk vocabulary

- **weakness**: a flaw in design or implementation that could be abused.
- **vulnerability**: a weakness that's actually reachable and abusable.
- **exploitability**: how hard it is to turn that vulnerability into an attack.
- **zero-day**: a vulnerability with no fix available yet, known to attackers before defenders.
- **actor**: whoever is acting on the system, benign or hostile.
- **threat**: the potential for harm. **threat event**: an attempt to realize it.
- **severity**: how bad the defect is on its own.
- **impact** / **loss**: what it costs you if the threat lands.
- **risk**: the thing you actually manage. Not severity.

## CVSS scores severity, not risk

CVSS tells you the likelihood an attacker *succeeds* once they try, and how much damage they can do. It says nothing about *when or whether* anyone attempts the exploit, what the impacted system is worth, or what the fix costs.

Risk is driven by three things CVSS ignores:

1. likelihood an attack even gets started,
2. value of the system or function under attack,
3. cost to mitigate.

Raw severity describes a defect well and manages risk badly. Two methods go further: **DREAD** (a rough scoring mnemonic) and **FAIR** (Factor Analysis of Information Risk), which models financial impact properly. FAIR is accurate but heavy. It needs computational and financial-modeling skill, not the security expertise your SMEs have. Don't run it live in a review session. Use a tool or a specialist if you adopt it.

## the properties you're protecting

**CIA**: confidentiality, integrity, availability.

- **confidentiality vs privacy** aren't the same. Confidentiality is controlled access to private info you've shared. Privacy is the right to *not have that info exposed* to unauthorized third parties. People say confidentiality when they mean privacy. Confidentiality is a prerequisite for privacy, not a synonym.
- **integrity**: the data or transaction hasn't been tampered with.
- **availability**: it's there when you need it.
- **safety**: freedom from unacceptable risk of physical injury or health damage, direct or through property or the environment. Matters once software touches the physical world.

## identity and access

Three steps, in order: **identification** (who you claim to be), **authentication** (proving it), **authorization** (what you're then allowed to do).

Access-control models:

- **MAC** (mandatory): a central policy decides, users can't override.
- **DAC** (discretionary): the resource owner grants access.
- **RBAC** (role-based): permissions attach to roles, users get roles.
- **capability-based**: holding an unforgeable token *is* the permission.

Backed by **logging** and **auditing** so access can be reviewed after the fact.

## design principles

- **zero trust**: no implicit trust from network location. Verify every request.
- **least privilege**: give each component the minimum access it needs, nothing spare.
- **defense in depth**: layered controls, so one failure isn't a breach.
- **keep it simple**: complexity hides flaws.
- **no secret sauce**: don't rely on obscurity. The design should hold even if every detail is public.
- **separation of privilege**: require more than one condition to grant access.
- **psychological acceptability**: humans are the weakest link. Security users find annoying gets routed around, so usability is a security constraint.
- **fail secure**: on failure, deny by default.
- **build in, not bolt on**: security designed from the start, not patched over a finished system.

## logging: what to capture, what to never log

A security analyst reviewing an event needs to answer three questions:

- **who** performed the action?
- **when** did it happen?
- **what** data or functionality did they touch?

**Nonrepudiation** (close to integrity): a tamper-evident record of who did what, so an actor can't later deny an action.

Knowing what *not* to log matters just as much. Never write to logs:

- **PII** in plain text.
- **sensitive content** passed in API or function calls.
- **clear-text** versions of anything encrypted.
- **cryptographic secrets**: passwords, keys.
