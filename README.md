#Last updated: Public Rest API for OMGFIN (2019-10-09)
#From: Public Rest API for OMGFIN (2018-10-20)
# General API Information
* The base endpoint is: **https://omgfin.com**
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  OMGFIN's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.
* Any endpoint can return an ERROR; the error payload is as follows:
```javascript
{
  "status": false,
  "message": "Unauthorized"
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.

# Query limitation
* For `GET` endpoints, request limitation number: 10 per second.
* For `POST`, `PUT`, and `DELETE` endpoints, request limitation number: 3 per second.
* A 401 `HTTP` error will be returned at over limit of limitation.

  
# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-API-KEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
USER_STREAM | Endpoint requires sending a valid API-Key.
MARKET_DATA | Endpoint requires sending a valid API-Key.


* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.
  
## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server

**It recommended to use a small recvWindow of 5000 or less!**

# Public API Endpoints
# ENUM definitions
**Order status:**

* NEW
* OPEN
* PENDING
* FILLED
* CANCELED
* WAITING

**Order types:**

* Limit buy
* Limit sell
* Market buy
* Market sell
* Stop-limit Buy
* Stop-limit sell

**Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

## General endpoints
### Test connectivity
```
GET /api/v1/ping
```
Test connectivity to the Rest API.

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Check server time
```
GET /api/v1/time
```
Test connectivity to the Rest API and get the current server time.

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{
  "serverTime": 1499827319559
}
```

### Exchange information
```
GET /api/v1/exchangeInfo
```
Current exchange trading rules and symbol information

**Parameters:**
NONE

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1508631584636,
  "symbols": [{
    "symbol": "ETHBTC",
    "status": "TRADING",
    "baseAsset": "ETH",
    "baseAssetPrecision": 8,
    "quoteAsset": "BTC",
    "quotePrecision": 8,
    "orderTypes": [
      // These are defined in the `ENUM definitions` section under `Order types (orderTypes)`.
      // All orderTypes are optional.
    ]
  }]
}
```

```
GET /api/v1/coinInfo/[symbol]
```
Current exchange symbol information


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | BTC ...

* If the symbol is not sent, config for all symbols will be returned in an array.

**example:**
```javascript
https://omgfin.com/api/v1/coinInfo/BTC
```

**Response:**
```javascript
[
  {
    "symbol": "BTC",
    "name": "Bitcoin",
    "active": true,
    "icon_path": "https://static.omgfin.com/omgfin/img/icon/btc.png",
    "can_deposit": true,
    "can_withdraw": true,
    "project_url": "https://bitcoin.org/",
    "coin_description": "\n\tBitcoin is the first cryptocurrency ever created. It is, basically, a digital form of money that is highly resistant to frauds and cannot be copied or destroyed. Computer and network technologies have advanced to its current state through one universal property of digital information: most of it can be easily copied. \n\n\tThe fast creation of pretty much everything, from the web to word processors to network programming, relies on the fact that a series of bits can be quickly and easily copied at close to zero cost. It was only a matter of time before computer scientists and developers started to wonder about the other half of the data economy. \n\n\tWhat if data couldn&rsquo;t be copied? What if there were such a thing as a unique piece of data, and what if it could be transmitted from user to user? The practical ramifications were pretty clear right off the bat. Unique data that can&#39;t be copied could be used as digital money. That is how the first cryptocurrency was invented. Most people have little or no experience with this particular kind of digital currencies, so they might ask &quot;what is Bitcoin?&quot; or might want to know how Bitcoin works. \n\n\tThe underlying technology that makes cryptocurrencies so unique is probably one of the most complex topics to the majority of us. Due to its various qualities and functionalities, Bitcoin as a word may be used to define many different things. First, Bitcoin as a cryptocurrency (BTC) is a distributed peer-to-peer (P2P) digital form of money. Second, this digital economic network is operated by an underlying set of rules, the Bitcoin Protocol. Third, the source code for such protocol and the respective software - which is running on many computers worldwide - may also be referred to as Bitcoin. Therefore, the word Bitcoin may be used to refer to the whole ecosystem, encompassing all the above-mentioned functionalities.\n"
  }
]
```

## Market Data endpoints
### Order symbols list
```
GET /api/v1/symbols
```

**Response:**
```javascript
[
    "ETHBTC",
    "UQCBTC",
    "UQCETH",
    "BCHBTC",
    "BCHUQC",
    "BCHETH",
    "NEOBTC",
    "NEOUQC",
    "NEOETH",
    "GASBTC",
    "GASUQC",
    "GASETH",
    "XVGBTC",
    "XVGUQC",
    "XVGETH",
    "BTCUSDT",
    "ETHUSDT",
    "UQCUSDT",
    "BCHUSDT",
    "NEOUSDT",
    "GASUSDT",
    "XVGUSDT"
]
```

### Order book
```
GET /api/v1/orderbook/[market_pair]
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market_pair | STRING | YES | ETHBTC or UQCBTC ...
limit | INT | NO | Default 20; max 500. Valid limits:[5, 10, 20, 50, 100, 500]

**example:**
```javascript
https://omgfin.com/api/v1/orderbook/ETH_BTC?limit=20
```

**Response:**
```javascript
{
  "timestamp": 1570591063,
  "bids": [
    [
      "8076.41",
      "1.65919412"
    ],
    [
      "8073.41",
      "1.45492549"
    ]
  ],
  "asks": [
    [
      "8283.3",
      "1.42864519"
    ],
    [
      "8339.11",
      "1.16695773"
    ]
  ]
}
```


### Kline/Candlestick data
```
GET /api/v1/klines
```
Kline/candlestick bars for a symbol.
Klines are uniquely identified by their open time.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | 1m, 1h, 1d, 1w, 1y
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 300; max 300.

* If startTime and endTime are not sent, the most recent klines are returned.

**example:**
```javascript
https://omgfin.com/api/v1/klines?symbol=ETHBTC&interval=4h&startTime=1531972492000&endTime=1539921292000
```

**Response:**
```javascript
[
  {
    "OT": 1499040000000,      // Open time
    "O": "0.01634790",        // Open
    "H": "0.80000000",        // High
    "L": "0.01575800",        // Low
    "C": "0.01577100",        // Close
    "V": "148976.11427815",   // Volume
    "CT": 1499644799999,      // Close time
    "QV": "2434.19055334",    // Quote asset volume
    "T": 308                  // Number of trades
  }
]
```

### Recent trades list
```
GET /api/v1/trades/[symbol]
```
Get recent trades (up to last 500).

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**example:**
```javascript
https://omgfin.com/api/v1/trades/ETH_BTC?limit=20
or
https://omgfin.com/api/v1/trades?symbol=ETHBTC&limit=20
```

**Response:**
```javascript
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
```

### Compressed/Aggregate trades list
```
GET /api/v1/aggTrades
```
Get compressed, aggregate trades. Trades that fill at the time, from the same
order, with the same price will have the quantity aggregated.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO | Timestamp in ms to get aggregate trades.
endTime | LONG | NO | Timestamp in ms to get aggregate trades.
limit | INT | NO | Default 500; max 1000.

**example:**
```javascript
https://omgfin.com/api/v1/aggTrades?symbol=ETHBTC&limit=20
```

**Response:**
```javascript
[
  {
    "a": 26129,         // Aggregate tradeId
    "p": "0.01633102",  // Price
    "q": "4.70443515",  // Quantity
    "f": 27781,         // First tradeId
    "l": 27781,         // Last tradeId
    "T": 1498793709153, // Timestamp
    "m": true,          // Was the buyer the maker?
    "M": true           // Was the trade the best price match?
  }
]
```

### Current average price
Current average price for a symbol.
```
GET /api/v1/avgPrice
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

**example:**
```javascript
https://omgfin.com/api/v1/avgPrice?symbol=ETHBTC
```

**Response:**
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```

### Overview of market data for all tickers
```
GET /api/v1/ticker/summary
```
Overview of market data for all tickers.

**example:**
```javascript
https://omgfin.com/api/v1/ticker/summary
```

**Response:**
```javascript
{
  "code": 200,
  "msg": "success",
  "data": {
    "BTC_USDT": {
      "last": "7940.98",
      "lowestAsk": "8007.16",
      "highestBid": "7909.22",
      "percentChange": "0",
      "baseVolume": "1.85044979",
      "quoteVolume": "14694.3847733942",
      "high24hr": "7940.98",
      "low24hr": "7940.98"
    }
  }
}
```

### The assets for each currency available on the exchange
```
GET /api/v1/ticker/summary
```
The assets endpoint is to provide a detailed summary for each currency available on the exchange.

**example:**
```javascript
https://omgfin.com/api/v1/assets
```

**Response:**
```javascript
{
  "BTC": {
    "name": "Bitcoin",
    "unified_cryptoasset_id": 1,
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.002",
    "max_withdraw": "10",
    "maker_fee": "0.001",
    "taker_fee": "0"
  },
  "ETH": {
    "name": "Ethereum",
    "unified_cryptoasset_id": 1027,
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.02",
    "max_withdraw": "50",
    "maker_fee": "0.001",
    "taker_fee": "0"
  }
}
```

### 24hr ticker price and volume each market pair available on the exchange
```
GET /api/v1/ticker
```
The ticker endpoint is to provide a 24-hour pricing and volume summary for each market pair available on the exchange.

**Weight:**
1 for a single symbol; **40** when the symbol parameter is omitted

**example:**
```javascript
https://omgfin.com/api/v1/ticker
```

**Response:**
```javascript
{
  "BTC_USDT": {
    "base_id": 1,
    "quote_id": 825,
    "last_price": "8148.28",
    "base_volume": "3.66948109",
    "quote_volume": "29516.3611345582"
  }
}
```

### 24hr ticker price change statistics
```
GET /api/v1/ticker/24hr
```
24 hour price change statistics. **Careful** when accessing this with no symbol.

**Weight:**
1 for a single symbol; **40** when the symbol parameter is omitted

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, tickers for all symbols will be returned in an array.

**example:**
```javascript
https://omgfin.com/api/v1/ticker/24hr?symbol=ETHBTC
```

**Response:**
```javascript
{
    "symbol": "ETH_BTC",
    "openPrice": "0.031785810800000000",
    "lastPrice": "0.031558302700000000",
    "priceChange": "-0.000227508100000000",
    "priceChangePercent": "-0.7209136123787798004739",
    "bidPrice": "0.031487810000000000",
    "askPrice": "0.031788850000000000",
    "highPrice": "0.031844763400000000",
    "lowPrice": "0.031489325800000000",
    "volume": "886.398512000000000000",
    "quoteVolume": "28.037312687091148100000000000000",
    "openTime": "1540108503106",
    "closeTime": "1540194903106",
    "numberOfTrades": "281"
}
```

### Symbol price ticker
```
GET /api/v1/ticker/price
```
Latest price for a symbol or symbols.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.

**example:**
```javascript
https://omgfin.com/api/v1/ticker/price?symbol=ETHBTC
```

**Response:**
```javascript
{
    "symbol": "ETH_BTC",
    "price": "0.056700000000000000"
}
```

### Symbol order book ticker
```
GET /api/v1/ticker/book
```
Best price/qty on the order book for a symbol or symbols.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, bookTickers for all symbols will be returned in an array.

**example:**
```javascript
https://omgfin.com/api/v1/ticker/book?symbol=ETHBTC
```

**Response:**
```javascript
{
    "symbol": "ETH_BTC",
    "bidPrice": "0.031487810000000000",
    "bidQty": "5.381762000000000000",
    "askPrice": "0.031788850000000000",
    "askQty": "1.569320000000000000"
}
```



