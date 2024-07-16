# getShippingLabel:

```xml
<service name="getShippingLabel" engine="java" auth="true"
         location="co.hotwax.canadapost.CanadaPostRequestServices" invoke="getShippingLabel">
    <implements service="getShippingLabelInterface"/>
</service>
```

The getShippingLabel method in the provided Java code snippet is responsible for initiating the label generation process from Canada Post for a given shipment. Here's a detailed explanation of its functionalities:

1. The method starts by retrieving the necessary context and dependencies, such as the delegator, dispatcher, and user login.
2. It then calls the getShippingDetails method from the ShippingAggregatorHelper class to retrieve the shipping details related to the given shipment. If an error occurs during this process, an error message is returned.
3. The shipping details response is processed to extract relevant information, such as the carrier party ID and facility ID.
4. The method creates a new service context for the createContractShipment service and sets the carrier party ID as a parameter. It then calls the createContractShipment service to initiate the label generation process. If an error occurs during this process, an error message is returned.
5. The response from the createContractShipment service is processed to extract the shipping label map and the label PDF link.
6. The method then calls the getArtifact service to retrieve the image of the label PDF using the label PDF link and the facility ID. If an error occurs during this process, an error message is returned.
7. The label image bytes are extracted from the response of the getArtifact service and updated in the shipping label map.
8. The shipment package sequence ID from the shipping details response map is also updated in the shipping label map.
9. (TODO) Ideally, the calling method should process the response and store the information in the database. However, since the old integration services do not follow the same practice, the logic for processing the response is temporarily implemented within the getShippingLabel method.
10. Finally, the method returns a success status indicating that the label generation process has been initiated successfully.

The provided code snippet is a part of a larger system and may not be complete or functional on its own. It is recommended to review the entire codebase and consider the context and dependencies when analyzing and understanding the code.


```java
public Map<String, Object> getShippingLabel(DispatchContext dctx, Map<String, Object> context) {
    Delegator delegator = dctx.getDelegator();
    LocalDispatcher dispatcher = dctx.getDispatcher();
    Map<String, Object> serviceResult = new HashMap<>();
    GenericValue userLogin = (GenericValue) context.get("userLogin");
    Map<String, Object> shippingDetailsResponse = ShippingAggregatorHelper.getShippingDetails(delegator, context);
    if (ServiceUtil.isError(shippingDetailsResponse)) {
        Debug.logError("Error in getting shipping details", MODULE);
        return ServiceUtil.returnError("Error in getting shipping details. Please try again!");
    }
    Map<String, Object> shippingDetailsResponseMap = UtilGenerics.checkMap(shippingDetailsResponse.get("response"));
    try {
        Map<String, Object> serviceContext = dctx.makeValidContext("createContractShipment", ModelService.IN_PARAM, context);
        serviceContext.put("carrierPartyId", shippingDetailsResponseMap.get("carrierPartyId"));
        serviceResult = dispatcher.runSync("createContractShipment", serviceContext);
        if (ServiceUtil.isError(serviceResult)) {
            Debug.logError("Error in initiating label generation from Canada Post. Please try again", MODULE);
            return ServiceUtil.returnError(ServiceUtil.getErrorMessage(serviceResult));
        }
        Map<String, Object> createContractShipmentResponse = UtilGenerics.checkMap(serviceResult.get("response"));
        Map<String, Object> shippingLabelMap = UtilGenerics.checkMap(createContractShipmentResponse.get("shippingLabelMap"));
        String labelPdfLink = (String) UtilGenerics.checkMap(UtilGenerics.checkList(shippingLabelMap.get("packages")).get(0)).get("labelData");
        String facilityId = (String) createContractShipmentResponse.get("facilityId");
        // Get the image of label pdf using getArtifact API
        serviceResult = dispatcher.runSync("getArtifact", UtilMisc.toMap("labelPdfLink", labelPdfLink, "facilityId", facilityId,"userLogin", userLogin, "carrierPartyId", shippingDetailsResponseMap.get("carrierPartyId")));
        if (ServiceUtil.isError(serviceResult)) {
            Debug.logError("Error in getting label from Canada Post. Please try again", MODULE);
            return ServiceUtil.returnError(ServiceUtil.getErrorMessage(serviceResult));
        }
        byte[] imageBytes = (byte[]) UtilGenerics.checkMap(serviceResult.get("response")).get("imageBytes");
        // Update labelData in shippingLabelMap as per the response of getArtifact API
        UtilGenerics.checkMap(UtilGenerics.checkList(shippingLabelMap.get("packages"))
            .get(0)).put("labelData", imageBytes);
        context.put("shippingLabelMap", shippingLabelMap);
        // Update shipmentPackageSeqId in shippingLabelMap with package detail of shippingDetailsResponseMap
        UtilGenerics.checkMap(UtilGenerics.checkList(shippingLabelMap.get("packages"))
            .get(0)).put("shipmentPackageSeqId",
        UtilGenerics.checkMap(UtilGenerics.checkList(shippingDetailsResponseMap.get("packages"))
            .get(0)).get("shipmentPackageSeqId"));
        /* TODO: Ideally the calling method should process the response and store the information in database, for example acceptShipment.
           But since the old integration services doesn't follow the same practice we cannot update the logic until refactoring.
           So, for now we are temporarily processing the response here using the generic helper method.
        */
        Map<String, Object> labelResult = ShippingAggregatorHelper.processShippingLabelResponse(delegator, context);
        if (ServiceUtil.isError(labelResult)) {
            return ServiceUtil.returnError("Error in updating shipping label details. Please try again!");
        }
    } catch (GenericServiceException e) {
        Debug.log(e.getMessage());
        return ServiceUtil.returnError("Error in initiating label generation from Canada Post. Please try again");
    }
    return ServiceUtil.returnSuccess();
}
```


# sendRequest:

The provided Java method, sendRequest, is responsible for sending HTTP requests to a specified URL using the RestClient library. The method follows a step-by-step approach to handle various aspects such as authentication, request type, content type, and template rendering.

Here's a step-by-step explanation of the method:

1. The method starts by checking if the required parameters are provided, such as the URL, authentication details, and request type. If any required parameters are missing, an error message is returned.
2. The method constructs the complete URL by appending the endpoint to the base URL.
3. A new RestClient instance is created and various configurations are set, including the authentication details, request type, headers, content type, and retry settings.
4. If the request type is POST or PUT, and a template content ID is provided, the method attempts to render the template using the ContentWorker. If the rendering fails, an error message is returned.
5. The method sends the HTTP request using the RestClient instance and checks for any HTTP errors. If an error occurs, an error message is returned.
6. If the request is successful, the method retrieves the response and checks if the content type is "application/pdf". If it is, the response bytes are stored in the result map. Otherwise, the response text is stored.
7. Finally, the method returns the result map containing the HTTP response.


Throughout the method, the code checks for potential errors and provides appropriate error messages to handle different scenarios. It also ensures that the method follows best practices by returning only the necessary information and avoiding any additional code beyond the immediate scope of the code block.

Please note that the provided code snippet is incomplete and may require additional context or dependencies to run successfully.
