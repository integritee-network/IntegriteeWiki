# Sidechain Demo

The sidechain demo can be a good starting point in understanding how the services and components in a sidechain interact.
The demo sets up an Integritee Node (layer 1 chain) and runs 2 workers that synchronize with a sidechain. The CLI client (called by a script) is then used to perform transactions and query state for the `Alice` and `Bob` accounts.

The sequence is as follows:

1. Issue funds to Alice's incognito account `(worker 1)`
2. Query Alice's incognito balance to verify balance `(worker 1)`
3. Transfer funds from Alice's incognito account to Bob's incognito account `(worker 1)`
4. Transfer funds from Alice's incognito account to Bob's incognito account `(worker 2)`
5. Query Alice's incognito balance to verify both transfers above have happened `(worker 1)`
6. Query Bob's incognito balance to verify both transfer above have happened `(worker 2)`

Each call is direct invocation, the invocation target is noted in brackets: e.g. `(worker 1)`. Because we're running a sidechain, it does not matter to which worker we send the direct invocation request, the result of the transaction will be shared across workers using the sidechain.

## Run the sidechain demo

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
COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 docker-compose -f docker-compose.yml -f demo-sidechain.yml build --build-arg WORKER_MODE_ARG=sidechain
```

Run the demo in the docker-compose setup
```
docker-compose -f docker-compose.yml -f demo-sidechain.yml up --exit-code-from demo-sidechain
```

You will see log output from the two workers and the demo service. First worker 1 will startup and initialize. Worker 2 waits until worker 1 is fully up and running and then starts up. The demo service again waits for worker 2 to be fully up before running. This whole process can take a couple of minutes and will automatically shut down once the demo script ran through.

Output like
```
integritee-worker-2    | 2022/07/20 13:50:56 Waiting for http://integritee-worker-1:4645/is_initialized: unexpected HTTP status code: 404.
```
is to be expected, it results from the service `integritee-worker-2` polling and waiting until `integritee-worker-1` is initialized (the same will appear for the demo service, with respect to `worker-2`).

A successful demo run will show
```
integritee-sidechain-demo | * Verifying Alice's balance
integritee-sidechain-demo | Alice's balance is correct (10000000000)
integritee-sidechain-demo |
integritee-sidechain-demo | * Verifying Bob's balance
integritee-sidechain-demo | Bob's balance is correct (40000000000)
integritee-sidechain-demo |
integritee-sidechain-demo exited with code 0
```
at the end of the log output, before the docker containers are stopped.

Use
```
docker-compose -f docker-compose.yml -f demo-sidechain.yml down
```
to clean up all the container context (including logs).