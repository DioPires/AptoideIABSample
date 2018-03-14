# AppCoins SDK

AppCoins SDK lets you sell digital content from inside applications.

## Abstract

This tutorial will guide through the process of integrating the AppCoins SDK.
You should be able to get it up and running in less than 10 minutes.

### Prerequisites

+ In order for the AppCoins SDK to work, you must have an AppCoins compliant wallet (trust link) installed.
+ The Android minimum API Level to use AppCoins SDK is 21 (Android 5.0).
+ Basic understanding of RxJava is advised, but now required.

## Getting Started

To integrate the AppCoins SDK, you only need an instance of the AppCoinsSdk interface.
For the sake of simplicity, in the sample code we just hold a static referecence to the AppCoins SDK instance in the Application class.

```
  public static AppCoinsSdk appCoinsSdk;

  private final String developerAddress = "0x4fbcc5ce88493c3d9903701c143af65f54481119";

  @Override public void onCreate() {
    super.onCreate();

    appCoinsSdk = new AppCoinsSdkBuilder(developerAddress).withSkus(buildSkus())
        .withDebug(true)
        .createAppCoinsSdk();
  }

  private List<SKU> buildSkus() {
    List<SKU> skus = new LinkedList<>();

    skus.add(new SKU(SKU_GAS_NAME, SKU_GAS, BigDecimal.valueOf(5)));
    skus.add(new SKU(SKU_PREMIUM_NAME, SKU_PREMIUM, BigDecimal.TEN));

    return skus;
  }
```

Here we use a convenient builder to create our AppCoinsSDK instance with a list consisting of two products.
The debug flag will set the AppCoins SDK to use the testnet (ropsten) instead of the mainnet.

Given the Android architecture, you will have to let the sdk know each time a Purchase Flow is finished. For that, just inform the sdk:

```
@Override protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    appCoinsSdk.onActivityResult(requestCode, requestCode, data);
}
```

AppCoins Sdk's onActivityResult will return true if handled, or false otherwise. In case you want to do some conditional here.

To start the purchase flow, you have to pass one of the previously defined SKU_IDs, and the activity that will be used to call the Wallet:

```
appCoinsSdk.buy(SKU_GAS, this);
```

And finally, we want to react to the purchase, so we can reflect that change in our's App's state.

Following is a good pattern to follow:

```
@Override protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (appCoinsSdk.onActivityResult(requestCode, requestCode, data)) {
      appCoinsSdk.getLastPayment()
          .subscribe(paymentDetails -> runOnUiThread(() -> {
            if (paymentDetails.getPaymentStatus() == PaymentStatus.SUCCESS) {
              String skuId = paymentDetails.getSkuId();
              // Now we tell the sdk to consume the skuId.
              appCoinsSdk.consume(skuId);

              // Purchase successfully done. Release the prize.
            }
          }));
    }
  }
```

First we check if it was the AppCoins SDK who made us leave the current activity, and if so, we will react to the purchase result.
In order to complete the purchase flow, we must consume the purchase.

### Sample Code

This repo holds a sample code bla bla bla lol