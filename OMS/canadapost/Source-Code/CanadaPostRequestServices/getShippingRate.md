**Package:**
```java
import co.hotwax.canadapost.CandaPostRequestServices.java
```

**Service Definition:**
```xml
<service name="getShippingRate" engine="java" auth="true"
     location="co.hotwax.canadapost.CanadaPostRequestServices" invoke="getShippingRate">
    <implements service="getShippingRateInterface"/>
    <attribute name="carrierPartyId" mode="IN" type="String" optional="true"/>
    <attribute name="shipmentMethodTypeId" mode="IN" type="String" optional="true"/>
</service>
```

**Code Explantion:**

Here's a step-by-step explanation of the code:

1. The getShippingRate method:
- Retrieves the necessary details from the context, such as the shipment ID, shipment method type ID, and carrier party ID.
- Validates the input parameters and retrieves the shipping details using the ShippingAggregatorHelper.getShippingDetails (TODO) method.
- Uses CanadaPostHelper.getConfig (TODO) method  to get configuration details
- From the result obtained form it, it prepares the request context for sending the rate request to Canada Post, including the necessary headers, template content ID, and endpoint.
- Sends the rate request to Canada Post using the ShippingAggregatorHelper.sendRequest(TODO) method.
- Handles the response from Canada Post and updates the shipment package route segment with any error messages.
- Returns the shipping rates in the response.

2. The createContractShipment method:
- Retrieves the necessary details from the context, such as the shipment ID, carrier party ID, and facility ID.
- Validates the input parameters and retrieves the shipping details using the ShippingAggregatorHelper.getShippingDetails method.
- Prepares the request context for creating the contract shipment on Canada Post, including the necessary headers, template content ID, endpoint, and service code.
- Sends the request to Canada Post using the ShippingAggregatorHelper.sendRequest method.
- Handles the response from Canada Post and updates the shipment package route segment with any error messages.
- Returns a success message if the contract shipment is created successfully.

4. The getShippingLabel method:
Retrieves the necessary details from the context, such as the shipment ID, carrier party ID, and facility ID.
Validates the input parameters and retrieves the shipping details using the ShippingAggregatorHelper.getShippingDetails method.
Prepares the request context for creating the shipping label on Canada Post, including the necessary headers, template content ID, endpoint, and label size.
Sends the request to Canada Post using the ShippingAggregatorHelper.sendRequest method.
Handles the response from Canada Post and updates the shipment package route segment with any error messages.
Returns a success message if the shipping label is created successfully.



```java

```
