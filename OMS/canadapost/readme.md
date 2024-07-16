The Canada Post plugin is used to integrate your eCommerce platform with Canada Post's shipping services. The plugin provides the following functionalities:

1. Get Shipping Rates: This API is used to fetch shipping rates from Canada Post for a given shipment and carrier. It takes into account the shipment details, origin and destination addresses, and other relevant parameters.
2. Get Shipping Label: This API is used to generate a shipping label for a given shipment and carrier. It retrieves the label data from Canada Post and converts it into a format suitable for display or storage.
3. Create Contract Shipment: This API is used to create a contract shipment with Canada Post for a given shipment and carrier. It sends the necessary details to Canada Post to create the shipment and receives the confirmation.
4. Get Artifact: This API is used to retrieve artifacts such as shipping labels from Canada Post. It sends a request to Canada Post to fetch the artifact and receives the response.

The plugin does not have a specific use for facility-wise configuration. It retrieves the necessary configuration details directly from the database or configuration files.

Please note that the provided code snippet is a part of the plugin and contains the implementation of the getShippingLabel and createContractShipment APIs. The complete plugin code would include additional functionalities and configurations.
