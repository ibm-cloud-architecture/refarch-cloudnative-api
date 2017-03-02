# API publication with IBM API Connect on Bluemix

*This project is part of the 'IBM Cloud Native Reference Architecture' suite, available at
https://github.com/ibm-cloud-architecture/refarch-cloudnative*

### Deploy using Bluemix DevOps Continuous Delivery Toolchain
[![api-toolchain](https://new-console.ng.bluemix.net/devops/graphics/create_toolchain_button.png)](https://new-console.ng.bluemix.net/devops/setup/deploy/?repository=https://github.com/ibm-cloud-architecture/refarch-cloudnative-api.git&branch=integration)

This project contains the API definitions for the entire BlueCompute application. There are three API Products: 
- OAuth Provider
- [Social Review](https://github.com/ibm-cloud-architecture/refarch-cloudnative-micro-socialreview)
- Store API (consisting of [Customer](https://github.com/ibm-cloud-architecture/refarch-cloudnative-micro-customer), [Catalog](https://github.com/ibm-cloud-architecture/refarch-cloudnative-micro-inventory), and [Orders](https://github.com/ibm-cloud-architecture/refarch-cloudnative-micro-orders) APIs)

Each API is defined in OpenAPI format. Each product is published to an API Connect instance on Bluemix in the `BlueCompute` catalog. The APIs uses DataPower gateway to invoke the REST endpoint exposed by [Netflix Zuul Proxy](https://github.com/ibm-cloud-architecture/refarch-cloudnative-netflix-zuul).

The API product definitions are defined in their own folders.  Each of the microservices should be provisioned before the API are usable.

Once the API is published, it can be consumed by the [BlueComputeMobile](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-mobile) and [BlueComputeWeb](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-web) applications.

## Review and Update the API

To review or edit the API definitions, you need to install IBM API connect developer toolkit (cli) at https://console.ng.bluemix.net/docs/cli/index.html#cli.  

### Review and update the OAuth Provider API

To review the OAuth Provider API,

  `$ cd oauthProvider`  
  `$ apic edit`  

Navigate to APIs -> oauth 1.0.0.  Locate the `OAuth 2` Tab in the left navigation panel.  
- Under `Identity Extraction` -> `Custom Form`, change the URL to match the [Auth Microservice](https://github.com/ibm-cloud-architecture/refarch-cloudnative-auth) public route, for example `https://us-auth-cloudnative-prod.mybluemix.net/login.html`.  This is the form shown to clients at the API Connect authentication URL using the `Implicit` grant, e.g. `https://api.us.apiconnect.ibmcloud.com/<org>-<space>/<catalog>/oauth20/authorize`.

- Under `Authenctication` -> `Authentication URL`, change the URL to match the [Auth Microservice](https://github.com/ibm-cloud-architecture/refarch-cloudnative-auth) public route, for example `https://us-auth-cloudnative-prod.mybluemix.net/authenticate` . This is the URL that API Connect uses to validate user credentials.  See the [Auth Microservice](https://github.com/ibm-cloud-architecture/refarch-cloudnative-auth) for more information.

### Review and update the Store API

To review all Store APIs,

  `$ cd store-api`  
  `$ apic edit`  

This will open a Browser with IBM API Designer and Sign in with your Bluemix account.

#### Set the `TARGET_HOST`

Navigate to APIs -> catalog 1.0.0. Locate the Properties tab from the left navigation panel. Click on the TARGET_HOST entry will expand the property definition.

Update the default value to the hostname of your Zuul Proxy (Its application route). For example:
  `https://us-netflix-zuul-cloudnative-prod.mybluemix.net`
  
**You can update the API swagger definition file directly as well**
For example, edit the `catalog.yaml` file, locate the `TARGET_HOST` property and change the value to your bff app endpoint:

```
properties:
  TARGET_HOST:
    value: 'https://us-netflix-zuul-cloudnative-prod.mybluemix.net'
    description: ''
    encoded: false
```

Perform the same procedure for `customer` and `orders` service.  The value of `TARGET_HOST` should be same (as all three of these microservices should be behind the same Zuul Proxy).

#### Set the OAuth Authorization URL for OAuth Protected API

For `orders` and `customer` service, the API are protected by the OAuth Provider.  The OAuth Authorization URL needs to be updated to the OAuth provider.

For example, 
1. navigate to APIs -> customer 1.0.0.  
2. Locate the `Security Definitions`.  
3. Under `apic-oauth-provider (OAuth)`, update the `Authorization URL` to point to the catalog host to `https://api.us.apiconnect.ibmcloud.com/<org>-<space>/<catalog>/oauth20/authorize` (e.g. `https://api.us.apiconnect.ibmcloud.com/centus-ibm-com-cloudnative-prod/bluecompute/oauth20/authorize`).  

If a client attempts to call the OAuth protected API without a valid Bearer token, it will be redirected to this URL.  Repeat the above for `orders`.

#### Set the JWT HS256 Shared Key

The solution uses JWT token to ensure only API Connect gateway can invoke Zuul proxied Microservices. The Store API and Social Review API use the IBM API Connect jwt-generate activity to generate and sign the JWT token. We use HS256 algorithm and need a shared key in your API. A sample key has been provided in the git projects, but if you would like to get your own key, please follow [the security instruction](https://github.com/ibm-cloud-architecture/refarch-cloudnative/blob/master/static/security.md#generate-jwt-shared-key) to get the shared key.

Once you have the key, locate the definition **- set: "hs256-key"** in the `customer.yaml` file, then replace the **k** property with the key generated above (replace the string VOKJrRJ4UuKbioNd6nIHXCpYkHhxw6-0Im-AupSk9ATvUwF8wwWzLWKQZOMbke-xxxx):

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

Repeat the above for `catalog.yaml` and `order.yaml`

### Review and Update the Social Review API

To update and review the Social Review,

  `$ cd socialreview`  
  `$ apic edit`  

### Set the `TARGET_HOST`

IBM API Connect proxies the Social Review microservice, which is implemented using OpenWhisk actions behind the experimental OpenWhisk API gateway.  Navigate to APIs -> socialreview 1.0.0. Locate the Properties tab from the left navigation panel. Click on the TARGET_HOST entry will expand the property definition.

Update the default value to the hostname of the Openwhisk experimental API gateway.  This can be retrieved using the command

```
# wsk api-experimental list
```

For more information, see the [Social Review](https://github.com/ibm-cloud-architecture/refarch-cloudnative-micro-socialreview) git repository.

#### Set the OAuth Authorization URL for OAuth Protected API

The API for posting review comments are protected by the OAuth Provider.  The OAuth Authorization URL needs to be updated to the OAuth provider.

1. navigate to APIs -> socialreview 1.0.0.  
2. Locate the `Security Definitions`.  
3. Under `apic-oauth-provider (OAuth)`, update the `Authorization URL` to point to the catalog host to `https://api.us.apiconnect.ibmcloud.com/<org>-<space>/<catalog>/oauth20/authorize` (e.g. `https://api.us.apiconnect.ibmcloud.com/centus-ibm-com-cloudnative-prod/bluecompute/oauth20/authorize`).  

If a client attempts to call the OAuth protected API without a valid Bearer token, it will be redirected to this URL.


## Deploy to Bluemix API Connect

IBM API Connect developer toolkit provides integrated command line utility to publish the APIs. To do this, you need to have your Bluemix API Connect service configured properly, if not, please reference the setup README at the root repository *https://github.com/ibm-cloud-architecture/refarch-cloudnative*

Assuming deploying to API connect environment at: us.apiconnect.ibmcloud.com/orgs/centusibmcom-cloudnative-dev.  Log in to APIC and set the correct catalog:

   `$ apic login --server us.apiconnect.ibmcloud.com`  
   `$ apic config:set catalog=apic-catalog://us.apiconnect.ibmcloud.com/orgs/centusibmcom-cloudnative-dev/catalogs/bluecompute`  

### Publish OAuth Provider API

Execute the following command:

   ```
   # cd oauthProvider
   # apic publish oauth-provider-product.yaml
   ```
   
This will deploy the OAuth provider APIs to Bluemix API Connect runtime.

### Publish Store API

Execute the following command:

   ```
   # cd store-api
   # apic publish store-api-product.yaml
   ```

This will deploy the store APIs to Bluemix API Connect runtime.

### Publish SocialReview API

   ```
   # cd socialreview
   # apic publish socialreview-product.yaml
   ```

This will publish the SocialReview API to Bluemix API Connect.

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
8. In the developer portal, navigate to `API Products`. You will see the currently available APIs. 
9. Click on the store-api (1.0.0) API.  
10. Click "Subscribe" in the API page.  Select the `MobileWeb-App-Dev` App.
![API Running](static/imgs/bluemix_18.png?raw=true)   
11. Repeat for the `oauth-provider` and `socialreview` API products.
12. Go back to the Apps -> BlueCompute-Mobile page, you will see that all APIs are subscribed in your Application page.  

Now, the APIs are ready to be consumed by the BlueCompute Mobile and Web applications.
