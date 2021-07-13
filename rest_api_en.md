- [Request processing](#request-processing)
- [Public data](#public-data)
  - [Getting active currencies list](#getting-active-currencies-list)
  - [Getting active pairs list](#getting-active-pairs-list)
  - [Getting prices](#getting-prices)
  - [Calculating exchange cost](#calculating-exchange-cost)
- [User data](#user-data)
  - [Creating account](#creating-account)
  - [Account types](#account-types)
  - [Authorization types](#authorization-types)
  - [JWT authorization](#jwt-authorization)
  - [Authorization by request signature](#authorization-by-request-signature)
  - [Getting balance](#getting-balance)
  - [Retrieving account settings](#retrieving-account-settings)
- [Orders](#orders)
  - [Getting order details](#getting-order-details)
  - [Getting order history](#getting-order-history)
  - [Callback notification](#callback-notification)
  - [Deposit order](#deposit-order)
    - [Operation scheme](#deposit-order-operation-scheme)
    - [Getting settings](#getting-deposit-order-settings)
    - [Getting replenishment address](#getting-replenishment-address)
    - [Specific parameters](#deposit-order-specific-parameters)
    - [Order details for callback notification](#order-details-for-callback-notification)
    - [Getting deposit order details](#getting-deposit-order-details)
  - [Withdrawal order](#withdrawal-order)
    - [Operation scheme](#withdrawal-order-operation-scheme)
    - [Getting settings](#getting-withdrawal-order-settings)
    - [Creating order](#creating-withdrawal-order)
    - [Cancelling order](#cancelling-withdrawal-order)
    - [Repeating order](#repeating-withdrawal-order)
    - [Specific parameters](#withdrawal-order-specific-parameters)
    - [Getting details](#getting-withdrawal-order-details)
    - [Order details for callback notification](#withdrawal-order-details-for-callback-notification) 
  - [Internal movement order](#internal-movement-order)
    - [Operation scheme](#internal-movement-order-operation-scheme)  
    - [Getting settings](#getting-internal-movement-order-settings)
    - [Creating order](#creating-internal-movement-order)
    - [Repeating order](#repeating-internal-movement-order)
    - [Getting order details](#getting-internal-movement-order-details)
    - [Order details for callback notification](#internal-movement-order-details-for-callback-notification)
  - [Invoice order(invoice)](#invoice-order) 
    - [Operation scheme](#invoice-order-operation-scheme)
    - [Getting settings](#getting-invoice-order-settings)
    - [Creating invoice](#creating-invoice)
    - [Getting details](#getting-invoice-details)
    - [Order details for callback notification](#order-for-invoice-order-details-for-callback-notification)
  - [Exchange order](#exchange-order)
    - [Exchange types](#exchange-types)
    - [Getting settings](#getting-exchange-order-settings)
    - [Creating exchange](#creating-exchange)
    - [Cancelling limit exchange](#cancelling-limit-exchange)
    - [Repeating limit exchange](#repeating-limit-exchange)
    - [Getting details](#getting-exchange-order-details)
    - [Order details for callback notification](#exchange-order-details-for-callback-notification)

## Request Processing
* Base api url - https://coinpay.org.ua/
* Content-Type – json

Each response contains a structure in which there is a ‘status’ field indicating if such response was processed successfully. A successfully processed response has 2XX status code, and an unsuccessfully processed one - either 4XXX or 5XX.

###### Example of a successfully processed request:

```javascript
 {"status": "success"}
```

###### Example of an unsuccessfully processed request:

```javascript
 {"status": "error", "error": <имя ошибки>}
```

There are no error codes; there is only a verbally described reason.
By default, all errors are returned in English.
It is possible to localize errors in Russian by adding the following title to the request itself – ‘Accept-Language’ with the value ‘ru’.

# Public Data
## Getting active currencies list

```javascript
 GET "/api/v1/currency"
```

###### Sample response:

```javascript
{
  "status": "success",
  "currencies": ["BTC","EUR",...]
}

```

The system returns a list of currencies active at the moment as well as the name of a currency itself, as identified in the system.

## Getting active pairs list
`` `javascript
GET "/ api / v1 / pair"
```

###### Sample response:
```javascript
{
  "status": "success",
  "pairs": [
    [
      {
        "currency_to_spend": "UAH",
        "currency_to_get": "BTC",
        "name": "BTC_UAH",
        "base_currency": "BTC"
      }
    ],
```

API endpoint returns a list of active pairs by which exchanges can be made.

** Explanation of a pair name: **

Usually two currencies are involved in an exchange; one of them is what someone spends, another one – what they receive.
In the current example, ‘BTC_UAH’ pair stands for buying BTC for UAH.
In order to purchase UAH for BTC, one needs to use another pair – ‘UAH_BTC’. In fact, we only have one pair; this is the direction of exchange.

## Getting prices

```javascript
 GET "api/v1/exchange_rate"
```

###### Sample response:

```javascript
{  
  "rates": [  
    {  
      "pair": "UAH_EUR",  
      "base_currency_price": 0.03800836,  
      "price": 26.31  
    }  
  ]  
}                              
																						
```

For the current API endpoint, you can get all the prices for all pairs for an exchange (ticker).


## Calculating exchange cost

```javascript
 GET "api/v1/exchange/calculate"
```

###### Parameters:

```javascript
currency_to_get - required. The currency a user receives
currency_to_spend - required. The currency a user spends
currency_to_get_amount, currency_to_spend_amount - In order to calculate, you need to specify one of the parameters, either ‘the amount of the currency one wants to receive’ or ‘the amount of the currency that a user wants to spend’
exchange_price is an optional parameter. You can specify an exchange price; if the price is not specified, market price is used.
```

###### Sample response:

```javascript
{
  "currency_to_get_amount": 1,
  "currency_to_get": "BTC",
  "currency_to_spend_amount": 202170.7475,
  "currency_to_spend": "UAH",
  "status": "success",
  "exchange_price": 202170.7475
}
```

At each calculation, a response always returns the calculated exchange (exchange price, how much will be spent, the currency, and how much will be received).

# User data
## Creating account

```javascript
  POST "api/v1/user/create"
```

###### Parameters:

```javascript
{
    "email": "string",
    "username": "string",
    "password": "string",
}                                                  

```

All the above fields are required. After API request is successfully completed, one needs to confirm their email (a mail will be sent to their email address); on can go through authorization only after confirming their email.
In the body of the mail, there is a link one needs to click to confirm their email. Link lifetime is 1 day.

## Account types
After a user has confirmed their email, their account becomes ‘NOT_VERIFIED’.
After verification, the account changes to ‘VERIFIED’.
The third account type is designated for business users (merchants) – ‘BUSINESS’.

## Authorization types
Currently, there are two types of API authorization - with JWT token and with request signature through api_key and api_secret.
Users choose the preferable authorization type themselves. We recommend using the request signature.

### JWT authorization


```javascript
POST "api/v1/user/obtain_token"
```
###### Parameters:

```javascript
{
  "email": "string",
  "password": "string"
}                                              

```

###### Example of a successful response (token received):
```javascript
{
  "username": "<username>",
  "token": "<token>"
}
```

After a token has been received, one can perform requests that require authorization.
The token must be specified in the header --header 'Authorization: Bearer <token> header. Token lifetime is 15 minutes.
The token for ‘Swagger’ (through Authorize button) is specified in the same format so that it reboots and shows all available methods allowed to a user.

After the token expires, it should be ‘reobtained’ through ‘obtain token’ method.

### Authorization by request signature
After passing the verification, api_key and api_secret will be provided.
In order to use authorization by request signature, one needs to provide two headers:
```javascript
--header 'Sign: <generated_signature>' --header 'Key: <api_key>'                                          
```

###### Signature creation algorithm:
** Step 1: ** You need to sort the request parameters in alphabetical order and transform nesting in parameters, if there is any.
** Example without nesting: **
```javascript
{
  "currency": "string",
  "amount": "string"
}							
```

In this case, you should get a dictionary, but with sorted keys:
```javascript
{
  "amount": "string",
  "currency": "string"
}							
```

** Example if there is nesting: **
```javascript
{
  "currency": "string",
  "amount": "string",
  "additional_info": {"client_ip": "8.8.8.8"}
}
```

** Transforms into the following dictionary: **
{
  "currency": "string",
  "amount": "string",
  "additional_info_client_ip": "8.8.8.8"
}

The keys of the received dictionary should be sorted in alphabetical order.

** Step 2: ** A line for signature is created in this format:
```javascript
{request.method} - {request.path} - {sorted_params}
```

** Step 3: ** The api_secret string is signed using the algorithm hmac_sha_256.
Similar manipulations are to be carried out with each request.

## Getting balance

```javascript
  GET "api/v1/user/balance"
```

###### Sample response:
```javascript
{
"wallets": {
    "USDT": {
      "address": "0x85E12eA5270C8848D9258FB76de475144dBC1E3e",
      "qr_file_data": img in base64
    },
    ...
  },
  "balance": {
    "EUR": {
      "currency": "EUR",
      "EUR": {
        "reserved": 0,
        "total": 0
      },
      "ETH": {
        "reserved": 0,
        "total": 0
      },
      "UAH": {
        "reserved": 0,
        "total": 0
      },
      "USDT": {
        "reserved": 0,
        "total": 0
      },
      "USD": {
        "reserved": 0,
        "total": 0
      },
      "RUB": {
        "reserved": 0,
        "total": 0
      },
      "BTC": {
        "reserved": 0,
        "total": 0
      }
    },
    ...
  }
}                                                  

```

The balance contains crypto addresses that can be replenished; the addresses are assigned to the users. The balance itself also comes in two fields, ‘total’ and ‘reserved’. ‘Total’ stands for the total amount of funds, ‘Reserved’ – for the funds already taken and located in the orders. Using the balance, you can find out how much money is currently available in a particular currency; moreover, you can get a conversion of funds from one currency to another.

**For example:**

1) The amount of ‘EUR’ in your account - balance [‘EUR’] [‘EUR’]

2) The amount of ‘EUR’ in ‘ETH’ - balance [‘EUR’] [‘ETH’]

## Getting account settings
```javascript
  GET "api/v1/user/account_info"
```

```javascript
  {
  "name": "Свят",
  "email": "svyatoslavkravchenko@gmail.com",
  "account_type": "VERIFIED",
  "is_email_verified": true,
  "referral_id": "89f67887-cae6-4a6f-a393-a89079878a57",
  "created": "2019-02-05T08:13:48.952334",
  "is_2fa_enabled": false,
  //limits, fee, processing_rules
  }
```

A response includes a username, email address, type of account, information on whether such mail has been verified, whether two-factor authorization is enabled, as well as limits, personal funds, etc. for each order type. Each order will be considered separately

# Orders
## Getting order details
When creating an order through API, the system gives the order_id of such order, thus giving the opportunity to get the details.
```javascript
GET "api/v1/orders/detail"
```
###### Параметры:

```javascript
{
  "order_id": $order_id
}						
```

The response provides order details such as whether the order has been created by a specific account or the account has participated in this order. There are orders in which several accounts participate.
The chapter provides an example of what exactly each order returns, for each order type.
## Getting order history
```javascript
GET "api/v1/orders/history"
```
###### Sample response:

```javascript
{
  "orders": [<orders_details_list>]
  "status": "success",
  "pages_count": 78
}
```

Order history has paging with 10 orders displayed on a page.

###### Filters:
  - order_type - the type of an order. Possible values are ‘DEPOSIT’, ‘WITHDRAWAL’, ‘EXCHANGE’, ‘INTERNAL_MOVEMENT’, ‘INVOICE’.

- order_status - the status of an order. Possible values are ‘NEW’, ‘ERROR’, ‘CLOSED’, ‘EXPIRED’, ‘CANCELLED’, ‘CANCELLING’, ‘WAITING_FOR_OPERATOR_CONFIRMATION’, ‘CONFIRMED_BY_OPERATOR’, ‘CANCELLED_BY_OPERATOR’, ‘WAITING_FOR_CONFIRMATION’, ‘WAITING_FOR_PRICE’, ‘PAYMENT_IN_PROGRESS’.
- order_sub_type - the subtype of an order. Some orders have their own subtypes that can be used to filter them.
- page - the page for withdrawal.
- from_timestamp, till_timestamp - filtering by time of order creation.
- currency - filtering by currency. It displays all orders for a particular currency.
- address finds all orders with an address. They can be either replenishment or withdrawal orders.

### Callback notification
When creating an order, it is possible to specify the url for notification when the status of such order changes.
With callback notification, a POST request is sent to the specified address. The 200th response will be a successful request processing, and if an error occurs, the system will try to send a callback 3 times with an interval of 5 minutes.

## Deposit order
### Deposit order operation scheme

In order to replenish an account, one first needs to get an address for replenishment, and then transfer money to this account; it can be either a crypto-currency address or a link to pay by card. A deposit order will be created after a successful replenishment 

#### Example of status transitions by cryptocurrencies (UAH):
Successful replenishment (enrolment) - "NEW-> WAITING_FOR_CONFIRMATION-> CLOSED"

As soon as there is a transaction in the network, the order gets the status ‘NEW’ followed by the status ‘WAITING_FOR_CONFIRMATION’, and will remain in this status until a certain number of confirmations for crediting money to the balance is reached. The order may become ‘EXPIRED’ if the required number of transaction confirmations is not reached within 24 hours.

#### Example of status transitions by fiat currencies (UAH):
Successful replenishment (enrolment) - "NEW-> CLOSED"

Replenishment failed - "NEW-> ERROR"
Link has expired "NEW> EXPIRED (CANCELLED)"

### Getting deposit order settings
```git
GET "api / v1 / user / account_info"
```
There are three types of settings – ‘fees’, ‘limits’, ‘processing_rules’.
#### Limits
```javascript
"deposit_order_limits": {
    "UAH": {
      "GATEWAY": {
        "P2P": {
          "max_amount": 14000,
          "min_amount": 10
        }
      }
    }
  }
 ``` 

As of now, only fiat currencies have limits for replenishment with an indicated limit for the operation. In this example, the limit for replenishment is shown as ‘UAH’, with ‘P2P’ type of replenishment.


#### Fee

```javascript
"deposit_order_fees": {
	"BTC": {
          "GATEWAY": {
            "P2P": {
             "payment_provider_static_fee": null,
             "static_fee": 0,
             "percent_fee": null
        }
      }
    },
}													
```
Shows fees available for a particular replenishment currency.
```javascript
static_fee shows the static value that is taken within the deposit.
percent_fee shows the value that is taken within the deposit in percentage terms.
payment_provider_static_fee means the cost of the provider’s replenishment work; it is usually indicated with some value (static_fee, or percent_fee).
```
    
#### Processing rules

```javascript
"deposit_order_processing_rules": {
	"BTC": {
          "GATEWAY": {
            "P2P": {
              "is_deposit_enabled": true,
              "confirmations_count": 2
        }
      }
    },
}													
```

Indicates whether the deposit is enabled; ‘confirmations_count’ is irrelevant for fiat currencies while for cryptocurrency it shows the number of confirmations for which the deposit is being credited.

### Getting replenishment address

```javascript
POST "api/v1/deposit/address"
```

###### Parameters:

```javascript
{
  "amount_to_spend": "string",
  "callback_url": "string",
  "payment_type": "string",
  "currency": "string",
  "amount_to_receive": "string",
  "additional_info": {},
  "comment": "string"
}
```

** Parameters description: **

1. `currency - currency, must be indicated (mandatory)`

2. `callback_url - url for notifications when creating a deposit order (optional)`

3. `amount_to_spend, amount_to_receive - you need to specify one of these parameters. In the first case, the system will create a link for replenishment taking into account the amount a client can spend, and in the second case, the balance will be replenished by this amount (amount_to_receive). These fields are mandatory only for fiat currencies replenishment`

4. `comment - a comment that will be indicated when creating an order at a specific address and received in callback - (optional parameter)`

5. `payment_type - replenishment method; if it is not specified, P2P will be taken by default`

6. `additional_info - a dictionary with additional parameters. customer_id is an optional parameter, client_ip is a required parameter, deposit_email or deposit_phone must be specified if one`s account status is ‘BUSINESS’

###### Example of obtaining a deposit address for cryptocurrency:

```javascript
{
  "currency": "BTC",
  "callback_url": "http://",
  "additional_info": {"client_ip": "1.1.1.1",
	              "customer_id": "<some_id>"}
}
```

###### Example of obtaining a deposit address for UAH:

```javascript
{
  "amount_to_spend": "10", 
  "callback_url": "http://",
  "currency": "UAH",
  "additional_info": {
	"client_ip": "62.80.170.214",
	"deposit_email": "<some_email>"
    }
}
```

###### Response analysis:

```javascript
{
   "qr": "base64 img",
   "addr": "адрес для пополнения или ссылка",
   "status": "success"
}
```

‘qr’ will be filled only for cryptocurrency

### Deposit order specific parameters

  1. `address - replenishment address`
  2. `status - order status`
  3. `order_id - order_id in the system using which you can get the details of an order`
  4. `customer_id - customer`s ID`
  5. `amount – replenishment amount`
  6. `received_amount - the amount credited to the account`
 
### Order details for callback notifications

```javascript
{"status": "CLOSED", 
 "customer_id": "37750", 
 "received_amount": 0.02206137, 
 "comment": null, 
 "address": "32ssDLdkZngu4gJ1gdmSFWp16AGmfaCnML", 
 "order_id": "0a6282ec-f63d-4a15-a16c-a128b46eb7eb", 
 "currency": "BTC", 
 "tr_hash": "ffb7c8fb3acc13b1cdd924c8ae1bc953c0c81c4eedc643ad104ecfd2f0e45481", 
 "amount": 0.02206137, 
 "confirmations_count": 1}
``` 

### Getting deposit order details

###### Example

```javascript
{
  "external_id": "92ba1fb2-b0af-4281-89e9-21f47ae16874",
  "order_type": "DEPOSIT",
  "status": "CLOSED",
  "fee": 5.15,
  "details": {
		"fee": 5.15,
		"address": "https://mapi.xpay.com.ua/ru/frame/widget/e89ce23c-4d5b-4124-897a-84e75ea972c9",
		"tr_id": "e89ce23c-4d5b-4124-897a-84e75ea972c9",
		"comment": null
	     },
  "currency": "UAH",
  "confirmations_count": null,
  "order_sub_type": "GATEWAY",
  "amount": 10,
  "dt": "2020-02-04 20:11:13.860751",
  "comment": null,
  "tr_hash": "e89ce23c-4d5b-4124-897a-84e75ea972c9"
}
```

## Withdrawal order
### Withdrawal order operation scheme

Upon successful creation of a withdrawal order, such order automatically switches to the status ‘NEW’.

Upon successful withdrawal, the order`s status changes to ‘CLOSED’. This is the final status, which means success.

In case of withdrawal error, the order`s status changes to ‘ERROR’.

It is possible to cancel the order if withdrawal transaction/transactions have not been completed in full.

Upon successful acceptance of a cancellation request, the status of the order changes to ‘CANCELLING’ and ‘CANCELLED’ when finally cancelled.

When withdrawing fiat, the withdrawal amount can be divided into several transactions, and there may be situations when some of the transactions are successful and some are not. In this case, two orders are created, one with ‘CLOSED’ status showing the amount of funds that was successfully withdrawn, and the second order with ‘ERROR’ status showing the amount of funds that has not been withdrawn.

### Getting withdrawal order settings

```git
 GET "api/v1/user/account_info"
```

###### Allows you to get to know the values of limits, fees, and whether withdrawal is enabled:

```javascript
withdrawal_order_processing_rules": {
	"USDT": {
		  "GATEWAY": {
		    "is_cancel_enabled": true,
		    "is_repeat_enabled": true,
		    "is_withdrawal_enabled": true
		   }
	},
	...
}
```

Indicates whether withdrawal is enabled, whether withdrawal cancellation is enabled, whether withdrawal repeat is enabled

```javascript
"withdrawal_order_fees": {
	"UAH": {
		"GATEWAY": {
		   "parent_currency_static_fee": null,
		   "static_fee": null,
		   "percent_fee": 0.5
		},
	},
	...
}
```

Indicates the set fees.

The following options are possible:

```javascript
"withdrawal_order_limits": {
	"USDT": {
		"GATEWAY": {
		    "min_amount": 10,
		    "max_amount": 10000
		}
	}, 
	...
}
```

In this case, the limits for the operation are shown.


### Creating withdrawal order

```git
 POST "api/v1/withdrawal"
```

###### Parameters:
```javascript
{
	"currency": "string",
	"withdrawal_type": "string",
	"comment": "string",
	"mode": "string",
	"callback_url": "string",
	"additional_info": {},
	"amount": 0,
	"wallet_to": "string"
}
```

** Parameter analysis: **

 - `currency - currency, required parameter`
 
 - `withdrawal_type - withdrawal type, required parameter. ‘GATEWAY’ should be indicated when withdrawing to cards or cryptocurrency addresses`
 
 - `comment - a comment to withdrawal, optional parameter`
 
 - `callback_url - url for notifications, optional parameter. If it is indicated, notifications with status updates on a specific order will be received`
 
 - `amount – withdrawal amount, required parameter`
 
 - `wallet_to - wallet address, required parameter`
 
 - `additional_info - dictionary with additional parameters. client_ip - optional, withdrawal_email must be specified if one`s account status is ‘BUSINESS’


** Examples: **

###### Creating withdrawal order (UAH):

```javascript
{
   "currency": "UAH",
   "withdrawal_type": "GATEWAY",
   "comment": "comment",
   "callback_url": "http://",
   "additional_info": {"withdrawal_email": $customer_email},
   "amount": 100,
   "wallet_to": $card_number
}
```

###### Creating withdrawal order (cryptocurrency):

```javascript
{
   "currency": "BTC",
   "withdrawal_type": "GATEWAY",
   "comment": "comment",
   "callback_url": "http://",
   "amount": 100,
   "wallet_to": $btc_addr
}
```

In response to creating an order, API returns order_id, ID of the order.

### Cancelling withdrawal order

A withdrawal order can be canceled if cancellation is allowed and the relevant order is being processed.
If withdrawal has already been completed (transaction has already been sent), such transaction cannot be canceled. You will have time to cancel before the actual withdrawal.

###### URI

```javascript
 POST "api/v1/withdrawal/cancel"
```

###### Parameters:

```javascript
{
  "order_id": "string"
}
```

If withdrawal cancellation has been successful, the system will return the 200th response, otherwise there will be an error.

### Repeating withdrawal order

It is possible to repeat a withdrawal order if repetition is enabled and such order has been processed.

```javascript
 POST " /api/v1/withdrawal/repeat            "
```

###### Parameters:

```javascript
{
  "amount": 0,
  "order_id": "string"
}
```

It is required to specify order_id. You can specify the amount; it will change the amount for withdrawal as part of a new order.

### Withdrawal order specific parameters

1. `amount - the amount of funds that has been withdrawn from the user's balance`
2. `amount_to_withdrawal - the actual withdrawal amount`
3. `withdrawal_transactions - list of successful transactions`
4. `address - withdrawal address. With fiat currency, the last 4 digits are indicated`
5. `dt - closing date for an order`

### Getting withdrawal order details
```javascript
{
      "details": {
        "comment": "",
        "address": "0997",
        "fee": 2.9
      },
      "amount": 582.9,
      "order_sub_type": "GATEWAY",
      "dt": "2020-04-09 13:43:47.531414",
      "comment": "",
      "status": "CLOSED",
      "external_id": "0bf77841-fe66-4a50-aa6a-129cde64c9aa",
      "amount_to_withdrawal": 580,
      "order_type": "WITHDRAWAL",
      "withdrawal_transactions": [
        580
      ],
      "currency": "UAH"
    },
```

### Withdrawal order details for callback notification

```javascript
{"withdrawal_transactions": [0.03562889], 
"comment": "some_comment", 
"order_type": "WITHDRAWAL", 
"currency": "BTC", 
"dt": "2020-04-09 16:14:12.076628", 
"status": "CLOSED", 
"order_sub_type": "GATEWAY", 
"amount_to_withdrawal": 0.03562889, 
"details": {"tr_id": "2a834817d524f8dde54fb9c4e1d5d3ba4839bc81495ed5d1e2bebb1ebeffc23d", 
            "comment": "some_comment", 
	    "tr_hash": "https://www.blockchain.com/btc/tx/2a834817d524f8dde54fb9c4e1d5d3ba4839bc81495ed5d1e2bebb1ebeffc23d",             "fee": 0.0002, 
	    "address": "14ckdLFzj3ba4FSQiW1fSGKURycqc5gNQE"
	    }, 
"amount": 0.03582889, 
"external_id": "02241379-109d-4c49-965d-6eea8de164c7"}
```

## Internal movement order
### Internal movement order operation scheme

This type of order involves the transfer of funds between users on the platform. For example, user A has 1000 USD; thus, knowing the recipient's email, you can instantly transfer 1000 USD.

Such orders may have the following statuses:

‘NEW’ – the order is being processed

‘CLOSED’ - successfully closed

‘ERROR’ – resulted in an error

### Getting internal movement order settings

```javascript
 GET "api/v1/user/account_info"
```

##### Allows you to get to know the values of limits, fees, and whether the movement of funds is enabled:

```javascript
"internal_movement_processing_rules": {
    "UAH": {
      "is_repeat_enabled": true,
      "is_enabled": true
    },
	...
}
```

Indicates whether the movement of funds within the platform is enabled, whether the repetition of the operation is enabled

```javascript
"internal_movement_fees": {
   "UAH": {
      "static_fee": 0,
      "percent_fee": 0
    },
	...
}
```

Shows available fees.
Fees are paid by the remitter.

The following options are possible:
- `static_fee - a static value is paid when transferring funds`  
- `percent_fee – percentage-based amount is paid`

```javascript
"internal_movement_limits": {
  "UAH": {
      "max_amount": 1000000,
      "min_amount": 0
    },
	...
}
```

In this case, the limits for the operation are shown. The amount of the transfer should not go beyond the specified numbers.


### Creating internal movement order

```javascript
 POST "api/v1/internal_movement"
```

###### Parameters:

```javascript
{
  "comment": "string",
  "amount": "string",
  "callback_url": "string",
  "currency": "string",
  "destination_account_email": "string"
}
```

** Parameter analysis: **
- `currency - currency, required parameter`
- `comment – a comment to withdrawal, optional parameter`
- `callback_url - url for notifications, optional parameter. If it is indicated, notifications with status updates on a specific order will be received `
- `amount – withdrawal amount, required parameter`
- `destination_account_email - email of the recipient of funds on the platform`

If successful, the response will contain the order_id of the created order.
 
  ### Repeating internal movement order

It is possible to repeat an internal movement order if repetition is enabled and such order has been processed.

`` `javascript
  POST "/ api / v1 / internal_movement / repeat"
`` ``

###### Parameters:

```javascript
{
  "amount": 0,
  "order_id": "string"
}
```
It is required to specify order_id. You can specify the amount; it will change the amount for transfer as part of a new order.

### Getting internal movement order details

```javascript
      "details": {
        "comment": "",
        "destination_account": "destination_account",
        "source_account": "source_account"
      },
      "amount": 266,
      "fee": 0,
      "dt": "2020-03-20 22:23:52.977529",
      "status": "CLOSED",
      "external_id": "021b33c6-060a-4018-be71-57088ce81d2e",
      "order_type": "INTERNAL_MOVEMENT",
      "currency": "USDT"
    },
```

### Internal movement order details for callback notification

```javascript
 {"details": 
    {"destination_account": "2@gmail.com", 
     "comment": "string", 
     "source_account": "1@gmail.com"
     }, 
  "fee": 0.01, 
  "dt": "2020-04-08 22:22:40.733016", 
  "currency": "UAH", 
  "order_type": "INTERNAL_MOVEMENT", 
  "external_id": "aaf4a18f-9148-4b09-a069-2f543a1f0c70", 
  "status": "CLOSED", 
  "amount": 1.01, 
  "order_sub_type": null
  }
 ```
 
## Invoice order
### Invoice order operation scheme
The platform allows you to create invoices for users to pay for the services.
For example, a merchant may issue an invoice for payment of goods or services and submit it to the user.

#### Invoice work stages:

1) an invoice is created (a link for payment is generated)
2) the user follows the link (logs in to coinpay.com.ua), choose in which way payment should be done and pay it


#### Possible statuses:

1) ‘NEW’ - invoice is created
2) ‘PAYMENT_IN_PROGRESS’ - payment is being processed. Awaiting the final status
3) ‘CLOSED’ - invoice successfully paid
4) ‘ERROR’ – invoice payment failed
5) ‘EXPIRED’ - invoice has ceased to exist. An invoice ceases to be active in 900 seconds
6) ‘WAITING_FOR_CONFIRMATION’ - waiting for payment on deposit address to pay invoice via  FIAT GATEWAY or CRYPTO GATEWAY

### Getting invoice order settings

```javascript
GET api/v1/user/account_info
```

#### Getting settings on whether invoice creation is active.

  ```javascript
 "invoice_order_processing_rules": {
    "UAH": {
      "is_enabled": true
           },
      }
  ```

Indicates whether invoice creation for a particular currency is enabled.
  
#### Getting fee settings

```javascript
"invoice_order_fees": {
    "UAH": {
      "percent_fee": 0
    },
  ```

Indicates the amount of fees and how such fees will be calculated for each invoice.
 
#### Getting operation limits settings

```javascript
"invoice_order_limits": {
    "UAH": {
      "max_amount": 148500,
      "min_amount": 81
    },
  ```

Indicates the amount and limits for invoice issuance.

### Creating invoice
 ```javascript
 POST api/v1/invoice
 ```
 
 #### Parameters
 ```javascript
 {
  "comment": "string" – optional parameter
  "amount": "string" – invoice amount (required parameter)
  "currency": "string" – required parameter
  "pay_account_email":  - required parameter
  "payment_option": restrict in which way user should pay invoice. Choices - ALL, COINPAY, FIAT, CRYPTO. Default value - ALL
  "additional_info": optional parameter(dict). Can include "callback_url", "success_url", "fail_url". 
  If "fail url" sets - customer will be redirected to client page with info about unsuccess payment.
  If "success_url" sets customer will be redirected to client page with info about success payment
}
```

 #### Payment option choices
 
 ALL - User can pay in all avialiable choices, which are present on COINPAY
 COINPAY - User can pay only if will be logged to coinpay
 FIAT - User can pay only from COINPAY or via FIAT GATEWAY(login to COINPAY not required)
 CRYPTO - User can pay only from COINPAY or via CRYPTO GATEWAY(login to COINPAY not required)

#### Sample response for a successfully created invoice
```javascript
 {
  "url": "https://coinpay.com.ua/invoice/cf700d3b-20b4-4b33-b195-38709b7e51fa",
  "order_id": "cf700d3b-20b4-4b33-b195-38709b7e51fa",
  "status": "success"
}
```

In order to pay, the user needs to follow the specified link, log in to coinpay.com.ua, choose the currency and go through the payment process.


### Getting invoice details
Invoice details can be seen by the account that created such invoice, and by the account that paid it.

```javascript
{
      "internal_id": 164554,
      "details": {
        "comment": null,
        "destination_account": "Svyat_merchant",
        "source_account": null
      },
      "url": "https://coinpay.com.ua/invoice/cf700d3b-20b4-4b33-b195-38709b7e51fa",
      "order_sub_type": null,
      "dt": "None",
      "amount": 100,
      "status": "NEW",
      "external_id": "cf700d3b-20b4-4b33-b195-38709b7e51fa",
      "order_type": "INVOICE",
      "fee": 0,
      "currency": "UAH"
    }
```

### Invoice order details for callback notification

```javascript
{"order_type": "INVOICE", 
 "dt": "2020-04-07 21:54:57.185155", 
 "url": "https://coinpay.com.ua/invoice/3542e3c2-4bab-48b8-bfbb-2b95b88c8a7a", 
 "internal_id": 163787, 
 "order_sub_type": null, 
 "amount": 100.0, 
 "status": "EXPIRED", 
 "fee": 0.0, 
 "currency": "UAH", 
 "details": {
       "destination_account": "Merchant_test", 
       "comment": "Bot_operation_100_UAH", 
       "source_account": <some_account>}, 
 "external_id": "3542e3c2-4bab-48b8-bfbb-2b95b88c8a7a"}
```

## Exchange order
### Exchange types
The platform allows you to make two different types of exchange:

1) Market exchange (order subtype – ‘MARKET_EXCHANGE’)
2) Exchange upon reaching the indicated price - (order subtype – ‘LIMIT_EXCHANGE’)

Market exchange has the following statuses:
1) ‘NEW’ – order is created
2) ‘CLOSED’ - exchange is successful
3) ‘ERROR’ – exchange resulted in an error

Limit exchange has the following statuses:
1) ‘NEW’, ‘WAITING_FOR_PRICE’ - waiting for the price on the platform
2) ‘CLOSED’ - exchange successfully completed
3) ‘ERROR’ – exchange resulted in an error
4) ‘CANCELLING’ – exchange is being canceled
5) ‘CANCELLED’ - exchange is canceled


### Getting exchange order settings

```javascript
GET api/v1/user/account_info
```

#### Getting settings for exchange transactions availability 

```javascript
"exchange_order_processing_rules": {
    "EUR_USD": {
      "is_market_exchange_enabled": true,
      "is_cancel_enabled_for_limit_exchange": false,
      "is_repeat_enabled_for_limit_exchange": false,
      "is_limit_exchange_enabled": true
    },
```

‘is_market_exchange_enabled’ - indicates whether market exchange is enabled

‘is_limit_exchange_enabled’ - indicates whether limit exchange is enabled

‘is_cancel_enabled_for_limit_exchange’ – indicates whether cancellation is enabled for limit exchange

‘is_repeat_enabled_for_limit_exchange’ – indicates whether repetition is enabled for limit exchange

#### Getting settings for exchange order limits

```javascript
"exchange_order_limits": {
    "EUR_USD": {
      "max_amount": 10000,
      "currency_to_spend": "USD",
      "min_amount": 5
    },
```

The limits for each pair are formed in the way that allows checking amounts of particular currency spent during exchange.
In the current example, you can spend 5 to 10.000 USD when buying euros.


### Creating exchange

```javascript
POST /api/v1/exchange
```

#### Parameters
```javascript
{
  "currency_to_spend": "string",
  "currency_to_get": "string",
  "currency_to_spend_amount": "string",
  "callback_url": "string",
  "currency_to_get_amount": "string",
  "exchange_price": "string"
}
```

#### Exchange creation rules
1) It is necessary to indicate the currency that is spent and the currency that is bought
2) You need to specify one of the amounts or make an exchange spending X currency, or make a purchase to get Y currency
3) If you want to purchase at the ordered price, you need to specify the following parameter value – ‘exchange_price’
4) You can specify callback_url to subscribe to notifications on changes in exchange status

#### Examples

1) Purchasing 1 BTC for UAH at the market price
```javascript
{
  "currency_to_spend": "UAH",
  "currency_to_get": "BTC",
  "currency_to_get_amount": "1"
}
```

2) Purchasing 1 BTC for UAH at the market price spending 1000 UAH on exchange
```javascript
{
  "currency_to_spend": "UAH",
  "currency_to_get": "BTC",
  "currency_to_spend_amount": "10000"
}
```

3) Purchasing 1 BTC for UAH at the price of 200.000
```javascript
{
  "currency_to_spend": "UAH",
  "currency_to_get": "BTC",
  "currency_to_get_amount": "1",
  "exchange_price": "200000"
}
```

### Cancelling limit exchange
```javascript
POST api/v1/exchange/cancel
```

#### Parameters
```javascript
{
  "order_id": "string"
}
```

Only a limit exchange order can be canceled if cancellation is enabled and the order is being processed.

### Repeating limit exchange

```javascript
POST /api/v1/exchange/repeat
```
#### Parameters
```javascript
{
  "order_id": "string"
}
```

Repeating the order is possible if repetition is enabled and the order is completed.


### Getting exchange order details

```javascript
{
      "details": {
        "currency_to_spend_amount": 92.42144177,
        "currency_to_get_amount": 2499.99999988,
        "price": 27.05
      },
      "order_sub_type": "MARKET_EXCHANGE",
      "dt": "2020-04-10 11:19:40.596357",
      "status": "CLOSED",
      "external_id": "d0a9823a-90e4-40af-9739-a380bdc13932",
      "order_type": "EXCHANGE",
      "currency": "UAH_USD"
    },
```

### Exchange order details for callback notification

```javascript
{"external_id": "ddd08d0a-198f-441e-bf45-a61fc889261f", 
"status": "CLOSED", 
"order_type": "EXCHANGE", 
"order_sub_type": "MARKET_EXCHANGE", 
"dt": "2020-04-09 14:05:19.373028", 
"currency": 
"RUB_USDT", 
"details": {"currency_to_get_amount": 40337.397594, 
            "price": 75.28443, 
	    "currency_to_spend_amount": 535.8}
	    }
}
```
