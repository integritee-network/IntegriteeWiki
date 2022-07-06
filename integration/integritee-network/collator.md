# Collator

## Overview

Collators maintain parachains by collecting parachain transactions from users and producing state transition proofs for Relay Chain validators. In other words, collators support parachains by aggregating parachain transactions into parachain block candidates and producing state transition proofs for validators based on those blocks.

While Integritee's network security and consensus (nPoS) are provided by Polkadot Relay chain's Validator set, the Collator set of Integritee will keep the network alive by collecting parachain transactions for validators to verify them. **Unlike validators, collators have nothing to do with the security of the network. By being a parachain, the network is by default trustless and decentralized, and a parachain only needs one honest collator to be censorship-resistant.** Read more on Collators [here](https://wiki.polkadot.network/docs/learn-collator).

Integritee's Collator node maintains a full-node service for the Polkadot Relay Chain and a full-node service for the Integritee network. Each service has its own http/ws RPC endpoints, P2P ports, etc. The base path of the Polkadot full-node service is located inside the base path of the Integritee full-node service.

## Collator Roll Out Plan

Integritee takes a phased approach to roll out the Collator operation. Since Collators are non-security critical, a parachain only needs one honest Collator to be censorship-resistant. Further, more collators are not necessarily good, e.g. too many might slow down the network. The Collator roll-out plan is designed first-and-foremost to ensure network stability and operation.

**Current Phase: Private Collator Set**

**Phase 0: Private Collator Set**\
The Genesis of the Integritee network was launched on 18th December 2021. Upon genesis, Integritee's network security and the consensus is provided by Polkadot Relay Chain's nominated Proof-of-Stake (nPoS) validators. Just like Statemine (the common-good asset parachain on Polkadot), Integritee's Collators initially will be run by Integritee, until the Collator software is stable and can be released to the wider community.

**Phase 1: Authorized Collator Set (We are here)**\
Through governance approval, Integritee will then open the collator set to an authorized set of collators. While there are no block rewards nor additional incentives for these authorized collators, they are paid a reasonable rate for their node service provisioning by Integritee.

These collators are known reputable node service providers who have a proven track-record of service levels and demonstrated a deep commitment to the network. It is expected that there will be much chaos and software upgrades during this phase still, and these collators are required to work closely with the core dev team to ensure network stability.

The Integritee network will maintain such an authorized collator set until the network is fully stabilized, the collator staking module and the reward scheme are fully implemented and audited.

**Phase 2: Public Collator Set**\
Through governance approval, Integritee will then enable the permissionless election of collators and enable collator rewards. As written above, collators are non-security critical, a parachain only needs a small set of collators to ensure liveness and censorship-resistance. The reward scheme will reflect this accordingly.

## Collator Node

### Run the node

See [parachain-on-kusama.md](parachain-on-kusama.md "mention")for how to run a full-node. There is only one difference. The parachain CLI (part before the `--`) needs the `--collator` flag.

### Collator Configuration

#### **Key Management**

The Aura Consensus works by registering a fixed set of authorities that will produce blocks in round-robin fashion. Aura keys are `sr2559` keys and can be created with [Subkey](https://core.tetcoin.org/docs/en/knowledgebase/integrate/subkey).

The Aura key needs to be injected into the node's keystore. The node will start to collate if it finds its key in the registered authority set. Key injection can be done with the RPC `aura_insertKey` or with Subkey.

```bash
# RPC
curl http://localhost:9936 -H "Content-Type:application/json;charset=utf-8" -d '{ "jsonrpc":"2.0", "id":1, "method":"author_insertKey", "params": [ "aura", <suri>, "<pubKey>" ] }'

# Subkey
subkey insert --suri <suri> --base-path /integritee/data --key-type aura
```

#### **Registration**

* Registering a new collator is currently a privileged low-level function call; `system.set_storage`. Therefore, registration needs to pass governance. The low-level call will be replaced with the introduction of the `pallet-collator-selection`. However, this will not change that updating the collator set needs governance.
* Authorized collator providers will have to submit the public key of the Aura session key for the collator, and the Integritee Council shall submit a proposal for the governance process.

