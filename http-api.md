# SWARM POS Integration HTTP API

Swarm provides a simple HTTP/JSON API that allows POS providers & developers to interact with Swarms loyalty engine and supports direct integration with point-of-sale software(POS).

- API access is over HTTPS using root endpoint: https://vsp.swarmapps.com/moonunit
- Data is sent and received using JSON

## Response codes

- Sending invalid JSON will result in a 400 Bad Request response
- Sending the wrong type of JSON values will result in a 400 Bad Request response
- Trying to access the API without authenticating or with an invalid auth token will result in a 401 Unauthorized response
- Trying to access resources which do not exist will result in a 404 Not Found response

## Authentication

In order to interact with Swarm's API you will need an `api-key` to authenticate - this will be provided to you as part of your api starter kit.

All HTTP requests to Swarms API should include your `api-key` in the request headers.

Curl Example of including your `api-key` as a header:
```
curl -H "api-key: <your-api-key>"
```

If you do not include your `api-key` or provide an incorrect `api-key` you will receive the following HTTP response:

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json
```
``` javascript
{"result": "error", "message": "Authentication failed: API Key invalid"}
```

## Your test customers

As part of your api starter kit you will be provided with 3 customer accounts which can be used for development & testing - each of these customers is identified by a unique account number.

To retrieve a list of account numbers for your test customers make an `GET` HTTP call to `/v1/test-customers` providing your `api-key` as in the request header:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/test-customers
```
Response: 

``` javascript
{"accountNumbers":["6L2NMOT","6L2NMOW","6L2NMOZ"]}
```

### Sample loyalty programs/rewards

During development you are provided with two sample loyalty programs(allows earning points towards a reward) namely `wholeEarthCafeCoffee` and `wholeEarthCafeCake`.

For the `wholeEarthCafeCoffee` loyalty program, including product SKUs `COFFCAP` or `COFFAMR` in basket data will earn 1 point each - when 7 points are reached a `wholeEarthCafeCoffee` reward is earned and the point balance reset to 0.

For the `wholeEarthCafeCake` loyalty program, including product SKUs `CHEESECK` or `CARROTCK` in basket data will earn 1 point each - when 7 points are reached a `wholeEarthCafeCake` reward is earned and the point balance reset to 0.

##### Sample rewards:

The `wholeEarthCafeCoffee` reward is setup to discount products `COFFCAP` or `COFFAMR`, up to a maximum value of R 22.00.

The `wholeEarthCafeCake` reward is setup to discount products `CHEESECK` or `CARROTCK`, up to a maximum value of R 32.00.

### Checking loyalty balance of your tests customers

During integration development it is useful to check a given customers loyal balance(current point balance & rewards) to validate if a transaction had the expected result.

For example - if you are testing a transaction that will earn 2 points on a given basket - you can perform the transaction and check the customers loyalty balance to ensure 2 points where added.

The current loyalty balance of a particular customer can be retrieved by making an `GET` HTTP call to `/v1/customers/:account-number/current-loyalty-balance` - `:account-number` should be replaced with the account number of a respective test customer

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/current-loyalty-balance
```

Response: 

``` javascript
{  
  "points":{  
    "wholeEarthCafeCoffee": 3,
    "wholeEarthCafeCake": 0
  },
  "vouchers":[  
    {  
      "voucherType":"/swarm/voucherTypes/wholeEarthCafeCoffee",
      "voucherNumber":"9139972",
      "id":"1a6ac63a9"
    }
  ]
}
```

### Customer codes for development and testing

To support testing & development a customers available codes, which are normally displayed on the customers mobile app, can be retrieved by making a `GET` request to `/v1/customer/<customer-account-number>/available-codes`, substituting `<customer-account-number>` for a one of your test customers accounts.

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/available-codes
```

``` javascript
{
  "brand":"swarm",
  "uid":"6L2NMOT",
  "earnCode":"8083260",
  "redeemCodes":[]
}
```


## Loyalty transactions

A loyalty transaction represents the interaction the consumer has with the cashier at the point-of-sale when the consumer wishes to earn points towards a reward or discount their purchase by redeeming a reward.

## Starting a transaction

A loyalty transaction starts at the "sub-total" phase of the sale - at this point the POS Software can start a transaction on Swarms API.

When a transaction is started `basket` information, the `saleTotal` in cents and any additional `saleInfo` should be provided.

##### UI Guidelines

- At this phase of the sale, when no further changes will be made, a "Swarm" button should be available in the POS software

- Clicking on the "Swarm" button will bring up a dialog window that allows the cashier to scan in(if a hardware barcode scanner is avaible) or manually enter codes to process loyalty.

##### API End-point

`/v1/transaction-started`

##### Required HTTP Headers: 

- "api-key: \<your-api-key\>" 
- "Content-Type: application/json"

##### Request body:

``` javascript
{
 // The basket is an array of sale items, including sku, unitPrice(total cents) and quantity
 "basket" [{"sku" "COFFCAP", "unitPrice" 2200, "quantity" 2}],
 "basketId" "123456" // A id that uniquely identifies the sale, should come from POS software
 "saleTotal" 4400, // Total cents
 "saleInfo" { // Sale info including any additional fields.
   "storeId" "1", // Required: An identified representing the Store the transaction is being started in.
   "posID" "1", // Required: An identifier for the Point-of-sale machine
   "cashierName" "Ruby Farish" // Required: An identifier representing the cashier processing the sale.
  }}
```

##### Curl Example:

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"basket\":[{\"sku\":\"COFFCAP\",\"unitPrice\":2200,\"quantity\":2}],\"basketId\":\"123456\",\"saleTotal\":4400,\"saleInfo\":{\"storeId\":\"1\",\"posId\":\"1\",\"cashierName\":\"Ruby\"}}" https://vsp.swarmapps.com/moonunit/v1/transaction-started
```

##### Response:

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json
Body: {"txId": "1a6ab37b6"}
```

By starting a new transaction you are provided with a transaction-id(`txId`) which will be used as a reference for further API calls relating to this transaction - `basket`, `saleTotal` and `sale-info` are stored and do not need to be included in further requests.

## Scanning in codes at the POS

Swarm's loyalty solution provides a mobile app that allow consumers to present codes at the POS which can either be scanned in(if a hardware barcode scanner is avaible) or manually entered by the cashier to process the consumers loyalty for a given sale.

Customers will generally scan in their codes at the point of sale to either redeem a reward, which discounts the purchase, or to earn points towards a reward on eligible products in their purchase. A customer can scan in one or more codes during a sale which are considered part of a loyalty transaction.

##### UI Guidelines

At this phase the POS UI should allow the cashier to scan in a code if a hardware barcode scanner is avaible or manually enter a the code presented by the consumer using their loyalty mobile app.

The screen should include:

- A single `code` text field, which should be populated by the scanned-in code if a hardware barcode scanner is avaible or allow the cashier to enter a code in this field. 

- A "Process" button, next to the `code` text field, which will action making a `/v1/code-scanned` API call to Swarm's API to process the code and receive results.


##### API End-point

`/v1/code-scanned`

##### Required HTTP Headers: 

- "api-key: \<your-api-key\>" 
- "Content-Type: application/json"

##### Request body:

* `txId`: The transaction id value which was returned when starting the transaction should be included:
* `code`: Either a valid earn code or redeem code. You can retrieve a list of valid codes to scan for a given customer by using the
<a href="https://gist.github.com/yawningman/8dce67d3c7abafd4d65bb29f78159176#customer-codes-for-development-and-testing" target="_blank">customer-codes api end-point</a>.

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
  "code" "1234567"   // The code scanned in or manually entered at the POS.
}
```

##### Curl Example for earning points

The customer will scan in an "earn" code at this point which is displayed on the consumers phone:

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"tx-id\":\"1a6ab37b6\",\"code\": \"1234567\"}" https://vsp.swarmapps.com/moonunit/v1/code-scanned
```

Response when earning points:

** a `messageToCashier` is included in the response which should be dislayed to the cashier which provides feedback on what loyalty was performed.

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json
Body:

{
"result":"success",
"txId":"1a6ac3cc6",
"brand":"swarm",
"uid":"6L2NMOT",
"messageToCashier": "Customer earned 2 X Coffee points"
"provisionalLoyalty":{
  "pointsEarned":true, // Indicates if points where earned or not
  "points":[ // A breakdown of points earned
    {"campaign": "whole-earth-cafe-coffee", "points": 2}
  ],
  "rewardsRedeemed": false, // Indicates if a reward was redeemed(implies a discount)
  "totalDiscountInCents": 0, // The total discount in cents that should be made to the purchase
  "discounts":[] // A breakdown of which products the discount applies to in the basket}
}
```

The response from `/code-scanned` will include a `provisionalLoyalty` field which outlines the expected loyalty result of the sale - no loyalty is committed at this point it is only calculated.

##### Curl Example for redeeming a reward

The customer will scan in an "voucher number" code at this point which is displayed on the consumers phone:

##### Request body:

* Note the `txId` value which was returned when starting the transaction should be included:

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
  "code" "1234567"   // The code scanned in or manually entered at the POS.
}
```

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"tx-id\":\"1a6ac6a78\",\"code\": \"2234567\"}" https://vsp.swarmapps.com/moonunit/v1/code-scanned
```

Response when redeeming a voucher:

** a `messageToCashier` is included in the response which should be dislayed to the cashier which provides feedback on what loyalty was performed.

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json

{  
  "result":"success",
  "txId":"1a6ac7771",
  "brand":"swarm",
  "uid":"6L2NMOT",
  "messageToCashier": "Customer redeemed 1 X Coffee reward"
  "provisionalLoyalty":{  
    "pointsEarned":true,
    "points":[
      {"campaign": "whole-earth-cafe-coffee", "points": 2}
     ],
    "rewardsRedeemed":true, // Indicates that a reward has been redeemed
    "totalDiscountInCents":2200, // THe total discount in sents that should be applied to the purchase
    "discounts":[ 
      {  
        "name":"Whole earth cafe coffee Reward #2234567", // The name of the reward which applied the discount
        "amount":2200, // The amount, in cents, of the discount applied to this product
        "product":[  // A break-down of products the discount applies to
          {  
            "sku":"COFFCAP",
            "quantity": 1,
            "discount": 2200
          }
        ]
      }
    ]
  }
}
```

##### Scanning in the same code twice

If a cashier happens to scan in the same code twice the API will respond as follows:

##### Response body:

``` javascript
{  
  "txId":"1a7099c11",
  "brand":"swarm",
  "uid":"6L2NMOT",
  "result":"code-already-scanned",
  "messageToCashier":"Code 1234567 has already been scanned in for this purchase."
}
```

##### Entering a code that is not recognised

If a cashier enters an arbitray or incorrect code that is not recognised by Swarm the API will respond as follows:

##### Response body:

``` javascript
{  
  "txId":"1a7099c11",
  "messageToCashier":"The code entered is not recognised as a valid code - please re-scan or manually re-enter the code and try again.",
  "result":"unrecognised-code"
}
```

## Voiding scanned in codes

By voiding a code that has been scanned in any loyalty actions associated with the given code will not be performed when the transaction is completed. This is most often used when a customer scans in a reward code where discounts apply and the cashier, for whatever reason, needs to remove this reward/discount from the purchase - by voiding the code that resulted in a discount applying the discount/reward will no longer apply to the sale.

##### API End-point

`/v1/code-voided`

##### Required HTTP Headers: 

- "api-key: \<your-api-key\>" 
- "Content-Type: application/json"

##### Request body:

* `txId`: The transaction id value which was returned when starting the transaction should be included:
* `code`: Either a valid earn code or redeem code.

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
  "code" "1234567"   // The code scanned in or manually entered at the POS.
}
```

##### Curl Example

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"tx-id\":\"1a6ab37b6\",\"code\": \"1234567\"}" https://vsp.swarmapps.com/moonunit/v1/code-voided
```

Response:

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json

Body:
  "result":"success",
  "brand":"swarm",
  "uid":"6L2NMOT",
  "txId":"1a6ab37b6",
  "code":"1234567",
  "messageToCashier":"Code 1234567 successfully voided"
}
```

## Completing a transaction and committing loyalty

At the Sub-total phase, when no further changes to the sale will be made, the transaction can be completed on Swarms API which will commit any relevant loyalty for the transaction.

##### UI Guidelines

A "Finished" button should be provided on the Swarm full screen - once the cashier is finished scanned in codes they can click this button to close the Swarm screen.

Once the customer has settled any outstanding amount and the sale is considered complete the POS should make the `transaction-completed` API call to indicate the sale is complete and that loyalty can be committed.

##### API End-point

`/v1/transaction-completed`

##### Required HTTP Headers: 

- "api-key: \<your-api-key\>" 
- "Content-Type: application/json"

##### Request body:

* Note the `txId` value which was returned when starting the transaction should be included:

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
}
```

In addition to providing only a `txId`, additional `saleInfo` can also be provided when the transaction is completed:

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
  "saleInfo" { // Optional sale info including any additional fields.
    "storeId" "1",
    "posID" "1",
    "cashierName" "Ruby Farish",
    "customerName": "Merv Pepler",
    "customerEmail": "merv@eatstatic.co.uk"
  }
}
```


##### Curl Example:

Completing a transaction:

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"tx-id\":\"1a6ab37b6\"}" https://vsp.swarmapps.com/moonunit/v1/transaction-completed
```

Response when earning points:

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json

Body:
{  
  "brand":"swarm",
  "uid":"6L2NMOT",
  "txId":"1a6ac3cc6",
  "messageToCashier": "Customer earned 2 X Coffee points"
  "earnResults":{  
    "pointsEarned":true,
    "points":
      [{"campaign":"whole-earth-cafe-coffee", "points": 1},
       {"campaign":"whole-earth-cafe-cake", "points": 0}
    ],
    "allBasketItems":
      [{"unitPrice":2200, "quantity":1, "sku":"COFFCAP"}],
    "allocatedBasketItems":[],
    "availableBasketItems":
      [{"unitPrice":2200, "quantity":1, "sku":"COFFCAP"}],
    "unallocatedBasketItems":[],
    "pointEarnBreakdown":[  
        {"campaign":"whole-earth-cafe-coffee", "pointsEarned":1,
         "basketEarnSkus": [{"unitPrice":2200, "quantity":1, "sku":"COFFCAP"}]},
        { "campaign":"whole-earth-cafe-cake", "pointsEarned":0,
          "basketEarnSkus":[]}
       ]
  },
  "redeemResults":{  
    "vouchersRedeemed":false,
    "totalDiscountInCents":0,
    "redeemableVouchers":[],
    "results":[]
  },
  "result":"success"
}
```

Response when redeeming a reward:

** a `messageToCashier` is included in the response which should be dislayed to the cashier which provides feedback on what loyalty was performed.

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json

Body:
{  
  "brand":"swarm",
  "uid":"6L2NMOT",
  "txId":"1a98cc32c",
  "result":"success"
  "messageToCashier": "Customer redeemed 1 X Coffee reward"
  "earnResults":{  
    "pointsEarned":false,
    "points":[  ],
    "allBasketItems":[  
      {  
        "unitPrice":2200,
        "quantity":1,
        "sku":"COFFCAP"
      }
    ],
    "allocatedBasketItems":[  // Allocated by reward redeem
      [  
        {  
          "quantity":1,
          "sku":"COFFCAP"
        }
      ]
    ],
    "availableBasketItems":[],
    "unallocatedBasketItems":[],
    "pointEarnBreakdown":[  
      {  
        "campaign":"whole-earth-cafe-coffee",
        "pointsEarned":0,
        "basketEarnSkus":[ ]
      },
      {  
        "campaign":"whole-earth-cafe-cake",
        "pointsEarned":0,
        "basketEarnSkus":[  

        ]
      }
    ]
  },
  "redeemResults":{  
    "vouchersRedeemed":true,
    "totalDiscountInCents":2200,
    "redeemableVouchers":[  
      {  
        "discountInCents":2200,
        "basketItem":[  
          {  
            "quantity":1,
            "sku":"COFFCAP"
          }
        ],
        "voucher":{  
          "id":"1234",
          "voucherNumber":"1621007",
          "voucherType":"\/swarm\/voucherTypes\/wholeEarthCafeCoffee"
        }
      }
    ]
  }
}
```

## Voiding a transaction

At the Sub-total phase, when no further changes to the sale will be made, the transaction can be voided for various reasons, including non-payment. Voiding the transactions will NOT commit any loyalty.

##### API End-point

`/v1/transaction-voided`

##### Required HTTP Headers: 

- "api-key: \<your-api-key\>" 
- "Content-Type: application/json"

##### Request body:

* Note the `txId` value which was returned when starting the transaction should be included:

``` javascript
{
  "txId" "1a6ab37b6" // The transaction id(txId) provided when you started the transaction.
}
```

##### Curl Example:

``` bash
curl -i -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"tx-id\":\"1a6ab37b6\"}" https://vsp.swarmapps.com/moonunit/v1/transaction-voided
```

Response:

``` javascript
HTTP/1.1 200 OK
Content-Type: application/json

Body:
{"result": "success", "messageToCashier": "The transaction has been voided successfully and no loyalty will be commited."}
```

