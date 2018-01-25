# Customer details: SWARM POS Integration HTTP API

Provides a simple HTTP/JSON API that allows POS providers & developers to query customer details.

- API access is over HTTPS using root endpoint: https://vsp.swarmapps.com/moonunit
- Data is sent & received using JSON

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

### Customer profile information

Customer profile information is provided by the `GET /customer/:id/profile` API end-point. `:id` being the customers account number.

Example:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/profile
```

---

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "profile": {
        "birthday": "1975-02-15",
        "city": "Cape Town",
        "country": "South Africa",
        "email": "eatstatic@planetdog.org",
        "facebookConnected": false,
        "firstName": "Merv",
        "gender": "male",
        "lastName": "Pepler",
        "mobile": "+27739772453",
        "region": "Western Cape"
    }
}
```

If an `:id` is supplied that does not belong to a valid customer the API will respond as follows:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/X123Y/profile
```
----

``` javascript

HTTP/1.1 404 Not Found
Content-Type: application/json

{
    "result": "error",
    "message": "No such customer: swarm X123Y"
}
```

### Customer point balance information

Customer point balance information is provided by the `GET /customer/:id/point-balance` API end-point. `:id` being the customers account number.

A customer may have one or more campaigns which included the following details:
- `campaignId` The internal Swarm identified for the campaign
- `name` The human friendly name for the campaign 
- `total` Represents the total point balance for campaign.

Example:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/point-balance
```

---

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "pointBalance": [
        {
            "campaignId": "whole-earth-cafe-coffee",
            "name": "Whole earth cafe coffee",
            "total": 2
        },
        {
            "campaignId": "whole-earth-cafe-cake",
            "name": "Whole earth cafe cake",
            "total": 0
        }
    ]
}
```

If an `:id` is supplied that does not belong to a valid customer the API will respond as follows:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/X123Y/point-balance
```
----

``` javascript

HTTP/1.1 404 Not Found
Content-Type: application/json

{
    "result": "error",
    "message": "No such customer: swarm X123Y"
}
```

### Customer reward balance information

Customer reward balance information is provided by the `GET /customer/:id/reward-balance` API end-point. `:id` being the customers account number.

A customer may have one or more rewards which include the following fields: 

- `id` The unique internal id
- `name` The human friendly name for the reward campaign
- `qrCode` Represents the value of the QR code displayed in the customer app - scanned in at POS.
- `voucherNumber` The number representing the reward - scanned in at POS.
- `voucherType` The internal Swarm voucher type of this reward.

Example:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/reward-balance
```

---

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "rewardBalance": [
        {
            "id": "1abca4b09",
            "name": "Whole earth cafe coffee",
            "qrCode": "6515249",
            "voucherNumber": "6515249",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCoffee"
        },
        {
            "id": "1abca4b08",
            "name": "Whole earth cafe cake",
            "qrCode": "6515242",
            "voucherNumber": "6515242",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCake"
        }
    ]
}
```

### Customer loyalty balance information

Both customers point and reward balance information is provided by the `GET /customer/:id/loyalty-balance` API end-point. `:id` being the customers account number.

The api response is a combination of the [customers points balance response](https://gist.github.com/yawningman/74a6f2e0809199e64c98afde9640173a#customer-point-balance-information)(`point-balance` field) & [reward balance response](https://gist.github.com/yawningman/74a6f2e0809199e64c98afde9640173a#customer-reward-balance-information)(`reward-balance`).

Example:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/loyalty-balance
```

---

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "pointBalance": [
        {
            "campaignId": "whole-earth-cafe-coffee",
            "name": "Whole earth cafe coffee",
            "total": 3
        },
        {
            "campaignId": "whole-earth-cafe-cake",
            "name": "Whole earth cafe cake",
            "total": 0
        }
    ],
    "rewardBalance": [
        {
            "id": "1abca4b09",
            "name": "Whole earth cafe coffee",
            "qrCode": "6515249",
            "voucherNumber": "6515249",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCoffee"
        },
        {
            "id": "1abca4b08",
            "name": "Whole earth cafe cake",
            "qrCode": "6515242",
            "voucherNumber": "6515242",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCake"
        }
    ]
}
```

### Customer available rewards

The `POST /customer/:id/available-rewards` API end-point(`:id` being the customers account number) provides the ability to determine which of a customers rewards are applicable to a given `basket`.

Basket data is supplied as JSON in the request body and should include a valid `sku`, `unitPrice`(total cents) and `quantity` for each basket item. If the `saleTotal` of a sale is available this should be included when making the API call.

Example JSON request body of a basket containing a single product:

``` javascript
{
 "basket": [{"sku": "COFFCAP", "unitPrice": 2200, "quantity": 1}],
 "saleTotal": 2200 // Total cents
}
```

Example JSON request body of a basket containing a 2 unique products:

``` javascript
{
 "basket": [{"sku": "COFFCAP", "unitPrice": 2200, "quantity": 1},
            {"sku": "CHEESECK", "unitPrice": 3500, "quantity": 1}],
 "saleTotal": 5700 // Total cents
}
```

Example JSON request body of a basket containing a 4 products(2 Coffee + 2 Cake items):

``` javascript
{
 "basket": [{"sku": "COFFCAP", "unitPrice": 2200, "quantity": 2},
            {"sku": "CHEESECK", "unitPrice": 3500, "quantity": 2}],
 "saleTotal": 114000 // Total cents
}
```

For example if a customer has two rewards using the `POST /customer/:id/available-rewards` API end-point with a given `basket` allows you to determine if one or more of the customers rewards are applicable to a basket.

- Example API call with single `COFFCAP` product in the basket with a customer who has a `wholeEarthCafeCoffee` and `wholeEarthCafeCake` reward:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"basket\":[{\"sku\":\"COFFCAP\",\"unitPrice\":2200,\"quantity\":1}],\"saleTotal\":2200}" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/available-rewards
```

---

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "hasApplicableVouchers": true,
    "totalApplicableRewards": 1,
    "totalDiscountInCents": 2200,
    "vouchers": [
        {
            "id": "1aa359b31",
            "name": "Whole earth cafe coffee",
            "qrCode": "7531138",
            "voucherNumber": "7531138",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCoffee"
        }
    ]    
    "discounts": [
        {
            "amount": 2200,
            "name": "Whole earth cafe coffee Reward #7531138",
            "products": [
                {
                    "discount": 2200,
                    "quantity": 1,
                    "sku": "COFFCAP"
                }
            ]
        }
    ]
}
```

In this case since the `basket` contains a `COFFCAP` product that is eligiable for the customers `wholeEarthCafeCoffee` reward the customers `wholeEarthCafeCoffee` reward is included in the response.

The following information is included in the response:

- `hasApplicableVouchers` Indicates if any of the customers vouchers are applicable to the `basket`
- `totalApplicableRewards` Indicates the total number of applicable customer vouchers to the `basket`
- `totalDiscountInCents` Indicates the total discount that applies to the entire sale
- `vouchers` An array of the customers vouchers that apply to the `basket`
- `discounts` A break-down of how the applicable vouchers applie to products in the basket.

----

- Example API call with single `CHEESECK` product in the basket with a customer who has a `wholeEarthCafeCoffee` and `wholeEarthCafeCake` reward:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"basket\":[{\"sku\":\"CHEESECK\",\"unitPrice\":3500,\"quantity\":1}],\"saleTotal\":3500}" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/available-rewards
```

----

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT",
    "hasApplicableVouchers": true,
    "totalApplicableRewards": 1,
    "totalDiscountInCents": 3200,
    "vouchers": [
        {
            "id": "1aa359b35",
            "name": "Whole earth cafe cake",
            "qrCode": "2093451",
            "voucherNumber": "2093451",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCake"
        }
    ],
    "discounts": [
        {
            "amount": 3200,
            "name": "Whole earth cafe cake Reward #2093451",
            "products": [
                {
                    "discount": 3200,
                    "quantity": 1,
                    "sku": "CHEESECK"
                }
            ]
        }
    ]
}
```

In this case since the `basket` contains a `CHEESECK` product that is eligiable for the customers `wholeEarthCafeCake` reward the customers `wholeEarthCafeCake` reward is included in the response.

----

- Example API call with two unique products `CHEESECK` and `COFFCAP` in the basket with a customer who has a `wholeEarthCafeCoffee` and `wholeEarthCafeCake` reward:

``` bash
curl -H "api-key: 1234" -H "Content-Type: application/json" -X POST -d "{\"basket\":[{\"sku\":\"COFFCAP\",\"unitPrice\":2200,\"quantity\":1}, {\"sku\":\"CHEESECK\",\"unitPrice\":3500,\"quantity\":1}],\"saleTotal\":5700}" https://vsp.swarmapps.com/moonunit/v1/customer/6L2NMOT/available-rewards
```

----

``` javascript
{
    "result": "success",
    "brand": "swarm",
    "uid": "6L2NMOT", 
    "hasApplicableVouchers": true,
    "totalApplicableRewards": 2,
    "totalDiscountInCents": 5400,
    "vouchers": [
        {
            "id": "1aa359b31",
            "name": "Whole earth cafe coffee",
            "qrCode": "7531138",
            "voucherNumber": "7531138",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCoffee"
        },
        {
            "id": "1aa359b35",
            "name": "Whole earth cafe cake",
            "qrCode": "2093451",
            "voucherNumber": "2093451",
            "voucherType": "/swarm/voucherTypes/wholeEarthCafeCake"
        }
    ],
    "discounts": [
        {
            "amount": 2200,
            "name": "Whole earth cafe coffee Reward #7531138",
            "products": [
                {
                    "discount": 2200,
                    "quantity": 1,
                    "sku": "COFFCAP"
                }
            ]
        },
        {
            "amount": 3200,
            "name": "Whole earth cafe cake Reward #2093451",
            "products": [
                {
                    "discount": 3200,
                    "quantity": 1,
                    "sku": "CHEESECK"
                }
            ]
        }
    ]
}
```

In this case since the `basket` contains both a `CHEESECK` and `COFFCAP` product that are eligiable for the customers `wholeEarthCafeCoffee` and `wholeEarthCafeCake` rewards, both are included in the response.

----
