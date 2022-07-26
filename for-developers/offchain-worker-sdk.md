# Offchain Worker SDK

## What is an offchain worker?

A TEE offchain worker is used to execute operations in a trusted manner. A client submits these operations to a substrate 'parentchain', which can be either a parachain or a solochain. Usually, this will be the Integritee Parachain. The operations are opaque to the parentchain because they are encrypted. Only the client and the TEE can see the operation's content.

Sending an encrypted operation to the parentchain, which is then picked up and executed by a TEE offchain worker, is called 'indirect invocation'. An offchain-worker only supports indirect invocation and 'getters'. In order to encrypt the operation, a client can query the offchain worker directly (with a 'getter') for the 'shielding key', which is an RSA 3072 public key.

Multiple offchain workers running in parallel share only the shielding key, but nothing else. They arrive at the same state independently by executing all the operations found on the parentchain, which guarantees ordering of operations.

### What is the difference between an offchain worker and a sidechain validateer?

An offchain-worker supports only 'indirect invocation', which means it executes only operations that are submitted to the parentchain (encrypted, in an extrinsic). As a result, the operation throughput is limited by the block production cycle on a parentchain. 'Direct invocation', available in a sidechain validateer, allows a client to send operations directly to a worker, allowing for much higher operation throughput. This is possible, because the sidechain allows sharing and synchronizing state between workers, without having to rely on the parentchain.

## How to use the SDK

1. Experiment with the template, [running the offchain-worker demo](./demos/offchain-worker-demo.md)
2. Fork the worker repository `https://github.com/integritee-network/worker.git` from the SDK release branch (`sdk-v0.1.0-polkadot-v0.9.26`)
3. Build the worker in offchain-worker mode
4. Write and integrate your own business logic, e.g. in a substrate pallet
5. Deploy on the [Integritee parachain](integrate-with-integritee-parachain.md) or you own solo chain

## Building the worker in offchain-worker mode

In order to build the worker in offchain worker mode, the corresponding cargo feature `offchain-worker` needs to be set. In the Makefiles, the environment variable `WORKER_MODE` is used to set the cargo features. 

In case you build with `make` directly, do so with:

```
WORKER_MODE=offchain-worker make
```

In case you use docker with our `build.Dockerfile`, use `--build-arg WORKER_MODE_ARG=offchain-worker` to set the corresponding docker `ARG`.

An example of a docker build command (as currently used for GitHub CI):

```
docker build -t integritee-worker --target deployed-worker --build-arg WORKER_MODE_ARG=offchain-worker -f build.Dockerfile .
``` 

## Customizing an offchain worker

### Business logic

This is the core part of the code changes necessary to turn the generic worker into your specific use-case. Read more in [this dedicated section](../for-developers/custom-business-logic-stf/README.md).

### RPC Interface

In case you want to extend the existing RPC interface, [this section](rpc-interface.md) tell you how.

### CLI Client

A simple [CLI client](https://github.com/integritee-network/worker/tree/master/cli) implementation is available in the worker repository, to show case communication with the worker, either by 'direct invocation' or 'indirect invocation'. For an offchain-worker, only 'indirect invocation' is available.

The CLI client uses [JSON RPC 2.0](https://www.jsonrpc.org/specification) over web-socket to communicate with a worker (direct invocation and trusted getters), as can be seen [here](https://github.com/integritee-network/worker/blob/a9a5afdb2de093de0062d7cb7ad302b8501e24a0/cli/src/trusted_operation.rs#L226) for example. The web-socket connection is secured by TLS with an enclave self-signed certificate.

#### Default client workflow for indirect invocation

Default workflow for executing operations on an offchain worker from a client's perspective:

1. Compose your trusted call, e.g. `TrustedCall::balance_transfer`
2. Get the shielding key from an offchain worker
    * In case there are multiple offchain workers, any of them will work, since they all share the same shielding key
3. Encode and encrypt the operation using the shielding key
4. Wrap the encrypted operation into a parentchain extrinsic
5. Send the extrinsic to the parentchain
6. Wait for the `ProcessedParentchainBlock` event, with the hash of the parentchain block including your extrinsic.

Examples of this workflow can be found in our CLI client implementation, [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_commands.rs#L167) and [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_operation.rs#L98).