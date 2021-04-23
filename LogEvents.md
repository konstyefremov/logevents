## nakijin

### HappyTravel.Nakijin.Api/Infrastructure/Logging/LogEvents.json
```json
[
  {"id": 90000, "name": "MappingAccommodationsStart", "level": "Information", "source": "AccommodationMapper", "isException": false}, 
  {"id": 90001, "name": "MappingAccommodationsOfSpecifiedCountryStart", "level": "Information", "source": "AccommodationMapper", "isException": false},
  {"id": 90002, "name": "MappingAccommodationsFinish", "level": "Information", "source": "AccommodationMapper", "isException": false},
  {"id": 90003, "name": "MappingAccommodationsOfSpecifiedCountryFinish", "level": "Information", "source": "AccommodationMapper", "isException": false},
  {"id": 90004, "name": "MappingAccommodationsCancel", "level": "Information", "source": "AccommodationMapper", "isException": false},
  {"id": 90005, "name": "MappingAccommodationsError", "level": "Error", "source": "AccommodationMapper", "isException": true},

  {"id": 90100, "name": "MappingLocationsStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90101, "name": "MappingLocationsFinish", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90102, "name": "MappingLocationsCancel", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90103, "name": "MappingLocationsError", "level": "Error", "source": "LocationMapper", "isException": true},
  {"id": 90104, "name": "MappingCountriesStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90105, "name": "MappingCountriesFinish", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90106, "name": "MappingLocalitiesStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90107, "name": "MappingLocalitiesFinish", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90108, "name": "MappingLocalitiesOfSpecifiedCountryStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90109, "name": "MappingLocalitiesOfSpecifiedCountryFinish", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90110, "name": "MappingLocalityZonesStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90110, "name": "MappingLocalityZonesFinish", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90111, "name": "MappingLocalityZonesOfSpecifiedCountryStart", "level": "Information", "source": "LocationMapper", "isException": false},
  {"id": 90112, "name": "MappingLocalityZonesOfSpecifiedCountryFinish", "level": "Information", "source": "LocationMapper", "isException": false},

  {"id": 90200, "name": "MergingAccommodationsDataStart", "level": "Information", "source": "AccommodationDataMerger", "isException": false},
  {"id": 90201, "name": "MergingAccommodationsDataFinish", "level": "Information", "source": "AccommodationDataMerger", "isException": false},
  {"id": 90202, "name": "MergingAccommodationsDataCancel", "level": "Information", "source": "AccommodationDataMerger", "isException": false},
  {"id": 90203, "name": "MergingAccommodationsDataError", "level": "Error", "source": "AccommodationDataMerger", "isException": true},

  {"id": 90300, "name": "PreloadingAccommodationsStart", "level": "Information", "source": "AccommodationPreloader", "isException": false},
  {"id": 90301, "name": "PreloadingAccommodationsFinish", "level": "Information", "source": "AccommodationPreloader", "isException": false},
  {"id": 90302, "name": "PreloadingAccommodationsCancel", "level": "Information", "source": "AccommodationPreloader", "isException": false},
  {"id": 90303, "name": "PreloadingAccommodationsError", "level": "Error", "source": "AccommodationPreloader", "isException": true},

  {"id": 90400, "name": "ConnectorClientError", "level": "Error", "source": "ConnectorClient", "isException": true},
  
  {"id": 90500, "name": "SameAccommodationInOneSupplierError", "level": "Error", "source": "AccommodationMapper", "isException": false},
  {"id": 90501, "name": "EmptyCoordinatesInAccommodation", "level": "Error", "source": "AccommodationMapper", "isException": false},
  
  {"id": 90502, "name": "LocationsPublished", "level": "Information", "source": "PredictionsUpdateService", "isException": false}
]
```

## columbus-connector

### HappyTravel.ColumbusConnector.Api/Infrastructure/Logging/LogEvents.json
```json
[
    {"id": 80001, "name": "WideAvailabilitySearchError", "level": "Warning", "source": "WideAvailabilitySearchService", "isException": false}
]
```

## travelgatex-channel

### HappyTravel.TravelgateXChannel.Api/Infrastructure/Logging/LogEvents.json
```json
﻿[
  {"id": 20001, "name": "ModelBindingFailure", "level": "Error", "source": "ClientRequestModelBinder", "isException": false},
  {"id": 20002, "name": "ErrorResponseReceived", "level": "Error", "source": "HttpClient", "isException": false},
  {"id": 20101, "name": "ClientCheckingFailure", "level": "Warning", "source": "ClientRequiredAttribute", "isException": false},
  {"id": 20102, "name": "AvailTimeoutReached", "level": "Warning", "source": "AvailabilityRequestExecutor", "isException": false},
  {"id": 20103, "name": "ParseOptionsFailure", "level": "Warning", "source": "OptionParametersConverter", "isException": false},
  {"id": 20201, "name": "ApiRequestSent", "level": "Information", "source": "HttpRequestSender", "isException": false},
  {"id": 20202, "name": "AvailabilityResultReturned", "level": "Information", "source": "AvailabilityRequestExecutor", "isException": false},
  {"id": 20202, "name": "ValuationResultReturned", "level": "Information", "source": "", "isException": false}
]
```

## rakuten-connector

### HappyTravel.RakutenConnector.Updater/Infrastructure/Logging/LogEvents.json
```json
[
    {"id": 30201, "name": "UpdatingRawPropertyCodes", "level": "Information", "source": "AccommodationUpdater", "isException": false}
]
```

### HappyTravel.RakutenConnector.Api/Infrastructure/Logging/LogEvents.json
```json
[
    {"id": 30001, "name": "RakutenRequestResult", "level": "Debug", "source": "RakutenShoppingClient", "isException": false},
    {"id": 30100, "name": "ApiBadRequest", "level": "Warning", "source": "RakutenShoppingClient", "isException": false},
    {"id": 30102, "name": "ApiRateExceeded", "level": "Warning", "source": "RakutenShoppingClient", "isException": false},
    {"id": 30103, "name": "ApiAuthorizationFailure", "level": "Critical", "source": "RakutenShoppingClient", "isException": false},
    {"id": 30104, "name": "ApiUnknownError", "level": "Critical", "source": "RakutenShoppingClient", "isException": false},
    {"id": 30105, "name": "ApiResponseDeserializationException", "level": "Critical", "source": "RakutenShoppingClient", "isException": true}
]
```

## osaka

### HappyTravel.Osaka.Api/Infrastructure/Logging/LogEvents.json
```json
[
    {"id": 2001, "name": "StartUploadingLocations", "level": "Information", "source": "LocationsManagementService", "isException": false},
    {"id": 2002, "name": "RemoveLocationsFromIndex", "level": "Information", "source": "LocationsManagementService", "isException": false},
    {"id": 2003, "name": "LocationsReceivedFromMapper", "level": "Information", "source": "LocationsManagementService", "isException": false},
    {"id": 2004, "name": "LocationsUploadedToIndex", "level": "Information", "source": "LocationsManagementService", "isException": false}, 
    {"id": 2005, "name": "CompleteUploadingLocations", "level": "Information", "source": "LocationsManagementService", "isException": false},
    {"id": 2006, "name": "UploadingError", "level": "Critical", "source": "LocationsManagementService", "isException": false},
    {"id": 2007, "name": "PredictionsQuery", "level": "Information", "source": "LocationsService", "isException": false}
]
```

## iwtx-connector

### HappyTravel.IwtxConnector.Common/Extensions/LogEvents.json
```json
[
  {"id": 20001, "name": "FillingDataException", "level": "Critical", "source": "", "isException": true},
  {"id": 20010, "name": "ClientRequestError", "level": "Error", "source": "", "isException": false},
  {"id": 20020, "name": "StaticDataUpdateException", "level": "Critical", "source": "UpdaterService", "isException": true},
  {"id": 20030, "name": "DataReadError", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 20031, "name": "DataWriteError", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 20041, "name": "IllusionsRequest", "level": "Debug", "source": "IllusionsClient", "isException": false},
  {"id": 20042, "name": "IllusionsResponse", "level": "Debug", "source": "IllusionsClient", "isException": false},
  {"id": 20050, "name": "StaticDataUpdateStart", "level": "Information", "source": "ProductionDataUpdaterService", "isException": false},
  {"id": 20051, "name": "StaticDataUpdateEnd", "level": "Information", "source": "ProductionDataUpdaterService", "isException": false},
  {"id": 20061, "name": "StaticDataUpdateCityProcessStart", "level": "Information", "source": "ProductionDataUpdaterService", "isException": false},
  {"id": 20062, "name": "StaticDataUpdateCityProcessEnd", "level": "Information", "source": "ProductionDataUpdaterService", "isException": false},
  {"id": 20070, "name": "SupplierInternalServerError", "level": "Critical", "source": "IllusionsClient", "isException": false },
  {"id": 20071, "name": "SupplierUnsuccessResponse", "level": "Information", "source": "IllusionsClient", "isException": false },
  {"id": 20072, "name": "SupplierResponseWithErrorMessage", "level": "Information", "source": "IllusionsClient", "isException": false },
  {"id": 20073, "name": "SupplierRequestException", "level": "Error", "source": "IllusionsClient", "isException": true },
  {"id": 20080, "name": "SupplierResponseDeserializationError", "level": "Error", "source": "IllusionsClient", "isException": false },
  {"id": 20090, "name": "GetHotelDetailsError", "level": "Error", "source": "HotelManager", "isException": false },
  {"id": 20091, "name": "GetHotelDetailsCodesMismatch", "level": "Error", "source": "HotelManager", "isException": false },
  {"id": 20092, "name": "GetEmptyCityCodeError", "level": "Error", "source": "HotelManager", "isException": false },
  {"id": 20100, "name": "SupplierDataProcessingError", "level": "Error", "source": "HotelManager", "isException": false }
]
```

## hiroshima

### HappyTravel.Hiroshima.DirectManager/Infrastructure/Logging/LogEvents.json
```json
﻿[
  {
    "id": 7001,
    "name": "BookingWebhookClientException",
    "level": "Error",
    "source": "BookingWebhookClient",
    "isException": true
  },
  {
    "id": 7002,
    "name": "InvitationCreated",
    "level": "Information",
    "source": "ManagerInvitationService",
    "isException": false
  },
  {
    "id": 7003,
    "name": "ManagerRegistrationFailed",
    "level": "Warning",
    "source": "ManagerRegistrationService",
    "isException": false
  },
  {
    "id": 7004,
    "name": "ManagerRegistrationSuccess",
    "level": "Information",
    "source": "ManagerRegistrationService",
    "isException": false
  }
]
```

## etg-connector

### HappyTravel.EtgConnector.Common/Infrastructure/Logging/LogEvents.json
```json
﻿[
  {"id": 5003, "name": "EtgWebhook", "level": "Information", "source": "BookingWebhookResponseService", "isException": false},
  {"id": 5020, "name": "BookingNotFoundInAudit", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5021, "name": "BookingNotFoundInCache", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5022, "name": "BookingNotFoundInOrderResponse", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5023, "name": "RoomContractSetNotFound", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5024, "name": "AccommodationDataNotFoundInCache", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5025, "name": "BookingReservationFailure", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5026, "name": "BookingFinalizationRequestFailure", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5027, "name": "BookingFinalizationFailure", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5028, "name": "BookingCancellationFailure", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5029, "name": "BookingOrderResponseFailure", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5030, "name": "IncorrectBookingOrderReceived", "level": "Error", "source": "BookingService", "isException": false},
  {"id": 5031, "name": "BookingOrderExists", "level": "Error", "source": "BookingService", "isException": false}
 ]
```

**Updated on: 4/23/2021 6:56:13 AM**
