# Sidechain SDK

## What is a sidechain (validateer)?

This question is mostly answered in the Integritee book, [here](https://book.integritee.network/sidechain.html).

### Which use-cases are suited for a sidechain validateer?

A sidechain validateer is typically used, when high transaction throughput of trusted transactions is required. The sidechain synchronizes validateers, so 'direct invocation' can be used to execute transactions. This means transactions do not need to be submitted to the layer 1 blockchain, but can directly be submitted to a specific validateer instance using an RPC interface. 

## How to use the SDK

1. Experiment with the template, [running the sidechain demo](./demos/sidechain-demo.md)
2. Fork the worker repository `https://github.com/integritee-network/worker.git` from the SDK release branch (`sdk-v0.1.0-polkadot-v0.9.26`)
3. Build the worker in sidechain mode
4. Write and integrate your own business logic, e.g. in a substrate pallet
5. Deploy on the [Integritee parachain](integrate-with-integritee-parachain.md) or you own solo chain

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

In case you want to extend the existing RPC interface, [this section](rpc-interface.md) tells you how.

### CLI Client

A simple [CLI client](https://github.com/integritee-network/worker/tree/master/cli) implementation is available in the worker repository, to show case communication with the worker, either by 'direct invocation' or 'indirect invocation'. For a sidechain validateer, 'direct invocation' will likely be the predominant way of communicating with a worker.

The CLI client uses [JSON RPC 2.0](https://www.jsonrpc.org/specification) over web-socket to communicate with a worker (direct invocation and trusted getters), as can be seen [here](https://github.com/integritee-network/worker/blob/a9a5afdb2de093de0062d7cb7ad302b8501e24a0/cli/src/trusted_operation.rs#L226) for example. The web-socket connection is secured by TLS with an enclave self-signed certificate.
