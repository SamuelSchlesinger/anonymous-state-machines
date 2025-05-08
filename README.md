# Anonymous State Machines

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
    |  === ISSUANCE PROTOCOL ===               |
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
    |  === TRANSITION PROTOCOL ===             |
    |                                           |
    |-- Generate:                               |
    |   - new secret blinding factor r'         |
    |   - new secret nullifier k'               |
    |                                           |
    |-- Create proof of valid state with        |
    |   authentication code σ                   |
    |                                           |
    |------- Transition Request -------------->|
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
    |<------ New Authentication Code σ' ---------|
    |        (for new state Γ(v,τ),             |
    |         new nullifier k', new blinding r') |
    |                                           |
```

## Summary

An Anonymous State Machine protocol allows an issuer to:
1. Issue authenticated initial states to clients after verifying they satisfy the predicate ψ
2. Process state transitions according to well-defined rules (Γ and ρ)
3. Prevent double-spending through nullifier tracking
4. Maintain client privacy by keeping state data with the client

This model enables privacy-preserving state management where:
- The client maintains their own state data
- Initial states are verified using the ψ predicate without revealing private data
- The issuer authenticates and validates state transitions using the ρ predicate
- Each state can only be transitioned once (via nullifier tracking)
- The issuer enforces state transition rules without necessarily learning all state values

This approach provides a powerful framework for privacy-preserving protocols where state needs to be maintained and validated while minimizing data exposure to the issuer.

## Examples

The [Anonymous Credit Tokens](https://github.com/SamuelSchlesinger/anonymous-credit-tokens)
are an example of an Anonymous State Machine. The state is a single element
vector which contains the number of credits. The function Γ(v, τ) = v - τ, and
the predicate ρ(v, τ) = v - τ ≤ 2<sup>l</sup>, where l is the maximum number of bits
allowed in token denominations.
