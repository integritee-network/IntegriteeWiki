# Offchain Worker SDK

## What is an offchain worker?

A TEE offchain worker is used to execute operations in a trusted manner. A client submits these operations to a substrate 'parentchain', which can be either a parachain or a solochain. The operations are opaque to the parentchain because they are encrypted. Only the client and the TEE can see the operation's content.

Sending an encrypted operation to the parentchain, which is then picked up and executed by a TEE offchain worker, is called 'indirect invocation'. An offchain-worker only supports indirect invocation and 'getters'. In order to encrypt the operation, a client can query the offchain worker directly (with a 'getter') for the 'shielding key', which is an RSA 3072 public key.

Multiple offchain workers running in parallel share only the shielding key, but nothing else. They arrive at the same state independently by executing all the operations found on the parentchain, which guarantees ordering of operations.


### What is the difference between an offchain worker and a worker with sidechain?

An offchain-worker supports only 'indirect invocation', which means it executes only operations that are submitted to the parentchain (encrypted, in an extrinsic). As a result, the operation throughput is limited by the block production cycle on a parentchain. 'Direct invocation', available in a worker with sidechain, allows a client to send operations directly to a worker, allowing for much higher operation throughput. This is possible, because the sidechain allows sharing and synchronizing state between workers, without having to rely on the parentchain.

### Default workflow for executing operations on an offchain worker

From a client's perspective:

1. Compose your operation, e.g. `balance_transfer`
2. Get the shielding key from an off-chain worker
    * In case there are multiple off-chain workers, any of them will work, since they all share the shielding key
3. Encode and encrypt the operation using the shielding key
4. Pack the encrypted operation into an extrinsic
5. Send the extrinsic to the parentchain
6. Wait for the `ProcessedParentchainBlock` event, with the block hash from your extrinsic.

## Building the worker in offchain worker mode

In order to build the worker in offchain worker mode, the corresponding cargo feature `offchain-worker` needs to be set. In the Makefiles, the environment variable `ADDITIONAL_WORKER_FEATURES` is used to set the cargo features. 

In case you build with `make` directly, do so with:

```
ADDITIONAL_WORKER_FEATURES=offchain-worker make
```

In case you use docker with our `build.Dockerfile`, use `--build-arg WORKER_MODE=offchain-worker` to set the corresponding docker `ARG`.

An example of a docker build command (as used for GitHub CI):

```
docker build -t integritee-worker --target deployed-worker --build-arg WORKER_MODE=offchain-worker -f build.Dockerfile .
``` 


## Customizing an offchain worker

Introducing your own business logic into an offchain worker, is mostly done in the STF crate [`ita-stf`](https://github.com/integritee-network/worker/tree/master/app-libs/stf). The `TrustedCall` and `TrustedGetter` structs are the definitions of the business logic and should be extended to match your use-case. The default implementation contains some basic logic for transferring, shielding and unshielding funds.

```
pub enum TrustedCall {
	balance_set_balance(...),
	balance_transfer(...),
	balance_unshield(...),
	balance_shield(...)
}
```

```
pub enum TrustedGetter {
	free_balance(...),
	reserved_balance(...),
	nonce(...),
}
```

Executing the business logic is done in [`stf_sgx.rs`](https://github.com/integritee-network/worker/blob/master/app-libs/stf/src/stf_sgx.rs), where you implement your business logic for each trusted call ([`fn execute(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L126)) and trusted getter ([`fn get_state(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L89)). In addition, 


### Example from Integritee book

The [Integritee book](https://book.integritee.network/introduction.html) also shows a concrete [example](https://book.integritee.network/howto_stf.html#integritee-worker) ([Encointer](https://encointer.org/)) of writing your own STF in the worker, with code samples.