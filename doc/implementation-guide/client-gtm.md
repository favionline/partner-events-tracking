# FAVI Partner Events Tracking

## Implementation guide - Google Tag Manager

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. [direct client-side (frontend) integration of FAVI script (called C2S)](client-direct.md),
2. client-side (frontend) integration using Google Tag Manager (called GTM),
3. [server-to-server integration using API (called S2S)](server-to-server.md).

One of the client side solutions is needed to measure campaign performance - such as attribution.

The S2S solution is most reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone.

You can pick any solution, or even combination of them - for example it is best to use a client side solution to create an order, but you can send more (or update) information about it later using the S2S solution.

This guide is for the GTM integration, but you can have a look at the other ones using appropriate links.

### GTM tag template

We have prepared a GTM tag template, which is published in the GTM *Community Template Gallery*, so it is ready to be used - you will just provide the needed data.

Please [follow this guide](https://github.com/favionline/partner-events-tracking-gtm-template/blob/master/doc/implementation-guide/readme.md) to see how to implement it. 
