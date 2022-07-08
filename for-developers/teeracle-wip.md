# TEEracle (WIP)

The TEEracle SDK facilitates submission of oracle data into the Integritee parachain.

## How-To: Get Your Currency Exchange Data

We already have a running TEEracle connected to the Integritee parachain that supplies the TEER-USD exchange rate. This exchange rate will be used to offer stable USD fees. Here, we quickly go through the relevant pieces of the code that need to be updated to fetch your own exchange rate.

Most of the integritee-worker's exchange oracle-specific code is found [here](https://github.com/integritee-network/teeracle/tree/master/app-libs/exchange-oracle).

Customizations mostly consist of:

* Defining currency pair
* Defining scheduling, i.e., interval of Web2 data fetches.
* Setting up your CoinGecko or Coinmarket Cap API key (if these sources are used) OR
* If the data comes from a new source: Implementing Web2 data fetches and response parsing

### Currency Pair

Changing the currency pair is fairly simple. The pair to use is currently hard-coded in the main execution file, [here](https://github.com/integritee-network/teeracle/blob/cdd376bcb4ee3c96f54bc0e6e1653759bad1ace9/service/src/main.rs#L576). In the  [Coinmarket Cap](https://github.com/integritee-network/teeracle/blob/master/app-libs/exchange-oracle/src/coin\_market\_cap.rs) and [CoinGecko](https://github.com/integritee-network/teeracle/blob/master/app-libs/exchange-oracle/src/coin\_gecko.rs) source files, we defined a mapping from the currency symbol to the respective ID for the request. If the respective token is not listed in our mapping already, it can be looked up in either:

* [https://coinmarketcap.com/api/documentation/v1/#section/Standards-and-Conventions](https://coinmarketcap.com/api/documentation/v1/#section/Standards-and-Conventions)
* [https://www.coingecko.com/en/api/documentation](https://www.coingecko.com/en/api/documentation)

### Scheduling

The scheduling is as simple as it could be, it is a CLI argument simply set the scheduling with the `-i` or `--interval` flag, e.g.:

```
./integritee-service run -i 5s15m2h1d
```

### Setup API Keys

TODO

### Web2 Data Fetching

This part is only needed if support for other data sources than Coinmarket Cap or CoinGecko is wished for. Our code for these two sources can be found in; [Coinmarket Cap](https://github.com/integritee-network/teeracle/blob/master/app-libs/exchange-oracle/src/coin\_market\_cap.rs) and [CoinGecko](https://github.com/integritee-network/teeracle/blob/master/app-libs/exchange-oracle/src/coin\_gecko.rs). Most of the code is very standard https-request code, revolving around performing the request and parsing the response data. However, it should be emphasized that we explicitly check the TLS-Root-Certificates of our sources to be correct. This gives us **non-repudiation,** i.e, it is guaranteed that the result is indeed from the advertised source.

The interesting part is the [OracleSource](https://github.com/integritee-network/teeracle/blob/70fc6196e3f2fb2ddfde13619fcb35cc057990de/app-libs/exchange-oracle/src/exchange\_rate\_oracle.rs#L32) trait. The generic abstraction for the oracle source data. This pattern is used to minimize the boilerplate code for new sources that should be added to the enclave. Currently, the abstraction level assumes that we are handling currency pairs. In the future, this will be generic enough to handle arbitrary data.

Here we see how the `OracleSource` trait is implemented for the `CoinGeckoSource`

```
impl OracleSource for CoinGeckoSource {
	// for prometheus monitoring
	fn metrics_id(&self) -> String {
		"coin_gecko".to_string()
	}

	fn request_timeout(&self) -> Option<Duration> {
		Some(COINGECKO_TIMEOUT)
	}

	fn base_url(&self) -> Result<Url, Error> {
		Url::parse(COINGECKO_URL).map_err(|e| Error::Other(format!("{:?}", e).into()))
	}

	fn root_certificate_content(&self) -> String {
		COINGECKO_ROOT_CERTIFICATE.to_string()
	}

	fn execute_exchange_rate_request(
		&self,
		rest_client: &mut RestClient<HttpClient<SendWithCertificateVerification>>,
		trading_pair: TradingPair,
	) -> Result<ExchangeRate, Error> {
		let fiat_id = trading_pair.fiat_currency.clone();
		let crypto_id = Self::map_crypto_currency_id(&trading_pair)?;

		let response = rest_client
			.get_with::<String, CoinGeckoMarket>(
				COINGECKO_PATH.to_string(),
				&[(COINGECKO_PARAM_CURRENCY, &fiat_id), (COINGECKO_PARAM_COIN, &crypto_id)],
			)
			.map_err(Error::RestClient)?;

		let list = response.0;
		if list.is_empty() {
			return Err(Error::NoValidData(COINGECKO_URL.to_string(), trading_pair.key()))
		}

		match list[0].current_price {
			Some(r) => Ok(ExchangeRate::from_num(r)),
			None => Err(Error::EmptyExchangeRate(trading_pair)),
		}
	}
}

```

There is a bit more code for the `CoinGeckoSource`, but this snipped mostly covers what is needed to add a new source.
