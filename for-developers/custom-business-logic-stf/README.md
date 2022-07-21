# Custom Business Logic / STF

When building your own solution, based on the [sidechain SDK](../sidechain-sdk.md) or the [offchain-worker SDK](../offchain-worker-sdk.md), a key task is to integrate your own business logic (also called STF, state transition function). Here we will show you how that is done in the Integritee worker.

## General Concept

Introducing your own business logic is mostly done in the STF crate [`ita-stf`](https://github.com/integritee-network/worker/tree/master/app-libs/stf). The `TrustedCall` and `TrustedGetter` structs are the declarations of the business logic and should be extended to match your use-case. 

The template declaration contains some basic logic for transferring, shielding and unshielding funds.

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

Implementing the business logic is done in [`stf_sgx.rs`](https://github.com/integritee-network/worker/blob/master/app-libs/stf/src/stf_sgx.rs), for each trusted call ([`fn execute(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L126)) and trusted getter ([`fn get_state(..)`](https://github.com/integritee-network/worker/blob/72d9ba960803b367a9cb4f0bc62d0f4a4b13fe6d/app-libs/stf/src/stf_sgx.rs#L89)). 


### Using Substrate Pallets

For developers who are used to writing substrate pallets, they can do so for the Integritee worker as well: Pallets can be imported and used inside the worker STF. This is possible thanks to our own adaptation of the substrate runtime to SGX (see [sgx-runtime repo](https://github.com/integritee-network/sgx-runtime)).

An example is the existing implementation of `balance_transfer`, using the standard substrate `Balance` pallet (imported [here](https://github.com/integritee-network/sgx-runtime/blob/cefb6991a5ddc7f8a1139da8f48a49b6379113df/runtime/src/lib.rs#L53)):

```rust
TrustedCall::balance_transfer(from, to, value) => {
    let origin = sgx_runtime::Origin::signed(from.clone());

    // ...
        
    sgx_runtime::BalancesCall::<Runtime>::transfer {
        dest: MultiAddress::Id(to),
		value,
	}
	.dispatch_bypass_filter(origin)
	.map_err(|e| {
		StfError::Dispatch(format!("Balance Transfer error: {:?}", e.error))
	})?;

	Ok(())
},
```

Code snippet taken from [here](https://github.com/integritee-network/worker/blob/a9a5afdb2de093de0062d7cb7ad302b8501e24a0/app-libs/stf/src/stf_sgx.rs#L155).

## Examples

