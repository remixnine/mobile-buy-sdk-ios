[![Circle CI](https://circleci.com/gh/Shopify/checkout-anywhere-ios/tree/develop.svg?style=svg&circle-token=bc81f8016a1c01955fb98204e59f01c418e02c4c)](https://circleci.com/gh/Shopify/checkout-anywhere-ios/tree/develop)

##What is this?

This is a client implementation of Checkout Anywhere for iOS that allows you to perform checkouts from an App. This also supports ApplePay.

This library also includes an implementation of Shopify's Storefront API. This is an API that allows you to fetch products and collections from a specific shop (i.e. ______.myshopify.com).

## Adding to a Project

You can either use CocoaPods or include this project in your workspace/project directly. However, you must do two steps:

1. Link against `libdstdc++.dylib` under **Project > Build Phases > Link Binary With Libraries**.
2. Add `-ObjC` in your `Other Linker Flags` under **Project > Build Settings.**.

## Flow

1. Create a cart.
2. Transform cart into a Checkout.
3. Add information to the Checkout.
4. Complete the checkout using an ApplePay token (or use a credit card).
6. Poll until the payment job completes.
7. Bask in the glory of a created order.

We highly recommend that you use ApplePay as the sole source of payment within your application, even if this SDK supports payment by credit card as well.
Please familiarize yourself with the [Mobile PCI Standards](https://www.pcisecuritystandards.org/documents/Mobile_Payment_Security_Guidelines_Developers_v1.pdf) before including a full credit card checkout flow in your application.

## Flow Examples

For a view of what a 'synchronous' view of checkout would look like: see the [integration tests](https://github.com/Shopify/checkout-anywhere-ios/blob/master/CheckoutAnywhere/CheckoutAnywhereTests/CHKAnywhereIntegrationTest.m)

Here is a simplistic view:

```
//Create the checkout
[_checkoutDataProvider createCheckout:checkout block:...]

//Fetch shipping rates as necessary
[_checkoutDataProvider getShippingRatesForCheckout:checkout block:...]

//Update the checkout with any updated information (refetch shipping rates if the addresses change)
[_checkoutDataProvider updateCheckout:checkout completion:...] (if necessary)

//Add a payment method and complete the checkout (see below for Options 1 and 2)
...


//Poll for completion -- this may take a few seconds.
//You will need to poll at a regular interval until this is complete (the status will be one of the many CHKStatus values).
[_checkoutDataProvider getCompletionStatusOfCheckout:checkout block:...]

//Update the checkout to obtain the final information if necessary
[_checkoutDataProvider getCheckout:checkout completion:...]
```

### Option 1: Completing a Checkout using ApplePay
```
//Or directly complete the payment using an ApplePay token (PKPaymentToken *)
[_checkoutDataProvider completeCheckout:checkout withApplePayToken:token block:...]
```

### Option 2: Completing a Checkout using a Credit Card
```
[_checkoutDataProvider storeCreditCard:creditCard checkout:checkout completion:...]
[_checkoutDataProvider completeCheckout:checkout block:...]
```

## Error Handling

Please ensure to verify the `error` object returned with every call back to Shopify. If there was an error along the way, the error object will be populated with a dictionary containing a mapping of fields->errors.
This will give you detailed information about what fields are missing, or what fields are wrong.

For example, if you were to complete a checkout with an **email**, the `error.userInfo` will be:

```
{
  "errors" = {
    "checkout" = {
      "email" = [              //One error associated with a "blank" (or missing) email.
        {
          "code" = "blank",
          "message" = "can't be blank"
        }],
       "line_items" = [],      //No errors associated with this
       "shipping_rate_id" = [] //No errors associated with this
  }
}
```

## Testing
To run the Shopify Anywhere SDK integration tests, you will need a Shopify shop that is publicly accessible (in other words, not hidden behind a password). Please note that the integration tests **will create an order** on that shop. This is to validate that the SDK works properly with Shopify. To have these tests run, fill out the information in `CHKTestConstants.h`.

The integration tests that exercise discount code functionality require the following discounts on your test shop:

1. **code:** "applicable" **amount:** 10% **conditions:** none
2. **code:** "inapplicable" **amount:** \<anything\> **conditions:** order value > $1,000,000

The integration tests that exercise the apply of gitfcards to orders will need you to create 2 giftcards:

1. a valid gift card - you'll need the gift card code 
2. an unexpired gift card - you'll need the both the gift card code and id

## Pods
This repo is also a CocoaPod.

Make sure to run `pod spec lint ShopifyCheckoutAnywhere.podspec` before publishing, and update this README.md to be public facing.
