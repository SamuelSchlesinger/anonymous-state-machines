# Anonymous State Machines

## Introduction

### The Problem: Privacy-Preserving Stateful Systems

In modern digital systems, users routinely interact with services that need to maintain state about them - from subscription services tracking usage limits to authentication systems recording login attempts. However, this state management traditionally requires users to trust service providers with their personal data, creating significant privacy concerns.

### Existing Solutions and Their Limitations

#### Anonymous Credentials

Anonymous credential systems (like U-Prove, Idemix, and zkSNARK-based approaches) allow users to prove properties about themselves without revealing their identity. However, these systems have a critical limitation: **they are fundamentally stateless**. Once issued, a credential cannot be updated - it can only be replaced with a newly issued credential. This requires reconnecting with the issuer for any state change, defeating true client-side state management.

#### Rate Limiting Solutions

Systems like Privacy Pass and rate-limiting tokens enable anonymous access while preventing abuse. However, these are often single-use tokens that can only be consumed once, not updated. This forces a choice between maintaining state centrally (sacrificing privacy) or issuing new credentials for each state change (sacrificing efficiency).

#### The Trust Gap

Due to these limitations, many systems resort to trusted server-based approaches, storing user state on servers and asking users to trust that their privacy won't be violated. This creates an unnecessary gap between theoretical privacy protection and practical implementations.

### Anonymous State Machines: Bridging the Gap

Anonymous State Machines (ASMs) address these limitations by enabling:
1. Client-held state that remains under user control
2. Server-validated state transitions without revealing all state data
3. Cryptographic verification of state validity without centralized storage
4. Prevention of replay attacks via nullifier tracking

Intuitively, an Anonymous State Machine is a way for a server to store state on
the client, all the while updating and querying that state and keeping it
authenticated by ensuring updates proceed according to a set of server-defined
rules.

An Anonymous State Machine consists of the following types and protocols:

## State

A state is a vector { v\_i } ∈ [n]. There are state variables which are known
to the issuer at issuance time P ⊆ [n]. There is a well known function Γ(v, τ)
= { v\_i } ∈ [n] for any transition τ ∈ T. There are two well known predicates:
- ρ(v, τ): defines valid transitions from state v using transition τ
- ψ(v): defines the validity of an initial state vector v

## Issue

Issuance takes place between a client and an issuer.

The issuer has:
- private key x
- issuance public state { v\_i } i ∈ P, where P is the set of indices of state
  variables which are public

The client has:
- public key w
- private values for state variables { v\_i } i ∈ [n] \ P (those not in P)

The issuance protocol:
1. Client requests an initial state from the issuer
2. Client generates:
   - secret blinding factor r
   - secret nullifier k (to be revealed in future transitions)
3. Issuer and client engage in a secure protocol where:
   - The issuer contributes the public state values { v\_i } i ∈ P
   - The client contributes the private state values { v\_i } i ∈ [n] \ P
   - No party learns the other's private inputs
4. Client provides a zero-knowledge proof ψ(v) that their initial state vector satisfies certain properties
   - This allows the issuer to verify the state is well-formed without learning private values
   - The predicate ψ defines valid initial states (e.g., range proofs for numeric values)
5. Issuer verifies ψ(v) is satisfied and then generates an authentication code σ for the complete state

By the end, the client will have an authentication code σ associated with {
v\_i }, k, r, without the issuer learning the client's private state values or nullifier.

## Transition

Transition takes place between a client and an issuer. The issuer maintains a
database db of seen nullifiers.

The issuer has:
- private key x
- intended transition τ

The client has:
- public key w
- all state variables { v\_ i } i ∈ \[n\]
- secret blinding factor r
- nullifier k to be revealed during transition
- authentication code σ for the above state
- intended transition τ

The transition protocol:
1. Client sends nullifier k and proof of valid state to the issuer
2. Issuer checks if nullifier k exists in database db
   - If k exists, reject the transition (prevents double-spending)
   - If k does not exist, add k to database db
3. Issuer verifies the state proof and that ρ(v, τ) is satisfied
4. Issuer issues new authentication code for the transitioned state

By the end, the client will have an authentication code σ' associated with Γ(v,
τ), k', r' for new nullifier k' and blinding factor r'. The issuer will be
certain that:
- there exists v, r, σ such that σ is a valid authentication code on v, k, r,
- the nullifier k has never been used before (preventing replay attacks)
- they only allowed the client to construct an authentication code σ' on Γ(v,
  τ),
- that ρ(v, τ) is satisfied

## Sequence Diagram

```
  Client                                      Issuer
    |                                           |
    |   === ISSUANCE PROTOCOL ===               |
    |                                           |
    |------- Request Initial State ------------>|
    |                                           |
    |<---- Request public state vars {v_i}i∈P --|
    |                                           |
    |-- Generate:                               |
    |   - secret blinding factor r              |
    |   - secret nullifier k                    |
    |   - private state vars {v_i}i∈[n]\P       |
    |                                           |
    |-- Compute zero-knowledge proof ψ(v) ----->|
    |   (proves initial state is valid)         |-- Verify ψ(v) is satisfied
    |                                           |   (without learning private values)
    |                                           |
    |<------ Issue Authentication Code σ -------|
    |        (for state v, nullifier k, r)      |
    |                                           |
    |   === TRANSITION PROTOCOL ===             |
    |                                           |
    |-- Generate:                               |
    |   - new secret blinding factor r'         |
    |   - new secret nullifier k'               |
    |                                           |
    |-- Create proof of valid state with        |
    |   authentication code σ                   |
    |                                           |
    |-------- Transition Request -------------->|
    |        with:                              |
    |        - nullifier k (revealed)           |-- Check if nullifier k 
    |        - transition τ                     |   exists in database
    |        - proof of valid state             |-- If exists, reject
    |        - proof that ρ(v,τ) is satisfied   |   If not, add to database
    |        - commitment to:                   |
    |          * new nullifier k'               |-- Verify state proof
    |          * new blinding factor r'         |-- Verify ρ(v,τ) is satisfied
    |          * new state Γ(v,τ)               |-- Compute new state Γ(v,τ)
    |                                           |
    |<----- New Authentication Code σ' ---------|
    |        (for new state Γ(v,τ),             |
    |         new nullifier k', new blinding r')|
    |                                           |
```

## Summary

### Bridging the Gap Between Privacy and Stateful Systems

Anonymous State Machines provide a cryptographic solution to a fundamental tension in privacy-preserving systems: how to maintain updateable state without sacrificing privacy. Unlike traditional anonymous credential systems that can only be replaced but not updated, ASMs enable genuine stateful operations while preserving anonymity.

An Anonymous State Machine protocol allows an issuer to:
1. Issue authenticated initial states to clients after verifying they satisfy the predicate ψ
2. Process state transitions according to well-defined rules (Γ and ρ)
3. Prevent double-spending through nullifier tracking
4. Maintain client privacy by keeping state data with the client

This model enables privacy-preserving state management where:
- The client maintains their own state data, reducing the need to trust servers with private information
- Initial states are verified using the ψ predicate without revealing private data
- The issuer authenticates and validates state transitions using the ρ predicate
- Each state can only be transitioned once (via nullifier tracking), preventing replay attacks
- The issuer enforces state transition rules without necessarily learning all state values

### Practical Applications

Anonymous State Machines can transform applications that currently require trusted servers for state management. By moving state to the client side while ensuring cryptographic integrity, ASMs enable:

- Rate-limited systems that preserve privacy
- Multi-use anonymous access tokens that can be decremented without revealing identity
- Loyalty systems that track points without tracking users
- Access control systems where permissions can be selectively disclosed
- Voting systems with provable eligibility but anonymous participation

This approach provides a powerful framework for privacy-preserving protocols where state needs to be maintained and validated while minimizing data exposure to the issuer.

## Examples

The following examples demonstrate how Anonymous State Machines can be applied to real-world problems.

### Anonymous Credit Tokens

The [Anonymous Credit Tokens](https://github.com/SamuelSchlesinger/anonymous-credit-tokens)
are an example of an Anonymous State Machine. The state is a single element
vector which contains the number of credits. The function Γ(v, τ) = v - τ, and
the predicate ρ(v, τ) = v - τ ≤ 2<sup>l</sup>, where l is the maximum number of bits
allowed in token denominations.

**Advantages:**
- Users can spend credits over time without revealing their identity across uses
- Server can enforce credit limits without maintaining user-linked accounts
- Credit balance is maintained client-side but cryptographically verified

### Anonymous Access Control

An Anonymous State Machine can implement privacy-preserving access control
systems.  In this example, the state vector v contains a set of permission
flags v = [v\_1, v\_2, ..., v\_n] where each v\_i represents a different permission
(e.g., read, write, admin).  The transition function Γ(v, τ) updates
permissions based on authorized actions, where τ might represent "grant write
access" or "revoke admin privileges".  The predicate ρ(v, τ) ensures that only
authorized parties can modify permissions (e.g., only users with admin
privileges can grant new permissions).  This allows a client to prove they have
specific permissions without revealing their identity or the full set of
permissions they hold.

**Advantages:**
- Users can selectively disclose only the permissions relevant to a specific resource
- Permissions can be updated without requiring full credential reissuance
- Organizations can delegate permissions without maintaining central user-permission databases
- Permission changes are cryptographically validated while preserving user privacy

### Anonymous Voting System

An Anonymous State Machine can power a private voting system where voters can
prove their eligibility without revealing their identity. The state is a single 
element vector containing the index of the current election. Transitions increment
this index by 1, representing a vote in the current election.

In this system, when a voter casts a ballot, their vote is bound to the transition validity
proof in the Fiat-Shamir transform by including it as an input to a hash function. The 
transition includes the election index, so the predicate ρ(v, τ) verifies that the transition
is valid for the current election index (preventing votes in past or future elections), while
the transition function Γ(v, τ) = v + 1 moves to the next election. This ensures that each
eligible voter can only vote once per election while keeping their identity anonymous.

**Advantages:**
- Voters remain anonymous while provably demonstrating voting eligibility
- Double-voting is cryptographically prevented without voter registration databases
- Voter status can be updated across multiple elections without reissuance
- Election administrators can ensure one-vote-per-person without tracking identities

### Anonymous Loyalty Program

An Anonymous State Machine can implement a privacy-preserving loyalty program that protects
customer purchase history while allowing point redemption. The state vector consists of
[total\_points, tier\_status], where total\_points tracks accumulated rewards and tier\_status 
represents the customer's loyalty level (e.g., bronze, silver, gold).

The transition function Γ(v, τ) handles three types of transitions:
1. Point earning: Γ([points, tier], τ\_earn(points)) = [points + amount, new\_tier] where new\_tier is 
   calculated based on the updated point balance
1. Tier update: Γ([points, tier], τ\_tier(new\_tier)) = [points, new\_tier]
2. Point redemption: Γ([points, tier], τ\_redeem(points)) = [points - cost, tier]
3. Tier verification: Γ([points, tier], τ\_verify(tier)) = \[points, tier\] (state remains unchanged)

The predicate ρ(v, τ) ensures valid transitions by verifying:
- For redemptions: points ≥ cost and the redemption amount doesn't exceed maximum allowed for tier
- For earnings: amount is within valid range and properly corresponds to purchase value
- For tier verification: always true, but allows proving tier membership to a verifier without 
  modifying state

The tier verification transition is particularly useful as it allows customers to selectively 
disclose only their loyalty tier to service providers (e.g., to access tier-specific benefits) 
without revealing their point balance or consuming their token. This transition uses a nullifier 
that can be regenerated deterministically for specific service providers, allowing repeated 
verification while maintaining unlinkability between different providers.

**Advantages:**
- Customers can accumulate and spend points without revealing their identity
- Businesses can prevent fraud and double-spending without tracking individual customers
- Tier benefits can be selectively disclosed without revealing point balances
- Customer purchase patterns remain private even as they earn and redeem rewards
- Different service providers cannot collude to track customer activities across venues
