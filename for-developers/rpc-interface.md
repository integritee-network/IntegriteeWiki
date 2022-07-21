# RPC Interface

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