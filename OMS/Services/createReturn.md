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

### Adding validations

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
- If customerId is empty and externalCustomerId is present then customerId is set to partyId retrieved using party's externalId
- Either companyId or externalCompanyId is required

  
### Iterating in items

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




