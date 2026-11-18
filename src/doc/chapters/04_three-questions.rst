
.. _chapter-three-questions:

Three Fundamental Questions
===========================

.. note::

   This chapter is structured around three questions: *what* should be
   attested for an AI agent, *how* an attestation should be produced and
   consumed, and *when* it should be produced. The text below frames each
   question from the position of the party that must **act on** an
   attestation — the relying party — because the consumption side is where
   agent attestation currently has the least shared vocabulary. The
   contributions below are offered as material for discussion, not as
   settled positions.

What to Attest?
---------------

An attestation is only useful if the party receiving it can determine what
claim it is entitled to draw from it. For AI agents, the candidate objects of
attestation are not mutually exclusive, and they are not equivalent in
strength:

- **The agent type or deployment.** Which agent implementation, build, or
  deployment is this — independent of any particular execution.

- **Code and configuration.** The instructions and settings that determine
  what the agent can do, and under which limits.

- **Model and weights.** The model artefact actually loaded, where the agent
  is model-driven.

- **Runtime state actually exercised.** The state, context and reachable
  tool surface at the moment of the action, which may differ from the state
  at load.

- **Behaviour observed during operation.** What the agent actually did, as
  distinct from what it is capable of doing.

- **The authored policy set.** The identity and version of the rules the
  agent's actions are evaluated against, and the authority under which those
  rules were authored. Behaviour is only meaningful against a reference, so the
  reference is itself a candidate object of attestation. It changes
  independently of code, model and runtime state; the timing side of that
  independence is the *At policy change* trigger in *When to Attest?* below.

The difficulty is that these objects have different lifetimes and different
verification costs, and a single attestation cannot carry all of them at
equal strength. The distinction that matters at the point of reliance is
between a **static claim** ("this artefact is what it says it is") and an
**instance claim** ("*this* execution, with *this* state, acted under *this*
authority"). For agents whose behaviour is probabilistic, a static claim
alone does not support a consequential decision.

One way to answer it is to separate three things that are often
conflated in a single identifier: the identifier of the agent *type*, the
identifier of the *instance*, and the statement of *authority* under which the
instance acted. Regimes in other regulated domains separate a type-level
identifier from an instance- or production-level identifier for exactly this
reason, and the separation is what allows a consumer to cache the first and
must re-check the second. The semantics of the third object are fixed
elsewhere: the charter of Theme #5 (FG-TIDA/themes#5), carried as PR #15
(FG-TIDA/themes#15), names *standing of authority* in its Scope section as
one property with two determinations, at origination and at act time. The
record-carriage requirements are fixed in that charter's scope element 2,
and the composition and revocation semantics in scope element 5; the
chapter claims nothing for the third object beyond what that charter fixes.

A **conformance verdict** is a behavioural form of the instance claim: the
result of evaluating one specific action against the authored policy set. It
is a claim by the engine that issued it, so it should carry the identity of
that engine. Carrying the issuer is necessary for the verdict to stand as an
instance claim, but it is not sufficient on its own. A verdict should also be
re-verifiable offline by the relying party from the evidence package: bound to
the action, to the version of the policy set and to the issuing engine, with
its evaluation parameters taken from the policy set. It should be reproducible,
so that a party holding the identified engine version and the policy set can
recompute it. Where a decision needs more than the producer's word, the
relying party's policy can require an issuer independent of the agent.

A verdict is more usable when it is drawn from a small closed vocabulary than
when it is free text, because a closed set can be compared across
heterogeneous evidence. One such set is the vocabulary stated in the themes
discussion on intent-based security policies (issue 6) — *permit*,
*remediate*, *block*, *escalate*, *indeterminate*. Which terms a particular
profile adopts is a matter for that profile; what matters here is that the
terms are fixed and shared, rather than chosen by each producer.

One distinction is worth keeping: a verifier's *no-assertion* result is
evidence-side, while an *indeterminate* verdict is action-side. The two
should not be read as the same state.

How to Attest?
--------------

For agentic AI, the binding constraint is **not the generation of an
attestation but its consumption**. An attestation produced by any mechanism
(a hardware quote, a trusted-execution report, a signed manifest) still has to
be *accepted* by a relying party that did not run the attestation itself.
Three properties must be established at the point of consumption, and each is
independently failable:

1. **Binding** — the evidence is bound to *this* agent instance, for *this*
   action, and not merely to a binary, an image, or a logical identity. Where
   behaviour is probabilistic, binding to a static artefact is insufficient;
   the evidence must reference the runtime state actually exercised.

2. **Freshness** — the evidence is not a replay. A valid attestation from an
   earlier session must not be accepted for a later action. This requires an
   explicit, verifiable notion of *when* the evidence was produced relative to
   the action (see *When to Attest?* below).

3. **Offline re-verifiability** — the relying party can check, offline and from
   the evidence package alone, that the verdict is bound to *this* action, to
   the version of the policy set and to the issuing engine, with its evaluation
   parameters taken from the policy set. The verdict should also be
   reproducible, so that a party holding the identified engine version and the
   policy set — an auditor, say — can recompute it. Neither step requires the
   producer's infrastructure to be online, and neither leaves the relying party
   to trust a verdict that only the producer is able to compute.

**How the three can be met together.** They are not in tension with one
another, and a single construction can carry all of them. Such a construction
moves away from one opaque attestation and towards a *verifiable evidence
package*: the evidence supporting an action is sealed under a digest, the
digest is published to a public transparency log so that its existence at a
given point in time can be checked by anyone, and the package is archived
under a persistent identifier so that its contents can be confirmed unmodified
after release. A relying party can then re-compute the package digest, look up
the log entry, and confirm the archive anchor — all of it offline, without
contacting the producer, and without having to trust a verdict that only the
producer is able to compute.

A construction of this kind is a proposal rather than a formal standard, and it
should scope its results accordingly: a pass means that no violation was found
within the observed evidence boundary, not that trustworthiness is asserted in
general. That scoping is part of the contribution. A verifier-side artefact
that does not state the limits of its own verdict is itself an instance of the
problem this chapter examines.

The shape is described here not as a mechanism proposed for adoption in this
chapter, but to record that the three properties above are jointly achievable
using existing, general-purpose building blocks.

**A minimal verifier-side checklist (proposed, for discussion).** For evidence
claiming to support a consequential agent action, a relying party should be
able to answer, from the package alone:

- Does the evidence bind to the specific agent instance *and* the specific
  action?
- Is the freshness window explicit, and is the evidence inside it?
- Is the verdict bound offline to the action, the policy set version and the
  engine, and reproducible by a party holding the engine version and the
  policy set?
- If verification of any of the above cannot be completed, is the default
  outcome to **deny** the action rather than to allow it?

The last point is where current practice appears weakest. The failure mode is
the one recorded in the Problem Statement section of this document,
and it is stated there rather than restated here: the check silently passed
because nothing was actually checked. What this chapter adds is not a second
statement of the mode but a concrete form for it. A catalogue of such cases —
verification steps that report success while checking nothing — would be a
modest but concrete contribution to this chapter, because failure semantics
cannot be specified before the failure modes are enumerated. Entries in such a
catalogue are worth recording in plain form: what happened, the root cause, and
the change that addressed it, so that others can extend the record with their
own cases.

Two instances are worth naming, because both were found by running a test
corpus rather than by review. In the first, a comparison silently collapsed two
distinct items into one before comparing them, and the check reported that
nothing had been altered. In the second, a conformance check scored a pass on
an empty input, because every assertion it made was vacuously true. Both belong
to the same class: the check ran, and it reported success while checking
nothing.

**Container and canonicalisation.** Two consequences of the properties above are
worth stating, because a profile inherits them whether it addresses them or not.

The first is *canonicalisation*. Any signed structure depends on exactly which
bytes are signed. The Entity Attestation Token (EAT) is published as RFC 9711
(April 2025), carried in CBOR, whose canonical encoding is specified in RFC
8949; a profile that signs a serialisation without fixing a canonical form
inherits a class of disagreement that a relying party cannot distinguish from
tampering. This is not specific to agents, and it is repeated here because a
verdict that cannot be re-derived offline does not satisfy property 3.

The second is that *the token is not the claim*. EAT, and the remote
attestation roles it assumes (RFC 9334), supply carriage and an evidence
architecture; they do not supply the vocabulary of authority, decision and
refusal this chapter asks for. An earlier attempt to add capability claims to
EAT remained an individual draft - `draft-huang-rats-agentic-eat-cap-attest`,
last revised 2025-06-13 - which expired without adoption. It is recorded here
as background only, and is not cited as a normative reference.

When to Attest?
---------------

Attestation timing for agents should be driven by **the moments at which the
authority boundary changes**, not by a fixed schedule. For a long-lived agent,
"attest at load" is insufficient: the agent's relevant state — its context,
its delegated authority, its reachable tools — changes during operation.

The trigger set can be expressed as *events*, each producing a fresh,
bound attestation:

- **At grant** — when authority is first conferred on the agent, the
  consuming side of Theme #5's two determinations: the attestation
  carries the determination the record carries, and makes none itself.
- **At (re)delegation** — when the agent hands a sub-task, and its authority,
  to another agent, likewise the consuming side of those determinations.
  Both triggers fire at the moments Theme #5 fixes, and the chapter claims
  nothing further for them.
- **At consequential action** — immediately before an action that is
  irreversible or that crosses an organisational boundary.
- **At policy change** — when the rules the agent is subject to are
  themselves updated.
- **At revocation** — when authority is withdrawn, suspended or rolled back.
  This trigger is the counterpart of *at grant*: what an attestation states at
  revocation is that the authority no longer binds, not what the agent's state
  is. Without this trigger, cached evidence silently outlives the grant it
  rests on.

This aligns attestation with a **decision-point** semantics: the question at
each trigger is not "is the agent trustworthy in general" but "may *this*
action proceed *now*". The consequence is that an attestation is only
meaningful **relative to a decision point**, and a verifier should be able to
state which decision point it is satisfied about. Two consequences follow.
First, an attestation whose decision point cannot be named is not usable.
Second, the *default* behaviour when a required attestation is missing must be
stated per decision point, because silence is currently interpreted
inconsistently.

**Change continuity across decision points.** The event triggers above produce fresh attestations at each boundary change, but the interval between triggers also matters: an agent's state may change between decision points, and a relying party holding a cached attestation cannot tell whether it still describes the state that is about to act. Three fields therefore belong with every attestation, in addition to the decision point it is satisfied about: (i) the **state version** the attestation is pinned to; (ii) the **changes since the previous attestation** (what changed, when, by whom), where determinable; and (iii) an explicit statement of what was **not evaluated** since the previous trigger - "not checked" must remain distinguishable from "checked and passed". With these fields, a relying party can decide whether a cached attestation is still usable without re-running the verification, and can attribute a stale attestation rather than a malicious agent when the gap is structural.

**Applicability at use time.** The fields above let a relying party decide
whether a cached attestation is still usable. Where the subject is unchanged, the
attestation continues to apply. Where a change occurred, it is the *relevance* of
that change to the policy set under which the decision is made that matters, not
the change itself: a policy-relevant change ends the attestation's applicability
and a fresh one is required; a change the policy set does not care about leaves it
applicable, and the attestation should say which changes it considered and
dismissed as irrelevant, so that the judgement is reviewable rather than
implicit. Where change evidence cannot be obtained at all, the outcome is
*indeterminate* and the required behaviour should be to escalate rather than to
pass, because the absence of change evidence is indistinguishable from an
unreported change.


Research Gaps
-------------

- **Consumption-side requirements are unstandardised.** Existing attestation
  building blocks specify how evidence is *produced*; they do not specify what
  a *relying party* must check. A profile that fixes the verifier's
  obligations — and the default outcome when those obligations cannot be
  met — is missing.

- **Failure semantics are undefined.** Profiles should state, for each failure
  class, whether the required behaviour on failure is to deny or to allow, and
  how a "check did not run" outcome is distinguished from "check ran and
  passed".

- **Freshness and replay for continuously operating agents.** There is no
  widely adopted vocabulary for expressing "this evidence is valid for *this*
  action and no other" in an agent context.

- **Binding to instance state.** Research on binding evidence to the state
  actually exercised by a probabilistic system remains thin, relative to work
  on binding evidence to static artefacts.

Standardization Gaps
--------------------

The entries below record absences observed while preparing this chapter. They
are not the result of an exhaustive survey, and a reader who knows of a
published convention covering one of them is invited to say so; the entry would
then be replaced by a reference to it.

- **No common verifier-side checklist.** Each attestation family implies its
  own acceptance procedure, so a relying party facing heterogeneous evidence
  has no shared minimum to apply.

- **No agreed treatment of "absence of evidence".** Whether a missing
  attestation blocks an action, permits it, or raises an exception is
  currently a local decision.

- **No layered identifier convention for agents.** Where a distinction between
  type-level and instance-level identification is needed, it is not yet
  expressed as a convention that a relying party can depend on. This chapter
  does not propose such a convention; it records its absence as a gap.

- **Timing triggers are not standardised.** "At load" versus "per action"
  versus "continuously" is treated as an implementation choice, so two
  conforming implementations can produce evidence that is not mutually
  comparable at a decision point.
- **No format for a negative attestation.** A refusal, a failed check, or the
  absence of a permission is behaviour a relying party needs to verify as
  reliably as a pass. Present formats carry positive claims well and negative
  ones poorly, so "no attestation was issued" and "an attestation was issued
  and said no" are not distinguishable by shape. Practices exist for logging
  refusals; a portable format for attesting one does not.

- **No convention for disclosing coverage.** What a set of attestations does
  *not* cover is left to prose, so completeness tends to be inferred from
  silence. A profile that states its coverage explicitly - the layers it
  attests and the layers it leaves out - would let a relying party discount it
  correctly instead of assuming it is complete.


Implementation observation (informative)
----------------------------------------

One implementation of the three properties has been published openly and is
recorded here as an observation, not as a recommendation for adoption, and not
as a claim that it satisfies the chapter.

What it does, in the terms this chapter uses: it pins a baseline by a manifest
covering several hundred files with a per-file digest, anchored to an external
commit, which is the layer the chapter calls the platform and artifact layer;
it runs three negative controls - inputs that must be rejected - so that a pass
is distinguishable from a check that silently checked nothing; and it publishes a
crosswalk against an external vocabulary that states, in its own body, one
partial mapping, fifteen items for which no mapping was found, and four gaps,
rather than in an appendix.

What it does not do, stated so that the observation can be discounted
correctly: the test corpus is from a single vendor; there has been no
third-party interoperability run; there is no formal proof of the properties;
and continuous integration covers one operating system only. A second
independent party running the same controls is what would move this from an
observation to evidence, and that has not happened.
