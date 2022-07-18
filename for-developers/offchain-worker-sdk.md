# Offchain Worker SDK

## What is an offchain worker?

A TEE offchain worker is used to execute operations in a trusted manner. A client submits these operations to a substrate 'parentchain', which can be either a parachain or a solochain. The operations are opaque to the parentchain because they are encrypted. Only the client and the TEE can see the operation's content.

Sending an encrypted operation to the parentchain, which is then picked up and executed by a TEE offchain worker, is called 'indirect invocation'. An offchain-worker only supports indirect invocation and 'getters'. In order to encrypt the operation, a client can query the offchain worker directly (with a 'getter') for the 'shielding key', which is an RSA 3072 public key.

Multiple offchain workers running in parallel share only the shielding key, but nothing else. They arrive at the same state independently by executing all the operations found on the parentchain, which guarantees ordering of operations.


### What is the difference between an offchain worker and a worker with sidechain?

An offchain-worker supports only 'indirect invocation', which means it executes only operations that are submitted to the parentchain (encrypted, in an extrinsic). As a result, the operation throughput is limited by the block production cycle on a parentchain. 'Direct invocation', available in a worker with sidechain, allows a client to send operations directly to a worker, allowing for much higher operation throughput. This is possible, because the sidechain allows sharing and synchronizing state between workers, without having to rely on the parentchain.

**Note:** More information on the difference between direct and indirect invocation can be found in [this](https://book.integritee.network/design.html) section of the Integritee book.

### Default client workflow

Default workflow for executing operations on an offchain worker from a client's perspective:

1. Compose your operation, e.g. `balance_transfer`
2. Get the shielding key from an off-chain worker
    * In case there are multiple off-chain workers, any of them will work, since they all share the shielding key
3. Encode and encrypt the operation using the shielding key
4. Pack the encrypted operation into an extrinsic
5. Send the extrinsic to the parentchain
6. Wait for the `ProcessedParentchainBlock` event, with the block hash from your extrinsic.

Examples of this workflow can be found in our CLI client implementation, [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_commands.rs#L167) and [here](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/cli/src/trusted_operation.rs#L98).

## Building the worker in offchain worker mode

In order to build the worker in offchain worker mode, the corresponding cargo feature `offchain-worker` needs to be set. In the Makefiles, the environment variable `WORKER_MODE` is used to set the cargo features. 

In case you build with `make` directly, do so with:

```
WORKER_MODE=offchain-worker make
```

In case you use docker with our `build.Dockerfile`, use `--build-arg WORKER_MODE_ARG=offchain-worker` to set the corresponding docker `ARG`.

An example of a docker build command (as currently used for GitHub CI):

```
docker build -t integritee-worker --target deployed-worker --build-arg WORKER_MODE=offchain-worker -f build.Dockerfile .
``` 


## Customizing an offchain worker

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