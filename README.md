# API publication with IBM API Connect on Bluemix

*This project is part of the 'IBM Cloud Native Reference Architecture' suite, available at
https://github.com/ibm-cloud-architecture/refarch-cloudnative*

This project contains the API definition for Inventory applications. It documents the Inventory items services in OpenAPI format. It will then be published to API Connect instance on Bluemix. The APIs uses DataPower gateway to invoke the REST endpoint exposed by inventory-bff-app.

The API definition is defined under the `inventory` folder.
You should have the [inventory-bff-app](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bff-inventory) project up and running before working on publishing the APIs.

Once the API is published, it can be consumed by the [BlueComputeMobile](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-mobile) and [BlueComputeWeb](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-web) applications.


The BlueCompuete reference applications uses another set of APIs - SocialReview APIs to add review comments to a product. The API definition is defined as part of the [socialreview-bff-app](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bff-socialreview) project because the BFF application is implemented as IBM API Connect Loopback application and created as an API connect project. Please pay attention to the instruction below when publishing the SocialReview APIs since it is located in a different project.

## Review and Update the API

To review or edit the API definition, you need to install IBM API connect developer toolkit (cli) at https://console.ng.bluemix.net/docs/cli/index.html#cli.

  `$ cd inventory`  
  `$ apic edit`  

This will open a Browser with IBM API Designer and Sign in with your Bluemix account.
Navigate to APIs -> inventory 0.0.1. Locate the Properties tab from the left navigation panel. Click on the TARGET_HOST entry will expand the property definition.

Update the default value to the hostname of your inventory-bff-app (Its application route). For example:
  `https://inventory-bff-app.mybluemix.net`

**You can update the API swagger definition file directly as well**
Edit the `inventory.yaml` file, locate the `TARGET_HOST` property and change the value to your bff app endpoint:

```
properties:
  TARGET_HOST:
    value: 'https://inventory-bff-app.mybluemix.net'
    description: ''
    encoded: false
```

The solution uses JWT token to ensure only API Connect gateway can invoke Zuul proxied Microservices. Both the Inventory and SocialReview APIs are using the IBM API Connect jwt-generate activity to generate and sign the JWT token. We use HS256 algorithm and need a shared key in your API. A sample key has been provided in the git projects, but if you would like to get your own key, please follow [the security instruction](https://github.com/ibm-cloud-architecture/refarch-cloudnative/blob/master/static/security.md#generate-jwt-shared-key) to get the shared key.

Once you have the key, locate the definition **- set: "hs256-key"** in the inventory.yaml file, then replace the **k** property with the key generated above:

```
- set-variable:
    title: "set-variable"
    actions:
    - set: "bluecompute.iss.claim"
      value: "apic"
    - set: "hs256-key"
      value: "{ \"alg\": \"HS256\",   \"kty\": \"oct\",   \"use\": \"sig\",  \
        \ \"k\": \"VOKJrRJ4UuKbioNd6nIHXCpYkHhxw6-0Im-AupSk9ATvUwF8wwWzLWKQZOMbke-xxxx\"\
        \ }"
    description: "Define JWT issuer claim"

```

Save the file.

You need to update the SocialReview APIs as well. You need to change the root directory to socialreview-bff-app:

  `$ cd ../../refarch-cloudnative-bff-socialreview/socialreview/definitions`  

Then, follow the same instructions above to update the socialreview.yaml file:
- SocialReview TARGET_HOST field
- hs256-key with key generated above

Save the file.

## Deploy to Bluemix API Connect

IBM API Connect developer toolkit provides integrated command line utility to publish the APIs. To do this, you need to have your Bluemix API Connect service configured properly, if not, please reference the setup README at the root repository *https://github.com/ibm-cloud-architecture/refarch-cloudnative*

Assuming deploying to API connect environment at: us.apiconnect.ibmcloud.com/orgs/centusibmcom-cloudnative-dev  
Use the following command to deploy the Loopback application

Make sure that you are under folder `refarch-cloudnative-api/inventory`, execute the following command:

   `$ apic login --server us.apiconnect.ibmcloud.com`  
   `$ apic config:set catalog=apic-catalog://us.apiconnect.ibmcloud.com/orgs/centusibmcom-cloudnative-dev/catalogs/bluecompute`  
   `$ apic publish inventory-product_0.0.1.yaml`

This will deploy the inventory APIs to Bluemix API Connect runtime.

You need to deploy the SocialReview APIs as well. You need to change the root directory to socialreview-bff-app:

  `$ cd ../../refarch-cloudnative-bff-socialreview`  
  `$ apic config:set catalog=apic-catalog://us.apiconnect.ibmcloud.com/orgs/centusibmcom-cloudnative-dev/catalogs/bluecompute`  
  `$ apic publish definitions/socialreview-product.yaml`  


This will publish the SocialReview API to Bluemix API Connect.
Next, you need to subscribe to the APIs via API Connect DeveloperPortal so that the Mobile and Web BlueCompute applications can use the APIs.

## Subscribe to the APIs in the Developer Portal

The API Connect Developer Portal enables API providers to build a customized consumer portal for their application developers. It also provides the interface for API consumers to discover APIs and subscribe to a consumption plan by which the API is consumed in either the Mobile or traditional Web application.

1. Open the API Connect Developer Portal. You will need to open your Portal URL. Obtain it by navigating to the ApicStore catalog from the Bluemix API manager console. Click "Settings", then "Portal". ![API Running](static/imgs/bluemix_15.png?raw=true)  
2. Open the API Connnect Portal in another browser window. You should see the portal home page with both the inventory and the socialreviews APIs highlighted.  
3. Click "Create an account" from the upper right menu bar.  
4. In the create an account wizard, enter the credentials of your Bluemix login ID and MobileWeb-App-Dev as your developer organization. Finally click "Create new account"  
5. In your developer portal, click "Login".  
6. Click on the menu tab "Apps" on top. Click on the link to "Register a new Application."  
![API Running](static/imgs/bluemix_16.png?raw=true)  
7. Enter the fields of "Title" and "Description". Enter "**org.apic://example.com**" for the OAuth URI redirection page as shown below then click submit.  
![API Running](static/imgs/bluemix_17.png?raw=true) ( Store the client ID, you will need it later)  
8. In the developer portal, navigate to Apps -> MobileWeb-App-Dev. Below the application, you will see a link to take you to the currently available APIs. Click on that.  
9. Click on the Inventory ( V1.0.0 ) API.  
10. Click "Subscribe" in the API page.  
![API Running](static/imgs/bluemix_18.png?raw=true)   
11. Subscribe to the social review API as well ( you can choose the silver or gold plan)  
12. Go back to the Apps -> BlueCompute-Mobile page, you will see that both APIs are subscribed in your Application page.  

Now, the APIs are ready to be consumed by the BlueCompute Mobile and Web applications.
