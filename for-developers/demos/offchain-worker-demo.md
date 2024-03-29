# Offchain-worker Demo

The offchain-worker demo shows indirect invocation with 2 offchain-worker instances running.
The demo sets up an Integritee Node (layer 1 chain) and runs 2 workers that both execute transactions submitted to the layer 1 chain (parentchain) - which is called indirect invocation. 

The CLI client (called by a script) is then used to perform transactions and query state for the `Alice` and `Bob` accounts. Transactions like `balance_transfer` are encrypted and then submitted to the parentchain. An offchain-worker then imports the parentchain block (upon finalization), decrypts and executes any transactions it finds. Both offchain-workers execute these transactions in parallel, there is no exchange of state or transaction information (like there is for the sidechain validateer).

The sequence in the demo is as follows:

1. Create incognito accounts for Alice and Bob, these are not disclosed to the public
2. Shield funds on Alice's account, i.e. moving funds from Alice's public, on-chain account to her incognito account
3. Transfer a fraction of the funds on Alice's incognito account to Bob's incognito account
4. Un-shield the remaining funds on Alice's incognito account, moving them back to Alice's on-chain account
5. Check balances on Alice's public on-chain account and her incognito account

The demo executes this sequence twice and verifies the balances are correct after each run.

For those who are interested in the demo script, can find it [here](TODO).

## Run the offchain-worker demo

In order to run the demo, clone the worker repository:
```
git clone https://github.com/integritee-network/worker.git
```

Change directory to
```
cd docker
```

Build all necessary images in the docker-compose setup, with (takes upwards of 10 minutes, depending on your hardware):
```
COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 docker-compose -f docker-compose.yml -f demo-indirect-invocation.yml build --build-arg WORKER_MODE_ARG=offchain-worker
```

Run the demo in the docker-compose setup
```
docker-compose -f docker-compose.yml -f demo-indirect-invocation.yml up --exit-code-from demo-indirect-invocation
```

You will see log output from the two workers and the demo service. First worker 1 will startup and initialize. Worker 2 waits until worker 1 is fully up and running and then starts up. The demo service again waits for worker 2 to be fully up before running. This whole process can take a couple of minutes and will automatically shut down once the demo script ran through.

Output like
```
integritee-worker-2    | 2022/07/20 13:50:56 Waiting for http://integritee-worker-1:4645/is_initialized: unexpected HTTP status code: 404.
```
is to be expected, it results from the service `integritee-worker-2` polling and waiting until `integritee-worker-1` is initialized (the same will appear for the demo service, with respect to `worker-2`).

A successful demo run will show
```
integritee-indirect-invocation-demo |
integritee-indirect-invocation-demo | Get balance of Alice's incognito account
integritee-indirect-invocation-demo | 20000000000
integritee-indirect-invocation-demo | * Get balance of Alice's on-chain account
integritee-indirect-invocation-demo | 599926837744000
integritee-indirect-invocation-demo |
integritee-indirect-invocation-demo | test passed (2nd time)
integritee-indirect-invocation-demo exited with code 0
```
at the end of the log output, before the docker containers are stopped.

Use
```
docker-compose -f docker-compose.yml -f demo-indirect-invocation.yml down
```
to clean up all the container context (including logs).