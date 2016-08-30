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

Save the file


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
Next, you need to integrate the Mobile and Web BlueCompute applications with APIs.
