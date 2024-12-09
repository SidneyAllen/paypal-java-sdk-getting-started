# How to Get Started

## Introduction
This guide walks you through installing PayPal's Java SDK as a dependency, obtaining your sandbox API credentials and creating your first order with PayPal's sandbox.

### Important Notes
⚠️ **Beta Release Notice**
This version is considered a beta release. While we have done our best to ensure stability and functionality, there may still be bugs, incomplete features, or breaking changes in future updates.

**Available Features**
This SDK currently contains only 3 of PayPal's API endpoints. Additional endpoints and functionality will be added in the future.

**API Changes**
Expect potential changes in APIs and features as we finalize the product.

#### Information
The PayPal Server SDK provides integration access to the PayPal REST APIs. The API endpoints are divided into distinct controllers:

* Orders Controller: [Orders API v2](https://developer.paypal.com/docs/api/orders/v2/)
* Payments Controller: [Payments API v2](https://developer.paypal.com/docs/api/payments/v2/)
* Vault Controller: [Payment Method Tokens API v3](https://developer.paypal.com/docs/api/payment-tokens/v3/) *Available in the US only.*

Find out more here: https://developer.paypal.com/docs/api/orders/v2/

## Setup your Project
In your chosen code editor (VSCode, IntelliJ, Eclipse, etc), create a new Java  project.

### Install the Package
Install the SDK by adding the following dependency in your project's pom.xml file:

``` xml
<dependency>
    <groupId>
        com.paypal.sdk
    </groupId>
    <artifactId>
        paypal-server-sdk
    </artifactId>
    <version>
        0.5.1
    </version>
</dependency>
```

You can also view the package at [Maven Central](https://central.sonatype.com/artifact/com.paypal.sdk/paypal-server-sdk/0.5.1) and find  snippets for Gradle, sbt and other common dependency managers.

###

## Environments
PayPal provides a sandbox environment for development and a production environment for live transactions. This guide explains how to start using SDKs using the **sandbox environment**. 

For more information checkout:
* [PayPal Sandbox Testing Guide](https://developer.paypal.com/tools/sandbox/)
* [PayPal Go Live Guide](https://developer.paypal.com/api/rest/production/).


## Get Client ID and Secret

PayPal SDKs use a client ID and secret to authenticate API calls: 

#### Keep the client secret safe.
Avoid hardcoding sensitive information like a client secret within your source code to prevent accidental exposure through commits or leaks. Access secrets through environment variables set during deployment.
  
Here's how to get your Sandbox client ID and secret:

1. Login to the [PayPal Developer Dashboard](https://www.paypal.com/signin?returnUri=https%3A%2F%2Fdeveloper.paypal.com%2Fdashboard%2Fapplications%2Fsandbox&intent=developer&ctxId=ul11a066f6c9f24569adf8595e023b30ee&_ga=2.56579371.1236056270.1733503268-692909966.1723047028)
2. Select Apps & Credentials.
3. New accounts come with a Default Application in the REST API apps section. To create a new project, select Create App.
4. Copy the client ID and  secret for your app.

### Initialize the API Client
Configure your API Client by using the following code and inserting your Sandbox client id and secret.

``` java
PaypalServerSDKClient client = new PaypalServerSDKClient.Builder()
    .clientCredentialsAuth(new ClientCredentialsAuthModel.Builder(
            "YOUR_CLIENT_ID",
            "YOUR_CLIENT_SECRET"
        )
        .build())
    .environment(Environment.SANDBOX)
    .build();
```

The following parameters are configurable for the API Client:

| Parameter  | Type | Description | 
| -------- | ------- | ---------- |
| environment  | Environment  |  The API environment. <br>**Default**: Environment.SANDBOX |
| httpClientConfig | Consumer<HttpClientConfiguration.Builder>  |  Set up Http Client Configuration instance. |
| loggingConfig    | Consumer<ApiLoggingConfiguration.Builder>  |   Set up Logging Configuration instance.    |
|  clientCredentialsAuth  |  [ClientCredentialsAuth](https://developer.paypal.com/serversdk/java/getting-started/how-to-get-started#oauth2-oauth-2-client-credentials-grant)  |   The Credentials Setter for OAuth 2 Client Credentials Grant  |

	
### Initialize an order controller
API methods are grouped together via a controller class.

Below is code showing how to initialize the Orders controller with your client.

``` java
OrdersController ordersController = client.getOrdersController();
```

### Initialize an order object

Endpoints that require a request body utilize  models in the SDK to populate an instance of the object for use when creating or updating a resource.

Below is code showing how an instance of the OrdersCreateInput object is built with two required properties country code and amount. 

*There are many additional properties you can set on the order object.*

``` java
 OrdersCreateInput ordersCreateInput = new OrdersCreateInput.Builder(
    null,
    new OrderRequest.Builder(
        CheckoutPaymentIntent.CAPTURE,
        Arrays.asList(
            new PurchaseUnitRequest.Builder(
                new AmountWithBreakdown.Builder(
                    "USD",
                    "19.99"
                )
                .build()
            )
            .build()
        )
    )
    .build()
)
.prefer("return=minimal")
.build();
```

### Create an order

Each controller has a series of methods that correspond to actions that can be performed on a resource.

The code below shows an asynchronoous method that creates an order by passing the ordersCreateInput object as the argument.

``` java
ordersController.ordersCreateAsync(ordersCreateInput).thenAccept(result -> {
    // TODO success callback handler
    System.out.println(result);
}).exceptionally(exception -> {
    // TODO failure callback handler
    exception.printStackTrace();
    return null;
});
```

## Complete code
The code below brings all the steps together into a runnable example with the appropriate  import statements.

``` java
package com.example;

import java.io.IOException;

import com.paypal.sdk.authentication.ClientCredentialsAuthModel;

import com.paypal.sdk.Environment;
import com.paypal.sdk.PaypalServerSDKClient;
import com.paypal.sdk.controllers.OrdersController;
import com.paypal.sdk.models.AmountWithBreakdown;
import com.paypal.sdk.models.CheckoutPaymentIntent;
import com.paypal.sdk.models.OrderRequest;
import com.paypal.sdk.models.OrdersCreateInput;
import com.paypal.sdk.models.PurchaseUnitRequest;
import java.util.Arrays;

public class Program {

    public static void main(String[] args) throws IOException {
    	
    	PaypalServerSDKClient client = new PaypalServerSDKClient.Builder()
		    .clientCredentialsAuth(new ClientCredentialsAuthModel.Builder(
		            "YOUR_CLIENT_ID",
		            "YOUR_CLIENT_SECRET"
		        )
		        .build())
		    .environment(Environment.SANDBOX)
		    .build();
  
    	OrdersController ordersController = client.getOrdersController();

        OrdersCreateInput ordersCreateInput = new OrdersCreateInput.Builder(
            null,
            new OrderRequest.Builder(
                CheckoutPaymentIntent.CAPTURE,
                Arrays.asList(
                    new PurchaseUnitRequest.Builder(
                        new AmountWithBreakdown.Builder(
                            "USD",
                            "19.99"
                        )
                        .build()
                    )
                    .build()
                )
            )
            .build()
        )
        .prefer("return=minimal")
        .build();

        ordersController.ordersCreateAsync(ordersCreateInput).thenAccept(result -> {
            // TODO success callback handler
            System.out.println(result.getResult().getStatus());
        }).exceptionally(exception -> {
            // TODO failure callback handler
            exception.printStackTrace();
            return null;
        });
    }
}
```

## What next

Provide a list of additional tutorials or resources for developers to use to continue expanding their knowldege of PayPal's APIs and SDKs.
	
