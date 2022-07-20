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

## Integrating you own business logic / STF into the sidechain



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


## Customizing a sidechain validateer

### Business logic

Introducing your own business logic into an offchain worker, is mostly done in the STF crate [`ita-stf`](https://github.com/integritee-network/worker/tree/master/app-libs/stf). The `TrustedCall` and `TrustedGetter` structs are the definitions of the business logic and should be extended to match your use-case. The default implementation contains some basic logic for transferring, shielding and unshielding funds.

```rust
pub enum TrustedCall {
	balance_set_balance(...),
	balance_transfer(...),
	balance_unshield(...),
	balance_shield(...)
}
```

```rust
pub enum TrustedGetter {
	free_balance(...),
	reserved_balance(...),
	nonce(...),
}
```

Executing the business logic is done in [`stf_sgx.rs`](https://github.com/integritee-network/worker/blob/master/app-libs/stf/src/stf_sgx.rs), where you implement your business logic for each trusted call ([`fn execute(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L126)) and trusted getter ([`fn get_state(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L89)). 



#### Example from Integritee book

The [Integritee book](https://book.integritee.network/introduction.html) also shows a concrete [example](https://book.integritee.network/howto_stf.html#integritee-worker) ([Encointer](https://encointer.org/)) of writing your own STF in the worker, with code samples.

*TODO: Internalize all docs from book into this Repo*

### RPC Interface
A worker provides a JSON RPC interface that runs on a secure websocket server inside the enclave. The websocket connection is secured by a TLS connection with a certificate signed with the enclave signing key.

By default, the JSON RPC interface of a worker provides functions to:
* Interact with the trusted operation pool
* Query state (getters)
* Get the shielding key (for encrypting trusted operations on the client side)

You can, of course, also add your own RPC functions by extending the `IoHandler` [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/enclave-runtime/src/rpc/worker_api_direct.rs#L57). Note that all JSON strings, incoming parameters and outgoing results, are expected to be hex encoded.

An example of a new RPC method called `my_own_rpc_method`, added to the `IoHandler` `io` :

```rust
	io.add_sync_method("my_own_rpc_method", move |params: Params| {

        // Get the hex encoded parameters (if any, otherwise skip this).
        let hex_encoded_params = params.parse::<Vec<String>>().map_err(|e| format!("{:?}", e))?;

        // Decode the parameters to your desired concrete object (`Request` in this example).
	    let request =
		    Request::from_hex(&hex_encoded_params[0].clone()).map_err(|e| format!("{:?}", e))?;

        // Process the request and generate response
        let result = call_my_function(request);

        // Return 
		Ok(json!(result.to_hex()))
	});
```
