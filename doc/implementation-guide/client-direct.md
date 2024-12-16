# FAVI Partner Events Tracking

## Implementation guide - Direct client-side integration

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. direct client-side (frontend) integration of FAVI script (called C2S),
2. [client-side (frontend) integration using Google Tag Manager (called GTM)](client-gtm.md),
3. [server-to-server integration using API (called S2S)](server-to-server.md).

The S2S solution is most feature rich, reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone. So pick the integration which most suits your situation.

This guide is for the C2S integration, but you can have a look at the other ones using appropriate links.

### Client-side tracking code

There is currently only one supported event - `createOrder`, so you will not need to load the tracking script on every page, only on the page(s) where an order has been confirmed.

You need to:

1. initialize the global variables,
2. send `createOrder` event,
3. load the tracking script.

In the example below, you can see all these steps:

```html
<script type="text/javascript">
    window.faviPartnerEventsTracking = window.faviPartnerEventsTracking || function() {
        window.faviPartnerEventsTracking.queue.push(arguments);
    };
    window.faviPartnerEventsTracking.queue = window.faviPartnerEventsTracking.queue || [];
    window.faviPartnerEventsTracking('init', '{FAVI-TRACKING-ID}');

    window.faviPartnerEventsTracking('createOrder', {
        orderId: '1251ddfd-4fd7-46b8-ac92-36d620e2f240',
        orderItems: [
            {
                product: {
                    id: '1251a241-19cb-4a45-af42-3575c066697f',
                    name: 'Acme 2.5m Square Arm Sofa with Reversible Cushions',
                },
            },
            {
                product: {
                    id: '18ce22cc-db0c-47a7-ba64-79f031d22574',
                    name: 'Wide Barrel Armchair by Acme',
                },
            },
        ],
        customer: {
            email: 'john.doe@example.com',
            name: 'John Doe',
        },
        expectedDeliveryDate: '2021-12-02',
    });
</script>
<script defer src='https://partner-events.favicdn.net/v1/partnerEventsTracking.js' type="text/javascript"></script>
```

where `{FAVI-TRACKING-ID}` is the *Tracking ID* you were assigned by your account manager at FAVI. Read about the `createOrder` object parameter below, to see what is required and in what format.

You can place this code anywhere on the order confirmation page. The higher it will be the better chance the event(s) will be sent before the user navigates away. But always initialize the global variables before loading the tracking script.

If you are doing something wrong, you should see error messages in web console / developer tools (usually F12 in browser). When you are all set up, you should see network requests being sent to `https://partner-events.favi.{XX}`, where `{XX}` is the country where your e-shop is registered.

Chrome and Chrome-based browsers may not show responses for failed requests, so in order to see helpful error messages from the API, please try using Firefox or the most reliable way is using a "copy as cURL" feature on the request.

If you have a dynamic client application that can send more orders for one page load, you can send multiple events, but always load the tracking script and initialize it just once.

If your e-shop is using Content Security Policy, see the dedicated section below.

### `createOrder` event

Use this event after an order is created on your e-shop with products that you advertise on FAVI. We will send a review request to the customer.

`createOrder` event is tracked by calling:

```js
window.faviPartnerEventsTracking('createOrder', order);
```

where `order` is an object with the following format:

`order`:

* `orderId`
  * required, string
  * your internal order ID
* `orderItems`
  * required, array of `orderItem` objects (see below)
* `customer`
  * required, `customer` object (see below)
* `expectedDeliveryDate`
  * optional, string
  * format conforming to `Y-m-d` according to [PHP date format](https://www.php.net/manual/en/datetime.format.php#refsect1-datetime.format-parameters)
  * will be used to decide when to send review request to customer

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

### Content Security Policy (CSP)

If your e-shop is using CSP, you will have to allow *FAVI Partner Events Tracking* to do its job by:

1. allowing execution of initialization and sending events code (`script-src`),
2. allowing loading tracking script from `https://partner-events.favicdn.net` (`script-src`),
3. allowing sending tracking data to `https://partner-events.favi.{XX}` (`connect-src`), where `{XX}` is the country where your e-shop is registered.

Depending on your CSP policy, you might want to initialize the globals and/or load the tracking script differently, which is fine, just remember, that you always have to initialize the global variables before loading the tracking script.

### Contact

If you have any questions regarding the implementation or need any help, please ask our account managers (you can find your contact in your partner dashboard after logging in on FAVI).
