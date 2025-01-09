# FAVI Partner Events Tracking

## Implementation guide - Direct client-side integration

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. direct client-side (frontend) integration of FAVI script (called C2S),
2. [client-side (frontend) integration using Google Tag Manager (called GTM)](client-gtm.md),
3. [server-to-server integration using API (called S2S)](server-to-server.md).

One of the client side solutions is needed to measure campaign performance - such as attribution.

The S2S solution is most reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone.

You can pick any solution, or even combination of them - for example it is best to use a client side solution to create an order, but you can send more (or update) information about it later using the S2S solution.

This guide is for the C2S integration, but you can have a look at the other ones using appropriate links.

### Client-side tracking code

You need to load the tracking script on every page, where an event should be sent.

You need to:

1. initialize the global variables,
2. create one or more events,
3. load the tracking script,
4. optionally you can continue sending more events.

In the example below, you can see all these steps:

```html
<script type="text/javascript">
    window.faviPartnerEventsTracking = window.faviPartnerEventsTracking || function() {
        window.faviPartnerEventsTracking.queue.push(arguments);
    };
    window.faviPartnerEventsTracking.queue = window.faviPartnerEventsTracking.queue || [];
    window.faviPartnerEventsTracking('init', '{FAVI-TRACKING-ID}', {
        debug: false,
    });

    window.faviPartnerEventsTracking('pageView');

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
        totalAmountWithoutVat: {
            value: '1234.56',
            currency: 'CZK',
        },
        customer: {
            email: 'john.doe@example.com',
            name: 'John Doe',
        },
        expectedDeliveryDate: '2021-12-02',
    });
</script>
<script defer src='https://partner-events.favicdn.net/v1/partnerEventsTracking.js' type="text/javascript"></script>
```

where `{FAVI-TRACKING-ID}` is the *Tracking ID* you were assigned by your account manager at FAVI. Read about the event objects below, to see what is required and in what format.

You can place this code anywhere on the page. The higher it will be the better chance the event(s) will be sent before the user navigates away. But always initialize the global variables before loading the tracking script.

If you have a dynamic client application that can send more orders for one page load, you can send multiple events, but always load the tracking script and initialize it just once.

### User consent

To measure campaign performance (e.g., attribution), we store a cookie in the user's browser. Consent to store this cookie is given (or declined) on the FAVI website, and this choice is passed to this tracking code. If consent is not given, no cookie is created.

This means that no further consent is required on your page, and loading this script should not be conditional on any other consent.

### How to verify that the events are triggering correctly

The best way to verify that the events are triggering correctly is to turn on the `debug` mode by passing the :

```js
    window.faviPartnerEventsTracking('init', '{FAVI-TRACKING-ID}', {
        debug: true,
    });
```

Then you will see all the events that are triggered and registered by the tracking script in the browser console / developer tools (usually F12 in browser).

If you are doing something wrong, you should see error messages in web console.

You should also see network requests being sent to `https://partner-events.favi.{XX}`, where `{XX}` is the country where your e-shop is registered. But not all triggered events are sent to the backend (see details in the description of particular events), so using the `debug` mode is more reliable.

Chrome and Chrome-based browsers may not show responses for failed requests, so in order to see helpful error messages from the API, please try using Firefox or the most reliable way is using a "copy as cURL" feature on the request.

If your e-shop is using Content Security Policy, see the dedicated section below.

### `pageView` event

This event should be triggered for every page shown on your web (except for pages that have a more specific view event type).

By default, the event sends current URL, so it is also important to trigger it before the URL is manipulated (for example removing analytical/marketing parameters).

If you need to trigger the event after the URL is manipulated, please store the original URL and pass it to the event explicitly (see event properties below).

If you show multiple pages that have their own URL using client side code, then you should send a view event for each of them.

This data (alongside the other events) is used to measure and optimize campaign performance.

Note that not all triggered events result in the event being sent to the server - if there is no information related to the campaign performance evaluation, the request is not sent. But if you turn on the `debug` mode, you will see even these events being logged in the console, to be able to check that the implementation is correct.

`pageView` event is tracked by calling:

```js
window.faviPartnerEventsTracking('pageView', pageView);
```

where `pageView` is an object with the following format:

`pageView`:

* `url`
    * optional, string
    * URL must include analytical/marketing query parameters
    * if URL is not set, current URL will be used

### `productDetailView` event

This event is a specialized version of the `pageView` event, so everything from the `pageView` event also applies here. This event should be triggered instead of (not in addition to) the `pageView` event on a product detail page, which allows for more precise campaign evaluation.

`productDetailView` event is tracked by calling:

```js
window.faviPartnerEventsTracking('productDetailView', productDetailView);
```

where `productDetailView` is an object with the following format:

`productDetailView`:

* `url`
    * optional, string
    * URL must include analytical/marketing query parameters
    * if URL is not set, current URL will be used
* `productId`
    * required, string
    * your internal product ID, the same you are sending to FAVI through XML feed

### `createOrder` event

Use this event after an order is created on your e-shop with products that you advertise on FAVI.

Also use this endpoint to update the same order (for example when expected delivery date changes) or if you want to provide additional data (for example add customer e-mail for review). Always send complete information for the particular time, not only changed properties.

If you send `customer` information (see below), FAVI will send a review request to the customer.

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
* `totalAmountWithoutVat`
  * optional, `totalAmountWithoutVat` object (see below)
  * total value of the order, including prices of all items, services, discounts, delivery, etc.
    * without VAT
* `customer`
  * optional, `customer` object (see below)
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

`totalAmountWithoutVat`:
  * `value`
    * required, string, format of string is either integer or decimal number
      * `.` used as decimal separator
      * no spaces or other characters
      * there can be no leading zeroes for the whole number part and no trailing zeroes for the fraction part (except for `0.1`, `1.0`, etc.)
    * number of decimal spaces is limited up to the number of digits the currency supports according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)
  * `currency`
    * required, string
    * 3-letter code according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)

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
