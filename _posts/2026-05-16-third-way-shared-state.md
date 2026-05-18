---
layout: post
title: "EC Protocol: Shared State Without Global Consensus, and Why Conflicts Become Self-Punishing"
date: 2026-05-16
---

Multiple parties often need to share records: payments, vouchers, votes, names, credentials, ownership claims.

The usual answer is simple: put the records in a database.

That works beautifully until the parties do not want the same operator to control the database. Whoever runs it can censor, rewrite, selectively enforce, lose, or surveil the shared state. Sometimes that trust is acceptable. Sometimes it is exactly the problem.

The other familiar answer is a global ledger: replicate the state widely, globally order transactions, and make the history publicly auditable.

That solves one class of trust problem, but it buys that solution with global coupling. Everyone is tied, directly or indirectly, to the same ordering machine. The system needs broad quorums, committees, fork-choice rules, permanent history, or incentive machinery to keep the whole thing moving in one direction.

This raises a question:

> Can we share state without trusting one operator, but also without forcing every record through one global consensus system?

EC Protocol is an experiment in that third way.

## Local Shared State

EC does not try to make every participant agree on every event.

Instead, it treats shared state as local. Records are assigned to token neighborhoods. A commit requires evidence from the relevant neighborhood, not from the whole world. The goal is not a single universal history of everything. The goal is that, for a given record, the relevant peers can detect conflicts and avoid accepting multiple incompatible outcomes.

That is a narrower and more achievable target.

EC is not "no consensus." It is no global consensus over all records.

The protocol's job is to make conflicts visible, attributable, and hard to profit from.

## The Threat Model Shift

A lot of distributed safety machinery is built around conflicting histories. If two incompatible transactions exist, the system needs some way to decide which one wins. That often leads to global ordering, fork choice, or Byzantine agreement.

But many real applications have an important property:

> A valid state transition must be signed by the owner.

A payment must be signed by the payer. A voucher transfer must be signed by the holder. A ballot must be signed by the voter's credential. A name update must be signed by the current controller.

The network can delay messages. It can censor messages. It can replay messages. It can route messages badly.

But it cannot manufacture a valid owner-signed conflict.

Only the key-holder can do that.

That changes the shape of the problem. A conflict is no longer anonymous noise injected by the infrastructure. It is signed evidence that a specific key equivocated.

The protocol does not need to prevent someone from signing twice. Cryptography cannot stop you from using your own key badly. What the protocol can do is make sure that signing twice does not silently create two valid outcomes.

## Why Conflicts Become Self-Punishing

In EC, the interesting safety claim is not "conflicts never happen."

Conflicts absolutely can happen. A malicious or careless owner can sign incompatible records.

The claim is that, in the application classes EC is designed for, a visible conflict usually harms the signer rather than helping them.

The network's job is to ensure that conflicting contenders become visible to the relevant neighborhood, and that applications can treat conflict evidence as invalidating, freezing, or burning the claim.

## Payments

Suppose a payer signs two incompatible payments using the same spendable claim.

In a naive system, that might look like an attempt to get free money. In an EC-style system, the conflict is evidence against the payer's own claim.

At redemption, the issuer or recipient does not merely ask "do I have a signed payment?" They ask whether the payment committed cleanly, and whether conflicting contenders exist.

If one payment wins, the conflict is visible. If neither resolves, the payer has created a stalled or suspicious claim. What they do not get is two clean redemptions.

The attack does not create money. It creates evidence.

## Vouchers

A voucher has the same structure.

The holder can sign a transfer. If they sign two incompatible transfers, they are not duplicating value. They are damaging their own entitlement.

The issuer can see that the voucher history contains a conflict and reject, freeze, or burn the voucher according to the application rules.

Again, the network did not need to make double-signing impossible. It needed to make double-signing visible.

## Voting

Voting is even cleaner.

A double ballot should not count as two votes. It should count as evidence that the credential equivocated.

The natural application rule is simple: if a voter credential signs conflicting ballots, the ballot counts as zero or is escalated into a dispute process.

The attacker's payoff is not "two votes." The attacker's payoff is, at best, destroying their own vote.

## Names and PKI

Name registries and public-key infrastructure have a similar pattern.

If the controller of a name signs conflicting ownership records or incompatible delegations, that conflict weakens the authority of the key that signed it.

A relying party does not need every name update in the world to be globally ordered forever. It needs conflict-visible evidence around the name or token being resolved.

That suggests a different architecture for name ownership: local verification, explicit delegation, visible equivocation, and bounded history.

## The Pattern

These examples have something in common.

They involve owner-signed state transitions where equivocation naturally damages the equivocator's own claim.

That does not make all attacks disappear. It does not solve censorship, key theft, denial of service, bad issuers, or bad application rules.

But it does mean that the most important conflict case has a different character.

The network is not trying to globally order away every possible contradiction. It is trying to expose contradictions so that the application can apply the obvious rule:

> If you signed incompatible claims, you do not get to profit from both.

## What We Saw in Simulation

The current EC simulator has been testing this under conflict-heavy conditions.

Under 30% conflict load across 2000 peers, meaning nearly a third of all tokens had competing signed transactions, the run produced:

- 0 lower-priority conflicting contender commits
- 0 multi-contender conflicting commits
- every observed majority selected the highest contender

In plain language: we did not observe cases where the losing side of a conflict committed, and we did not observe multiple incompatible winners.

There was also an important measurement correction.

An earlier metric made convergence look much worse than it was. We were undercounting success because committed blocks that had already been extended were dropping out of the success measurement. After correcting that, apparent convergence moved from roughly 7% to roughly 93%.

That correction matters. This project is still research-stage, and the point is not to polish the numbers into a sales pitch. The point is to make the protocol measurable enough that wrong interpretations can be found and fixed.

## What This Enables

If this model holds, it opens up an architectural space between centralized databases and global ledgers.

You do not need one operator to own the database.

You also do not need every participant to store, order, and validate every record forever.

Instead, EC aims for:

- local neighborhoods rather than a global committee
- conflict-visible commits rather than a universal transaction order
- bounded retention rather than permanent global history
- application-level consequences for equivocation
- no token required purely for incentive alignment

That last point is important. EC is not trying to build a new economy around consensus. It is trying to make shared state cheap enough and local enough that many applications can use it directly.

Bounded retention is part of the design philosophy too. Many systems treat deletion, expiry, and the right to be forgotten as awkward layers on top of permanent infrastructure. EC explores the opposite direction: shared state with retention limits built into the protocol shape.

## Scope and Honest Gaps

This is not a claim that EC is production-ready.

It is not a claim that local neighborhoods magically solve all adversarial behavior.

It is not a claim that every application fits this model.

And it is definitely not a claim that networks cannot censor, delay, partition, or attack routing.

The narrower claim is the interesting one:

> For owner-signed state transitions, conflict can be treated as attributable evidence instead of something that must always be prevented by global ordering.

That is the idea worth testing.

EC Protocol is currently a Rust reference implementation and simulator suite. The simulator works. The claims are being sharpened. The gaps are real.

Where does local neighborhood consensus fail under adversarial topology, churn, censorship, or key compromise?

Which applications do not fit the self-punishing conflict model?

What breaks this reasoning entirely?

The repo, simulator reports, and design documents are at [EcProtocol/EcNode](https://github.com/EcProtocol/EcNode). The README is the best starting point.
