# Governance Overview

### Mechanism

To make any changes to the network, the idea is to compose active token holders and the council together to administrate a network upgrade decision. No matter whether the proposal is proposed by the public (token holders) or the council, it finally will have to go through a referendum to let all holders, weighted by stake, make the decision. There is also the technical committee, which can't submit proposals, but safeguard against malicious proposals.

## Referenda

Referenda are a simple, inclusive, stake-based voting scheme. A proposal takes the form of a privileged function call.

Referenda can be started in one of several ways:

* Publicly submitted proposals.
* Proposals submitted by the council.
* Proposals submitted as part of the enactment of a prior referendum.
* Emergency proposals submitted by the Technical Committee and approved by the Council.

All referenda have an _enactment delay_ associated with them. This is the period between the referendum ending and, assuming the proposal was approved, the changes being enacted. Emergency proposals (e.g. fix urgent network issues) can be "fast-tracked" to have a shorter enactment period.

&#x20;Read more at the \[Polkadot Wiki]\([https://wiki.polkadot.network/docs/learn-governance/#referenda](https://wiki.polkadot.network/docs/learn-governance/#referenda)).

In every new voting period, a proposal from one of the queues will be taken in alternating fashion and transformed into the next referendum.

## Council

The council represents passive stakeholders. It is initially assigned by Integritee.

Along with [controlling the treasury](https://wiki.polkadot.network/docs/learn-treasury), the council is called upon primarily for three tasks of governance: proposing sensible referenda (i.e. runtime upgrades, network upgrades), cancelling uncontroversially dangerous or malicious referenda, and electing the technical committee.

Once the network is sufficiently bootstrapped, stabilized, and security measures are in place, governance will be moves to an Elected Council where the candidacy of councillors is an open coin-voting process.

## Technical Committee

The Technical Committee(TC) consists of technical experts with a lot of knowledge about the Integritee architecture. Members are added or removed from the TC via a simple majority vote of the [#council](./#council "mention").

The purpose of the TC is to safeguard against malicious referenda, implement bug fixes, reverse faulty runtime updates, or add new but battle-tested features. The TC has the power to fast-track proposals by using the Democracy pallet, and is the only origin that can trigger the fast-tracking functionality. We can consider the TC to be a "unique origin" that cannot generate proposals, but can fast track existing proposals.

Fast-tracked referenda are the only type of referenda that can be active alongside another active referendum. Thus, with fast-tracked referenda, it is possible to have two active referenda at the same time. Voting on one does not prevent a user from voting on the other.

