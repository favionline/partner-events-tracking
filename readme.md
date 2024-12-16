# FAVI Partner Events Tracking

This tracking code allows FAVI's partnering shops to

* participate in FAVI Extra and collect feedback from customers of their online shop,
* measure and optimize campaign performance.

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. [direct client-side (frontend) integration of FAVI script (called C2S)](doc/implementation-guide/client-direct.md),
2. [client-side (frontend) integration using Google Tag Manager (called GTM)](doc/implementation-guide/client-gtm.md),
3. [server-to-server integration using API (called S2S)](doc/implementation-guide/server-to-server.md).

One of the client side solutions is needed to measure campaign performance - such as attribution.

The S2S solution is most reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone.

You can pick any solution, or even combination of them - for example it is best to use a client side solution to create an order, but you can send more (or update) information about it later using the S2S solution.

### Contact

If you have any questions regarding the implementation or need any help, please ask our account managers (you can find your contact in your partner dashboard after logging in on FAVI).
