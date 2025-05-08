# Anonymous State Machines

Intuitively, an Anonymous State Machine is a way for a server to store state on
the client, all the while updating and querying that state and keeping it
authenticated by ensuring updates proceed according to a set of server-defined
rules.

An Anonymous State Machine consists of the following types and protocols:

## State

A state is a vector { v\_i } ∈ [n]. There are state variables which are known
to the issuer at issuance time P ⊆ [n]. There is a well known function Γ(v, τ)
= { v\_i } ∈ [n] for any transition τ ∈ T. There is a well known predicate
ρ(v, τ).

## Issue

Issuance takes place between a client and an issuer.

The issuer has:
- private key x
- issuance public state { v\_i } i ∈ P, where S is the set of indices of state
  variables which are public

The client has:
- public key w
- all state variables { v\_ i } i ∈ \[n\]
- secret blinding factor r
- secret nullifier k

By the end, the client will have an authentication code σ associated with {
v\_i }, k, r.

## Transition

Transition takes place between a client and an issuer. The issuer maintains a
database db.

The issuer has:
- private key x
- revealed nullifier k
- intended transition τ

The client has:
- public key w
- all state variables { v\_ i } i ∈ \[n\]
- secret blinding factor r
- now public nullifier k
- authentication code σ for the above state
- intended transition τ

By the end, the client will have an authentication code σ' associated with Γ(v,
τ), k', r' for new nullifier k' and blinding factor r'. The issuer will be
certain that:
- there exists v, r, σ such that σ is a valid authentication code on v, k, r,
- they only allowed the client to construct an authentication code σ' on Γ(v,
  τ),
- that ρ(v, τ) is satisfied

## Summary

An Anonymous State Machine protocol allows a server to issue authenticated
initial states, then transition them according to known transition rules. This
allows an issuer to issue and query client state, potentially modifying it with
every query, while maintaining as much privacy as is possible.

## Examples

The [Anonymous Credit Tokens](https://github.com/SamuelSchlesinger/anonymous-credit-tokens)
are an example of an Anonymous State Machine. The state is a single element
vector which contains the number of credits. The function Γ(v, τ) = v - τ, and
the predicate ρ(v, τ) = v - τ ≤ 2<sup>l</sup>, where l is the maximum number of bits
allowed in token denominations.
