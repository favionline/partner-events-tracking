# FAVI Partner Events Tracking

## Implementation guide - Server-to-server integration

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. [direct client-side (frontend) integration of FAVI script (called C2S)](client-direct.md),
2. [client-side (frontend) integration using Google Tag Manager (called GTM)](client-gtm.md),
3. server-to-server integration using API (called S2S).

One of the client side solutions is needed to measure campaign performance - such as attribution.

The S2S solution is most reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone.

You can pick any solution, or even combination of them - for example it is best to use a client side solution to create an order, but you can send more (or update) information about it later using the S2S solution.

This guide is for the S2S integration, but you can have a look at the other ones using appropriate links.

### Server-to-server integration

This integration allows you to send data directly from your server, for example as part of creating the order in your e-shop. All the client-side integrations rely on JavaScript, which can easily be blocked in the browser, so the intended data would never be sent.

It is also the most secure one, because in all the client-side integrations your *Tracking ID* is publicly available in the code of the webpage, so in theory anyone can see this and start sending events as if it were you. We also provide additional security in the form of a *Server-Side Token* - see dedicated section below for more information.

Using server-to-server integration offers all the possibilities of the client-side integrations and on top of that gives you access to features, which are not available in the client-side integrations:

* you can change the expected delivery date of the order,
* you can cancel the order.

### Server-side token

We provide additional security in the form of a *Server-Side Token* - a secret token, which you can send when calling the API.

This prevents anyone else, who has access only to your *Tracking ID*, to send events. Typically, if you were using any kind of client-side integration, the *Tracking ID* would have been part of the page and therefore anyone could have seen it.

For the same reasons **never use *Server-Side Token* in a public context** - it should ideally never leave your server.

This can be combined with requiring the *Server-Side Token* for any request for your e-shop. But this also means, that the client side solutions are no longer usable and therefore currently this would not support the campaign performance measurement and optimization capabilities of FAVI Partner Events Tracking.

If you want to turn on requiring the *Server-Side Token* for all requests, please contact our account managers, the implementation should follow these steps:

1) account manager will provide you matching *Server-Side Token* for your *Tracking ID*,
2) then you verify, that you are able to send events (API verifies the token any time you send it),
3) ask the account manager to enable requiring *Server-Side Token* for all requests - after this any request without a *Server-Side Token* will be rejected.

Thanks to these steps it is safe to enable requiring the token even if you originally were not - you can try it before all requests are affected.

If your *Server-Side Token* was compromised, please ask our account managers to generate a new one for you.

### Using the API

The API is available at `https://partner-events.favi.{XX}/api/v1` (where `{XX}` is the country where your e-shop is registered) and all the paths below are relative to this.

All requests:

* use `{FAVI-TRACKING-ID}` which should be replaced by the *Tracking ID* you were assigned by your account manager at FAVI
* can optionally send the *Server-Side Token* in the `X-Favi-Partner-Events-Server-Side-Token` header (see dedicated section above about this functionality)
* should be using JSON body

Common types used in the API (both requests and responses):

* `date-string`
  * format conforming to `Y-m-d` according to [PHP date format](https://www.php.net/manual/en/datetime.format.php#refsect1-datetime.format-parameters)
* `time-string`
  * format conforming to `Y-m-d\TH:i:s.u\Z` ([ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) variant) according to [PHP date format](https://www.php.net/manual/en/datetime.format.php#refsect1-datetime.format-parameters)
  * note that the time includes microseconds (6 decimal places, including trailing zeroes)
  * note that the time is in UTC (Zulu) time zone
* `money`:
  * `value`
    * required, string, format of string is either integer or decimal number
      * `.` used as decimal separator
      * no spaces or other characters
      * there can be no leading zeroes for the whole number part and no trailing zeroes for the fraction part (except for `0.1`, `1.0`, etc.)
    * number of decimal spaces is limited up to the number of digits the currency supports according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)
  * `currency`
    * required, string
    * 3-letter code according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)

Errors from the API are returned as JSON in this basic format - i.e., it can return multiple errors at once:

There are two main kinds of errors differentiated by the returned HTTP response code:

* `400 Bad Request`
  * this means the request breaks a requirement of the API endpoint (something described in this documentation) - i.e., the implementation is probably wrong
  * example:

```json
{
    "errors": [
        {
            "code": "request-object-validation-error",
            "field": "orderId",
            "message": "This value should not be blank."
        },
        {
            "code": "request-object-validation-error",
            "field": "customer",
            "message": "This value should not be null."
        }
    ]
}
```

* `422 Unprocessable Entity`
  * this means that the request cannot be fulfilled based on the state of the data on the server
  * you can use the `code` field, dedicated to explaining what situation occurred, to react to it on the client
  * example:

```json
{
    "errors": [
        {
            "code": "tracking-id-does-not-match"
        }
    ]
}
```

### Create/edit order

`POST /tracking/{FAVI-TRACKING-ID}/orders`

Use this endpoint whenever an order is created on your e-shop with products that you advertise on FAVI.

Also use this endpoint to update the same order (for example when expected delivery date changes) or if you want to provide additional data (for example add customer e-mail for review). Always send complete information about the order (as if it was new) for the particular time, not only changed properties.

If you send `customer` information (see below), FAVI will send a review request to the customer.

JSON body properties:

* `orderId`
  * required, string
  * your internal order ID
* `orderItems`
  * required, array of `orderItem` objects (see below)
* `totalAmountWithoutVat`
  * optional, `money` object
  * total value of the order, including prices of all items, services, discounts, delivery, etc.
    * without VAT
* `customer`
  * optional, `customer` object (see below)
* `expectedDeliveryDate`
  * optional, `date-string`
  * will be used to decide when to send review request to customer
* `orderCreatedTime`
  * optional, `time-string`
    * for backwards compatibility reasons it is allowed to omit microseconds, but it is deprecated, so it is strongly advised to include them
  * orders can be sent with up to 48 hours delay - e.g., if you are sending orders asynchronously, you have 2 days to send a particular order

`orderItem`:

* `product` - required, object:
  * `id`
    * required, string
    * your internal product ID, the same you are sending to FAVI through XML feed
  * `name`
    * required, string
    * your product name, must match the name you are using in XML feed provided to FAVI

`customer`:

* `email`
  * required, string
  * will be used to send review request to customer
* `name`
  * optional, string
  * will be used to show name in the review, customer will have to confirm sharing it publicly, but this eases the review filling process

Example:

```bash
curl --request POST 'https://partner-events.favi.xx/api/v1/tracking/{FAVI-TRACKING-ID}/orders' \
--header 'Content-Type: application/json' \
--data-raw '{
    "orderId": "1251ddfd-4fd7-46b8-ac92-36d620e2f240",
    "orderItems": [
        {
            "product": {
                "id": "1251a241-19cb-4a45-af42-3575c066697f",
                "name": "Acme 2.5m Square Arm Sofa with Reversible Cushions"
            }
        },
        {
            "product": {
                "id": "18ce22cc-db0c-47a7-ba64-79f031d22574",
                "name": "Wide Barrel Armchair by Acme"
            }
        }
    ],
    "totalAmountWithoutVat": {
        "value": "1234.56",
        "currency": "CZK",
    },
    "customer": {
        "email": "john.doe@example.com",
        "name": "John Doe"
    },
    "expectedDeliveryDate": "2021-12-02"
}'
```

Expected `422 Unprocessable Entity` error codes:

* `tracking-id-does-not-match`
  * using invalid *Tracking ID*
  * please check that you are sending the requests to correct country
  * if it was working before, your credentials might have changed based on your request
* `server-side-token-is-missing`
* `server-side-token-does-not-match`
* `expected-delivery-date-is-before-order-created-time`
* `order-created-time-exceeds-allowed-delay`
* `order-created-time-cannot-be-in-the-future`
* `unsupported-currency`
  * Currency is not currently supported by FAVI. If you need to use it, please contact your contact at FAVI.

### Cancel order

`POST /tracking/{FAVI-TRACKING-ID}/cancel-order`

If an order, that you sent to FAVI, is cancelled, please, let us know using this endpoint. This will for example prevent asking the customer for review of product, that he/she never received.

Once the order is cancelled, this can not be reversed in any way.

JSON body properties:

* `orderId`
  * required, string
  * your internal order ID that you used when calling *Create Order* endpoint

Example:

```bash
curl --request POST 'https://partner-events.favi.xx/api/v1/tracking/{FAVI-TRACKING-ID}/cancel-order' \
--header 'Content-Type: application/json' \
--data-raw '{
    "orderId": "1251ddfd-4fd7-46b8-ac92-36d620e2f240"
}'
```

Expected `422 Unprocessable Entity` error codes:

* `tracking-id-does-not-match`
  * using invalid *Tracking ID*
  * please check that you are sending the requests to correct country
  * if it was working before, your credentials might have changed based on your request
* `server-side-token-is-missing`
* `server-side-token-does-not-match`

### Change expected delivery date

`POST /tracking/{FAVI-TRACKING-ID}/change-expected-order-delivery-date`

This endpoint is deprecated, do not use it anymore. Instead, use the `Create/edit order` endpoint again and send complete information about the order to replace the current state.

### Contact

If you have any questions regarding the implementation or need any help, please ask our account managers (you can find your contact in your partner dashboard after logging in on FAVI).
