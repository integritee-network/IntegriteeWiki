# Sidechain SDK

## What is a sidechain (validateer)?

This question is mostly answered in the Integritee book, [here](https://book.integritee.network/sidechain.html).

### Which use-cases are suited for a sidechain validateer?

A sidechain validateer is typically used, when high transaction throughput of trusted transactions is required. The sidechain synchronizes validateers, so 'direct invocation' can be used to execute transactions. This means transactions do not need to be submitted to the layer 1 chain, but can directly be submitted to a specific validateer instance using an RPC interface. 

## How to use the SDK

1. Experiment with the template, [running the sidechain demo](./demos/sidechain-demo.md)
2. Fork the worker repository `https://github.com/integritee-network/worker.git`
3. Build the worker in sidechain mode
4. Write and integrate your own business logic, e.g. in a substrate pallet
5. Deploy on the Integritee parachain or you own solo chain

## Building the worker in sidechain validateer mode

In order to build the worker in sidechain mode, the corresponding cargo feature `sidechain` needs to be set. In the Makefiles, the environment variable `WORKER_MODE` is used to set the cargo features.

In case you build with `make` directly, do so with:

```
WORKER_MODE=sidechain make
```

In case you use docker with our `build.Dockerfile`, use `--build-arg WORKER_MODE_ARG=sidechain` to set the corresponding docker `ARG`.

An example of a docker build command (as currently used for GitHub CI):

```
docker build -t integritee-worker --target deployed-worker --build-arg WORKER_MODE_ARG=sidechain -f build.Dockerfile .
```

## Customizing a sidechain validateer

### Business logic / STF

This is the core part of the code changes necessary to turn the generic worker into your specific use-case. Read more in [this dedicated section](../for-developers/custom-business-logic-stf/README.md).

### RPC Interface

[this section](rpc-interface.md)

### Default client workflow

Default workflow for executing operations on a sidechain validateer from a client's perspective (direct invocation):

1. Compose your operation, e.g. `balance_transfer`
2. Get the shielding key from an off-chain worker
    * In case there are multiple off-chain workers, any of them will work, since they all share the shielding key
3. Encode and encrypt the operation using the shielding key
4. Wrap the encrypted operation into a parentchain extrinsic
5. Send the extrinsic to the parentchain
6. Wait for the `ProcessedParentchainBlock` event, with the hash of the parentchain block including your extrinsic.

Examples of this workflow can be found in our CLI client implementation, [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_commands.rs#L167) and [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_operation.rs#L98).





#### Example from Integritee book

The [Integritee book](https://book.integritee.network/introduction.html) also shows a concrete [example](https://book.integritee.network/howto_stf.html#integritee-worker) ([Encointer](https://encointer.org/)) of writing your own STF in the worker, with code samples.

*TODO: Internalize all docs from book into this Repo*
