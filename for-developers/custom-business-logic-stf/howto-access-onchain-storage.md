# How To Access On-Chain Storage From Within The Enclave Trustlessly

Integritee isolates *confidential state* (what the STF `TrustedCall` operates on inside the SGX enclave) from *on-chain state* (what is plaintext readable by the entire network of integritee-nodes). Some use cases, however, require read access to on-chain storage for their `TrustedCall`s. As the enclave can't trust its worker service, it has to request and verify read proofs from the integritee-node.

Our goal is that you can use the same pallets that you use on-chain also inside Integritee enclaves. Therefore, we are mapping storage keys directly between confidential state and on-chain state. Your `TrustedCall` has to specify what storage keys it requires and these will be mapped to the confidential state before executing the call.

For this to work, the [sgx-runtime](https://github.com/integritee-network/sgx-runtime/tree/master/runtime) must be compatible with the [node-runtime](https://github.com/integritee-network/integritee-node/tree/master/runtime). This means that the same substrate version must be used. However, it does not mean that the same pallets must be instantiated.

Until [#113](https://github.com/integritee-network/worker/issues/113) is resolved, we also have the restriction that `StorageMap` and `StorageDoubleMap` must use `StorageHasher::Blake2_128Concat`.

## Trusted Time Example

Inside the enclave we don't have a trusted time source (We could use Intel's AESM with `sgx_get_trusted_time` but that would extend our trust assumptions). The blockchain delivers trusted time because every block includes a UTC timestamp which is agreed upon by consensus (within a certain tolerance).

For this example, we access on-chain time using substrate's [timestamp](https://crates.parity.io/pallet_timestamp/index.html) pallet. More precisely, we will enable you to call `Timestamp::<T>::now()` from any pallet in your STF. You will get the UTC timestamp from the block that includes your `TrustedCall`.

### Key Mapping

In your STF, you'll have to define what on-chain storage keys shall be mapped for each `TrustedCall`:

```rust
pub fn get_storage_hashes_to_update(call: &TrustedCall) -> Vec<Vec<u8>> {
    let mut key_hashes = Vec::new();
    match call {
        TrustedCall::your_time_aware_call(_) => {
            key_hashes.push(
                storage_value_key("Timestamp","Now"));
        }
        // more calls ....
    };
    key_hashes
}
```

This part of the code can be found [here](https://github.com/integritee-network/worker/blob/a9a5afdb2de093de0062d7cb7ad302b8501e24a0/app-libs/stf/src/stf_sgx.rs#L300) in the worker repository.

In your pallet you can now query timestamp as usual

```rust
decl_module! {
    pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        #[weight = 10_000]
        pub fn your_time_aware_call(origin) -> dispatch::DispatchResult {
            ensure!(Timestamp::<T>::now() > EARLIEST_TIME_OF_EXECUTION,
                "too early to call this");
            // ...
```
