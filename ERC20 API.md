ERC20 API
=========

Version
-------

All API in this reference accept version argument which specifies version of API requested. The latest version of API is described in this document. Previous versions are used for backward compatibility during transitioning periods and deprecated as long as no server is using them. version argument is optional and if not specified, it resolves to the latest version.

**Current API version**: 2

GetPrimaryCoins
---------------

**Description**

Retrieves a list of primary coins available where each coin represents an entirely new market on the exchange.

| GET parameter | Value | Notes |
|---------------|-------|-------|
|```action```|```get_primary_coins```|Should be always ```get_primary_coins``` for this API call|


**Output**

An array of symbols for primary coins.

**Example**

[http://api.erc20.exchange/index.php?action=get\_primary\_coins](http://api.erc20.exchange/index.php?action=get_primary_coins)

**Response**

```json
["ETC"]
```

GetSecondaryCoins
-----------------

**Description**

Retrieves a list of secondary coins available for trading against a primary coin. As an input primary coin should be provided. A list of primary coins (markets) may be retrieved with [GetPrimaryCoins](#bookmark=id.r5jpkypgzd1f) API.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_secondary_coins```|Should be always ```get_secondary_coins``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|



**Output**

Returns a list of secondary coins where every coin element is described by the following fields (all fields must present in the output):

* ```Symbol```: A symbolic name which represents the coin (text)
* ```Name```: A full name of the coin (text)
* ```Approved```: An indicator that tells if coin has been reviewed by staff member (0 or 1)

**Example**

[http://api.erc20.exchange/index.php?action=get\_secondary\_coins&primary=ETC](http://api.erc20.exchange/index.php?action=get_secondary_coins&primary=ETC)

**Response**

```json
[{
	"Symbol": "BAG",
	"Name": "Bagholder Coin",
	"Approved": 0
}, {
	"Symbol": "LGD",
	"Name": "Legendary Coin",
	"Approved": 1
}]
```

GetFees
-------

**Description**

Retrieves fees applied on the exchange for trading.


|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_fees```|Should be always ```get_fees``` for this API call|
|```primary```|Optional: A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API|


**Output**

If primary coin is not specified, then always returns two float numbers:

* ```MakerFee```: A float value which represents percent taken from the order for makers.
* ```TakerFee```: A float value which represents percent taken from  the order for takers.

If primary coin is specified, then always returns one float numbers:

* ```DeployFee```: A float value (in primary coin units) which is a fee taken for coin creation.

**Example**

[http://api.erc20.exchange/index.php?action=get_fees](http://api.erc20.exchange/index.php?action=get_fees) 

**Response**

```json
[{
	"MakerFee": 0.05,
	"TakerFee": 0.15
}]
```

GetBalance
----------

**Description**

Retrieves how many primary and secondary coins are available for trading for a given address (level of allowance for secondary coins, actual balance for primary).

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_balance```|Should be always ```get_balance``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```address```|A user address which is checked for coins balance|A hexadecimal string representing user address.|

**Output**

Always returns:

* ```Balance```: The amount of primary coins available for a given address.
* All balances for secondary coins where field names match tickers of secondary coins.
* Future balances for coins in case if there is a pending transaction which is planning to change user’s balance. This data is contained in ```Pending``` dictionary. Every key in this dictionary is one of secondary coins listed for the marked. Not every secondary coin is listed in “Pending” dictionary, but only the ones which are expected to change user’s balance. ```Pending``` dictionary is always present in the response.

**Example**

[http://api.erc20.exchange/?action=get_balance&primary=ETC&address=0xef7b8Ec0afAc38faf6b89480767878811f66A7b8](http://api.erc20.exchange/?action=get_balance&primary=ETC&secondary=BAG&address=0xef7b8Ec0afAc38faf6b89480767878811f66A7b8)  

**Response**

```json
[{
	"Balance": 10.1283,
	"Coins": {
		"BAG": 33.1,
		"TOP": 3.1,
		"BUG": 303.1,
		"LBC": 33.11
	},
	"Pending": {
		"BAG": 330
	}
}]
```

GetCoinInfo
-----------

**Description**

Retrieves real-time information about coins: current price, 24 hours volume, 24 hours change in price.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_coin_info```|Should be always ```get_coin_info``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|

**Output**

Always returns the following fields:

* ```Status```: Status of the market (One of the following: ```Active```, ```Disabled```, ```Slow```, ```Very Slow```).
* ```Reason```: Optional field which is available only when market is disabled. Contains code of the problem (pls, see all codes and their meaning below).
* ```Coins```: Array which contains all secondary coins where each element of array contains the following fields:
	* ```Ticker```: Ticker of secondary coin.
	* ```Status```: ```Active``` or ```Disabled``` depending on if trades are running for a coin.
	* ```Price```: Current price of the coin (a float number expressed in units of primary coin).
	* ```Volume```: 24 hours trading volume (a float number expressed in units of primary coin).
	* ```Change```: 24 hours change in price (percent).

**Reasons**

|Code|Description|
|--- |--- |
|1|Backend node is not functioning properly.|
|2|Backend node is not syncing.|
|3|Infrastructure error.|
|4|Node is lagging behind the network.|


**Example**

[http://api.erc20.exchange/index.php?action=get\_coin\_info&primary=ETC](http://api.erc20.exchange/index.php?action=get_coin_info&primary=ETC)

**Response**

```json
{
	"Status": "Active",
	"Coins": [{
		"Ticker": "PTOY",
		"Status": "Active",
		"Price": 0.1,
		"Volume": 0.1,
		"Change": -10
	}, {
		"Ticker": "LBC",
		"Status": "Active",
		"Price": 0.1,
		"Volume": 0.1,
		"Change": -10
	}]
}
```

GetOpenOrders
-------------

**Description**

Retrieves all open orders for a given secondary coin. If address is provided then output will consists of open orders belonging to a given address. 

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_open_orders```|Should be always ```get_open_orders``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```secondary```|A secondary coin symbol name.|One of the coins returned in response to GetSecondaryCoins API.|
|```address```|Optional: A hexadecimal string representing user address|If provided, only open orders associated with a given address will be returned.|

**Output**

Returns an array of open orders where each of them consists of the following fields:

* ```Id```: A integer number to represent order ID which unique for an order. Negative Id means that order is being cancelled, but it is not confirmed yet.
* ```Date```: A integer number of seconds since 1st, January 1970 (Epoch Unix Time).
* ```Type```: Can be one of the following values:
	* ```Sell```: A sell order registered on the blockchain.
	* ```Buy```: A buy order registered on the blockchain.
	* ```Incoming Sell```: A sell order which is found in pending tx.
	* ```Incoming Buy```: A buy order which is found in pending tx.
* ```Price```: Price of the coin (a float number expressed in primary coin units).
* ```Amount```: Amount of coins bought or sold with an order (float number).

**Example**

[http://api.erc20.exchange/index.php?action=get\_open\_orders&primary=ETC&secondary=BAG](http://api.erc20.exchange/index.php?action=get_open_orders&primary=ETC&secondary=BAG)

**Response**

```json
[{
	"Id": 10,
	"Date": 23872383,
	"Type": "Sell",
	"Price": 0.00000019,
	"Amount": 10.1283
}]
```

GetHistory
----------

**Description**

Retrieves information about executed orders. If address is provided then output will consists of executed orders belonging to a given address.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_history```|Should be always ```get_history``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```secondary```|A secondary coin symbol name.|One of the coins returned in response to GetSecondaryCoins API.|
|```address```|Optional: A hexadecimal string representing user address|If provided, only executed orders associated with a given address will be returned.|


**Output**

Returns an array of open orders where each of them consists of the following fields:

* ```Date```: A integer number of seconds since 1st, January 1970 (Epoch Unix Time).
* ```Type```: Either ```Sell``` or ```Buy``` representing a type of the order.
* ```Price```: Price of the coin (a float number expressed in primary coin units).
* ```Amount```: Amount of coins bought or sold with an order (float number).

**Example**

[http://api.erc20.exchange/index.php?action=get_history&primary=ETC&secondary=BAG](http://api.erc20.exchange/index.php?action=get_history&primary=ETC&secondary=BAG)

**Response**

```json
[{
	"Date": 23872383,
	"Type": "Sell",
	"Price": 0.00000019,
	"Amount": 10.1283
}]
```

GetChartData
------------

**Description**

Retrieves real-time information about secondary coin price. Data is requested by providing a starting point in time, a type of the interval and number of interval to report back from API. Intervals are returned in sorted order starting from the oldest.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_chart_data```|Should be always ```get_chart_data``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```secondary```|A secondary coin symbol name.|One of the coins returned in response to GetSecondaryCoins API.|
|```start```|Timestamp for the first interval|Timestamp is expressed in “YYYY-MM-DD HH:MM” format.|
|```interval```|Type of the interval as a string (e.g., ```5min```).|The following types of intervals are supported: ```5min```, ```15min```, ```30min```, ```1h```, ```3h```, ```6h```, ```12h```, ```1d```, ```3d```, ```1w```, ```3w```|
|```count```|A integer number|Number of interval to report back|


**Output**

Returns an array of intervals where each of them consists of the following fields:

* ```Date```: Start date of the interval in "YYYY-MM-DD HH:MM" format.
* ```Open```: Opening price (a float number).
* ```High```: The highest price for the interval (a float number).
* ```Low```: The lowest price for the interval (a float number).
* ```Close```: Closing price (a float number).
* ```Volume```: Volume expressed in primary coin units (a float number).

**Response**

[http://api.erc20.exchange/index.php?action=get\_chart\_data&primary=ETC&secondary=BAG&start=1993-01-29 16:00&interval=1h&count=2](http://api.erc20.exchange/index.php?action=get_chart_data&primary=ETC&secondary=BAG&start=1993-01-29 16:00&interval=1h&count=2)

**JSON output:**

```json
[
    {
        "Date":"1993-01-29 16:00",
        "Open":43.97,
        "High":43.97,
        "Low":43.75,
        "Close":43.94,
        "Volume":1003200
    },
    {
        "Date":"1993-01-29 17:00",
        "Open":43.97,
        "High":44.25,
        "Low":43.97,
        "Close":44.25,
        "Volume":480500
    }
]
```

GetCoinStaticInfo
-----------------

**Description**

Retrieves static information about the coin: ABI, address, number of decimals, etc.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_coin_static_info```|Should be always ```get_coin_static_info``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|

**Output**

Always returns the following:

* ```Digits```: A number of fraction digits used for coin representation.
* ```Address```: Address of the ERC20 contract published for this primary coin.
* ```ABI```: ABI of published ERC20 contract to use for contract function invocation.
* ```MinimumOrder```: A minimal order which is accepted by the exchange.
* ```TxLink```: Transaction link pattern to use for transaction tracking.
* ```MarkBlock```: Which block needs to be tested to recognize the coin.
* ```MarkHash```: Hash associated with a MarkBlock block number.
* ```Trezor```: ```Yes``` if this market is active for Trezor users, ```No``` otherwise.
* ```TrezorPath```: Optional and available only if Trezor field equals ```Yes```. Specified Trezor path for transaction signing.
* ```ERC20_ABI```: It is an ABI to use with all token contracts
* ```ERC20```: An array describing all secondary coins where each elements contains the following fields:
    * ```Digits```: A number of fraction digits used for coin representation.
    * ```Address```: Address of the coin contract represented as a text string.
    * ```MinimumOrder```: A minimal order which is accepted by the exchange.

**Example**

[http://api.erc20.exchange/index.php?action=get\_coin\_static\_info&primary=ROPSTEN](http://api.erc20.exchange/index.php?action=get_coin_static_info&primary=ROPSTEN)

**Response**

```json
[{
	"Digits": 18,
	"Address": "0x52a8c02f5472c59422498C3e1cCeCcc1d0d3C57E",
	"ABI": "...",
	"MinimumOrder": 0.000000000000000000,
	"TxLink": "https://ropsten.etherscan.io/tx/#TX#",
	"MarkBlock": 1920000,
	"MarkHash": "0x9a5fc0d8ac0fb3f73e781256fe0119daf54324bef79c5b387662944b13e5a9b0",
	"Trezor": "Yes",
	"ERC20_ABI": "...",
	"ERC20": [{
		"Ticker": "TOP",
		"Digits": 8,
		"Address": "0x0f006d0E4928Da6A0e361F5D51ac9cfA16Db07a5",
		"MinimumOrder": 0.00000000
	}, {
		"Ticker": "COOL",
		"Digits": 8,
		"Address": "0x6dC33bCD0657eE434A7C1e78629e653B95E0fbB2",
		"MinimumOrder": 0.00000000
	}, {
		"Ticker": "ABC",
		"Digits": 8,
		"Address": "0xC695381457C00E09917a686c3A04ffB03484356B",
		"MinimumOrder": 0.00000000
	}],
	"TrezorPath": "m/44'/1'/0'/0/0"
}]
```

Broadcast
---------

**Description**

Broadcasts signed transaction to the network. This API is used to execute "buy", “sell”, “create coin”, “set allowance” functionality as they all represent transaction to a contract.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```broadcast```|Should be always ```broadcast``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```data```|A hexadecimal string representing signed transactions|Transaction signed by a user to be broadcasted to a primary coin network.|

**Output**

Returns a transaction number:

* ```tx```: A hexadecimal string representing transaction number.

**Example GET request:**

[http://api.erc20.exchange/index.php?action=broadcast&primary=ETC&data=0x76487237692388236](http://api.erc20.exchange/index.php?action=broadcast&primary=ETC&data=0x76487237692388236)

**Response**

```json
[{
	"tx": "0xef7b8Ec0afAc38faf6b89480767878811f66A7b8"
}]
```

GetTxData
---------

**Description**

Provides transaction data required for signing for a desired type of user’s action.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```get_tx_data```|Should be always ```get_tx_data``` for this API call.|
|```primary```|A primary coin symbol name|One of the coins returned in response to GetPrimaryCoins API.|
|```tx_type```|User’s action which requires signing|One of the following: ```set_allowance```, ```add_coin```, ```buy```, ```sell```, ```cancel```.|
|```address```|User’s wallet address|User’s address which is used for signing.|
|```secondary```|Optional: specifies secondary coin selected for transaction. Required for all tx types, except ```add_coin```.|One of the coins returned in response to GetSecondaryCoins API.|
|```allowance```|Required only for ```set_allowance``` tx type.|A string representing a big integer which defines allowance parameter for a contract call.|
|```coin_ticker```|Required only for ```add_coin``` tx type.|Ticker of the coin to add.|
|```coin_name```|Required only for ```add_coin``` tx type.|Name of the coin to add.|
|```coin_address```|Required only for ```add_coin``` tx type.|Contract address of the coin.|
|```coin_digits```|Required only for ```add_coin``` tx type.|Number of digits after decimal point.|
|```price```|Required only for ```buy``` and ```sell``` tx types.|Price of an order as a big integer.|
|```amount```|Required only for ```buy``` and ```sell``` tx types.|Amount of coins in an order as a big integer.|
|```order_id```|Required only for ```cancel``` tx types.|Order id to cancel.|

**Output**

Returns parameters of the transaction to sign with Trezor. Output always includes the following fields:

* ```Address```: Address on Trezor for tx signing.
* ```Nonce```: Number of sender’s transactions as a hexadecimal string.
* ```GasPrice```: Gas price to pay for the transaction as a hexadecimal string.
* ```GasLimit```: Gas limit for the transaction as a hexadecimal string.
* ```To```: Destination address as a hexadecimal string.
* ```Value```: Value in Wei which will be sent with transaction as a hexadecimal string.
* ```Data```: Contract data as a hexadecimal string.
* ```ChainId```: Id of the chain where this transaction needs to be deployed as an integer.

**Response**

[http://api.erc20.exchange/index.php?action=get\_tx\_data&primary=ROPSTEN&secondary=TOP&tx\_type=set\_allowance&address=0x6A55986C77A97cD0618D42a50be9c5ff9f3b6dbb&allowance=100000000](http://api.erc20.exchange/index.php?action=get_tx_data&primary=ROPSTEN&secondary=TOP&tx_type=set_allowance&address=0x6A55986C77A97cD0618D42a50be9c5ff9f3b6dbb&allowance=100000000)

**Response**

```json
{
	"Address": "m/44'/1'/0'/0/0",
	"Nonce": "0x7b",
	"GasPrice": "0x3b9aca00",
	"GasLimit": "0x7666",
	"To": "0x0f006d0E4928Da6A0e361F5D51ac9cfA16Db07a5",
	"Value": "0x0",
	"Data": "0x095ea7b30000000000000000000000003c1b67af99471ba77fe2bcd46ca3507685f055cc0000000000000000000000000000000000000000000000000000000005f5e100",
	"ChainId": 1
}
```

TestNewCoin
-----------

**Description**

Tests if a token with a given ticker may be added on the exchange. Race condition is possible (but highly unlikely on practice) when two users tests at the same time, get positive response and both submits a transaction. One tx will eventually fail.

|GET parameter|Value|Notes|
|--- |--- |--- |
|```action```|```test_new_coin```|Should be always ```test_new_coin``` for this API call.|
|```primary```|Primary coin market|One of the coins returned in response to GetPrimaryCoins API.|
|```ticker```|Ticker of a coin which user is about to add.||

**Output**

Output always contain the following field:

* ```Result```: Code which defines if coin with proposed ticker can be added to the exchange. 

|Code|Description|
|--- |--- |
|0|Coin can be added to the exchange.|
|-1|There is already an ongoing transaction to add a coin with the same ticker.|
|-2|Coin with this ticker already scheduled for the review.|
|-3|Coin with this ticker already listed on the exchange.|

**Example**

[http://api.erc20.exchange/index.php?action=test\_new\_coin&ticker=ABC](http://api.erc20.exchange/index.php?action=test_new_coin&ticker=ABC) 

**Response**

```json
{
	"Result": 0
}
```
