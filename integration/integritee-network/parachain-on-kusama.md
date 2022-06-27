---
description: Please read carefully
---

# 🚀 Parachain on Kusama

{% hint style="warning" %}
Due to a configuration error, a trivial extrinsic stalls the parachain. \
Go to [troubleshooting](parachain-on-kusama.md#troubleshooting) for more info!
{% endhint %}

## Spec Requirement

Same as the Polkadot full node requirements. See: [https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#requirements](https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#requirements)

## Run from Source Code

* Clone the repo: [https://github.com/integritee-network/parachain](https://github.com/integritee-network/parachain)
* Or checkout one tag from here: [https://github.com/integritee-network/parachain/tags](https://github.com/integritee-network/parachain/tags)
* Install dependencies `rustup update` \
  More info: [https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#node-prerequisites-install-rust-and-dependencies](https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#node-prerequisites-install-rust-and-dependencies)
* Build Integritee: `cargo build --release`
* Run `./target/release/integritee --chain=integritee-kusama \` \
  `-- --chain=kusama`

## Using Docker

* Image: `integritee/parachain:latest` or `integritee/parachain:[version number]`
* `docker run integritee/parachain:latest --chain=integritee-kusama \` \
  `-- --chain=kusama`

## Common CLI

* CLI is mostly the same as any Substrate-based chain such as Polkadot and Polkadot
* Because there are two node services are running, `--` is used to split the CLI. Arguments before `--` are passed to the parachain full-node service and arguments after `--` is passed to the Relay Chain full-node service.
  * For example `--chain=parachain.json --ws-port=9944 -- --chain=relaychain.json --ws-port=9945` means
    * The parachain service is using `parachain.json` as the chain spec and the web socket RPC port is 9944
    *   The Relay Chain service is using `relaychain.json` as the chain spec and the web socket

        RPC port is 9945
* It is recommended to explicitly specify the ports for both services to avoid confusion
  * For example `--listen-addr=/ip4/0.0.0.0/tcp/30333 --listen-addr=/ip4/0.0.0.0/tcp/30334/ws -- --listen-addr=/ip4/0.0.0.0/tcp/30335 --listen-addr=/ip4/0.0.0.0/tcp/30336/ws`
* It is recommended to add `--execution=wasm` for parachain service to avoid syncing issues.

## Example CLI

```
./target/release/integritee
--base-path=/integritee/data
--chain=integritee-kusama
--name=integritee-rpc
--pruning=archive
--ws-external
--rpc-external
--rpc-cors=all
--ws-port=9944
--rpc-port=9933
--ws-max-connections=2000
--execution=wasm
--
--chain=kusama
```

## Troubleshooting

{% hint style="warning" %}
Due to a configuration error, the sync stalls at #38220.\
More info: [https://github.com/integritee-network/parachain/issues/84](https://github.com/integritee-network/parachain/issues/84)
{% endhint %}

### The Fix

#### Build the fix yourself

1. Checkout our fix branch [https://github.com/integritee-network/parachain/tree/skip-signature-verification](https://github.com/integritee-network/parachain/tree/skip-signature-verification)
2. Build the code as it is described in [Run from source code](parachain-on-kusama.md#run-from-source-code) section
3. Once the sync left #38220 it is safe to stop and continue with the latest version

#### Using our pre-built binary

1. Download [https://github.com/integritee-network/parachain/releases/download/1.5.15/integritee-collator-accept-all-signatures](https://github.com/integritee-network/parachain/releases/download/1.5.15/integritee-collator-accept-all-signatures) , replace it with integritee binary. Follow the instructions in [Run from source code](parachain-on-kusama.md#run-from-source-code) section.
2. Once the sync continued, you can use the latest binary

#### Download our initial DB

Once parachain sync stalls at #38220 you can replace the DB with our one:

1. Download [https://github.com/integritee-network/parachain/releases/download/1.5.13/integritee-kusama-48375.RocksDb.tar.lz4](https://github.com/integritee-network/parachain/releases/download/1.5.13/integritee-kusama-48375.RocksDb.tar.lz4)
2. Extract it \
   `curl -o - -L https://github.com/integritee-network/parachain/releases/download/1.5.13/integritee-kusama-48375.RocksDb.tar.lz4 | lz4 -c -d - | tar -x -C /path/chains/integritee-kusama/`
3. Run the binary as described above

