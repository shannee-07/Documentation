This service creates returns of products belonging to one or multiple orders. It means items from different orders are possible.

# How this service works at code level:

### Getting the data from payload

This service receives the data to create return in a key "payload" from the context and stores it in returnMap variable

```java
 Map<String, Object> returnMap = UtilGenerics.checkMap(context.get("payLoad"));
```

Then, it gets data from this map

```java
String externalId = (String) returnMap.get("externalId");
String returnHeaderTypeId = (String) returnMap.get("type");
String customerId = (String) returnMap.get("customerId");
String externalCustomerId = (String) returnMap.get("externalCustomerId");
String customerIdentificationType = (String) returnMap.get("customerIdentificationType");
String customerIdentificationValue = (String) returnMap.get("customerIdentificationValue");
String companyId = (String) returnMap.get("companyId");
String externalCompanyId = (String) returnMap.get("externalCompanyId");
String returnStatusId = (String) returnMap.get("status");
String currencyCode = (String) returnMap.get("currencyCode");
String note = (String) returnMap.get("note");
String returnDate = (String) returnMap.get("returnDate");
Map<String, Object> shipFrom = UtilGenerics.checkMap(returnMap.get("shipFrom"));
Map<String, Object> shipTo = UtilGenerics.checkMap(returnMap.get("shipTo"));
List<Map<String, Object>> items = UtilGenerics.checkList(returnMap.get("items"));
List<Map<String, Object>> transactions = UtilGenerics.checkList(returnMap.get("returnPaymentPref"));
List<Map<String, Object>> returnAdjustments = UtilGenerics.checkList(returnMap.get("returnAdjustment"));
List<Map<String, String>> returnIdentifications = UtilGenerics.checkList(returnMap.get("returnIdentifications"));
List<String> tagsList = UtilGenerics.checkList(returnMap.get("tags"));
```

## Validations

### Order level validations

Several validation are added in the data received from payload.

It checks if the data that is received exists in the database or not. It Validates following fields in it:

- externalId
- returnHeaderTypeId
- companyId (partyId)
- externalCompanyId (Party's externalId)
- returnStatusId (StatusItem's statusId)
- currencyCode


Besides, it applies different validations in these fields:

- **Customer ID:** For customer Id, either one of these three is required:
  - customerId
  - externalCustomerId
  - customerIdentificationType and customerIdentificationValue
- If `customerIdentificationType` and `customerIdentificationValue` are available, then it is checked if they are present in **PartyIdentification** entity. If they are present, `partyId` received from **PartyIdentification** entity is stored in `customerId` 
- If `customerId` is empty and `externalCustomerId` is present then `customerId` is set to `partyId` retrieved using party's `externalId`
- Either `companyId` or `externalCompanyId` is required

  
### Item Validations

If items exist in payload, it iterates in it. In each iteration:

We are receiving these values from each of the item:

```java
Map<String, Object> returnItemCtx = new HashMap<>();
Map<String, Object> returnItemAdjustmentCtx = new HashMap<>();
String productId = (String) item.get("id");
String sku = (String) item.get("sku");
String idType = (String) item.get("idType");
String idValue = (String) item.get("idValue");
String orderId = (String) item.get("orderId");
String orderExternalId = (String) item.get("orderExternalId");
```

#### Item validations

1. Product:

```java
if (UtilValidate.isEmpty(sku) && UtilValidate.isEmpty(productId) && (UtilValidate.isEmpty(idType) && UtilValidate.isEmpty(idValue))) {
     errorList.add(UtilProperties.getMessage("CommerceUiLabels", "EitherProductOrSkuOrIdTypeAndIdValueRequired", locale));
}
```
The code checks if the SKU (stock keeping unit), productId, and ID type and ID value for the return item are all empty. If all these fields are empty, it means that the system cannot identify the product associated with the return item and an error message is added in it.

After that in the below code validation is applied using DB query based on available data:

- Either productId or sku is available (first and second condition)
- Both sku and productId are available (third condition)
- sku and productId both are not available but idType and idValue are available (fourth condition)

```java
if (UtilValidate.isEmpty(productId) && UtilValidate.isNotEmpty(sku)) {
    long products = EntityQuery.use(delegator).from("Product").where("internalName", sku).queryCount();
    if (products == 0) {
        errorList.add(UtilProperties.getMessage("CommerceUiLabels", "InvalidOrderItemSku", locale));
    }
}
if (UtilValidate.isEmpty(sku) && UtilValidate.isNotEmpty(productId)) {
    long products = EntityQuery.use(delegator).from("Product").where("productId", productId).queryCount();
    if (products == 0) {
        errorList.add(UtilProperties.getMessage("CommerceUiLabels", "InvalidProductId", locale));
    }
}

if (UtilValidate.isNotEmpty(sku) && UtilValidate.isNotEmpty(productId)) {
    long product = EntityQuery.use(delegator).from("Product").where("internalName", sku, "productId", productId).queryCount();
    if (product == 0) {
        errorList.add(UtilProperties.getMessage("ApiUiLabels", "SkuAndProductIdNotAssociated", locale));
    }
}

if (UtilValidate.isEmpty(sku) && UtilValidate.isEmpty(productId) && (UtilValidate.isNotEmpty(idType) && UtilValidate.isNotEmpty(idValue))) {
    if ("SKU".equals(idType)) {
        idType = null;
    }
    productId = NamespaceHelper.getProductId(delegator, idValue, idType);
    if (UtilValidate.isEmpty(productId)) {
        errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidProductIdTypeAndIdValue", UtilMisc.toMap("idType", idType, "idValue", idValue), locale));
    }
}
```

2. Order and item:

Four validations are applied on order items based on available data:
- If orderExternalId or orderId are available (first and second condition)
- If both are available, then it is checked if they are associated to each other or not (third condition)
- If both orderId or orderExternalId are not available but orderName is available

```java
if (UtilValidate.isNotEmpty(orderExternalId)) {
    long orderCount = EntityQuery.use(delegator).from("OrderHeader").where("externalId", orderExternalId).queryCount();
    if (orderCount == 0) {
        errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidOrderExternalId", locale));
    }
}
if (UtilValidate.isNotEmpty(orderId)) {
    long orderCount = EntityQuery.use(delegator).from("OrderHeader").where("orderId", orderId).queryCount();
    if (orderCount == 0) {
        errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidOrderId", locale));
    }
}

if (UtilValidate.isNotEmpty(orderId) && UtilValidate.isNotEmpty(orderExternalId)) {
    long orderCount =
            EntityQuery.use(delegator).from("OrderHeader").where("orderId", orderId, "externalId", orderExternalId).queryCount();
    if (orderCount == 0) {
        errorList.add(UtilProperties.getMessage("ApiUiLabels", "OrderExternalIdAndOrderIdNotAssociated", locale));
    }
}

if (UtilValidate.isEmpty(orderId) && UtilValidate.isEmpty(orderExternalId) && UtilValidate.isNotEmpty(item.get("orderName"))) {
    long orderCount = EntityQuery.use(delegator).from("OrderHeader").where("orderName", item.get("orderName")).queryCount();
    if (orderCount == 0) {
        errorList.add("Invalid Order Name.");
    }
}
```

### Return Adjustment Validation

If return adjusments are available, then we are iterating in the `returnAdjustments`. For each return adjustment, code verifies if there is a return adjustment type with the given type in the system. If no such return adjustment type exists, an error message is added to the error list.

```java
if (UtilValidate.isNotEmpty(returnAdjustments)) {
    for (Map<String, Object> returnAdjustment : returnAdjustments) {
        Map<String, Object> returnAdjustmentCtx = new HashMap<>();
        String type = (String) returnAdjustment.get("type");

        if (UtilValidate.isNotEmpty(type)) {
            long adjustmentType =
                    EntityQuery.use(delegator).from("ReturnAdjustmentType").where("returnAdjustmentTypeId", type).queryCount();
            if (adjustmentType == 0) {
                errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidReturnAdjustmentType" + type, locale));
            }
        }
    }
}
```


### Ship From validation

Validates Contact Mechanisms. The code retrieves the ship from information and the postal address associated with it. 

It checks if the postal address has an `externalId` or a `contactMechId`. If the postal address has an `externalId`, the code verifies if there is a contact mech with the given `externalId` in the system. If no such contact mech exists, an error message is added to the error list.

If the postal address has a `contactMechId`, the code verifies if there is a contact mech with the given `contactMechId` in the system. If no such contact mech exists, an error message is added to the error list.

```java
if (UtilValidate.isNotEmpty(shipFrom)) {
    long shipFromContactMech;
    Map<String, Object> shipFromPostalAddress = UtilGenerics.checkMap(shipFrom.get("postalAddress"));
    String shipFromPostalAddressContactMechId = (String) shipFromPostalAddress.get("id");
    String shipFromPostalAddressExternalId = (String) shipFromPostalAddress.get("externalId");
    if (UtilValidate.isNotEmpty(shipFromPostalAddressExternalId)) {
        shipFromContactMech =
                EntityQuery.use(delegator).from("ContactMech").where("externalId", shipFromPostalAddressExternalId).cache().queryCount();
        if (shipFromContactMech == 0) {
            errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidShipFromPostalAddressExternalId", UtilMisc.toMap("externalId"
                    , shipFromPostalAddressExternalId), locale));
        }
    }
    if (UtilValidate.isNotEmpty(shipFromPostalAddressContactMechId)) {
        shipFromContactMech =
                EntityQuery.use(delegator).from("ContactMech").where("contactMechId", shipFromPostalAddressContactMechId).cache().queryCount();
        if (shipFromContactMech == 0) {
            errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidShipFromPostalAddressId", UtilMisc.toMap("contactMechId",
                    shipFromPostalAddressContactMechId), locale));
        }
    }
    /*Validation for telecom number and email address is not handled because we do not store both of then in the return data model*/
}
```

### Ship To Validation

Checks if facilityId and externalFacilityId exists in the system if given.

```java
if (UtilValidate.isNotEmpty(shipTo)) {
    long facility;
    String facilityId = (String) shipTo.get("facilityId");
    String externalFacilityId = (String) shipTo.get("externalId");
    if (UtilValidate.isNotEmpty(externalFacilityId)) {
        facility = EntityQuery.use(delegator).from("Facility").where("externalId", externalFacilityId).cache().queryCount();
        if (facility == 0) {
            errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidShipToExternalFacilityId", UtilMisc.toMap("externalId",
                    externalFacilityId), locale));
        }
    }
    if (UtilValidate.isNotEmpty(facilityId)) {
        facility = EntityQuery.use(delegator).from("Facility").where("facilityId", facilityId).cache().queryCount();
        if (facility == 0) {
            errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidShipToFacilityId", UtilMisc.toMap("facilityId", facilityId),
                    locale));
        }
    }
}
```

### Return identification Validation

If returnIdentifications is available then each of the returnIdentification in it is validated:
- `returnIdentificationTypeId` is mandatory in every returnIdentification, if it does not exist then error message is added
- A query is perfomed on **Enumeration** table to validate `returnIdentificationTypeId` for enumTypeId="RETURN_IDENTITY"
- Error messgae is added if `returnIdentificationTypeId` is not found in the system or idValue does not exist in the returnIdentification

```java
if (UtilValidate.isNotEmpty(returnIdentifications)) {
    for (Map<String, String> returnIdentification : returnIdentifications) {
        if (UtilValidate.isNotEmpty(returnIdentification.get("returnIdentificationTypeId"))) {
            long returnIdentificationCount = EntityQuery.use(delegator).from("Enumeration").where("enumId", returnIdentification.get(
                    "returnIdentificationTypeId"), "enumTypeId", "RETURN_IDENTITY").queryCount();
            if (returnIdentificationCount == 0) {
                errorList.add(UtilProperties.getMessage("ApiUiLabels", "InvalidReturnIdentificationTypeId", locale));
            }
            if (UtilValidate.isEmpty(returnIdentification.get("idValue"))) {
                errorList.add(UtilProperties.getMessage("ApiUiLabels", "IdValueCannotBeEmpty", locale));
            }
        } else {
            errorList.add(UtilProperties.getMessage("ApiUiLabels", "ReturnIdentificationTypeIdRequired", locale));
        }
    }
}
```

Finally in the validation process, if errorList contains any item means one or more validation is failed then error is returned with errorList:

```java
if (UtilValidate.isNotEmpty(errorList)) {
    return ServiceUtil.returnError(errorList);
}
```
