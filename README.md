# Anonymous State Machines

An Anonymous State Machine consists of the following protocols:

## State

A state is a vector { v\_i } ∈ [n]. There are state variables which are known
to the issuer at issuance time, and there are state variables which are known
to the issuer at verification time.

## Issue

Issuance takes place between a client and an issuer.

The issuer has:
- private key x
- issuance public state { v\_i } i ∈ S, where S is the set of indices of state
  variables which are public

The client has:
- public key w = g^x
- all state variables { v\_ i } i ∈ \[n\]
- secret blinding factor r
- secret nullifier k

By the end, the client will have a MAC of r, k, v\_0, ..., v\_{N-1}.
