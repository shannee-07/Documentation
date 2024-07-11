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


Besides, it applies different validations in these fields:

- **Customer ID:** For customer Id, either one of these three is required:
  - customerId
  - externalCustomerId
  - customerIdentificationType and customerIdentificationValue
- If `customerIdentificationType` and `customerIdentificationValue` are available, then it is checked if they are present in **PartyIdentification** entity. If they are present, `partyId` received from **PartyIdentification** entity is stored in `customerId` 
 
