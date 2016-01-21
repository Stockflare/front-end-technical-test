# Front End Technical Test

Stockflare's page for a stock is one of the key elements of our site, Apple's stock page can be found here https://stockflare.com/stocks/AAPL.O

We have a number of api endpoints that provide the stock information, we would like you to write a stock web page of your own design to display some of the information available from our apis.

You can:
*  Choose any platform, language, framework as long as the entire project can be delivered in a single github repo.
*  Choose to display whatever information you want about the stock as long as you display the `short_name`, `ric` `rating` and `price`.

Make sure that:
* The page can display any stock using either `ric` or `sic` as the key in the url so your urls need to be along the lines of http://mysite.com/stocks/:sic or http://mysite.com/stocks?sic=xxx, choose one of the key fields, you don't need to support both.
* Anything displayed that is available from one of our apis must be retrieved by the page from the api, not hard coded.
* We can run the project, the build and running instructions must be part of the README.md of the project.  We will run your code on a clean LINUX VM so you can feel free to use any tech that can be easily installed on linux.
* Submit the test as a github repo, either public or private, if private then please add https://github.com/stratmm and https://github.com/davidkelley as collaborators.

Things of note:
* Don't feel the need to spend a huge amount of time on this test.
* We want to find out what your coding / work style is like and how you choose to organize a project.
* We don't want you to feel you have to produce a huge volume of code, if you have code from previous projects that will save you time then feel free to use it.
* Don't feel the need to add in every conceivable CSS flourish, the page just needs to look clean and nice.


## APIs

There are three apis available for you to use.  We front the APIs though the Amazon AWS-API-Gateway product, because we use a lot of array and object based parameters and AWS-API-Gateway does not support this in `GET` calls we generally publicly expose the endpoints as `PUT` calls so that the parameters can be sent in the request body.

The `Search` API is a convenient method of retrieving all stock data and shows the latest data for any stock.  The `Instrument` API proved basic details of a stock while the `Historical` API provides time series data on stocks for values that change every day and is useful for return graphs.

Use any or all of these API as you choose, they are there simply for your convenience and don't feel obliged to use them all.

### Search API

https://dozlacmd51.execute-api.us-east-1.amazonaws.com/v1/search

The Search API aims to abstract and simplify specific calls to the Elasticsearch backend. The API provides a number of endpoints facilitating the requests that are required by the Stockflare software stack. Such endpoints simplify searching for a company by name, for example.

#### Getting a list of stocks by text search term

PUT https://dozlacmd51.execute-api.us-east-1.amazonaws.com/v1/search with body
```
{
  "term": "apple",
  "select": "_all"
}
```

This will give you a list of stocks with `apple` in their name.

#### Getting a single stock by id

PUT https://dozlacmd51.execute-api.us-east-1.amazonaws.com/v1/search/filter with body
```
{
  "conditions": {
      "_id": "6c8227be-6855-11e4-98bf-294717b2347c"
  },
  "select": "_all"
}
```

This example will return an array as before however it will contain only one element which will be Apple.  in this case due to how Elasticsearch works we need to use `_id` however the value of `_id` is always the same as the `sic` (Stockflare Instrument Code) attribute.

### Full API documentation

#### GET or PUT: `/search`

Performs a full text search over the fields provided, with results weighted by their associated values.

| Parameter    | Type     | Values                                    | Default                       |
|:-------------|:---------|:------------------------------------------|:------------------------------|
| `term`       | `string` | `apple, etc.`                             | `nil`                         |
| `conditions` | `hash`   | `{ long_term_growth: "1..50" }`           | `nil`                         |
| `select`     | `Array`  | `['ric', 'long_name', 'price']` or `_all` | `sic`                         |
| `sorting`    | `hash`   | `{ price: 'desc' }`                       | `{ market_value_usd: 'desc'}` |

The conditions hash supports a custom field type that enables the definition of a range of field conditions that are converted into their associated structure before being sent to Elasticsearch.

#### GET or PUT: `/filter`

Filters records based upon the parameters received. Parameters are parsed in a number of ways, providing a greater flexibility of querying.

| Parameter    | Type    | Values                                    | Default                       |
|:-------------|:--------|:------------------------------------------|:------------------------------|
| `conditions` | `hash`  | `{ long_term_growth: "1..50" }`           | `nil`                         |
| `select`     | `Array` | `['ric', 'long_name', 'price']` or `_all` | `sic`                         |
| `sorting`    | `hash`  | `{ price: 'desc' }`                       | `{ market_value_usd: 'desc'}` |

The conditions hash supports a custom field type that enables the definition of a range of field conditions that are converted into their associated structure before being sent to Elasticsearch.

**Note:** Any fields can be defined inside the `conditions` hash.

#### GET or PUT: `/aggregate`

Performs the desired aggregation over the field specified, using the list of matching records returned and as specified from the `fields` key.

| Parameter    | Type     | Values                          | Default |
|:-------------|:---------|:--------------------------------|:--------|
| `type`       | `string` | `"avg"`                         | `nil`   |
| `field`      | `string` | `market_value_usd`              | `nil`   |
| `conditions` | `hash`   | `{ long_term_growth: "1..50" }` | `nil`   |

**Note:** The provided `field` determines the data that is used within the aggregation.

The `?type=` field supports `avg`, `min`, `max`, `sum` and `cardinality` (count unique values).

---

##### Condition Value Structures

The values provided for each field within the `conditions` hash can take on values similar to the examples defined below.

|  Example Value  | Type                   | Matched By           |
|:---------------:|:-----------------------|:---------------------|
|   `"50..70"`    | `Between`              | `/[0-9]+\.\.[0-9]+/` |
|     `">20"`     | `GreaterThan`          | `/\A\\>[0-9]+\Z/`    |
|     `"<80"`     | `LessThan`             | `/\A\\<[0-9]+\Z/`    |
|    `"<=60"`     | `LessThanOrEqualTo`    | `/\A\\<\=[0-9]+\Z/`  |
|     `">=3"`     | `GreaterThanOrEqualTo` | `/\A\\>\=[0-9]+\Z/`  |
| `"usa,gbr,fra"` | `List`                 | `/(.+(,|\Z)/`        |
|    `"true"`     | `Boolean`              | `/\A(true|false)\Z/` |
|     `"usd"`     | `Value`                | `/\A.+\Z/`           |
|   `"abc*ddd"`   | `Wildcard`             | `/\*/`               |

### Instrument API

Returns data pertaining to Instruments (Stocks, ETFs, REITs, etc.) and their associated Companies. This API responds with typically static, non-time-series information for specific instruments, such as the company name "Apple Inc", when the RIC AAPL.O is requested.

Instruments can be requested using an `isin` code, a `ric` or `repo_no` parameter. The last two being specific to Reuters. This API supports requesting multiple or single ISIN codes in one call. However, when using a `ric` or `repo_no` parameter, only a single instrument can be requested at once.

#### Get the details of an instrument by SIC

PUT https://dozlacmd51.execute-api.us-east-1.amazonaws.com/v1/instruments with body
```
{
    "sic": "6c8227be-6855-11e4-98bf-294717b2347c"
}
```

This will return the details of Apple

#### Full API Documentation

##### `GET:/` or `PUT:/`

| Parameter | Required? | Type          | Description                                                                            |
|:----------|:----------|:--------------|:---------------------------------------------------------------------------------------|
| `ric`     | no        | String        | The Reuters identification code for a specific instrument                              |
| `repo_no` | no        | String        | The Reuters fundamentals repository number for a specific instrument                   |
| `isin`    | no        | String        | The recognised International Securities Identification Number (ISIN) for an instrument |
| `sic`     | no        | String        | The Stockflare Ident Code for the instrument                                           |
| `sics`    | no        | Array[String] | (An array form of the singular parameter)                                              |

**Note:** All parameters are mutually exclusive.

### Historical API

Returns time-series based historical information for all Instruments tracked by Stockflare (Equities, ETFs, REITs, etc.). This API offers flexible querying for a large tableset of data, but is limited to returning specific fields as determined by the `select` key.

Historical data can only be requested using a `sic` (Stockflare Instrument Code) parameter.

#### Get the history of a price for a stock
PUT https://dozlacmd51.execute-api.us-east-1.amazonaws.com/v1/historical with body
```
{
    "sic": "6c8227be-6855-11e4-98bf-294717b2347c",
    "after": 0,
    "select": ["price", "market_value_usd"]
}
```

This will return an array of the stock `price` and `market_value_usd` over time.  The updated_at is a UNIX timestamp of the pricing date.

#### Full API Documentation
##### `<GET|PUT>:/`

| Parameter | Required? | Type          | Description                                          | Default                   |
|:----------|:----------|:--------------|:-----------------------------------------------------|:--------------------------|
| `sic`     | yes       | String        | The Stockflare Identification code for an instrument |                           |
| `select`  | no        | Array[String] | The time-series data keys to return data for         | `["price"]` or `["_all"]` |
| `after`   | yes       | Integer       | Return data after this specific epoch timestamp      |                           |
| `before`  | no        | Integer       | Return data before this specific epoch timestamp     | (Time.now.utc.to_i)       |

Below is an example response:

```
[
  ...,
  {
    "price": "110.45",
    "updated_at": 1444144597
  },
  ...
]
```

Below is a full map of the fields currently tracked and available to the `select` parameter along with example values:

```
{
  "high_growth": true,
  "cheaper": true,
  "upside": true,
  "profitable": true,
  "updated_at": 1444144597,
  "dividends": true,
  "growing": true,
  "rating": 5,
  "price": 110.77,
  "fifty_two_week_high": 134.53,
  "fifty_two_week_low": 92,
  "ten_day_average_volume": 55.84,
  "pe_ratio": 12.79,
  "eps": 8.65,
  "book_value": 22.02,
  "cash_flow_per_share": 10.37,
  "dps": 1.92,
  "recommendation": 1.9375,
  "target_price": 147.19,
  "eps_next_quarter": 1.87,
  "forecast_pe": 12.13,
  "forecast_eps": 9.12,
  "forecast_dps": 1.98,
  "forecast_dividend_yield": 0.01,
  "price_to_book_value": 5.02,
  "shares_outstanding": 5702722048,
  "long_term_gain": 0.20,
  "peer_average_long_term_growth": 13.62,
  "peer_average_forecast_pe": 17.49,
  "recommendation_text": "buy",
  "momentum": 0.35,
  "price_change": 0.36,
  "one_month_forecast_eps_change": 0.00,
  "three_month_forecast_eps_change": 0.00,
  "one_month_price_change": -0.01,
  "three_month_price_change": -0.11,
  "free_cash_flow_yield": 0.1076,
  "seven_year_gain": 1.6177,
  "per_year_gain": 0
  "net_cash": false,
  "enterprise_value": 651462.625,
  "market_value": 631747.625,
  "long_term_growth": 14.23
  "forecast_sales": 233233.21875,
  "sales_next_quarter": 51060.9765625,
  "forecast_net_profit": null,
  "return_on_equity": 41.14
  "latest_sales": 224337,
  "operating_profit": 77879,
  "net_profit": 50737,
  "gross_margin": 39.71
  "enterprise_value_to_sales": 2.90
  "market_value_usd": 631747.625,
  "enterprise_value_to_operating_profit": 8.36
}
```