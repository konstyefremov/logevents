## travelgatex-connector

### HappyTravel.TravelgateXConnector.Api/Controllers/AvailabilityController.cs
```json
﻿using System;
using System.Net;
using System.Threading;
using System.Threading.Tasks;
using CSharpFunctionalExtensions;
using HappyTravel.EdoContracts.Accommodations;
using HappyTravel.TravelgateXConnector.Api.Services;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace HappyTravel.TravelgateXConnector.Api.Controllers
{
    [ApiController]
    [ApiVersion("1.0")]
    [Route("api/{v:apiVersion}/accommodations")]
    [Produces("application/json")]
    public class AvailabilityController : BaseController
    {
        public AvailabilityController(IWideAvailabilitySearchService wideAvailabilitySearchService, 
            IRoomSelectionService roomSelectionService, IBookingEvaluationService bookingEvaluationService)
        {
            _wideAvailabilitySearchService = wideAvailabilitySearchService;
            _roomSelectionService = roomSelectionService;
            _bookingEvaluationService = bookingEvaluationService;
        }
        
        
        /// <summary>
        /// Searches for accommodations with available room contract sets. The 1st search step.
        /// </summary>
        /// <param name="request">Availability search request</param>
        /// <param name="cancellationToken">Cancellation token</param>
        /// <returns></returns>
        [HttpPost("availabilities")]
        [ProducesResponseType(typeof(Availability), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> Get([FromBody] AvailabilityRequest request, CancellationToken cancellationToken)
        {
            var (isSuccess, _, availability, error) = await _wideAvailabilitySearchService.Get(request, LanguageCode, cancellationToken);
            return isSuccess
                ? Ok(availability)
                : BadRequestWithProblemDetails(error);
        }


        /// <summary>
        /// Searches for specific accommodation with available room contract sets. The 2nd search step. 
        /// </summary>
        /// <param name="accommodationId">Supplier accommodation id</param>
        /// <param name="availabilityId">Availability Id</param>
        /// <param name="cancellationToken">Cancellation token</param>
        /// <returns></returns>
        [HttpPost("{accommodationId}/availabilities/{availabilityId}")]
        [ProducesResponseType(typeof(AccommodationAvailability), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> Get([FromRoute] string accommodationId, [FromRoute] string availabilityId, CancellationToken cancellationToken)
        {
            var (isSuccess, _, availability, error) = await _roomSelectionService.Get(availabilityId, accommodationId, cancellationToken);
            return isSuccess
                ? Ok(availability)
                : BadRequestWithProblemDetails(error);
        }


        /// <summary>
        /// Searches exact accommodation with specific room contract sets.
        /// </summary>
        /// <param name="availabilityId">Availability Id</param>
        /// <param name="roomContractSetId">Id of selected room contract set</param>
        /// <param name="cancellationToken"></param>
        /// <returns></returns>
        [HttpPost("availabilities/{availabilityId}/room-contract-sets/{roomContractSetId}")]
        [ProducesResponseType(typeof(RoomContractSetAvailability?), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> Get([FromRoute] string availabilityId, [FromRoute] Guid roomContractSetId, CancellationToken cancellationToken)
        {
            var (isSuccess, _, availability, error) = await _bookingEvaluationService.Evaluate(availabilityId, roomContractSetId, cancellationToken);
            return isSuccess
                ? Ok(availability)
                : BadRequestWithProblemDetails(error);
        }


        /// <summary>
        /// Method not supported. The deadline date is included into a response of the availability search.
        /// </summary>
        /// <param name="availabilityId"></param>
        /// <param name="roomContractSetId"></param>
        /// <param name="cancellationToken"></param>
        /// <returns></returns>
        [HttpGet("availabilities/{availabilityId}/room-contract-sets/{roomContractSetId}/deadline")]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public IActionResult GetDeadline([FromRoute] string availabilityId, [FromRoute] Guid roomContractSetId, CancellationToken cancellationToken)
        {
            return StatusCode((int) HttpStatusCode.MethodNotAllowed,
                "Our current implementation and settings shows deadlines on the first two steps");
        }


        private readonly IWideAvailabilitySearchService _wideAvailabilitySearchService;
        private readonly IRoomSelectionService _roomSelectionService;
        private readonly IBookingEvaluationService _bookingEvaluationService;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/appsettings.json
```json
{
  "AllowedHosts": "*",
  "Authority": {
    "Options": "travelgatex-connector/authority"
  },
  "Vault": {
    "Endpoint": "HTDC_VAULT_ENDPOINT",
    "Engine": "secrets",
    "Role": "travelgatex-connector",
    "Token": "HTDC_VAULT_TOKEN"
  },
  "TravelgateXOptions": "travelgatex-connector/options",
  "Jaeger": {
    "AgentHost": "JAEGER_AGENT_HOST",
    "AgentPort": "JAEGER_AGENT_PORT"
  },
  "Database": {
    "ConnectionString": "Server={0};Port={1};Database=travelgatex;Userid={2};Password={3};",
    "Options": "travelgatex-connector/connection-string"
  },
  "Fukuoka": {
    "Options": "fukuoka/options"
  },
  "Redis": {
    "Endpoint": "HTDC_REDIS_HOST"
  },
  "Sentry": {
    "Endpoint": "HTDC_TRAVELGATEX_SENTRY_ENDPOINT"
  },
  "CompanyInfo": {
    "Contacts": "travelgatex-connector/contacts"
  }
}
```

### HappyTravel.TravelgateXConnector.Api/Controllers/BaseController.cs
```json
﻿using System.Globalization;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace HappyTravel.TravelgateXConnector.Api.Controllers
{
    public class BaseController : ControllerBase
    {
        protected static string LanguageCode => CultureInfo.CurrentCulture.Name;
        
        protected IActionResult BadRequestWithProblemDetails(string details)
            => BadRequest(new ProblemDetails
            {
                Detail = details,
                Status = StatusCodes.Status400BadRequest
            });
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Amenity.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Amenity
    {
        public string Code { get; set; } = string.Empty;
        public string AmenityCodeSupplier { get; set; } = string.Empty;
        public ApplicationAreaTypes ApplicationAreaType { get; set; }
        public string Value { get; set; } = string.Empty;
        public string Texts { get; set; } = string.Empty;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Rule.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Rule
    {
        public string Id { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
        public MarkupRuleTypes MarkupRuleType { get; set; }
        public decimal Value { get; set; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Surcharge.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Surcharge
    {
        public string Code { get; set; } = string.Empty;
        public ChargeTypes ChargeType { get; set; }
        public bool Mandatory { get; set; }
        public Price Price { get; set; } = new();
        public string Description { get; set; } = string.Empty;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/MarkupRuleTypes.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public enum MarkupRuleTypes
    {
        PERCENT,
        IMPORT,
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/Extensions/AvailabilityRequestExtensions.cs
```json
﻿using System.Collections.Generic;
using System.Linq;
using HappyTravel.EdoContracts.Accommodations;
using HappyTravel.EdoContracts.Accommodations.Internals;
using HappyTravel.Money.Enums;

namespace HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions
{
    public static class AvailabilityRequestExtensions
    {
        public static Dictionary<string, object> ToSearchCriteria(this AvailabilityRequest request, List<string> accommodationIds)
        {
            return new()
            {
                {"checkIn", request.CheckInDate.ToTravelgateXFormat() },
                {"checkOut", request.CheckOutDate.ToTravelgateXFormat() },
                {"currency", Currencies.USD },
                {"nationality", request.Nationality },
                {"hotels", accommodationIds },
                {"occupancies", GenerateOccupancies(request.Rooms)}
            };
        }


        private static List<object> GenerateOccupancies(List<RoomOccupationRequest> rooms)
        {
            var result = new List<object>();

            foreach (var room in rooms)
            {
                var paxes = new List<object>();
                var adults = Enumerable.Range(1, room.AdultsNumber);
                paxes.AddRange(adults.Select(_ => new Dictionary<string, object>{{ "age", DefaultAdultAge }}));
                paxes.AddRange(room.ChildrenAges.Select(x => new Dictionary<string, object>{{ "age", x }}));
                
                var occupancy = new Dictionary<string, object>
                {
                    {"paxes", paxes}
                };
                
                result.Add(occupancy);
            }

            return result;
        }


        private const int DefaultAdultAge = 33;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/ChargeTypes.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public enum ChargeTypes
    {
        INCLUDE,
        EXCLUDE
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Promotion.cs
```json
﻿using System;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Promotion
    {
        public string Code { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
        public DateTime Start { get; set; }
        public DateTime End { get; set; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Room.cs
```json
﻿using System.Collections.Generic;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Room
    {
        public int OccupancyRefId { get; set; }
        public string LegacyRoomId { get; set; } = string.Empty;
        public string Code { get; set; } = string.Empty;
        public string SupplierCode { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public bool Refundable { get; set; }
        public int Units { get; set; }
        public RoomPrice RoomPrice { get; set; } = new();
        public List<Bed> Beds { get; set; } = new();
        public List<RatePlan> RatePlans { get; set; } = new();
        public List<Promotion> Promotions { get; set; } = new();
        public List<Surcharge> Surcharges { get; set; } = new();
        public List<Feature> Features { get; set; } = new();
        public List<Amenity> Amenities { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Data/HappyTravel.TravelgateXConnector.Data.csproj
```json
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net5.0</TargetFramework>
        <Nullable>enable</Nullable>
    </PropertyGroup>

    <ItemGroup>
      <PackageReference Include="HappyTravel.EdoContracts" Version="1.6.8" />
      <PackageReference Include="Microsoft.EntityFrameworkCore" Version="5.0.5" />
    </ItemGroup>

</Project>

```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Markup.cs
```json
﻿using System;
using System.Collections.Generic;
using HappyTravel.Money.Enums;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Markup
    {
        public string Channel { get; set; } = string.Empty;
        public Currencies Currency { get; set; }
        public bool Binding { get; set; }
        public decimal Net { get; set; }
        public decimal Gross { get; set; }
        public Exchange Exchange { get; set; } = new();
        public List<Rule> Rules { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Data/Models/Accommodation.cs
```json
﻿using NetTopologySuite.Geometries;

namespace HappyTravel.TravelgateXConnector.Data.Models
{
    public class Accommodation
    {
        public string Code { get; set; }
        public string Name { get; set; }
        public string Country { get; set; }
        public string Locality { get; set; }
        public Point Coordinates { get; set; } = new Point(0, 0);
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Exchange.cs
```json
﻿using HappyTravel.Money.Enums;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Exchange
    {
        public Currencies Currency { get; set; }
        public decimal Rate { get; set; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Services/WideAvailabilitySearchService.cs
```json
﻿using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;
using CSharpFunctionalExtensions;
using GraphQL;
using GraphQL.Client.Http;
using GraphQL.Client.Serializer.SystemTextJson;
using GraphQL.Query.Builder;
using HappyTravel.EdoContracts.Accommodations;
using HappyTravel.EdoContracts.GeoData.Enums;
using HappyTravel.TravelgateXConnector.Api.Infrastructure;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Options;
using HappyTravel.TravelgateXConnector.Api.Models.Tgx;
using HappyTravel.TravelgateXConnector.Data;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Options;
using NetTopologySuite.Geometries;
using Location = HappyTravel.EdoContracts.GeoData.Location;

namespace HappyTravel.TravelgateXConnector.Api.Services
{
    public class WideAvailabilitySearchService : IWideAvailabilitySearchService
    {
        public WideAvailabilitySearchService(IHttpClientFactory httpClientFactory, IOptions<TravelgateXRequestSettings> options, 
            GeometryFactory geometryFactory, TravelgateXContext context)
        {
            var httpClient = httpClientFactory.CreateClient(HttpClientNames.TgxClient);
            _client = new GraphQLHttpClient(new GraphQLHttpClientOptions(), new SystemTextJsonSerializer(), httpClient);
            _settings = options.Value;
            _geometryFactory = geometryFactory;
            _context = context;
        }
        
        
        public async Task<Result<Availability>> Get(AvailabilityRequest request, string languageCode, CancellationToken cancellationToken)
        {
            return await GetAccommodationCodes(request)
                .Bind(GetHotelResults)
                .Map(ToContracts);


            async Task<Result<HotelX>> GetHotelResults(List<string> accommodationIds)
            {
                var query = new Query<HotelX>("hotelX")
                    .AddField(
                        selector => selector.Search,
                        searchBuilder => searchBuilder
                            .AddArguments(new Dictionary<string, object>
                            {
                                {"criteria", request.ToSearchCriteria(accommodationIds)},
                                {"settings", _settings.ToDictionary()}
                            })
                            .AddField(search => search.Context)
                            .AddField(
                                search => search.Options, 
                                optionsBuilder => optionsBuilder
                                    .AddField(f => f.AccessCode)
                                    .AddField(f => f.HotelCode)
                                    .AddField(f => f.HotelName)
                                    .AddField(f => f.SupplierCode)
                                    .AddField(f => f.BoardCodeSupplier)
                                    .AddField(f => f.HotelCodeSupplier)
                            )
                    );
                
                var rq = new GraphQLRequest { Query = "{" + query.Build() + "}" };
                var rs = await _client.SendQueryAsync<HotelX>(rq, cancellationToken);
                return rs.Data;
            }


            Availability ToContracts(HotelX hotelX)
            {
                return default;
            }
        }
        
        
        private async Task<Result<List<string>>> GetAccommodationCodes(AvailabilityRequest request)
        {
            if (request.AccommodationIds.Any())
                return request.AccommodationIds;

            var location = request.Location.Value;
            return location.Type switch
            {
                LocationTypes.Accommodation => await GetAccommodationsByName(location) ?? await GetAccommodationsByLocation(location),
                LocationTypes.Location => await GetAccommodationsByLocation(location),
                LocationTypes.Destination => await GetAccommodationsByLocation(location),
                LocationTypes.Landmark => await GetAccommodationsByCoordinates(location) ?? await GetAccommodationsByLocation(location),
                _ => throw new ArgumentException($"Invalid location type", nameof(location))
            };
            
            
            async Task<List<string>?> GetAccommodationsByName(Location location)
            {
                var accommodations = await _context.Accommodations
                    .Where(ac => EF.Functions.ILike(ac.Name, $"%{location.Name}%"))
                    .Select(ac => ac.Code)
                    .ToListAsync();

                return accommodations.Any()
                    ? accommodations
                    : null;
            }
            
            
            async Task<List<string>?> GetAccommodationsByCoordinates(Location location)
            {
                var point = _geometryFactory.CreatePoint(new Coordinate(
                    location.Coordinates.Longitude,
                    location.Coordinates.Latitude)
                );

                var accommodations = await _context.Accommodations
                    .Where(ac => ac.Coordinates.Distance(point) <= location.Distance)
                    .Select(ac => ac.Code)
                    .ToListAsync();

                return accommodations.Any()
                    ? accommodations
                    : null;
            }
            
            
            Task<List<string>> GetAccommodationsByLocation(Location location)
            {
                return (from accommodation in _context.Accommodations
                    where EF.Functions.ILike(location.Name, accommodation.Country) ||
                          EF.Functions.ILike(location.Locality, accommodation.Locality)
                    select accommodation.Code).ToListAsync();
            }
        }


        private readonly GraphQLHttpClient _client;
        private readonly TravelgateXRequestSettings _settings;
        private readonly GeometryFactory _geometryFactory;
        private readonly TravelgateXContext _context;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Controllers/AccommodationsController.cs
```json
﻿using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Threading;
using System.Threading.Tasks;
using CSharpFunctionalExtensions;
using HappyTravel.EdoContracts.Accommodations;
using HappyTravel.TravelgateXConnector.Api.Services;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace HappyTravel.TravelgateXConnector.Api.Controllers
{
    [ApiController]
    [ApiVersion("1.0")]
    [Route("api/{v:apiVersion}/accommodations")]
    public class AccommodationsController : BaseController
    {
        public AccommodationsController(IAccommodationService accommodationService)
        {
            _accommodationService = accommodationService;
        }


        /// <summary>
        /// Returns an accommodation details.
        /// </summary>
        /// <param name="accommodationId">Accommodation Id</param>
        /// <param name="cancellationToken"></param>
        /// <returns>Accommodation details</returns>
        [HttpGet("{accommodationId}")]
        [ProducesResponseType(typeof(Accommodation), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> GetAccommodationDetails([Required] [FromRoute] string accommodationId, CancellationToken cancellationToken)
        {
            var (isSuccess, _, accommodation, error) = await _accommodationService.Get(accommodationId, cancellationToken);
            return isSuccess
                ? Ok(accommodation)
                : BadRequestWithProblemDetails(error);
        }


        /// <summary>
        /// Returns a list of accommodation details.
        /// </summary>
        /// <param name="skip">Skip</param>
        /// <param name="top">Top</param>
        /// <param name="modificationDate">Last modification date</param>
        /// <param name="cancellationToken"></param>
        /// <returns>List of accommodations</returns>
        [HttpGet]
        [ProducesResponseType(typeof(List<MultilingualAccommodation>), StatusCodes.Status200OK)]
        public async Task<IActionResult> Get([FromQuery] int skip = 0, [FromQuery] int top = 10, 
            [FromQuery(Name = "modification-date")] DateTime? modificationDate = null, CancellationToken cancellationToken = default)
        {
            return Ok(await _accommodationService.Get(skip, top, modificationDate, cancellationToken));
        }


        private readonly IAccommodationService _accommodationService;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/RoomPrice.cs
```json
﻿using System.Collections.Generic;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class RoomPrice
    {
        public Price Price { get; set; } = new();
        public List<PriceBreakdown> Breakdown { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/PriceBreakdown.cs
```json
﻿using System;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class PriceBreakdown
    {
        public DateTime Start { get; set; }
        public DateTime End { get; set; }
        public Price Price { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Controllers/BookingsController.cs
```json
﻿using System.Threading;
using System.Threading.Tasks;
using CSharpFunctionalExtensions;
using HappyTravel.EdoContracts.Accommodations;
using HappyTravel.TravelgateXConnector.Api.Services;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace HappyTravel.TravelgateXConnector.Api.Controllers
{
    [ApiController]
    [ApiVersion("1.0")]
    [Route("api/{v:apiVersion}/accommodations")]
    public class BookingsController : BaseController
    {
        public BookingsController(IBookingService bookingService)
        {
            _bookingService = bookingService;
        }
        
        
        /// <summary>
        /// Makes an asynchronous booking request to the supplier api.
        /// </summary>
        /// <param name="bookingRequest">Request</param>
        /// <param name="cancellationToken">Cancellation token</param>
        /// <returns></returns>
        [HttpPost("bookings")]
        [ProducesResponseType(typeof(Booking), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> Book([FromBody] BookingRequest bookingRequest, CancellationToken cancellationToken)
        {
            var (isSuccess, _, availability, error) = await _bookingService.Book(bookingRequest, cancellationToken);
            return isSuccess
                ? Ok(availability)
                : BadRequest(error);
        }


        /// <summary>
        /// Makes a cancellation request to the supplier api
        /// </summary>
        /// <param name="referenceCode">Booking reference code</param>
        /// <param name="cancellationToken">Cancellation token</param>
        /// <returns></returns>
        [HttpPost("bookings/{referenceCode}/cancel")]
        [ProducesResponseType(StatusCodes.Status204NoContent)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> Cancel(string referenceCode, CancellationToken cancellationToken)
        {
            var (isSuccess, _, error) = await _bookingService.Cancel(referenceCode, cancellationToken);
            return isSuccess
                ? Ok()
                : BadRequestWithProblemDetails(error);
        }


        /// <summary>
        /// Returns information about a booking order.
        /// </summary>
        /// <param name="referenceCode">Booking reference code</param>
        /// <param name="cancellationToken">Cancellation token</param>
        /// <returns></returns>
        [HttpGet("bookings/{referenceCode}")]
        [ProducesResponseType(typeof(Booking), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> GetDetails(string referenceCode, CancellationToken cancellationToken)
        {
            var (isSuccess, _, booking, error) = await _bookingService.Get(referenceCode, cancellationToken);
            return isSuccess
                ? Ok(booking)
                : BadRequestWithProblemDetails(error);
        }


        private readonly IBookingService _bookingService;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/Extensions/TravelgateXRequestSettingsExtensions.cs
```json
﻿using System.Collections.Generic;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Options;

namespace HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions
{
    public static class TravelgateXRequestSettingsExtensions
    {
        public static Dictionary<string, object> ToDictionary(this TravelgateXRequestSettings settings)
        {
            return new()
            {
                {"client", settings.Client},
                {"context", settings.Context},
                {"auditTransactions", settings.AuditTransactions},
                {"testMode", settings.TestMode},
                {"timeout", settings.Timeout},
            };
        }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Startup.cs
```json
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Reflection;
using System.Threading.Tasks;
using CacheFlow.Json.Extensions;
using FloxDc.CacheFlow;
using FloxDc.CacheFlow.Extensions;
using HappyTravel.ErrorHandling.Extensions;
using HappyTravel.StdOutLogger.Extensions;
using HappyTravel.TravelgateXConnector.Api.Infrastructure;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Environment;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions;
using HappyTravel.TravelgateXConnector.Api.Services;
using HappyTravel.TravelgateXConnector.Data;
using HappyTravel.VaultClient;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.AspNetCore.Localization;
using Microsoft.AspNetCore.Localization.Routing;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.OpenApi.Models;
using NetTopologySuite.Geometries;

namespace HappyTravel.TravelgateXConnector.Api
{
    public class Startup
    {
        public Startup(IConfiguration configuration, IHostEnvironment hostEnvironment)
        {
            Configuration = configuration;
            HostEnvironment = hostEnvironment;
        }
        

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            using var vaultClient = new VaultClient.VaultClient(new VaultOptions
            {
                BaseUrl = new Uri(EnvironmentVariableHelper.Get("Vault:Endpoint", Configuration)),
                Engine = Configuration["Vault:Engine"],
                Role = Configuration["Vault:Role"]
            });
            
            vaultClient.Login(EnvironmentVariableHelper.Get("Vault:Token", Configuration)).GetAwaiter().GetResult();
            
            services.AddResponseCompression()
                .AddCors()
                .AddLocalization()
                .AddMemoryCache()
                .AddMemoryFlow()
                .AddStackExchangeRedisCache(options => { options.Configuration = EnvironmentVariableHelper.Get("Redis:Endpoint", Configuration); })
                .AddDoubleFlow()
                .AddCacheFlowJsonSerialization()
                .AddTracing(HostEnvironment, Configuration);
            
            services.AddControllers();
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "HappyTravel.TravelgateXConnector.Api", Version = "v1" });
            });
            
            services.AddHealthChecks()
                .AddDbContextCheck<TravelgateXContext>()
                .AddRedis(EnvironmentVariableHelper.Get("Redis:Endpoint", Configuration));

            services.AddApiVersioning(options =>
            {
                options.AssumeDefaultVersionWhenUnspecified = false;
                options.DefaultApiVersion = new ApiVersion(1, 0);
                options.ReportApiVersions = true;
            });
            
            services.AddSwaggerGen(options =>
            {
                options.SwaggerDoc("v1.0", new OpenApiInfo {Title = "HappyTravel.com TravelgateX API", Version = "v1.0"});

                var xmlCommentsFileName = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
                var xmlCommentsFilePath = Path.Combine(AppContext.BaseDirectory, xmlCommentsFileName);
                options.IncludeXmlComments(xmlCommentsFilePath);
                options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    Description = "JWT Authorization header using the Bearer scheme. Example: \"Authorization: Bearer {token}\"",
                    Name = "Authorization",
                    In = ParameterLocation.Header,
                    Type = SecuritySchemeType.ApiKey
                });
                options.AddSecurityRequirement(new OpenApiSecurityRequirement()
                {
                    {
                        new OpenApiSecurityScheme
                        {
                            Reference = new OpenApiReference
                            {
                                Type = ReferenceType.SecurityScheme,
                                Id = "Bearer"
                            },
                            Scheme = "oauth2",
                            Name = "Bearer",
                            In = ParameterLocation.Header,
                        },
                        Array.Empty<string>()
                    }
                });
            });
            
            services.AddProblemDetailsErrorHandling();

            services.AddMvcCore()
                .AddAuthorization()
                .AddControllersAsServices()
                .AddFormatterMappings()
                .AddApiExplorer()
                .AddDataAnnotations()
                .AddNewtonsoftJson();
            
            services.AddOptions()
                .Configure<RequestLocalizationOptions>(o =>
                {
                    o.DefaultRequestCulture = new RequestCulture("en");
                    o.SupportedCultures = new[]
                    {
                        new CultureInfo("bg"), new CultureInfo("de"), new CultureInfo("el"), new CultureInfo("en"), new CultureInfo("es"),
                        new CultureInfo("fr"), new CultureInfo("bg"), new CultureInfo("de"), new CultureInfo("el"), new CultureInfo("en"),
                        new CultureInfo("es"), new CultureInfo("it")
                    };

                    o.RequestCultureProviders.Insert(0, new RouteDataRequestCultureProvider {Options = o});
                })
                .Configure<FlowOptions>(options =>
                {
                    options.DataLoggingLevel = DataLogLevel.Normal;
                    options.SuppressCacheExceptions = false;
                    options.CacheKeyDelimiter = "::";
                    options.CacheKeyPrefix = "HappyTravel::TravelgateX";
                });

           

            services
                .AddHttpClients(Configuration, vaultClient)
                .ConfigureDatabaseOptions(Configuration, vaultClient)
                .ConfigureAuthentication(Configuration, vaultClient)
                .ConfigureTravelgateRequestSettings(Configuration, vaultClient)
                .AddTransient<GeometryFactory>()
                .AddTransient<IAccommodationService, AccommodationService>()
                .AddTransient<IWideAvailabilitySearchService, WideAvailabilitySearchService>()
                .AddTransient<IRoomSelectionService, RoomSelectionService>()
                .AddTransient<IBookingEvaluationService, BookingEvaluationService>()
                .AddTransient<IBookingService, BookingService>();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILoggerFactory loggerFactory)
        {
            var logger = loggerFactory.CreateLogger<Startup>();
            app.UseProblemDetailsExceptionHandler(env, logger);
            app.UseHttpsRedirection();
            app.UseHealthChecks("/health");
            app.Use(async (context, next) =>
            {
                if (context.Request.Path.StartsWithSegments("/robots.txt"))
                {
                    context.Response.ContentType = "text/plain";
                    await context.Response.WriteAsync("User-agent: * \nDisallow: /");
                }
                else
                {
                    await next();
                }
            });
            
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseSwagger();
                app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "HappyTravel.TravelgateXConnector.Api v1"));
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
            
            app.UseHttpContextLogging(options => options.IgnoredPaths = new HashSet<string> {"/health"});

            app.UseSwagger()
                .UseSwaggerUI(options =>
                {
                    options.SwaggerEndpoint("/swagger/v1.0/swagger.json", "HappyTravel.com TravelgateX Connector API");
                    options.RoutePrefix = string.Empty;
                });
        }
        
        
        private IConfiguration Configuration { get; }
        private IHostEnvironment HostEnvironment { get; }
    }
}

```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/PaymentTypes.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public enum PaymentTypes
    {
        MERCHANT,
        DIRECT,
        CARD_BOOKING,
        CARD_CHECK_IN,
        PAYX
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/ApplicationAreaTypes.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public enum ApplicationAreaTypes
    {
        HOTEL,
        ROOM,
        SERVICE,
        GENERAL,
    }
}
```

### HappyTravel.TravelgateXConnector.Api/HappyTravel.TravelgateXConnector.Api.csproj
```json
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net5.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
    <NoWarn>1701;1702;1591</NoWarn>
    <DocumentationFile>..\HappyTravel.RakutenConnector.Api\HappyTravel.TravelgateXConnector.Api.xml</DocumentationFile>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="AspNetCore.HealthChecks.Redis" Version="5.0.2" />
    <PackageReference Include="CSharpFunctionalExtensions" Version="2.15.1" />
    <PackageReference Include="FloxDc.CacheFlow" Version="1.9.1" />
    <PackageReference Include="FloxDc.CacheFlow.Json" Version="1.9.1" />
    <PackageReference Include="GraphQL.Client" Version="3.2.3" />
    <PackageReference Include="GraphQL.Client.Serializer.SystemTextJson" Version="3.2.3" />
    <PackageReference Include="GraphQL.Query.Builder" Version="1.2.1" />
    <PackageReference Include="HappyTravel.EdoContracts" Version="1.6.8" />
    <PackageReference Include="HappyTravel.ErrorHandling" Version="1.2.0" />
    <PackageReference Include="HappyTravel.StdOutLogger" Version="1.0.19" />
    <PackageReference Include="HappyTravel.VaultClient" Version="1.1.0" />
    <PackageReference Include="IdentityModel" Version="5.1.0" />
    <PackageReference Include="IdentityServer4.AccessTokenValidation" Version="3.0.1" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="5.0.5" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Versioning" Version="5.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="5.0.5" />
    <PackageReference Include="Microsoft.Extensions.Caching.StackExchangeRedis" Version="5.0.1" />
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="5.0.1" />
    <PackageReference Include="Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore" Version="5.0.5" />
    <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="5.0.5" />
    <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite" Version="5.0.5" />
    <PackageReference Include="OpenTelemetry" Version="1.1.0-beta1" />
    <PackageReference Include="OpenTelemetry.Exporter.Jaeger" Version="1.1.0-beta1" />
    <PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.0.0-rc3" />
    <PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.0.0-rc3" />
    <PackageReference Include="OpenTelemetry.Instrumentation.Http" Version="1.0.0-rc3" />
    <PackageReference Include="OpenTelemetry.Instrumentation.StackExchangeRedis" Version="1.0.0-rc3" />
    <PackageReference Include="Sentry.AspNetCore" Version="3.3.1" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.1.3" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\HappyTravel.TravelgateXConnector.Data\HappyTravel.TravelgateXConnector.Data.csproj" />
  </ItemGroup>

</Project>

```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/Extensions/ServiceCollectionExtensions.cs
```json
﻿using System;
using System.Net;
using System.Net.Http;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Environment;
using HappyTravel.TravelgateXConnector.Api.Infrastructure.Options;
using HappyTravel.TravelgateXConnector.Data;
using HappyTravel.VaultClient;
using IdentityServer4.AccessTokenValidation;
using Microsoft.AspNetCore.Builder;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using StackExchange.Redis;

namespace HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions
{
    public static class ServiceCollectionExtensions
    {
        public static IServiceCollection AddTracing(this IServiceCollection services, IHostEnvironment environment, IConfiguration configuration)
        {
            string agentHost;
            int agentPort;
            if (environment.IsLocal())
            {
                agentHost = configuration["Jaeger:AgentHost"];
                agentPort = int.Parse(configuration["Jaeger:AgentPort"]);
            }
            else
            {
                agentHost = EnvironmentVariableHelper.Get("Jaeger:AgentHost", configuration);
                agentPort = int.Parse(EnvironmentVariableHelper.Get("Jaeger:AgentPort", configuration));
            }

            var connection = ConnectionMultiplexer.Connect(EnvironmentVariableHelper.Get("Redis:Endpoint", configuration));

            services.AddOpenTelemetryTracing(builder =>
            {
                builder.AddAspNetCoreInstrumentation()
                    .AddHttpClientInstrumentation()
                    .AddRedisInstrumentation(connection)
                    .AddJaegerExporter(options =>
                    {
                        options.AgentHost = agentHost;
                        options.AgentPort = agentPort;
                    })
                    .SetResourceBuilder(ResourceBuilder.CreateDefault())
                    .SetSampler(new AlwaysOnSampler());
            });

            return services;
        }


        public static IServiceCollection ConfigureDatabaseOptions(this IServiceCollection services,
            IConfiguration configuration, VaultClient.VaultClient vaultClient)
        {
            var databaseOptions = vaultClient.Get(configuration["Database:Options"]).GetAwaiter().GetResult();
            
            return services.AddDbContextPool<TravelgateXContext>(options =>
            {
                var host = databaseOptions["host"];
                var port = databaseOptions["port"];
                var password = databaseOptions["password"];
                var userId = databaseOptions["userId"];
            
                var connectionString = configuration["Database:ConnectionString"];
                options.UseNpgsql(string.Format(connectionString, host, port, userId, password), builder =>
                {
                    builder.UseNetTopologySuite();
                    builder.EnableRetryOnFailure();
                });
                options.UseInternalServiceProvider(null);
                options.EnableSensitiveDataLogging(false);
                options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
            }, 16);
        }
        
        
        public static IServiceCollection ConfigureAuthentication(this IServiceCollection services, IConfiguration configuration, IVaultClient vaultClient)
        {
            var authorityOptions = vaultClient.Get(configuration["Authority:Options"]).GetAwaiter().GetResult();
            
            services.AddAuthentication(IdentityServerAuthenticationDefaults.AuthenticationScheme)
                .AddIdentityServerAuthentication(options =>
                {
                    options.Authority = authorityOptions["authorityUrl"];
                    options.ApiName = authorityOptions["apiName"];
                    options.RequireHttpsMetadata = true;
                    options.SupportedTokens = SupportedTokens.Jwt;
                });

            return services;
        }


        public static IServiceCollection ConfigureTravelgateRequestSettings(this IServiceCollection services, IConfiguration configuration, IVaultClient vaultClient)
        {
            var secrets = vaultClient.Get(configuration["TravelgateXOptions"]).GetAwaiter().GetResult();
            
            return services.Configure<TravelgateXRequestSettings>(options =>
            {
                options.Client = secrets["client"];
                options.Context = secrets["context"];
                options.Timeout = 25000;
                options.AuditTransactions = false;
                options.TestMode = true;
            });
        }


        public static IServiceCollection AddHttpClients(this IServiceCollection services, IConfiguration configuration, IVaultClient vaultClient)
        {
            var secrets = vaultClient.Get(configuration["TravelgateXOptions"]).GetAwaiter().GetResult();
            
            services.AddHttpClient(HttpClientNames.TgxClient, client =>
            {
                client.BaseAddress = new Uri("https://api.travelgatex.com/");
                client.DefaultRequestHeaders.Add("Authorization", $"ApiKey {secrets["token"]}");
            }).ConfigurePrimaryHttpMessageHandler(_ => new HttpClientHandler
            {
                AutomaticDecompression = DecompressionMethods.Deflate | DecompressionMethods.GZip
            });

             return services;
        }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/Options/TravelgateXRequestSettings.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Infrastructure.Options
{
    public class TravelgateXRequestSettings
    {
        public string Client { get; set; } = string.Empty;
        public string Context { get; set; } = string.Empty;
        public bool AuditTransactions { get; set; } = false;
        public bool TestMode { get; set; } = true;
        public int Timeout { get; set; } = 25000;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Occupancy.cs
```json
﻿using System.Collections.Generic;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public readonly struct Occupancy
    {
        public List<Pax> Paxes { get; init; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/Extensions/DateTimeExtensions.cs
```json
﻿using System;

namespace HappyTravel.TravelgateXConnector.Api.Infrastructure.Extensions
{
    public static class DateTimeExtensions
    {
        public static string ToTravelgateXFormat(this DateTime dateTime)
        {
            return dateTime.ToString("yyyy-MM-dd");
        }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Infrastructure/HttpClientNames.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Infrastructure
{
    public static class HttpClientNames
    {
        public const string TgxClient = "travelgatex-client";
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/StatusTypes.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public enum StatusTypes
    {
        OK,
        RQ,
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Search.cs
```json
﻿using Newtonsoft.Json;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Search
    {
        [JsonProperty("context")]
        public string Context { get; init; } = string.Empty;
        [JsonProperty("options")]
        public HotelOptionSearch Options { get; init; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Data/TravelgateXContext.cs
```json
﻿using HappyTravel.TravelgateXConnector.Data.Models;
using Microsoft.EntityFrameworkCore;

namespace HappyTravel.TravelgateXConnector.Data
{
    public class TravelgateXContext : DbContext
    {
        public TravelgateXContext(DbContextOptions<TravelgateXContext> options) : base(options)
        { }
        
        
        public DbSet<Accommodation> Accommodations { get; set; }


        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Accommodation>(b =>
            {
                b.HasKey(a => a.Code);
            });
            
            
            base.OnModelCreating(modelBuilder);
        }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/HotelOptionSearch.cs
```json
﻿using System.Collections.Generic;
using Newtonsoft.Json;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class HotelOptionSearch
    {
        [JsonProperty("supplierCode")]
        public string SupplierCode { get; init; } = string.Empty;
        [JsonProperty("accessCode")]
        public string AccessCode { get; init; } = string.Empty;
        [JsonProperty("hotelCode")]
        public string HotelCode { get; init; } = string.Empty;
        [JsonProperty("hotelCodeSupplier")]
        public string HotelCodeSupplier { get; init; } = string.Empty;
        [JsonProperty("hotelName")]
        public string HotelName { get; init; } = string.Empty;
        [JsonProperty("boardCodeSupplier")]
        public string BoardCodeSupplier { get; init; } = string.Empty;
        public PaymentTypes PaymentType { get; init; }
        public StatusTypes StatusType { get; init; }
        public List<Occupancy> Occupancies { get; init; } = new();
        public List<Room> Rooms { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/HotelX.cs
```json
﻿using Newtonsoft.Json;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class HotelX
    {
        [JsonProperty("search")]
        public Search Search { get; init; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Pax.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public readonly struct Pax
    {
        public int Age { get; init; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Feature.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Feature
    {
        public string Code { get; set; } = string.Empty;
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Bed.cs
```json
﻿namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Bed
    {
        public string Type { get; set; } = string.Empty;
        public int Count { get; set; }
        public bool Shared { get; set; }
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/Price.cs
```json
﻿using System.Collections.Generic;
using HappyTravel.Money.Enums;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class Price
    {
        public Currencies Currency { get; set; }
        public bool Binding { get; set; }
        public decimal Net { get; set; }
        public decimal Gross { get; set; }
        public Exchange Exchange { get; set; } = new();
        public List<Markup> Markups { get; set; } = new();
    }
}
```

### HappyTravel.TravelgateXConnector.Api/Models/Tgx/RatePlan.cs
```json
﻿using System;

namespace HappyTravel.TravelgateXConnector.Api.Models.Tgx
{
    public class RatePlan
    {
        public string Code { get; set; } = string.Empty;
        public string SupplierCode { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
        public DateTime Start { get; set; }
        public DateTime End { get; set; }
    }
}
```

## odawara

### HappyTravel.Odawara.Api/Pages/Account/Login.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@using HappyTravel.Odawara.Api.Services
@inject IViewLocalizer Localizer
@model Account.LoginModel

<div class="small">
    @if (HolidayService.IsHolidayOrEve(DateTime.Today))
    {
        <div class="picture">
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
            <div class="snow"></div>
        </div>
    }
    <form method="post">
        <h2>
            @Localizer["Sign In"]
        </h2>
        <div class="paragraph">
            <div>@Localizer["Don`t have an Account?"] <a asp-page="./Register" asp-route-returnUrl="@Model.ReturnUrl" class="link">@Localizer["Create a new one"]</a></div>
        </div>
        <div class="form">
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span>@Localizer["Login"]</span>
                    </div>
                    <div class="auth-input">
                        <input placeholder="@Localizer["Please enter your Login"]" asp-for="Input.UserName" />
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span>@Localizer["Password"]</span>
                    </div>
                    <div class="auth-input">
                        <input placeholder="@Localizer["Please enter your Password"]" asp-for="Input.Password" />
                    </div>
                </div>
            </div>
            <div class="paragraph action">
                <a asp-page="./ForgotPassword" asp-route-returnUrl="@Model.ReturnUrl" class="link">@Localizer["Forgot Password?"]</a>
            </div>
            <div class="paragraph action">
                <a class="link" asp-page="./ResendEmailConfirmation">@Localizer["Resend confirmation email"]</a>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">@Localizer["Sign In"]</button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>
```

### HappyTravel.Odawara.Api/wwwroot/images/logo/logo-small-gray.png
```json
�PNG

   IHDR   �   k   2���   	pHYs  %  %IR$�   sRGB ���   gAMA  ���a  �IDATx�]v�ƒ����s�!�l t��1����,`"g��\IY��<��ـ�, �g���|�<\3+��t�D��V5�?�H$�;aH�@��P��x|v��p8�A����pd�@SG��D' ���3����898q8rp�p�����A�8+
:q8l ����p���L,��k"gb9Y(��8�!џlbi�A
�8�mp8)�F� N�8i�󽖂�s�#E�;�|�4��a#��(��'�?��SG���HE�t7���Hb\# :�w�p8��D&?��Emp87�[$ �H��q�	����3��O0.q�(x�ܮ@��ҹ
G���h�7�3��I?k./�%�����8L�=8��G��]>\�A�Cp8�5�]�k	5�H��F�/��_oU4yG��PO��m��<{��I[T�q�9|�d��Ki��iSw ��cP�>��q# Ap����?w0�����f�������� ?�dg�y�� 	����r�/��:9y����;u���:���G�������u�����ښB�*�z[uޏϣ��<�l��r=�ݯ��h���k;���E�}��x�Z�m�k�&u�P����+�
k�}��Pp�Sֵ��\Z:�?c�G��y7uX����M�w�����EP�G�N'H���Y��l���6_ѡm�������k��.�{;o�y�V�6n�py�QJ@Z'g������vn���<@���?���˔����ۺ�����8�2yߏO��w�Ts8�p��o��m �[?����[$f?�G��n-Qd�ʸ^F8L[K�	E[�?/�
G|y0��W4F�p�6���I�i������%}�%�C{���l툠&�;k?�a�1���P�����&7�W�0B��o��;"%����r�<y̠ǉ�Ԡ_�7`�ܑ�_|/�7�i���fkM�f9��TN�MCcF¼����}b��U.$�O����
w�;'�l����@���b������Iw䵗:7kܕŇ餭b�_`��{��D��>��y���I�����\#=EP;�\�$�jA���g���CV��h7���5��S~\~�H�1Ҳu�a����5��5�4�����i����K~m�h��m/�BoA�d!�ٮ[�(����=�G>v���"D|^"�;7)<L�����, �cl�&% ��Ν	��-h	c���ȸ���Z��/e�ē������[�ö|F�n��g盺SޏH"�{���b����7�����@�O[�m���@�Y��i���������Ý�1bF��o�BֺZd���W�?|	����D�9������|��o�ӕŁ�#3K�7F$�2�4�f��H7����
�Dy���Oy��ԁ�*�����pW��ؑ�f�#Sk�o"3�0j�h�z�y������Y�r�9��G�rʸX��?�D���Ғ4,�cr 94��a�DaA�;�Ӣ���Ý�(£��a6�Ě�I��`�a�������p�&�$՝ �au�Y�_��߬��f߶�^�W�{p'�ᒒ�]��ԓ�n0(�_&Kο���/r��W�6Yr@�!k���̤ސoҗ�vE��|c5���T>�Ue�K��{�>�>��J�e�i�}Hi9
�k
t:k�v%�؀y�-6�JȬ�.�,�*y�K��!fP~��~�$���!<m(�w�i�� �Uo�����4���J�B���UL��Q4��Գl�B��ĺ� �mVf��i�fE�̀�,l��D����AH��<� �աO �Q4S��5��) ҉�:�D��	��]��'�ɼ�oaH(�g�ms΢;��X�sG��Hr�@��5��Q^�����ߠX��5���l��^r��8��D��@>�m7�-�u��t(F�7B��n35��	��9Q��&w�^6�n4G�6ۖa)>0�ki�����{�l )�cr{�q㮮ͼ������Q��I�J�EP��z۞�6uC^�ܳe凥��R����%���vA�;�"��c%�%2�3��Yz��y��j�4F(l7!�Y��^F)��y�r# ���<;W�H�@ڕ�P�g�Qhl���|n�qH'����s�_
�x�ϐ_�|A����X�s�Iw��v*�(�^ݐ��-���b&�.r~�-����2�X�~-y�珿�F������(��5�㮺����V��^�?t�I���ɳ���`:�Vg�wr��U^'�����-�{����O�R�9];9=?���a�.� [��Y�ky���P���J��t�]��qW���^J�:9�S85��dz���_JI1�Bt
۔[��ݜF��c����,\_��y-��Ȝr�f"k�[����B��������|��R��W��(��e	q��tտ��܋W9I�c� ����a��G�v|���e�QR8��.k �-�=�r&ts ��GU!Vg�ȣ�Rk(�{Ĕ�h7��ڸ)�y�=���h���E�%��B�]4�<h��>H��E�{ϫ_�O�d���B����=�U
����N��2K��?���� |Ԗ�wy~1R��U�/Pad\��iq|v��(��H{a�?�I!�b�҅֔����'y�+�����6��2�9�;� "� -�G�L�?�~�j"~ٳa�v�A *�vtz�dV��RkJ��ϡ�%I��
��@�.���E;�Dc��
0���3��������D��e���+3[�Q1����ĺF�zO�✱m�Ϛ�m��Զ��)|}�ծu�)�a�/|/W�o.G���U���R�5�L�86"����'���>� /�����Q!���dN
���ۈ�0�Tj7���^�@��tw��np�xzs�S�8�a�V�έ$�x��w��$!�o㎜E�an�>�A�/�4�m�,�C����
��j��ҦӇ��ܔ��U��o�#�ۮ�}�Mng���_�L��#�R�:�\|��w\�9�Z@�%�'��~���|NU��2�4��pEc�6�bf��B}XXq]���+K��E��S�,}�̬,ob fX�J��*(HM�IԻ"������R����b뀶Y�|=���i�rk1�ͳe���3Ib�g/?���M�8���ˮ�n-B��w%	��G�ȖJ(�SK���1���H�H#�h�a׶�̘�;])��gVj���{�]���R�LL#ڋ���)�g�V��_Nr;'�y�q��$��R+�3��rr��$�/��?�P���,�� �_~�U�$�_�$v�=lp�����h���u��Z!�R;H��ȸ+�Fwe{��O�'�m��y"��G֪�!�nۙ��܂N�#b4&VI N�TlJ� 
|fN"E��{v|��ړ�Q�yW�j�v�H^�f��fم�BK&fɚ�rg�?�d-(��l�Tj:}���:z�]eM��(���HE��e��$Gk��`?��MU.�E��j�,F* ��Ί?�R	�2,��b��a�G���cD e� ��\��C�R;dh��zW�r�7�{�r��@U����������Ͼ�W��)E:{�P��H�Y!6϶�ݽ�1v'|��G*�����Z�[g�;)��+�G���Eic(]$0�)�:������_ml,�X���W
��̊�9k�ȧ�u������N�b��<-K,e��D���5y����#!�Ƶ��	��4��2�f@�~��Vc�A-l$7jYT����<ֻ"�=cՙf��h����DR���OP��9���h�;)�򪞹L�7j�<��}�$lo*��J�X��!g.I�Pë��J�s�&C�Y��"��{�����q|�^����ڎ�j�ZY~4���ؽ.�D��g)��j��]إ~X�є�l5����JUj�B�z��}���Õ��$E�����k� �zT(�b2�x'�;	K��gEH��dT�������H�Y� � ����Lȷ��93�2@��Z�����ԗܹ��t��u���\�~����R73Bw<+w����	!�U�c��@C�.�˼�Wn�Hb^����,�9��/��릹6�/�hO��߽B,��ޞt�J$
�X8��� ��c�x]yt0���7�:�y�E�Y�*�(�k/���?+t|߫-��������
;�8�2Ŭ]?9& g~fF��)8="�!_!k�~�:9/�{��y�/�L�		�0���ITp���t+�7����S�_�6˅v��ϯ�T�7mǕ��:�%J3A�&6v`�Bb��Nh�v)mݯ��"$�:�����߳Q�*� �Y� ����/S�>f�J�&�\�~ʄ�CI-��OE}�D�,X"C�<�������68ujzw��4a��ޫ���_���	]/Y����K\*t�� Ë�i �cΨ<���h�[dߑ��[:�ټ�a���bv(����t8C������:}?�������P)����c,D�r�8�p*' �L��囙���(�8���۬a?.*9��'��4L�t��D���	�PIҋ�Kf���6*/ �Y䒼W��Ĥ�������d܋���&�[#qл��z�C�
�gy7�-�M��O��A+QN��� ����N�L�5���6��N���|��3E�FS- 1&�E_Dr�讕b�vL93! �z����5Q؜
:;UG5
fF@b��LI�Q�N�������'(w��uS�g�1��e<̺`�̼��8A����x��fJe17	J�撍��0���jwV��̝��b�x��N��ama��F�z�ͨ<�Z@bL���Q}��&Ȝp�����y�6��$�M0�'�]�mvM�x��b3�VpV*��5�,�Y��:"��@�]:��j͔Ǚ�!�#~��Gv��@��	�-�e��y+R	]�_!����f4��E��&�w ����q2b��,���WM�9�VAQ���َh��0QǗ���ڄ�'��?j�B(�}:AN@&ȵ0Af����'˿ ��\-��6    IEND�B`�
```

### HappyTravel.Odawara.Api/wwwroot/images/favicon.ico
```json
         h     (                                             ��  �� �� ��O ��p ��p ��O �� �� ��                  ��  ��  �� ��t ��� ��� ��� ��� ��� ��� ��t �� ��  ��      ��  ��  �� ���!���C���C���T���@���J���Y���%��� ��� �� ��  ��  ��  �� ��� ���T���������������������������N��� ������ �� ��  ��  ��[���%���U���{���g���s���K���`���1���;���)��������[��  ��	 ���N�������������������������������K���d���~���[���3��� ��	 ��$ ���0���A���-���8���5���1���:���>���L���������������Y���5��$ ��9 ��� ��� ��� ��� ��� ��� ���	���3���]���������������i���:��9 ��9 ��� ��� ��� ��� ��� ��� ���	���<���m���������������P���.��9 ��$ ��� ��� ��� ��� ��� ��� ���������D���_���m���S���0���#��$ ��	 ��� ��� ��� ��� ��� ��� ��� ���������*���1���%��������	 ��  ��[ ��� ��� ��� ��� ��� ��� ��� ���������������	��[��  ��  �� ��� ��� ��� ��� ��� ��� ��� ��� ��� ��� ��� ��� �� ��  ��  ��  �� ��� ��� ��� ��� ��� ��� ��� ��� ��� ��� �� ��  ��      ��  ��  �� ��t ��� ��� ��� ��� ��� ��� ��t �� ��  ��                  ��  �� �� ��O ��p ��p ��O �� �� ��             �  �  �  �  �                          �  �  �  �  �  
```

### HappyTravel.Odawara.Api/Pages/Account/Logout.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.LogoutModel

<div class="small">
    <h2>
        @Localizer["You were successfully logged out!"]
    </h2>
    <div class="paragraph">
        <a href="https://happytravel.com" class="link">@Localizer["Go back to Happytravel.com"]</a>
    </div>
</div>

```

### HappyTravel.Odawara.Api/Pages/Account/Register.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.RegisterModel

<script src="~/js/password.js"></script>

<div>
    <div class="breadcrumbs">
        <div class="links">
            <a href="@Model.BackToLoginUrl">@Localizer["Sign In"]</a>
            <span class="small-arrow-right"></span>
            @Localizer["Registration"]
        </div>
    </div>
    <h2>
        @Localizer["Get started with your new account"]
    </h2>
    <div class="paragraph">
        <div>@Localizer["Create a Happytravel.com account and start booking"]</div>
        @Localizer["Already have an account?"] <a href="@Model.BackToLoginUrl" class="link">@Localizer["Sign In here"]</a>
    </div>
    <div class="accent-frame">
        <div class="data only">
            <div><b>@Localizer["Please keep in mind we don't store your credentials"]</b></div>
            @Localizer["You must remember them or use a password management software"]
        </div>
    </div>
    <form method="post">
        <div class="form">
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span class="required">
                            @Localizer["Email"]
                        </span>
                    </div>
                    <div class="auth-input">
                        <input placeholder="@Localizer["Please enter your Email"]" asp-for="RegisterInput.Email" />
                        <span class="error-field" asp-validation-for="RegisterInput.Email"></span>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span class="required">
                            @Localizer["Login"]
                        </span>
                    </div>
                    <div class="auth-input">
                        <input placeholder="@Localizer["Please enter your Login"]" asp-for="RegisterInput.UserName" />
                        <span class="error-field" asp-validation-for="RegisterInput.UserName"></span>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="label with-actions">
                        <span class="required">
                            @Localizer["Password"]
                        </span>
                        <span>
                            <a onclick="generatePassword()" class="link" id="generate-link">@Localizer["Generate Password"]</a>
                            &#8226
                            <a onclick="copyToClipboard()" class="link" id="copy-link">@Localizer["Copy to Clipboard"]</a>
                        </span>
                    </div>
                    <div class="auth-input">
                        <input placeholder="@Localizer["Choose your Password"]" asp-for="RegisterInput.Password" id="password" onkeyup="validateComplexity(this)">
                        <span class="error-field" asp-validation-for="RegisterInput.Password"></span>
                    </div>
                    <div class="password-rules">
                        <div id="password-rules">
                            <div class="caption-wrapper">
                                <div class="caption"></div>
                            </div>
                            <div class="title">@Localizer["Password Complexity"]:</div>
                            <div class="columns">
                                <div class="rule-lowercase">
                                    <span>&#8226</span> @Localizer["A lowercase Latin character"]
                                </div>
                                <div class="rule-uppercase">
                                    <span>&#8226</span> @Localizer["An uppercase Latin character"]
                                </div>
                                <div class="rule-number">
                                    <span>&#8226</span> @Localizer["A digit"]
                                </div>
                                <div class="rule-symbol">
                                    <span>&#8226</span> @Localizer["A special character"]
                                </div>
                                <div class="rule-length">
                                    <span>&#8226</span> @Localizer["10 characters minimum"]
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">@Localizer["Continue Registration"]</button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>

```

### HappyTravel.Odawara.Api/Pages/Account/Login.cshtml.cs
```json
using System;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;
using HappyTravel.Odawara.Data.Model;
using HappyTravel.Odawara.Api.Infrastructure;
using IdentityServer4.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Localization;

namespace HappyTravel.Odawara.Api.Pages.Account
{
    public class LoginModel : PageModel
    {
        public LoginModel(UserManager<User> userManager, SignInManager<User> signInManager, IIdentityServerInteractionService interaction,
            IStringLocalizer<LoginModel> localizer)
        {
            _interaction = interaction;
            _localizer = localizer;
            _userManager = userManager;
            _signInManager = signInManager;
        }


        [BindProperty]
        public LoginInputModel Input { get; set; }

        public string ReturnUrl { get; set; }


        public async Task<IActionResult> OnGetAsync(string returnUrl = null)
        {
            // Clear current user authorization if exists - new login requested
            await _signInManager.SignOutAsync();
            ReturnUrl = returnUrl;

            var isCustomRedirectUrlDefined = TryGetCustomReturnUrl(returnUrl, out var customReturnUrl);
            if (isCustomRedirectUrlDefined)
                return RedirectToPage("./Login", new {returnUrl = customReturnUrl});

            if (NeedRedirectToRegistration(returnUrl, out var changedReturnUrl))
                return RedirectToPage("./Register", new {returnUrl = changedReturnUrl});

            return Page();
        }


        private static bool TryGetCustomReturnUrl(string returnUrl, out string editedReturnUrl)
        {
            var customReturnUrl = ReturnUrlHelper
                .GetAndRemoveQueryParameter(returnUrl, "customRedirectUrl", out editedReturnUrl);

            if (customReturnUrl == null)
                return false;

            editedReturnUrl = ReturnUrlHelper.SetQueryParameter(editedReturnUrl, "redirect_uri", customReturnUrl);
            return true;
        }


        private static bool NeedRedirectToRegistration(string returnUrl, out string editedReturnUrl)
        {
            var redirectToRegister = ReturnUrlHelper
                .GetAndRemoveQueryParameter(returnUrl, "redirectToRegister", out editedReturnUrl);

            return bool.TrueString.Equals(redirectToRegister, StringComparison.OrdinalIgnoreCase);
        }


        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            ReturnUrl = returnUrl;
            if (!ModelState.IsValid)
                return Page();

            var user = await _userManager.FindByNameAsync(Input.UserName);
            if (user == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect Login or Password"]);
                return Page();
            }

            if (!user.EmailConfirmed)
                return RedirectToPage("Confirm", new
                {
                    returnUrl,
                    userId = user.Id
                });

            var signInResult = await _signInManager.PasswordSignInAsync(user, Input.Password, false, false);
            if (!signInResult.Succeeded)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect Login or Password"]);
                return Page();
            }

            var context = await _interaction.GetAuthorizationContextAsync(returnUrl);
            if (context == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Wrong query parameters in URL"]);
                return Page();
            }

            return LocalRedirect(returnUrl);
        }


        private readonly IIdentityServerInteractionService _interaction;
        private readonly IStringLocalizer _localizer;
        private readonly SignInManager<User> _signInManager;
        private readonly UserManager<User> _userManager;


        public class LoginInputModel
        {
            [Required]
            [Display(Name = "Username")]
            public string UserName { get; set; }

            [Required]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }
        }
    }
}
```

### HappyTravel.Odawara.Api/wwwroot/images/other/mc.png
```json
�PNG

   IHDR   �   u   Np   KiTXtXML:com.adobe.xmp     <?xpacket begin="﻿" id="W5M0MpCehiHzreSzNTczkc9d"?>
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="Adobe XMP Core 5.6-c140 79.160451, 2017/05/06-01:08:21        ">
 <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
  <rdf:Description rdf:about=""/>
 </rdf:RDF>
</x:xmpmeta>
<?xpacket end="r"?>-CD�  �IDATx��y�U��?��������@6B�2������	��ฎ��(.x<(���#*!(΀��(A�@$	�hBv�����{u�_U^��u�_��U���=�N���uﭺ���o����:ꨣ�:ꨣ�:�0	e��טb�ڡ 4������q@��*���lڀ�@�p;�����pkbЂ\�$`� ���ד6 �ܿm��V�k�=����j����,�`���p`.�B]��u�_��m�S��I`r=���9lu9�5�V���^�JOkQ!�\�B�d�Hw�s�7��oC��8��S��)��s��rO�B�0��.u��u�U����	�d�$�|8ׅv�~��/�b�jIpp0�P��V���3���@�t�u��/F#��4�v�u��$�z���iuM��n�0$t�)�v��܌�|_H���;Pb�刔�8��x80���a��A�{�[�	��لGl?b����ߗS��"�r�,u��C��@�܆讗��n��^��gS����ܗ�P��N#������R	N�� SK�Mu���)X��ç�q�y�2������/CAX<
��VD�}�ݟ	�oC�8d<�z)c�Ur_�����#U�,�Zb��a�%��8�q>2>�ݑ�P���)ph��4�ԌΦt���쭝M(U&��q(2N7�ݑ�PKp�<pc��&2���߆R{m�㐟�(��� 4 _� 7��v�/j��	$p�;R=(�G��pb�H��5�������(D M > �ܗ��$�I���y�4��*�^TR���Jls{q_4�dk�P�r�:�NT�y4��:�T���Jls���_��gM lr��"�"9�K$��r�d�mh���vt�M�i�� ����E|B��z��m�i?�ޞ(<���!�#4r�n�Bj��P��<M��C%��'$X@Ni�;*2oB���0;�S��hNmC�m�c�b@VT[�����xN<��E���0��>FV�Ȁ�X�fv�8�k�S�,�vva��D��6����"�
���G��Mc�(b�zI�6C�����Ԇ��Vd���"�8�$w�#3	�,��d�vb�)Ί��@l['h%��8d�A7$��.	�=��(�{H��9�Հ��N�}=Q2.A��ܠ�܍��U3ѫ�B)Mf~�xG�e* ��_� 'R�e�� �ܧ��U��!Y�S�՟Bk��7��/5��|��!(r"�v��4�:Ĉ4b["'�!`A�々�4����{�����^��=Q��>� ���d��û �+�Ы�wu���S](�������c����Q��"�M���Ց���>�q�&���D�mEl|/��|5���<V[/:Z�H@���>��d�(���|�WzxMo<�0I�I�v5��N�O�`=6�{�Q�B�T��@51IX�j,`�̫$�
��s~�Ir�G֖���Ğ��RX�=�o$	�n��q�XCu����`�$�P��k'3���=
Y�42P�&6�l�,,�[{�&�Af�4�l����ȊQ�����D�����#
��C@���`��$5,���Q�}��L�{��zC�Fa7@��l���o�O��)r�5Toh��BT4�����7�<�K�AB%���3(�0OL���P��!lW\��=剩[91c��e����2�<��<�:VC>��� =�Z�*��=(h
���T�bQ��)r��]t.D9���JT�����e���P�L�(OL�{��zC�ΪP�d�}C0�S��1Toh(t��#���̄&?������74���GR�6�Sw��0Tw�Phr{�C���R8��-�Ӂ�O��"�>`���C��i�dC`��P�T�P���)r�^2Tw8Њ��x(�[7ģ�y	�1�"w/��h��M�Ҡ_�:���YM՟�����F���|g��ɨ-�E�a&����`�����i����t�Fg"�܈��(L�f`����B�������*�ӒDǭ�啬&�@�Irk���>�q8�M� �_NK�v���y�Ư�����n�mM~g��y��n��G%���ލ��8L�{'��p�A�ɵ%�mO�w	j(�$)4&�F��/�#��.�U5(4����n��<����FEHAxpWP�A�'��h'(4=�R�w�hp�8cRP��C�� w�~ ����&Q~jcsȿOpk��}XP[ơ��lNKP��)8c�8��QӵW <A������Vt=�$Zd5%����&�#%�����,E��ˀ_؞Q( �Z��+Y���CaR�%%�6ȸ/�� ɝ�h�M��hGѽ�	���߻ :c�?�	m�(E$ۑq��p�9n�7ܦ1(4����7��4TMB6�5���hyHnF�=p�����?��vաp�Z�H|r�Č!~^�3>EaJ#D��eː�8��6"R|-���Ø��>l
�m#PZ��T�}��Ņ#�r3[�4@=��q�Ҟ�"�7SLw �Y�k��	�z6�BW��'G�d����q�ܬQ�t��뫏nd|����B���ӝ3Q���\��!�����nO���fOi� ߆w@[���8�"��*2�~t �ى0�Ј���PU(4�/5ѵ�Iv�ww]�5Q���`�w�q���^� |�^���]�5ѽ�A�n�E�V��@~z8NРj�2�5�����_�f���U-t�� ���#vnV����BƱf�е29���7�TZ+:�nAw�d�k;@���f	��@���)A-Hn]�u��ͪ6�����XقS���i!?�Y� ��s�x]��_M����0�-����B�K�Qh�/7d�:�?T�ҐE���!�t5��i������1���*�4W�y%��T�!�$�~�Z%7Ȍ�E���W� q7p>������ݡ�hx�B�c�Qc)�Y��+�1�jp�Ln���,g��^F������c���W��F4r�"�1���"W���f]��k��xZ8G݈�d�A��<���$���9H��_����;�؊����P�����+ �b�X�f~� t�x\Q���;��tU��Wv��$ߺf41��ns��F���<~1�Em8��+�DB�+�ӆ��*c=�Bز!��%b���7�A���s��R�D���9�������pϓ��$�vg�Mr��Fp�����j����e����4���$ُ�C�:�D�M�E�e@K�� ~���,�|r{���iϪR+A/�� �{*X_D)H&�o���U����������is��P'��b&1�0�,Z�hǩ��I�PD�- i�|������"�:�In��y���ۀ��5U�� �"o�W����$Sy�y�{�ge��qjb�\A���HЍ�娀���Ȅ�y�8`�He6FB�}�T^�H�g�"C�.wȞg%���G�R'#�}4� 4WPO��ۋ,i�"����w�`�+Ʉb��{�kY?�Cm9�j��~|�Q|�f��4���,���D�M�h���}	$ ����6!+�nd�aehD�A�:x�'�K�ݣ�cP��ÁÀi��;�=��Gԋm#�[c��ې[G��餂������K��
5Gn�;�Dl��ɝ�a�^ �
�*NĊ!$4y��NE�k{+<8�r��XB?��_���SR,>9�%gfȵ;���i���N`�������<'XZ���]��T7��1k5I��4Ўf:1���i�*�D3��ʽ,ÂBȼ���i廌��l:q���%�b���<��7~CRq�|d,S�X��,QR�=���j6l��
,h�p���$3?Y�����u��ز'OG���5I���b��	�&�~
�e�E��r��gx� M(�X|�v�s��=s�[M�;���lۂ3�$��ƳeG��Ų��3��� ݝ��kd��X\�$K*t��k���7-m�/dQ
�5��]F�=�!·h�22Lt���慒d����hlJm�.4��O�h�6(���*z���#a�	i�ߜcî�Z�W�e�:��Lj�X��rKK�[ӓ��%s:mE��<ޔ�9m��ڢc�RГ�ܸ�6u�J1"���M$��Mx�,{K��sH1�u��"Ҽ� O��7X'(,'��X����3�I�x~��IP��؍7߻�5�O�-ض��[s4�,6�uT0�7�b|��5�[OQ��Ԫ���o9&�#�ǑSC랍�W�"��rhX��2���s��B�K����6��H�`f���if#	-�)� _{c����>�T����Y�)���<E�g�(�#�C"�Y$�ap1mی$�?떝\���-�/HNG�$p~?ɗ~Q! !�MH�΃B$�p}����HxzB��H�8�Ba�{nҭg+��ў�$C����J�eܺ��G�������e-$��B`�[v#�0r�J�Ĺ�}�������0�,d%��1Q���H��
�T$������\��l${�����!-����^�ɁE�H��ǐ��u%�4��L�]�))߉�F��[|�$�\�|�v4�e� ���2��۾�-�HX��\'�Q�:�f��+ O#���;	& ?�.~��G�3n�bd˔}����&D������~�c���.w�5�@ƫ\�p�D��7�����
~�{��E���$�)�ab�k�9kIZ��7|�}���y8�#R���q���4+�`��~v#��W�}��Id�����W~v����$5y�;<I���Y�n��O#	U6�钲���RJ�-�%��2�m@�h�StWI���u�^��ܻ����=!P�w�9�-ȍ�G����[_�$��_��ڙ����:���˃�,2Y��͘��p�Fc�6�yx4�KD�9�g���D���q%���\�<�y�f�甒�߁#�7�G�ϔܛ�O�,$5�^�y�� ����L{�W׫�%�������Q(����#�|��En���侱�X��j���_H�X����S�M�N_��Q���(�;��%�����(E���2ǿ��ϻ�}~r��0�8�;��7�Ieʤ(����e�.����ܯ!v�1�_�ז��Dp�J�0�%�@���������R�����I���[o.��=��,��ʳ��A�������P���w�Np��c���[�N�6�8�<
��f��HP����e�d�z��>��m�b�L������(�7 ��9�Ij��ua�����r�At�RW[�Є��e�{u�"�?���
yЛ�b8�C����So��8��d����	�4��*
�H =��^�>{��kD��;��u�_䩁��a��L���������7�~ʌƖ��\��<y}NA\r���DǬ$��+[@<B/"*�@\�p�g���kL�C/�yx�R~��t�o�~i��3����,ʫ��"
�-Ȕ�f��z�WAކ�
��a�a� ���6��<��Z�"����r�26�M:ۃx+�!��ш�Ώ"5{_�E�����r~���"��s��t%�n�q""BE"�[(�}��i��Ƞ�@���5�Qy�O��l_y��w]@��'ODԢ������c
����d�h�!���b�؎xC@賂'�p����Y��"��պ� .�و�x&�?���=�F���M�;73ݺ"n!�=��yKZ)o�?D10��X�=O���G_ߴ��S���9���������w��1x�w���k�7������=������Q�������IHt�_�v����JQ�/�)�8��^��<�ˏ4�����;�ϝ���/n3B�Y�s��\����"z��Q�|���QH�����"��#��PLF:;˜�"�h��q9E��{��i��<�<���=�w�˴�a:}]q���	i!ʎ2�V �G���91��{.�r�!��!x�����^V�)���E��G#�������B�FoC��_%���#�c"A�Cq	�Q�D�B�3����s"�OG�/������8�b�(���$E��a�ۧ���F���Ry��+�[de?킸� *ȡ�~���J�:� ��1�U��2�����=~+�P��%�Y��}���Qy�t�|��&#jc;�y�rn�:ꨣ�:ꨣ�:�c���pj���h    IEND�B`�
```

### HappyTravel.Odawara.Api/Pages/Account/ResetPassword.cshtml.cs
```json
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;
using HappyTravel.Odawara.Data.Model;
using HappyTravel.Odawara.Api.Infrastructure.Emails;
using IdentityServer4.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Localization;

namespace HappyTravel.Odawara.Api.Pages.Account
{
    public class ResetPassword : PageModel
    {
        public ResetPassword(UserManager<User> userManager, IIdentityServerInteractionService interaction, ResetPasswordMailSender mailSender,
            IStringLocalizer<ResetPassword> localizer)
        {
            _interaction = interaction;
            _localizer = localizer;
            _mailSender = mailSender;
            _userManager = userManager;
        }


        public async Task<IActionResult> OnGet(string userId, string code, string returnUrl)
        {
            var user = await _userManager.FindByIdAsync(userId);

            if (user is null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect Email"]);
                return Page();
            }

            ViewData["Email"] = user.Email;
            ModelState.Clear();

            return Page();
        }


        public async Task<IActionResult> OnPostAsync(string userId, string code, string returnUrl)
        {
            if (string.IsNullOrWhiteSpace(userId) || string.IsNullOrWhiteSpace(code) || string.IsNullOrWhiteSpace(returnUrl))
                return Page();

            if (!ModelState.IsValid)
                return Page();

            var authorizationContext = await _interaction.GetAuthorizationContextAsync(returnUrl);
            if (authorizationContext == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Wrong query parameters in URL"]);
                return Page();
            }

            var user = await _userManager.FindByIdAsync(userId);
            if (user is null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Login does not exist"]);
                return Page();
            }

            var result = await _userManager.ResetPasswordAsync(user, code, ResetPasswordInput.Password);

            if (!result.Succeeded)
            {
                foreach (var error in result.Errors)
                    ModelState.AddModelError(string.Empty, error.Description);

                return Page();
            }

            await _mailSender.SendSuccessInfo(user);

            return LocalRedirect(returnUrl);
        }


        [BindProperty(SupportsGet = true)]
        public ResetPasswordInputModel ResetPasswordInput { get; set; }


        private readonly IIdentityServerInteractionService _interaction;
        private readonly IStringLocalizer _localizer;
        private readonly ResetPasswordMailSender _mailSender;
        private readonly UserManager<User> _userManager;


        public class ResetPasswordInputModel
        {
            [Required]
            [StringLength(100, ErrorMessage = "The {0} must have a minimum of {2} and a maximum of {1} characters.", MinimumLength = 10)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }
        }
    }
}
```

### HappyTravel.Odawara.Api/Pages/Account/ResendEmailConfirmation.cshtml.cs
```json
﻿using System.Threading.Tasks;
using HappyTravel.Odawara.Data.Model;
using HappyTravel.Odawara.Api.Infrastructure.Emails;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Localization;

namespace HappyTravel.Odawara.Api.Pages.Account
{
    public class ResendEmailConfirmation : PageModel
    {
        public ResendEmailConfirmation(UserManager<User> userManager, ConfirmationMailSender confirmationMailSender, IStringLocalizer<ResendEmailConfirmation> localizer)
        {
            _confirmationMailSender = confirmationMailSender;
            _userManager = userManager;
            _localizer = localizer;
        }
        
        [BindProperty]
        public string Username { get; set; }
        
        public void OnGet()
        {
            
        }

        public string BackToLoginUrl => Url.Page("Login", new {returnUrl = HttpContext.Request.Query["returnUrl"]});


        public async Task<IActionResult> OnPostAsync()
        {
            var user = await _userManager.FindByNameAsync(Username);
            if (user == null || user.EmailConfirmed)
            {
                ModelState.AddModelError(string.Empty, _localizer["Invalid username or email already confirmed"]);
                return Page();
            }
            await _confirmationMailSender.Send(Url, user, "Account/Confirm");
            
            return RedirectToPage("/Account/Confirm", new
            {
                userId = user.Id
            });
        }
        
        
        private readonly ConfirmationMailSender _confirmationMailSender;
        private readonly UserManager<User> _userManager;
        private readonly IStringLocalizer _localizer;
    }
}
```

### HappyTravel.Odawara.Api/wwwroot/js/password.js
```json
﻿function getFirstPasswordField() {
    var inputs = document.getElementsByTagName("input");
    for (var i = 0; i < inputs.length; i++) {
        if ("password" == inputs[i].type)
            return inputs[i];
    }
    return inputs[0];
}

function generatePassword() {
    let div = document.createElement("div");
    div.className = "loader full-page";
    div.innerHTML =
        '<div class="inside"><span></span><span></span><span></span><span>' +
        '</span><span></span><span></span><span></span></div>';
    document.body.appendChild(div);
    getFirstPasswordField().style.color = "#eee";
    setTimeout(function() {
        document.body.removeChild(div);
        getFirstPasswordField().style.color = "#231f20";
    }, 430);

    fetch("Register?handler=GeneratePassword")
        .then(function (response) {
            return response.json();
        }).then(function (result) {
            let password = getFirstPasswordField();
            password.value = result;
            clearCopyResult();
            validateComplexity(null, result);
            let generateLink = document.getElementById("generate-link");
            generateLink.textContent = "Re-generate Password";
        });
}

function copyToClipboard() {
    let copyLink = document.getElementById("copy-link");

    let password = getFirstPasswordField();

    let dummy = document.createElement("input");
    document.body.appendChild(dummy);

    dummy.value = password.value;
    dummy.select();
    document.execCommand("copy");
    document.body.removeChild(dummy);
    copyLink.textContent = "Copied to Clipboard";
}

function clearCopyResult() {
    let copyLink = document.getElementById("copy-link");
    copyLink.textContent = "Copy to Clipboard";
}

function validateComplexity(target, generationResult) {
    let passwordRulesNode = window.document.getElementById("password-rules"),
        classes = [],
        value = (target ? target.value : generationResult) || "";

    if (value.match(/[a-z]/g))
        classes.push("ok-lowercase");

    if (value.match(/[A-Z]/g))
        classes.push("ok-uppercase");

    if (value.match(/[0-9]/g))
        classes.push("ok-number");

    if (value.match(/[^a-zA-Z0-9]/g))
        classes.push("ok-symbol");

    if (value.length >= 10)
        classes.push("ok-length");

    classes.push("total-" + classes.length);

    passwordRulesNode.className = classes.join(" ");
    clearCopyResult();
}

```

### HappyTravel.Odawara.Api/Pages/Account/Register.cshtml.cs
```json
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;
using HappyTravel.Odawara.Data.Model;
using HappyTravel.Odawara.Api.Events;
using HappyTravel.Odawara.Api.Infrastructure;
using HappyTravel.Odawara.Api.Infrastructure.Emails;
using IdentityServer4.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Localization;

namespace HappyTravel.Odawara.Api.Pages.Account
{
    public class RegisterModel : PageModel
    {
        public RegisterModel(UserManager<User> userManager, SignInManager<User> signInManager, IEventService events,
            IIdentityServerInteractionService interaction, ConfirmationMailSender confirmationMailSender, PasswordGenerator passwordGenerator,
            IStringLocalizer<RegisterModel> localizer)
        {
            _confirmationMailSender = confirmationMailSender;
            _events = events;
            _interaction = interaction;
            _localizer = localizer;
            _passwordGenerator = passwordGenerator;
            _signInManager = signInManager;
            _userManager = userManager;
        }


        public async Task OnGetAsync(string returnUrl)
        {
            ReturnUrl = returnUrl;
            await _signInManager.SignOutAsync();

            var isEmailDefined = TryGetEmailFromQuery(out var email);
            if (isEmailDefined)
                RegisterInput.Email = email;

            ModelState.Clear();
        }


        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            if (!ModelState.IsValid)
                return Page();

            var authorizationContext = await _interaction.GetAuthorizationContextAsync(returnUrl);
            if (authorizationContext == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Wrong query parameters in URL"]);
                return Page();
            }

            var user = new User {UserName = RegisterInput.UserName, Email = RegisterInput.Email};
            var result = await _userManager.CreateAsync(user, RegisterInput.Password);
            if (!result.Succeeded)
            {
                foreach (var error in result.Errors)
                    ModelState.AddModelError(string.Empty, error.Description);

                return Page();
            }

            await _events.RaiseAsync(new UserRegisterSuccessEvent(user.Id, user.UserName, authorizationContext.Client.ClientId));
            await _confirmationMailSender.Send(Url, user, returnUrl);

            return RedirectToPage("/Account/Confirm", new
            {
                userId = user.Id,
                returnUrl
            });
        }


        public string BackToLoginUrl => Url.Page("Login", new {returnUrl = HttpContext.Request.Query["returnUrl"]});


        [BindProperty(SupportsGet = true)]
        public RegisterInputModel RegisterInput { get; set; }


        public string ReturnUrl { get; set; }


        public JsonResult OnGetGeneratePassword() => new JsonResult(_passwordGenerator.Generate());


        private bool TryGetEmailFromQuery(out string email)
        {
            email = ReturnUrlHelper.GetQueryParameter(ReturnUrl, "userMail");
            return email != null;
        }


        private readonly ConfirmationMailSender _confirmationMailSender;

        private readonly IEventService _events;
        private readonly IIdentityServerInteractionService _interaction;
        private readonly IStringLocalizer _localizer;
        private readonly PasswordGenerator _passwordGenerator;
        private readonly SignInManager<User> _signInManager;
        private readonly UserManager<User> _userManager;


        public class RegisterInputModel
        {
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            [Required]
            [Display(Name = "Username")]
            public string UserName { get; set; }

            [Required]
            [StringLength(100, ErrorMessage = "The {0} must have a minimum of {2} and a maximum of {1} characters.", MinimumLength = 10)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }
        }
    }
}
```

### HappyTravel.Odawara.Api/Pages/Account/ResendEmailConfirmation.cshtml
```json
﻿@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.ResendEmailConfirmation

<div class="small">
    <div class="breadcrumbs">
        <div class="links">
            <a href="@Model.BackToLoginUrl">@Localizer["Sign In"]</a>
            <span class="small-arrow-right"></span>
            @Localizer["Registration"]
        </div>
    </div>
    <h2>
        @Localizer["Resend confirmation email"]
    </h2>
    <form method="post">
        <div class="form">
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span class="required">
                            @Localizer["Login"]
                        </span>
                    </div>
                    <div class="auth-input">
                        <input asp-for="Username" placeholder="@Localizer["Login"]" />
                    </div>
                </div>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">@Localizer["Resend"]</button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>
```

### HappyTravel.Odawara.Api/Pages/Error.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model ErrorModel

<div>
    <h2>
        @Localizer["Oops!"]
    </h2>
    <div class="paragraph">
        @Localizer["An error occurred while processing your request."]
    </div>
</div>
```

### HappyTravel.Odawara.Api/Pages/Account/Confirm.cshtml.cs
```json
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;
using HappyTravel.Odawara.Data.Model;
using IdentityServer4.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Localization;

namespace HappyTravel.Odawara.Api.Pages.Account
{
    public class ConfirmEmailModel : PageModel
    {
        public ConfirmEmailModel(UserManager<User> userManager, SignInManager<User> signInManager, IIdentityServerInteractionService interaction,
            IStringLocalizer<ConfirmEmailModel> localizer)
        {
            _interaction = interaction;
            _localizer = localizer;
            _signInManager = signInManager;
            _userManager = userManager;
        }


        public async Task<IActionResult> OnGetAsync(string userId, string code, string returnUrl)
        {
            if (string.IsNullOrWhiteSpace(userId) || string.IsNullOrWhiteSpace(code) || string.IsNullOrWhiteSpace(returnUrl))
                return Page();

            var user = await _userManager.FindByIdAsync(userId);
            if (user == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect URL parameters"]);
                return Page();
            }

            var context = await _interaction.GetAuthorizationContextAsync(returnUrl);
            if (context == null)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect URL parameters"]);
                return Page();
            }

            var confirmResult = await _userManager.ConfirmEmailAsync(user, code.Trim());
            if (!confirmResult.Succeeded)
            {
                ModelState.AddModelError(string.Empty, _localizer["Incorrect confirmation Code"]);
                return Page();
            }

            await _signInManager.SignInAsync(user, false);

            return LocalRedirect(returnUrl);
        }


        public IActionResult OnPost(string userId, string returnUrl)
        {
            var code = Input.Code;
            return RedirectToPage("Confirm", new {userId, returnUrl, code});
        }


        [BindProperty]
        public ManualConfirmInputModel Input { get; set; }

        public string BackToLoginUrl => Url.Page("Login", new {returnUrl = HttpContext.Request.Query["returnUrl"]});


        private readonly IIdentityServerInteractionService _interaction;
        private readonly IStringLocalizer _localizer;
        private readonly SignInManager<User> _signInManager;
        private readonly UserManager<User> _userManager;


        public class ManualConfirmInputModel
        {
            [Required]
            [Display(Name = "Confirmation Code")]
            public string Code { get; set; }
        }
    }
}
```

### HappyTravel.Odawara.Api/Pages/Account/ResetPassword.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.ResetPassword

<script src="~/js/password.js"></script>

<div>
    <h2>
        @Localizer["Reset Password"]
    </h2>
    <div class="paragraph">
        @Localizer["Please set a new password for"] @ViewData["Email"]
    </div>
    <form method="post">
        <div class="form">
            <div class="row">
                <div class="field">
                    <div class="label with-actions">
                        <span class="required">
                            @Localizer["Password"]
                        </span>
                        <span>
                            <a onclick="generatePassword()" class="link" id="generate-link">@Localizer["Generate Password"]</a>
                            &#8226
                            <a onclick="copyToClipboard()" class="link" id="copy-link">@Localizer["Copy to Clipboard"]</a>
                        </span>
                    </div>
                    <div class="auth-input">
                        <input asp-for="ResetPasswordInput.Password" placeholder="@Localizer["Please enter your Password"]" onkeyup="validateComplexity(this)" />
                    </div>
                    <div class="password-rules">
                        <div id="password-rules">
                            <div class="caption-wrapper">
                                <div class="caption"></div>
                            </div>
                            <div class="title">@Localizer["Password Complexity"]:</div>
                            <div>
                                <div class="label rule-lowercase">
                                    <span>&#8226</span> @Localizer["A lowercase Latin character"]
                                </div>
                                <div class="label rule-uppercase">
                                    <span>&#8226</span> @Localizer["An uppercase Latin character"]
                                </div>
                                <div class="label rule-number">
                                    <span>&#8226</span> @Localizer["A digit"]
                                </div>
                                <div class="label rule-symbol">
                                    <span>&#8226</span> @Localizer["A special character"]
                                </div>
                                <div class="label rule-length">
                                    <span>&#8226</span> @Localizer["10 characters minimum"]
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">@Localizer["Apply"]</button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>

```

### HappyTravel.Odawara.Api/Views/Shared/_Layout.cshtml
```json
﻿@using HappyTravel.Odawara.Api.Services

<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <meta charset="utf-8"/>
    <title>Happytravel.com – Identity</title>
    <link rel="stylesheet" href="/css/main.css"/>
    <link rel="shortcut icon" href="/images/favicon.ico"/>
    <link rel="icon" href="/images/icons/icon-32x32.png" sizes="32x32" />
    <link rel="icon" href="/images/icons/icon-192x192.png" sizes="192x192" />
    <link rel="apple-touch-icon" href="/images/icons/icon-180x180.png" />
    <meta name="msapplication-TileImage" content="/images/icons/icon-270x270.png" />
    <meta name="viewport" content="width=device-width">
    @if (HolidayService.IsHolidayOrEve(DateTime.Today))
    {
        <link rel="stylesheet" href="/css/snow.css"/>
    }
</head>
<body>
<div class="body-wrapper">
    <div class="block-wrapper">
        <div class="account block" style="background-image: url('@(BackgroundImageService.GetUrl())')">
            <header>
                <section>
                    <div class="logo-wrapper">
                        <div class="logo"><a href="/" class="image"></a><div class="underline"></div></div>
                    </div>
                </section>
            </header>
            <section class="section">
                @RenderBody()
            </section>
        </div>
    </div>
    <footer>
        <section>
            <div class="company">
                <div class="logo-wrapper"><a href="https://happytravel.com/" class="logo"></a></div>
            </div>
            <div class="column"><h3>Information</h3>
                <ul>
                    <li><a href="https://happytravel.com/about">About Us</a></li>
                    <li><a href="https://happytravel.com/contact">Contacts</a></li>
                    <li><a href="https://happytravel.com/terms">Terms &amp; Conditions</a></li>
                    <li><a href="https://happytravel.com/privacy">Privacy Policy</a></li>
                </ul>
            </div>
            <div class="column"><h3>Contacts</h3>
                <ul>
                    <li><span>Email:</span> <a href="mailto:info@happytravel.com">info@happytravel.com</a></li>
                    <li><span>Phone:</span> @ViewBag.CompanyInfo.Phone</li>
                    <li>
                        <span>Address:</span>
                        @ViewBag.CompanyInfo.Name,<br>
                        @ViewBag.CompanyInfo.Address<br>
                        P.O. @ViewBag.CompanyInfo.PostalCode<br>
                        @ViewBag.CompanyInfo.City, @ViewBag.CompanyInfo.Country
                    </li>
                    <li><span>TRN:</span> @ViewBag.CompanyInfo.Trn</li>
                    <li><span>IATA:</span> @ViewBag.CompanyInfo.Iata</li>
                    <li><span>Trade License:</span> @ViewBag.CompanyInfo.TradeLicense</li>
                </ul>
            </div>
        </section>
        <section>
            <div class="payments">
            <img src="/images/other/mc.png" class="near transparent" alt="Mastercard">
            <img src="/images/other/mc-sec.png" class="interval-big transparent" alt="Mastercard Id Check">
            <img src="/images/other/visa.png" alt="Visa">
            <img src="/images/other/visa-sec.png" class="interval" alt="Visa Secure">
            <img src="/images/other/amex.png" alt="American Express"></div>
        </section>
        <section class="copyright">
            <div>All Rights Reserved © 2019 — 2021 @ViewBag.CompanyInfo.Name</div>
            <div class="service-info column">
                <span>API – @ViewBag.Version</span>
            </div>
        </section>
    </footer>
</div>
</body>
</html>
```

### HappyTravel.Odawara.Api/en.po
```json
﻿msgid "10 characters minimum"
msgstr "10 characters minimum"

msgid "A digit"
msgstr "A digit"

msgid "A special character"
msgstr "A special character"

msgid "A lowercase Latin character"
msgstr "A lowercase Latin character"

msgid "An error occurred while processing your request."
msgstr "An error occurred while processing your request."

msgid "An uppercase Latin character"
msgstr "An uppercase Latin character"

msgid "Account Confirmation"
msgstr "Account Confirmation"

msgid "Address Enter"
msgstr "Address Enter"

msgid "Agency Information"
msgstr "Agency Information"

msgid "Agent Information"
msgstr "Agent Information"

msgid "Already have an account?"
msgstr "Already have an account?"

msgid "Apply"
msgstr "Apply"

msgid "Back"
msgstr "Back"

msgid "Choose your Password"
msgstr "Choose your Password"

msgid "Confirmation"
msgstr "Confirmation"

msgid "Confirmation Code"
msgstr "Confirmation Code"

msgid "Confirm The Email Address"
msgstr "Confirm The Email Address"

msgid "Continue Registration"
msgstr "Continue Registration"

msgid "Copy to Clipboard"
msgstr "Copy to Clipboard"

msgid "Create a Happytravel.com account and start booking"
msgstr "Create a Happytravel.com account and start booking"

msgid "Create a new one"
msgstr "Create a new one"

msgid "Don`t have an Account?"
msgstr "Don`t have an Account?"

msgid "Please enter your Password"
msgstr "Please enter your Password"

msgid "Please enter your Login"
msgstr "Please enter your Login"

msgid "Enter Code here"
msgstr "Enter Code here"

msgid "Please enter your Email"
msgstr "Please enter your Email"

msgid "Email"
msgstr "Email"

msgid "Forgot Password?"
msgstr "Forgot Password?"

msgid "Generate Password"
msgstr "Generate Password"

msgid "Get started with your new account"
msgstr "Get started with your new account"

msgid "Go back to Happytravel.com"
msgstr "Go back to Happytravel.com"

msgid "Incorrect confirmation Code"
msgstr "Incorrect confirmation Code"

msgid "Incorrect URL parameters"
msgstr "Incorrect URL parameters"

msgid "Incorrect Login or Password"
msgstr "Incorrect Login or Password"

msgid "Sign In"
msgstr "Sign In"

msgid "Sign In here"
msgstr "Sign In here"

msgid "Login Information"
msgstr "Login Information"

msgid "Oops!"
msgstr "Oops!"

msgid "Password"
msgstr "Password"

msgid "Password Complexity"
msgstr "Password Complexity"

msgid "Please enter the email address connected with your Happytravel.com account in the field below to begin password restoration."
msgstr "Please enter the email address connected with your Happytravel.com account in the field below to begin password restoration."

msgid "Please keep in mind we don't store your credentials"
msgstr "Please keep in mind we don't store your credentials"

msgid "Please set a new password for"
msgstr "Please set a new password for"

msgid "Query parameters in URL are missing. Sorry, we can`t find your account without this info. Have you got here by the direct link?"
msgstr "Query parameters in URL are missing. Sorry, we can`t find your account without this info. Have you got here by the direct link?"

msgid "Registration"
msgstr "Registration"

msgid "Reset"
msgstr "Reset"

msgid "Reset Password"
msgstr "Reset Password"

msgid "To confirm the address, manually enter the code from the message into the field below:"
msgstr "To confirm the address, manually enter the code from the message into the field below:"

msgid "Login does not exist"
msgstr "Login does not exist"

msgid "Login"
msgstr "Login"

msgid "We have sent a message with a confirmation link and a confirmation code to your email address. The message will arrive within 5&ndash;10 minutes."
msgstr "We have sent a message with a confirmation link and a confirmation code to your email address. The message will arrive within 5&ndash;10 minutes."

msgid "We have sent a message with a password recovery link to your email. The message usually arrives within 5&ndash;10 minutes. The link will be available for 24 hours."
msgstr "We have sent a message with a password recovery link to your email. The message usually arrives within 5&ndash;10 minutes. The link will be available for 24 hours."

msgid "Incorrect Email"
msgstr "Incorrect Email"

msgid "Wrong query parameters in URL"
msgstr "Wrong query parameters in URL"

msgid "You must remember them or use a password management software"
msgstr "You must remember them or use a password management software"

msgid "You were successfully logged out!"
msgstr "You were successfully logged out!"

msgid "Resend"
msgstr "Resend"

msgid "Resend confirmation email"
msgstr "Resend confirmation email"

msgid ""
msgstr ""
```

### HappyTravel.Odawara.Api/wwwroot/css/main.css
```json
* {
  outline: none;
  box-sizing: border-box; }[dir] * {
  margin: 0;
  padding: 0; }

menu, nav, li, ul {
  list-style: none; }

a, a[href] {
  color: inherit;
  text-decoration: none; }

[dir] a, [dir] a[href] {
  cursor: pointer; }

h1, h2, h3, h4, h5, h6 {
  font-weight: 400; }


@-webkit-keyframes Appear {
  0% {
    opacity: 0; }
  100% {
    opacity: 1; } }

@keyframes Appear {
  0% {
    opacity: 0; }
  100% {
    opacity: 1; } }

body {
  color: #231f20;
  letter-spacing: 0.232143px;
  line-height: 16px;
  font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", "Arial Neue", sans-serif;
  font-size: 13px; }

[dir] body {
  background: #ffffff; }

textarea, pre, input, button {
  font-family: inherit;
  letter-spacing: inherit; }

.link, a.link, a[href].link {
  color: #1267fb; }

[dir] .link, [dir] a.link, [dir] a[href].link {
  cursor: pointer; }
  .link:hover, a.link:hover, a[href].link:hover {
    color: #074BC5; }
  .link.underlined, a.link.underlined, a[href].link.underlined {
    text-decoration: underline; }

h1 {
  font-size: 30px;
  line-height: 36px; }

h2 {
  font-size: 24px;
  line-height: 29px;
  font-weight: 400; }
  h2 span {
    color: #777777; }

section {
  max-width: 1290px;
  width: 100%; }

[dir] section {
  margin: 0 auto;
  padding: 0 30px; }
  @media (max-width: 1139.5px) {
    [dir] section {
      padding: 0 20px; } }
  @media (max-width: 767.5px) {
    [dir] section {
      padding: 0 10px; } }

.hide {
  display: none !important; }

.dual {
  display: flex;
  width: 100%;
  justify-content: space-between; }
  [dir=ltr] .dual .second {
  text-align: right; }
  [dir=rtl] .dual .second {
    text-align: left; }
  .dual.column {
    flex-direction: column; }
    [dir=ltr] .dual.column .second {
  text-align: left; }
    [dir=rtl] .dual.column .second {
      text-align: right; }

b, strong {
  font-weight: 500; }

.block-wrapper {
  min-height: 100vh; }

[dir] .block-wrapper {
  padding-top: 81px; }
  .block-wrapper header {
    position: static; }
  [dir] .block-wrapper header {
    margin-top: -81px;
    box-shadow: none !important; }

.hide-desktop {
  display: none; }

.hide-mobile {
  display: block; }

@media (max-width: 1139.5px) {
  [dir] .with-search .block-wrapper {
    padding-top: 161px; }
  .hide-desktop {
    display: block; }
  .hide-mobile {
    display: none; } }

.accent-frame {
  position: relative;
  min-height: 60px;
  height: auto;
  overflow: hidden;
  display: flex;
  line-height: 18px; }[dir] .accent-frame {
  margin: 20px 0 35px;
  border-radius: 8px;
  background: #F7F8FC;
  padding: 16px 0 14px; }
  [dir] .accent-frame + .accent-frame {
    margin-top: -20px; }
  [dir] .accent-frame .before {
    padding: 9px 25px 9px 25px; }
  [dir=ltr] .accent-frame .before {
  border-right: 1px solid #e5e5e5; }
  [dir=rtl] .accent-frame .before {
    border-left: 1px solid #e5e5e5; }
    .accent-frame .before.Cancelled .icon {
      -webkit-filter: grayscale(1);
              filter: grayscale(1);
      opacity: .6; }
  .accent-frame .data {
    flex-grow: 1;
    display: flex;
    justify-content: space-between; }
  [dir] .accent-frame .data {
    padding: 4px 27px; }
    .accent-frame .data.only {
      flex-direction: column;
      color: #818283; }
      .accent-frame .data.only div {
        font-weight: 500;
        line-height: 17px;
        color: #231f20; }
      [dir] .accent-frame .data.only div {
        margin-bottom: 8px; }
    .accent-frame .data .first, .accent-frame .data .second {
      font-weight: 500;
      line-height: 26px; }
    [dir=ltr] .accent-frame .data .first {
  padding-right: 20px; }
    [dir=rtl] .accent-frame .data .first {
      padding-left: 20px; }
    [dir=ltr] .accent-frame .data .second {
  text-align: right; }
    [dir=rtl] .accent-frame .data .second {
      text-align: left; }
    @media (max-width: 650px) {
      .accent-frame .data {
        flex-direction: column; }
        [dir=ltr] .accent-frame .data .second {
    text-align: left; }
        [dir=rtl] .accent-frame .data .second {
          text-align: right; } }
  [dir=ltr] .accent-frame .after {
  padding: 4px 22px 4px 5px; }
  [dir=rtl] .accent-frame .after {
    padding: 4px 5px 4px 22px; }
    .accent-frame .after .status-updater {
      vertical-align: middle;
      overflow: hidden;
      display: inline-block;
      width: 70px;
      line-height: 30px;
      font-size: 40px; }
    [dir] .accent-frame .after .status-updater {
      text-align: center; }
      .accent-frame .after .status-updater button {
        width: 50px;
        height: 50px; }
      [dir] .accent-frame .after .status-updater button {
        border-radius: 6px; }
        .accent-frame .after .status-updater button .icon {
          -webkit-filter: grayscale(1);
                  filter: grayscale(1); }
        .accent-frame .after .status-updater button:hover .icon {
          -webkit-filter: none;
                  filter: none; }

.allotment li {
  font-size: 14px;
  line-height: 22px; }[dir] .allotment li {
  margin-bottom: 12px;
  background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTAiIGhlaWdodD0iMTIiIHZpZXdCb3g9IjAgMCAxMCAxMiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGQ9Ik05LjYzMDc4IDcuMTMxQzkuNTc0MjggNy4xODkgOS4zNjA5NCA3LjQzNyA5LjE2MjIgNy42NDFDNy45OTcwOCA4LjkyNCA0Ljk1NzYyIDExLjAyNCAzLjM2Njc4IDExLjY2NUMzLjEyNTE4IDExLjc2OCAyLjUxNDM3IDExLjk4NiAyLjE4ODAyIDEyQzEuODc1MyAxMiAxLjU3NzIgMTEuOTI4IDEuMjkyNzQgMTEuNzgyQzAuOTM4MTM5IDExLjU3OCAwLjY1MzY3OCAxMS4yNTcgMC40OTc4MDggMTAuODc4QzAuMzk3NDY3IDEwLjYxNSAwLjI0MTU5OCA5LjgyOCAwLjI0MTU5OCA5LjgxNEMwLjA4NTcyODIgOC45NTMgMCA3LjU1NCAwIDYuMDA4QzAgNC41MzUgMC4wODU3MjgyIDMuMTkzIDAuMjEzMzQ2IDIuMzE5QzAuMjI3OTU5IDIuMzA1IDAuMzgzODI5IDEuMzI3IDAuNTU0MzExIDAuOTkyQzAuODY3MDI0IDAuMzggMS40Nzc4NCAwIDIuMTMxNTEgMEgyLjE4ODAyQzIuNjEzNzQgMC4wMTUgMy41MDkwMSAwLjM5NSAzLjUwOTAxIDAuNDA5QzUuMDE0MTMgMS4wNTEgNy45ODM0NCAzLjA0OCA5LjE3NjgxIDQuMzc1QzkuMTc2ODEgNC4zNzUgOS41MTI5MSA0LjcxNiA5LjY1OTA0IDQuOTI5QzkuODg2OTkgNS4yMzUgMTAgNS42MTQgMTAgNS45OTNDMTAgNi40MTYgOS44NzIzOCA2LjgxIDkuNjMwNzggNy4xMzFaIiBmaWxsPSJ1cmwoI3BhaW50MF9saW5lYXIpIi8+CiAgICA8ZGVmcz4KICAgICAgICA8bGluZWFyR3JhZGllbnQgaWQ9InBhaW50MF9saW5lYXIiIHgxPSI1IiB5MT0iMCIgeDI9IjUiIHkyPSIxMiIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPgogICAgICAgICAgICA8c3RvcCBzdG9wLWNvbG9yPSIjRkJBQzE5Ii8+CiAgICAgICAgICAgIDxzdG9wIG9mZnNldD0iMSIgc3RvcC1jb2xvcj0iI0ZFQ0IwQyIvPgogICAgICAgIDwvbGluZWFyR3JhZGllbnQ+CiAgICA8L2RlZnM+Cjwvc3ZnPgo=) 3px 5px no-repeat; }[dir=ltr] .allotment li {
  padding-left: 24px; }[dir=rtl] .allotment li {
  padding-right: 24px; }
  [dir] .allotment li .primary + .additional {
    margin-top: 5px; }
  .allotment li .additional {
    color: #777777;
    font-weight: 300; }
  .allotment li.gray {
    -webkit-filter: grayscale(1);
            filter: grayscale(1); }
  [dir] .allotment li.important {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTQiIHZpZXdCb3g9IjAgMCAxNiAxNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJNOS43MDQ1MyAxLjEyMTYzTDE1LjM1OCAxMC45MzM0QzE1LjQ4MjQgMTEuMjI2MyAxNS41MzY5IDExLjQ2NDQgMTUuNTUyNCAxMS43MTE4QzE1LjU4MzUgMTIuMjg5OCAxNS4zODE0IDEyLjg1MTcgMTQuOTg0OCAxMy4yODRDMTQuNTg4MiAxMy43MTQ4IDE0LjA1MTYgMTMuOTY5MiAxMy40NjgzIDE0SDIuMDgzNTlDMS44NDI1MiAxMy45ODU0IDEuNjAxNDUgMTMuOTMwNiAxLjM3NTkzIDEzLjg0NTlDMC4yNDgzNDYgMTMuMzkxMSAtMC4yOTYwMDcgMTIuMTExOCAwLjE2MjgwNCAxMS4wMDI4TDUuODU1MTggMS4xMTQ2OUM2LjA0OTU5IDAuNzY3MTA1IDYuMzQ1MSAwLjQ2NzMwMyA2LjcxMDU5IDAuMjc0NjI5QzcuNzY4MTkgLTAuMzExODczIDkuMTEzNTIgMC4wNzM0NzYyIDkuNzA0NTMgMS4xMjE2M1pNOC40NTMxNyA3LjU4NzlDOC40NTMxNyA3Ljk1Nzg0IDguMTQ5ODkgOC4yNjY4OSA3Ljc3NjYxIDguMjY2ODlDNy40MDMzNCA4LjI2Njg5IDcuMDkyMjkgNy45NTc4NCA3LjA5MjI5IDcuNTg3OVY1LjQwNzZDNy4wOTIyOSA1LjAzNjg5IDcuNDAzMzQgNC43MzcwOSA3Ljc3NjYxIDQuNzM3MDlDOC4xNDk4OSA0LjczNzA5IDguNDUzMTcgNS4wMzY4OSA4LjQ1MzE3IDUuNDA3NlY3LjU4NzlaTTcuNzc2NjEgMTAuOTAyNUM3LjQwMzM0IDEwLjkwMjUgNy4wOTIyOSAxMC41OTM1IDcuMDkyMjkgMTAuMjI0M0M3LjA5MjI5IDkuODUzNiA3LjQwMzM0IDkuNTQ1MzIgNy43NzY2MSA5LjU0NTMyQzguMTQ5ODkgOS41NDUzMiA4LjQ1MzE3IDkuODQ2NjYgOC40NTMxNyAxMC4yMTU4QzguNDUzMTcgMTAuNTkzNSA4LjE0OTg5IDEwLjkwMjUgNy43NzY2MSAxMC45MDI1WiIgZmlsbD0iIzU5QkE3MSIvPgo8L3N2Zz4K); }
  [dir=ltr] .allotment li.important {
  background-position: left 4px; }
  [dir=rtl] .allotment li.important {
    background-position: right 4px; }
  [dir] .allotment li.warning {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTQiIHZpZXdCb3g9IjAgMCAxNiAxNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJNOS43MDQ1MyAxLjEyMTYzTDE1LjM1OCAxMC45MzM0QzE1LjQ4MjQgMTEuMjI2MyAxNS41MzY5IDExLjQ2NDQgMTUuNTUyNCAxMS43MTE4QzE1LjU4MzUgMTIuMjg5OCAxNS4zODE0IDEyLjg1MTcgMTQuOTg0OCAxMy4yODRDMTQuNTg4MiAxMy43MTQ4IDE0LjA1MTYgMTMuOTY5MiAxMy40NjgzIDE0SDIuMDgzNTlDMS44NDI1MiAxMy45ODU0IDEuNjAxNDUgMTMuOTMwNiAxLjM3NTkzIDEzLjg0NTlDMC4yNDgzNDYgMTMuMzkxMSAtMC4yOTYwMDcgMTIuMTExOCAwLjE2MjgwNCAxMS4wMDI4TDUuODU1MTggMS4xMTQ2OUM2LjA0OTU5IDAuNzY3MTA1IDYuMzQ1MSAwLjQ2NzMwMyA2LjcxMDU5IDAuMjc0NjI5QzcuNzY4MTkgLTAuMzExODczIDkuMTEzNTIgMC4wNzM0NzYyIDkuNzA0NTMgMS4xMjE2M1pNOC40NTMxNyA3LjU4NzlDOC40NTMxNyA3Ljk1Nzg0IDguMTQ5ODkgOC4yNjY4OSA3Ljc3NjYxIDguMjY2ODlDNy40MDMzNCA4LjI2Njg5IDcuMDkyMjkgNy45NTc4NCA3LjA5MjI5IDcuNTg3OVY1LjQwNzZDNy4wOTIyOSA1LjAzNjg5IDcuNDAzMzQgNC43MzcwOSA3Ljc3NjYxIDQuNzM3MDlDOC4xNDk4OSA0LjczNzA5IDguNDUzMTcgNS4wMzY4OSA4LjQ1MzE3IDUuNDA3NlY3LjU4NzlaTTcuNzc2NjEgMTAuOTAyNUM3LjQwMzM0IDEwLjkwMjUgNy4wOTIyOSAxMC41OTM1IDcuMDkyMjkgMTAuMjI0M0M3LjA5MjI5IDkuODUzNiA3LjQwMzM0IDkuNTQ1MzIgNy43NzY2MSA5LjU0NTMyQzguMTQ5ODkgOS41NDUzMiA4LjQ1MzE3IDkuODQ2NjYgOC40NTMxNyAxMC4yMTU4QzguNDUzMTcgMTAuNTkzNSA4LjE0OTg5IDEwLjkwMjUgNy43NzY2MSAxMC45MDI1WiIgZmlsbD0idXJsKCNwYWludDBfbGluZWFyKSIvPgogICAgPGRlZnM+CiAgICAgICAgPGxpbmVhckdyYWRpZW50IGlkPSJwYWludDBfbGluZWFyIiB4MT0iMC4zMTExMTEiIHkxPSI3IiB4Mj0iMTUuODY2NyIgeTI9IjciIGdyYWRpZW50VW5pdHM9InVzZXJTcGFjZU9uVXNlIj4KICAgICAgICAgICAgPHN0b3Agc3RvcC1jb2xvcj0iI0ZBQUIxOSIvPgogICAgICAgICAgICA8c3RvcCBvZmZzZXQ9IjEiIHN0b3AtY29sb3I9IiNGRUNEMEEiLz4KICAgICAgICA8L2xpbmVhckdyYWRpZW50PgogICAgPC9kZWZzPgo8L3N2Zz4K); }
  [dir=ltr] .allotment li.warning {
  background-position: left 4px; }
  [dir=rtl] .allotment li.warning {
    background-position: right 4px; }
  [dir] .allotment li.warn {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNiAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJNMTYgOEMxNiAxMi40MTUyIDEyLjQxNiAxNiA4IDE2QzMuNTc2IDE2IC0zLjEzMzkzZS0wNyAxMi40MTUyIC02Ljk5MzgyZS0wNyA4Qy0xLjA4NTUxZS0wNiAzLjU4MzIgMy41NzYgMS4wODYxNGUtMDYgOCA2Ljk5MzgyZS0wN0MxMi40MTYgMy4xMzMyM2UtMDcgMTYgMy41ODMyIDE2IDhaTTguNzA1MzYgMTEuMDMyQzguNzA1MzYgMTEuNDE1MiA4LjM4NTM2IDExLjczNiA4LjAwMTM2IDExLjczNkM3LjYxNzM2IDExLjczNiA3LjMwNTM2IDExLjQxNTIgNy4zMDUzNiAxMS4wMzJMNy4zMDUzNiA3LjQ5NjA1QzcuMzA1MzYgNy4xMTEyNSA3LjYxNzM2IDYuODAwMDUgOC4wMDEzNiA2LjgwMDA1QzguMzg1MzYgNi44MDAwNSA4LjcwNTM2IDcuMTExMjUgOC43MDUzNiA3LjQ5NjA1TDguNzA1MzYgMTEuMDMyWk03Ljk5MjQzIDQuMjU1MTNDOC4zODQ0MyA0LjI1NTEzIDguNjk2NDMgNC41NzUxMyA4LjY5NjQzIDQuOTU5MTNDOC42OTY0MyA1LjM0MzEzIDguMzg0NDMgNS42NTUxMyA4LjAwMDQzIDUuNjU1MTNDNy42MDg0MyA1LjY1NTEzIDcuMjk2NDMgNS4zNDMxMyA3LjI5NjQzIDQuOTU5MTNDNy4yOTY0MyA0LjU3NTEzIDcuNjA4NDMgNC4yNTUxMyA3Ljk5MjQzIDQuMjU1MTNaIiBmaWxsPSJ1cmwoI3BhaW50MF9saW5lYXIpIi8+CiAgICA8ZGVmcz4KICAgICAgICA8bGluZWFyR3JhZGllbnQgaWQ9InBhaW50MF9saW5lYXIiIHgxPSI4IiB5MT0iMTYiIHgyPSI4IiB5Mj0iNi45OTM4MmUtMDciIGdyYWRpZW50VW5pdHM9InVzZXJTcGFjZU9uVXNlIj4KICAgICAgICAgICAgPHN0b3Agc3RvcC1jb2xvcj0iI0ZCQUMxOSIvPgogICAgICAgICAgICA8c3RvcCBvZmZzZXQ9IjEiIHN0b3AtY29sb3I9IiNGRUNCMEMiLz4KICAgICAgICA8L2xpbmVhckdyYWRpZW50PgogICAgPC9kZWZzPgo8L3N2Zz4K); }
  [dir=ltr] .allotment li.warn {
  background-position: left 1px; }
  [dir=rtl] .allotment li.warn {
    background-position: right 1px; }
  .allotment li .guests {
    color: #231f20;
    font-size: 15px; }
    .allotment li .guests strong {
      font-weight: 400; }

.amenities {
  display: flex; }
  .amenities .allotment {
    width: 33%; }
  [dir=ltr] .amenities .allotment {
  padding-right: 20px; }
  [dir=rtl] .amenities .allotment {
    padding-left: 20px; }
    [dir] .amenities .allotment li {
      margin-bottom: 8px; }

.billet {
  font-size: 14px;
  line-height: 17px; }[dir] .billet {
  border: 1px solid #e5e5e5;
  padding: 38px 30px 28px;
  border-radius: 25px; }
  .billet img {
    max-width: 100%; }
  .billet.sticky {
    position: -webkit-sticky;
    position: sticky;
    top: 160px; }
  [dir] .billet.sticky {
    margin-bottom: 50px; }
  [dir] .billet h2 {
    margin-bottom: 20px; }
  [dir] .billet h3, [dir] .billet h4 {
    margin-bottom: 25px; }
  .billet h4 {
    font-size: 20px;
    line-height: 24px; }
  [dir] .billet h4 {
    margin-top: 40px; }
  .billet .line {
    font-size: 14px;
    line-height: 22px;
    font-weight: 500; }
  [dir] .billet .line {
    margin-bottom: 12px; }
    .billet .line .icon {
      position: relative;
      top: 3px; }
    [dir=ltr] .billet .line .icon {
  margin-right: 15px; }
    [dir=rtl] .billet .line .icon {
      margin-left: 15px; }
  .billet .select .line {
    font-weight: 400;
    line-height: 34px !important; }
  [dir] .billet .select .line {
    margin-bottom: 0; }
  .billet .select span {
    color: #1CBE69; }
  .billet .select img {
    max-width: 25px;
    max-height: 18px;
    vertical-align: middle; }
  [dir=ltr] .billet .select img {
  margin-left: 12px; }
  [dir=rtl] .billet .select img {
    margin-right: 12px; }
    [dir=ltr] .billet .select img:last-child {
  margin-left: 15px; }
    [dir=rtl] .billet .select img:last-child {
      margin-right: 15px; }
  .billet .button {
    width: 100%; }
  [dir] .billet .button {
    margin-top: 15px; }
  [dir] .billet .dual {
    margin-bottom: 15px; }
    .billet .dual .first {
      color: #777777;
      font-weight: 300; }
    [dir=ltr] .billet .dual .first {
  margin-right: 6px; }
    [dir=rtl] .billet .dual .first {
      margin-left: 6px; }
    .billet .dual .second {
      color: #231f20; }
  .billet .subtitle {
    color: #777777; }
  [dir] .billet .subtitle {
    margin-bottom: 3px; }
  [dir] .billet .title {
    margin-top: 0; }
  [dir] .billet .title + .subtitle {
    margin: -15px 0 15px; }
  .billet .photo-holder {
    max-height: 200px;
    overflow: hidden; }
  [dir=ltr] .billet .photo-holder {
  border-top-left-radius: 25px;
  border-bottom-right-radius: 25px; }
  [dir=rtl] .billet .photo-holder {
    border-top-right-radius: 25px;
    border-bottom-left-radius: 25px; }
    .billet .photo-holder .photo {
      height: 300px; }
    [dir] .billet .photo-holder .photo {
      background: #F0F5F9 center center no-repeat;
      background-size: cover; }
    .billet .photo-holder img {
      object-fit: cover; }
    [dir=ltr] .billet .photo-holder img {
  float: left; }
    [dir=rtl] .billet .photo-holder img {
      float: right; }
  .billet .checkbox-holder {
    font-size: 13px; }
  [dir] .billet .checkbox-holder {
    margin-top: 20px; }
    .billet .checkbox-holder .checkbox:hover {
      color: #231f20; }
    .billet .checkbox-holder a, .billet .checkbox-holder a:hover {
      color: #f9a51a; }
    [dir=ltr] .billet .checkbox-holder a, [dir=ltr] .billet .checkbox-holder a:hover {
  margin-left: 4px; }
    [dir=rtl] .billet .checkbox-holder a, [dir=rtl] .billet .checkbox-holder a:hover {
      margin-right: 4px; }
  .billet .total-price .second {
    color: #1CBE69;
    font-size: 16px;
    font-weight: 600; }
  [dir] .billet .restricted-rate {
    text-align: center; }

.billet-wrapper {
  display: flex;
  justify-content: space-between;
  flex-direction: row-reverse;
  align-items: flex-start; }
  @media (max-width: 1139.5px) {
    .billet-wrapper {
      flex-direction: column; } }
  .billet-wrapper .another {
    flex-grow: 1; }
    @media (max-width: 1139.5px) {
      .billet-wrapper .another {
        width: 100%; } }
  .billet-wrapper .billet {
    min-width: 360px;
    max-width: 420px;
    width: 100%; }
  [dir=ltr] .billet-wrapper .billet {
  margin-left: 30px; }
  [dir=rtl] .billet-wrapper .billet {
    margin-right: 30px; }

.breadcrumbs {
  color: #777777;
  font-size: 14px;
  line-height: 17px;
  position: relative;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  font-weight: 300; }[dir] .breadcrumbs {
  margin-top: 40px;
  cursor: default; }
  [dir=ltr] .breadcrumbs .links {
  padding-right: 30px; }
  [dir=rtl] .breadcrumbs .links {
    padding-left: 30px; }
  .breadcrumbs a:hover {
    color: #231f20; }
  .breadcrumbs .back-button {
    min-width: 60px; }
  [dir] .breadcrumbs .back-button {
    cursor: pointer; }
  [dir=ltr] .breadcrumbs .back-button {
  text-align: right; }
  [dir=rtl] .breadcrumbs .back-button {
    text-align: left; }
    [dir=ltr] .breadcrumbs .back-button span {
  margin-right: 4px; }
    [dir=rtl] .breadcrumbs .back-button span {
      margin-left: 4px; }
    .breadcrumbs .back-button:hover {
      color: #231f20; }
  @media (max-width: 767.5px) {
    [dir] .breadcrumbs {
      margin-top: 30px; } }

.small-arrow-right,
.small-arrow-left {
  width: 5px;
  height: 5px;
  display: inline-block;
  line-height: 0;
  position: relative;
  top: -2px; }

[dir] .small-arrow-right, [dir] .small-arrow-left {
  border: 1px solid;
  border-top: 0;
  margin: 0 4px; }

[dir=ltr] .small-arrow-right, [dir=ltr] .small-arrow-left {
  border-left: 0;
  -webkit-transform: rotate(-45deg);
          transform: rotate(-45deg); }

[dir=rtl] .small-arrow-right, [dir=rtl] .small-arrow-left {
  border-right: 0;
  -webkit-transform: rotate(45deg);
          transform: rotate(45deg); }

[dir] .small-arrow-left {
  margin: 0; }

[dir=ltr] .small-arrow-left {
  -webkit-transform: rotate(135deg);
          transform: rotate(135deg); }

[dir=rtl] .small-arrow-left {
  -webkit-transform: rotate(-135deg);
          transform: rotate(-135deg); }

button {
  display: inline-block;
  min-width: 0;
  min-height: 0;
  outline: 0;
  font: inherit; }[dir] button {
  cursor: pointer;
  border: 0;
  background: transparent; }

.button {
  display: inline-flex;
  height: 54px;
  line-height: 20px;
  font-size: 13px;
  color: #231f20;
  justify-content: center;
  flex-direction: column;
  align-items: center; }

[dir] .button {
  border: 1px solid #e5e5e5;
  border-radius: 30px;
  cursor: pointer;
  background: #ffffff;
  padding: 3px 26px; }
  .button:hover {
    color: #FF9D19; }
  [dir] .button:hover {
    border-color: #FF9D19; }
  .button.main {
    color: #ffffff; }
  [dir] .button.main {
    background: #f9a51a;
    border: 0; }
  [dir=ltr] .button.main {
  background: -webkit-gradient(linear, left top, right top, from(#FFCD0B), to(#FBAC19));
  background: -webkit-linear-gradient(left, #FFCD0B 0%, #FBAC19 100%);
  background: linear-gradient(90deg, #FFCD0B 0%, #FBAC19 100%); }
  [dir=rtl] .button.main {
    background: -webkit-gradient(linear, right top, left top, from(#FFCD0B), to(#FBAC19));
    background: -webkit-linear-gradient(right, #FFCD0B 0%, #FBAC19 100%);
    background: linear-gradient(-90deg, #FFCD0B 0%, #FBAC19 100%); }
    [dir] .button.main:hover {
      background: #FF9D19; }
    [dir=ltr] .button.main:hover {
  background: -webkit-gradient(linear, left top, right top, from(#FF9D19), to(#F9A51B));
  background: -webkit-linear-gradient(left, #FF9D19 0%, #F9A51B 100%);
  background: linear-gradient(90deg, #FF9D19 0%, #F9A51B 100%); }
    [dir=rtl] .button.main:hover {
      background: -webkit-gradient(linear, right top, left top, from(#FF9D19), to(#F9A51B));
      background: -webkit-linear-gradient(right, #FF9D19 0%, #F9A51B 100%);
      background: linear-gradient(-90deg, #FF9D19 0%, #F9A51B 100%); }
  .button.gray {
    color: #b0b0b0; }
    .button.gray:hover {
      color: #f9a51a; }
  .button.green {
    color: #1CBE69; }
  [dir] .button.green {
    border-color: #1CBE69; }
    [dir] .button.green:hover {
      border-color: #f9a51a; }
  .button.transparent {
    text-transform: none;
    line-height: 30px;
    height: 30px;
    color: #ffffff;
    font-weight: 400; }
  [dir] .button.transparent {
    border: 1px solid #ffffff;
    background: transparent;
    padding: 0 15px; }
    .button.transparent:hover {
      color: #FF9D19; }
    [dir] .button.transparent:hover {
      background: #ffffff; }
  .button.round {
    min-width: 26px;
    min-height: 26px; }
  [dir] .button.round {
    border-radius: 300px !important;
    padding: 0 !important; }
    .button.round .icon {
      vertical-align: middle; }
  .button.small {
    height: auto;
    line-height: 14px;
    font-size: 10px;
    font-weight: 500; }
  [dir] .button.small {
    padding: 6px 18px; }
  .button.mini-label {
    font-size: 9px;
    height: 20px;
    line-height: 20px;
    display: inline-block;
    font-weight: 500; }
  [dir] .button.mini-label {
    cursor: default;
    padding: 0 8px; }
  .button.disabled, .button.disabled:hover, .button.disabled:active {
    color: #aaa; }
  [dir] .button.disabled, [dir] .button.disabled:hover, [dir] .button.disabled:active {
    cursor: default !important;
    background: #f2f2f2;
    border: 0; }
    .button.disabled.main, .button.disabled:hover.main, .button.disabled:active.main {
      color: #fff; }
    [dir] .button.disabled.main, [dir] .button.disabled:hover.main, [dir] .button.disabled:active.main {
      background: #d9d9d9; }

.file-upload {
  font-size: 14px; }

[dir] .file-upload {
  padding: 0 30px; }
  .file-upload input {
    display: none; }

.DayPicker {
  display: block;
  font-weight: 500;
  position: relative;
  line-height: 16px; }
  .DayPicker-wrapper {
    -webkit-user-select: none;
       -moz-user-select: none;
        -ms-user-select: none;
            user-select: none; }
  .DayPicker-NavButton {
    position: absolute;
    -webkit-filter: brightness(0);
            filter: brightness(0);
    box-sizing: content-box;
    width: 6px;
    height: 10px;
    top: -7px; }
  [dir] .DayPicker-NavButton {
    background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNiIgaGVpZ2h0PSIxMSIgdmlld0JveD0iMCAwIDYgMTEiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CiAgICA8cGF0aCBkPSJNNS43Mjc3MyA4LjYyMTY4QzYuMDg4NzIgOC45NzA2NCA2LjA4ODcyIDkuNTM2MTkgNS43Mzk3NyA5Ljg5NzE5QzUuNTU5MjcgMTAuMDc3NyA1LjMzMDY0IDEwLjE2MTkgNS4wODk5OCAxMC4xNjE5QzQuODYxMzUgMTAuMTYxOSA0LjYzMjcyIDEwLjA3NzcgNC40NjQyNiA5Ljg5NzE5TDAuMjY0NzI2IDUuNzIxNzJDMC4wOTYyNjM2IDUuNTUzMjUgLTIuMzI3NDdlLTA3IDUuMzI0NjMgLTIuMjIyMjdlLTA3IDUuMDgzOTZDLTIuMTE3MDhlLTA3IDQuODQzMyAwLjA5NjI2MzcgNC42MTQ2OCAwLjI2NDcyNiA0LjQ0NjIxTDQuNDY0MjYgMC4yNzA3NDRDNC44MTMyMiAtMC4wOTAyNDggNS4zNzg3NyAtMC4wOTAyNDc5IDUuNzM5NzcgMC4yNzA3NDRDNi4wODg3MiAwLjYxOTcwMiA2LjA4ODcyIDEuMTk3MjkgNS43Mjc3MyAxLjU0NjI1TDIuMTc3OTggNS4wODM5Nkw1LjcyNzczIDguNjIxNjhaIiBmaWxsPSIjZjlhNTFhIi8+Cjwvc3ZnPgo=) center center no-repeat;
    border: 12px solid transparent;
    cursor: pointer; }
    .DayPicker-NavButton:hover {
      -webkit-filter: none;
              filter: none; }
    .DayPicker-NavButton--interactionDisabled {
      display: none; }
    [dir=ltr] .DayPicker-NavButton--prev {
  left: 0; }
    [dir=rtl] .DayPicker-NavButton--prev {
      right: 0; }
    [dir] .DayPicker-NavButton--next {
      -webkit-transform: scaleX(-1);
              transform: scaleX(-1); }
    [dir=ltr] .DayPicker-NavButton--next {
  right: 0; }
    [dir=rtl] .DayPicker-NavButton--next {
      left: 0; }
  .DayPicker-Months {
    display: flex;
    justify-content: space-between; }
  .DayPicker-Caption {
    display: table-caption;
    font-size: 16px;
    line-height: 19px; }
  [dir] .DayPicker-Caption {
    text-align: center;
    padding-bottom: 16px; }
  .DayPicker-Month {
    display: table;
    border-spacing: 0;
    border-collapse: collapse; }
    @media (max-width: 767.5px) {
      .DayPicker-Month:last-child {
        display: none; } }
  .DayPicker-Weekdays {
    display: table-header-group; }
  .DayPicker-WeekdaysRow {
    display: table-row; }
  .DayPicker-Weekday abbr[title] {
    text-decoration: none; }
  [dir] .DayPicker-Weekday abbr[title] {
    border-bottom: none; }
  .DayPicker-Body {
    display: table-row-group; }
  .DayPicker-Week {
    display: table-row; }
  .DayPicker-Weekday, .DayPicker-Day {
    display: table-cell;
    width: 40px; }
  [dir] .DayPicker-Weekday, [dir] .DayPicker-Day {
    text-align: center; }
  .DayPicker-Weekday {
    color: #777777; }
  [dir] .DayPicker-Weekday {
    padding: 12px 5px; }
  .DayPicker-Day {
    vertical-align: middle;
    position: relative;
    z-index: 2;
    overflow: hidden; }
  [dir] .DayPicker-Day {
    cursor: pointer;
    background: white; }
    [dir] .DayPicker-Day div {
      margin: 3px 0;
      padding: 9px 5px; }
    .DayPicker-Day:hover div {
      color: #f9a51a !important; }
  .DayPicker-Day--disabled {
    color: #aaaaaa; }
  [dir] .DayPicker-Day--disabled {
    cursor: default; }
  .DayPicker-Day--selected:not(.DayPicker-Day--start):not(.DayPicker-Day--end):not(.DayPicker-Day--outside) div {
    color: #231f20; }
  [dir] .DayPicker-Day--selected:not(.DayPicker-Day--start):not(.DayPicker-Day--end):not(.DayPicker-Day--outside) div {
    background: #F4F4F4; }
  .DayPicker-Day--start:not(.DayPicker-Day--outside), .DayPicker-Day--end:not(.DayPicker-Day--outside) {
    position: relative; }
    .DayPicker-Day--start:not(.DayPicker-Day--outside):after, .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
      content: '';
      display: block;
      width: 17px;
      height: 34px;
      position: absolute;
      top: 3px;
      z-index: 1; }
    [dir] .DayPicker-Day--start:not(.DayPicker-Day--outside):after, [dir] .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
      background: #F4F4F4; }
    [dir=ltr] .DayPicker-Day--start:not(.DayPicker-Day--outside):after, [dir=ltr] .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
  right: 0; }
    [dir=rtl] .DayPicker-Day--start:not(.DayPicker-Day--outside):after, [dir=rtl] .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
      left: 0; }
    .DayPicker-Day--start:not(.DayPicker-Day--outside) div, .DayPicker-Day--end:not(.DayPicker-Day--outside) div {
      position: relative;
      z-index: 2;
      color: #F0F8FF !important; }
    [dir] .DayPicker-Day--start:not(.DayPicker-Day--outside) div, [dir] .DayPicker-Day--end:not(.DayPicker-Day--outside) div {
      border-radius: 50% !important;
      margin: 3px; }
    [dir=ltr] .DayPicker-Day--start:not(.DayPicker-Day--outside) div, [dir=ltr] .DayPicker-Day--end:not(.DayPicker-Day--outside) div {
  padding-left: 2px;
  padding-right: 2px;
  background: -webkit-gradient(linear, right top, left top, from(#FAAB19), to(#FECD0A));
  background: -webkit-linear-gradient(right, #FAAB19 0%, #FECD0A 100%);
  background: linear-gradient(270deg, #FAAB19 0%, #FECD0A 100%); }
    [dir=rtl] .DayPicker-Day--start:not(.DayPicker-Day--outside) div, [dir=rtl] .DayPicker-Day--end:not(.DayPicker-Day--outside) div {
      padding-right: 2px;
      padding-left: 2px;
      background: -webkit-gradient(linear, left top, right top, from(#FAAB19), to(#FECD0A));
      background: -webkit-linear-gradient(left, #FAAB19 0%, #FECD0A 100%);
      background: linear-gradient(-270deg, #FAAB19 0%, #FECD0A 100%); }
  [dir=ltr] .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
  left: 0;
  right: auto; }
  [dir=rtl] .DayPicker-Day--end:not(.DayPicker-Day--outside):after {
    right: 0;
    left: auto; }
  .DayPicker-Day--only:after {
    display: none !important; }

[dir="rtl"] .DayPicker-Weekday {
  padding-left: 1px;
  padding-right: 1px;
  font-size: 9px; }

[dir="rtl"] .DayPicker-NavButton--prev {
  -webkit-transform: scaleX(-1);
          transform: scaleX(-1); }

[dir="rtl"] .DayPicker-NavButton--next {
  -webkit-transform: scaleX(1);
          transform: scaleX(1); }

.filters h3 {
  font-size: 18px;
  line-height: 22px; }[dir] .filters h3 {
  margin-top: 23px;
  margin-bottom: 16px; }

.filters h4 {
  font-size: 13px;
  line-height: 16px;
  font-weight: 300;
  color: #777777; }

[dir] .filters h4 {
  margin-top: -6px;
  margin-bottom: 7px; }

[dir] .filters .checkbox {
  margin-bottom: 14px; }

[dir] .filters .price-range {
  margin-bottom: 50px; }

.filtration {
  display: flex;
  flex-wrap: wrap; }

[dir] .filtration {
  margin-bottom: 3px; }
  .filtration > .item {
    position: relative;
    display: inline-block; }
  [dir] .filtration > .item {
    margin-bottom: 10px; }
  [dir=ltr] .filtration > .item {
  margin-right: 10px; }
  [dir=rtl] .filtration > .item {
    margin-left: 10px; }
    .filtration > .item .button {
      flex-direction: row;
      font-weight: 400;
      -webkit-user-select: none;
         -moz-user-select: none;
          -ms-user-select: none;
              user-select: none; }
    [dir] .filtration > .item .button {
      padding: 0 18px; }
      .filtration > .item .button .icon {
        -webkit-filter: brightness(0.3);
                filter: brightness(0.3); }
        .filtration > .item .button .icon.icon-remove {
          -webkit-filter: grayscale(1);
                  filter: grayscale(1); }
      [dir=ltr] .filtration > .item .button.after-icon {
  padding-right: 12px; }
      [dir=rtl] .filtration > .item .button.after-icon {
        padding-left: 12px; }
        [dir=ltr] .filtration > .item .button.after-icon .icon {
  margin-left: 6px; }
        [dir=rtl] .filtration > .item .button.after-icon .icon {
          margin-right: 6px; }
      [dir=ltr] .filtration > .item .button.leading-icon .icon {
  margin-right: 8px; }
      [dir=rtl] .filtration > .item .button.leading-icon .icon {
        margin-left: 8px; }
      .filtration > .item .button:hover .icon {
        -webkit-filter: none;
                filter: none; }
    .filtration > .item .input, .filtration > .item .button {
      line-height: 36px !important;
      height: 36px !important; }
  .filtration .dropdown.select {
    min-width: 190px; }

.flag {
  width: 15px;
  height: 10px;
  position: relative;
  display: inline-block;
  font-size: 11px; }
  .flag span {
    outline: 1px solid rgba(128, 128, 128, 0.22); }
    .flag span.np {
      outline: 0; }
  .flag .flag-icon {
    width: 100%;
    height: 100%; }

footer {
  color: #777777;
  width: 100%;
  min-width: 200px;
  position: relative; }[dir] footer {
  background: #f7f8fc;
  padding-top: 35px; }
  footer a:hover, footer a[href]:hover {
    color: #231f20; }
  footer .company {
    flex-grow: 1; }
    footer .company .logo-wrapper {
      height: 68px;
      position: absolute;
      top: 0; }
    [dir=ltr] footer .company .logo-wrapper {
  text-align: left; }
    [dir=rtl] footer .company .logo-wrapper {
      text-align: right; }
      footer .company .logo-wrapper .logo {
        width: 100px;
        height: 53px;
        display: inline-block; }
      [dir] footer .company .logo-wrapper .logo {
        background: url("/images/logo/logo-small-gray.png");
        background-size: cover; }
        footer .company .logo-wrapper .logo:hover {
          -webkit-filter: brightness(0.75);
                  filter: brightness(0.75); }
  footer section {
    display: flex;
    justify-content: space-between; }
  footer .column {
    max-width: 265px;
    width: 100%; }
    footer .column h3 {
      font-size: 12px;
      text-transform: uppercase;
      color: #231f20; }
    [dir] footer .column h3 {
      margin-bottom: 19px; }
    footer .column li {
      font-size: 14px;
      line-height: 18px; }
    [dir] footer .column li {
      margin-bottom: 18px; }
    footer .column span {
      color: #231f20; }
  footer .payments {
    line-height: 0; }
  [dir] footer .payments {
    padding-bottom: 30px;
    margin-top: -60px; }
    footer .payments img {
      height: 40px; }
    [dir] footer .payments img {
      background: #ffffff; }
    [dir=ltr] footer .payments img {
  margin-right: 13px; }
    [dir=rtl] footer .payments img {
      margin-left: 13px; }
      [dir=ltr] footer .payments img.interval {
  margin-right: 18px; }
      [dir=rtl] footer .payments img.interval {
        margin-left: 18px; }
      [dir=ltr] footer .payments img.interval-big {
  margin-right: 25px; }
      [dir=rtl] footer .payments img.interval-big {
        margin-left: 25px; }
      [dir=ltr] footer .payments img.near {
  margin-right: 5px; }
      [dir=rtl] footer .payments img.near {
        margin-left: 5px; }
      [dir] footer .payments img.transparent {
        background: transparent; }
      [dir=ltr] footer .payments img:last-child {
  margin-right: 0; }
      [dir=rtl] footer .payments img:last-child {
        margin-left: 0; }
  footer .copyright {
    line-height: 14px;
    font-weight: 300;
    font-size: 12px; }
  [dir] footer .copyright {
    padding-top: 20px;
    padding-bottom: 20px; }
  footer .service-info {
    font-size: 10px; }
    footer .service-info span {
      color: #777777; }
    [dir=ltr] footer .service-info span {
  margin-right: 5px; }
    [dir=rtl] footer .service-info span {
      margin-left: 5px; }
  @media (max-width: 1139.5px) {
    [dir] footer {
      padding-top: 90px; }
    [dir=ltr] footer {
    padding-left: 20px;
    padding-right: 20px; }
    [dir=rtl] footer {
      padding-right: 20px;
      padding-left: 20px; }
      footer .company {
        max-width: 0; }
      footer .middle {
        flex-direction: column;
        justify-content: flex-start; }
      footer .copyright {
        display: flex;
        flex-direction: column-reverse; }
      [dir] footer .copyright {
        margin: 0 auto; }
        [dir] footer .copyright div {
          margin-bottom: 20px; }
      [dir=ltr] footer .company .logo-wrapper {
    left: 50%;
    -webkit-transform: translateX(-50%);
            transform: translateX(-50%); }
      [dir=rtl] footer .company .logo-wrapper {
        right: 50%;
        -webkit-transform: translateX(50%);
                transform: translateX(50%); }
      [dir] footer .payments {
        margin-top: 20px;
        padding-bottom: 10px; } }
  @media (max-width: 767.5px) {
    footer section {
      flex-direction: column; }
    [dir] footer .column {
      margin-bottom: 30px; }
      footer .column:nth-child(2) {
        order: 1; }
    footer ul {
      justify-content: center;
      flex-wrap: wrap; }
      [dir=ltr] footer ul li:last-child {
    margin-right: 0 !important; }
      [dir=rtl] footer ul li:last-child {
        margin-left: 0 !important; }
    footer .copyright {
      max-width: 300px; }
    [dir] footer .payments {
      text-align: center; }
    footer .contact {
      width: 280px; }
    [dir] footer .contact {
      margin: 40px auto 0; } }

.gallery {
  width: 100%;
  height: 414px;
  display: flex;
  justify-content: space-between;
  position: relative; }
  .gallery .big {
    width: 49.5%;
    height: 400px;
    position: relative;
    overflow: hidden; }
  [dir] .gallery .big {
    margin-top: 5px; }
  [dir=ltr] .gallery .big {
  border-top-left-radius: 50px; }
  [dir=rtl] .gallery .big {
    border-top-right-radius: 50px; }
    @media (max-width: 767.5px) {
      .gallery .big {
        display: none; } }
  .gallery .thumbs {
    height: 400px;
    width: 49.5%;
    overflow: hidden;
    display: flex;
    position: relative; }
    @media (max-width: 767.5px) {
      .gallery .thumbs {
        width: 100%; } }
    .gallery .thumbs.scroll {
      overflow-x: scroll;
      height: 423px;
      flex-direction: column;
      flex-wrap: wrap; }
      .gallery .thumbs.scroll::-webkit-scrollbar {
        height: 5px;
        line-height: 0; }
      [dir] .gallery .thumbs.scroll::-webkit-scrollbar-track {
        background: transparent; }
      [dir] .gallery .thumbs.scroll::-webkit-scrollbar-thumb {
        background-color: #f9a51a; }
    .gallery .thumbs .item {
      display: inline-block;
      position: relative;
      min-width: 49%;
      max-width: 49%;
      height: 195px; }
    [dir] .gallery .thumbs .item {
      cursor: pointer; }
    [dir=ltr] .gallery .thumbs .item {
  margin: 5px 2% 5px 0; }
    [dir=rtl] .gallery .thumbs .item {
      margin: 5px 0 5px 2%; }
      [dir] .gallery .thumbs .item:last-child {
        margin-bottom: 0; }
      @media (min-width: 768px) {
        .gallery .thumbs .item:nth-child(1) {
          order: 1; } }
  .gallery .corner {
    position: absolute;
    top: 355px;
    width: 50px;
    height: 50px;
    overflow: hidden; }
  [dir=ltr] .gallery .corner {
  right: 0; }
  [dir=rtl] .gallery .corner {
    left: 0; }
    .gallery .corner div {
      position: absolute;
      bottom: 0;
      width: 100px;
      height: 100px; }
    [dir] .gallery .corner div {
      background: transparent;
      border-radius: 50px;
      box-shadow: 0 0 0 30px #fff; }
    [dir=ltr] .gallery .corner div {
  right: 0; }
    [dir=rtl] .gallery .corner div {
      left: 0; }
  .gallery .sizer {
    width: 100%;
    height: 100%;
    justify-content: center;
    align-items: center;
    overflow: hidden; }
  [dir] .gallery .sizer {
    background: #F0F5F9 center center no-repeat;
    background-size: cover; }

header {
  z-index: 1001;
  position: fixed;
  width: 100%;
  height: 80px;
  line-height: 22px;
  font-size: 14px;
  -webkit-transition: box-shadow 0.2s;
  transition: box-shadow 0.2s; }[dir] header {
  background: #ffffff;
  box-shadow: 0 .5px 0 #ffffff; }
  header.fixed {
    -webkit-transition: box-shadow 1.2s;
    transition: box-shadow 1.2s; }
  [dir] header.fixed {
    box-shadow: 0 0.5px 0 #B9B9B9; }
  header a {
    display: inline-block;
    vertical-align: top; }
  header section {
    display: flex;
    justify-content: space-between; }
  header .search-wrapper {
    position: relative; }
  [dir] header .search-wrapper {
    margin-top: 16px; }
  header .agent-menu {
    min-width: 240px; }
  [dir] header .agent-menu {
    padding: 17px 0; }
  [dir=ltr] header .agent-menu {
  text-align: right; }
  [dir=rtl] header .agent-menu {
    text-align: left; }
    header .agent-menu .button {
      height: 46px; }
      [dir] header .agent-menu .button:hover {
        border-color: #f9a51a;
        box-shadow: none; }
  header .agent-link {
    display: inline-block;
    height: 46px; }
  [dir] header .agent-link {
    padding: 5px; }
  [dir=ltr] header .agent-link {
  margin-left: 14px; }
  [dir=rtl] header .agent-link {
    margin-right: 14px; }
    [dir] header .agent-link .icon {
      margin: 13px 9px 0 9px; }
    [dir=ltr] header .agent-link .icon {
  float: left; }
    [dir=rtl] header .agent-link .icon {
      float: right; }
    header .agent-link:hover .icon {
      -webkit-filter: none;
              filter: none; }
    [dir] header .agent-link .avatar {
      background: url(data:image/svg+xml;base64,Cjxzdmcgd2lkdGg9IjMwIiBoZWlnaHQ9IjMwIiB2aWV3Qm94PSIwIDAgMzAgMzAiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPgo8bWFzayBpZD0ibWFzazAiIG1hc2stdHlwZT0iYWxwaGEiIG1hc2tVbml0cz0idXNlclNwYWNlT25Vc2UiIHg9IjAiIHk9IjAiIHdpZHRoPSIzMCIgaGVpZ2h0PSIzMCI+CjxjaXJjbGUgY3g9IjE1IiBjeT0iMTUiIHI9IjE0LjUiIGZpbGw9IiNDNEM0QzQiIHN0cm9rZT0iI0Q4REFEQyIvPgo8L21hc2s+CjxnIG1hc2s9InVybCgjbWFzazApIj4KPGNpcmNsZSBjeD0iMTUiIGN5PSIxNSIgcj0iMTQuNSIgc3Ryb2tlPSIjRDhEQURDIi8+CjxyZWN0IHg9Ii00LjE2NjYzIiB5PSItNC4xNjY2OSIgd2lkdGg9IjM4LjMzMzMiIGhlaWdodD0iMzcuNSIgZmlsbD0idXJsKCNwYXR0ZXJuMCkiLz4KPC9nPgo8ZGVmcz4KPHBhdHRlcm4gaWQ9InBhdHRlcm4wIiBwYXR0ZXJuQ29udGVudFVuaXRzPSJvYmplY3RCb3VuZGluZ0JveCIgd2lkdGg9IjEiIGhlaWdodD0iMSI+Cjx1c2UgeGxpbms6aHJlZj0iI2ltYWdlMCIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoMCAtMC4wMDE4NTU1Nikgc2NhbGUoMC4wMDIwMTIwNykiLz4KPC9wYXR0ZXJuPgo8aW1hZ2UgaWQ9ImltYWdlMCIgd2lkdGg9IjQ5NyIgaGVpZ2h0PSI0ODgiIHhsaW5rOmhyZWY9ImRhdGE6aW1hZ2UvcG5nO2Jhc2U2NCxpVkJPUncwS0dnb0FBQUFOU1VoRVVnQUFBZkVBQUFIb0NBWUFBQUJaNjlZdUFBQWdBRWxFUVZSNEFlM2RDVk1ieTdZbTBFUUl4SXlOejNTbjF4MzkvMy9TaXhjOXZMakR1VDYyR1FWQ2lJNWRXSGlTUUVBaFZXYXVDaE5nRFZXWmF5ZDhxbm50UC8vUDc3ZkpSSUFBQVFJRUNHUW4wTXV1eFJwTWdBQUJBZ1FJTkFKQzNFQWdRSUFBQVFLWkNnanhUQXVuMlFRSUVDQkFRSWdiQXdRSUVDQkFJRk1CSVo1cDRUU2JBQUVDQkFnSWNXT0FBQUVDQkFoa0tpREVNeTJjWmhNZ1FJQUFBU0Z1REJBZ1FJQUFnVXdGaEhpbWhkTnNBZ1FJRUNBZ3hJMEJBZ1FJRUNDUXFZQVF6N1J3bWsyQUFBRUNCSVM0TVVDQUFBRUNCRElWRU9LWkZrNnpDUkFnUUlDQUVEY0dDQkFnUUlCQXBnSkNQTlBDYVRZQkFnUUlFQkRpeGdBQkFnUUlFTWhVUUlobldqak5Ka0NBQUFFQ1F0d1lJRUNBQUFFQ21Rb0k4VXdMcDlrRUNCQWdRRUNJR3dNRUNCQWdRQ0JUQVNHZWFlRTBtd0FCQWdRSUNIRmpnQUFCQWdRSVpDb2d4RE10bkdZVElFQ0FBQUVoYmd3UUlFQ0FBSUZNQllSNHBvWFRiQUlFQ0JBZ0lNU05BUUlFQ0JBZ2tLbUFFTSswY0pwTmdBQUJBZ1NFdURGQWdBQUJBZ1F5RlJEaW1SWk9zd2tRSUVDQWdCQTNCZ2dRSUVDQVFLWUNRanpUd21rMkFRSUVDQkFRNHNZQUFRSUVDQkRJVkVDSVoxbzR6U1pBZ0FBQkFrTGNHQ0JBZ0FBQkFwa0tDUEZNQzZmWkJBZ1FJRUJBaUJzREJBZ1FJRUFnVXdFaG5tbmhOSnNBQVFJRUNBaHhZNEFBQVFJRUNHUXFJTVF6TFp4bUV5QkFnQUFCSVc0TUVDQkFnQUNCVEFXRWVLYUYwMndDQkFnUUlDREVqUUVDQkFnUUlKQ3BnQkRQdEhDYVRZQUFBUUlFaExneFFJQUFBUUlFTWhVUTRwa1dUck1KRUNCQWdJQVFOd1lJRUNCQWdFQ21Ba0k4MDhKcE5nRUNCQWdRRU9MR0FBRUNCQWdReUZSQWlHZGFPTTBtUUlBQUFRSkMzQmdnUUlBQUFRS1pDZ2p4VEF1bjJRUUlFQ0JBUUlnYkF3UUlFQ0JBSUZNQklaNXA0VFNiQUFFQ0JBZ0ljV09BQUFFQ0JBaGtLaURFTXkyY1poTWdRSUFBQVNGdURCQWdRSUFBZ1V3RmhIaW1oZE5zQWdRSUVDQWd4STBCQWdRSUVDQ1FxWUFRejdSd21rMkFBQUVDQklTNE1VQ0FBQUVDQkRJVkVPS1pGazZ6Q1JBZ1FJQ0FFRGNHQ0JBZ1FJQkFwZ0pDUE5QQ2FUWUJBZ1FJRUJEaXhnQUJBZ1FJRU1oVVFJaG5XampOSmtDQUFBRUNRdHdZSUVDQUFBRUNtUW9JOFV3THA5a0VDQkFnUUVDSUd3TUVDQkFnUUNCVEFTR2VhZUUwbXdBQkFnUUlDSEZqZ0FBQkFnUUlaQ29neERNdG5HWVRJRUNBQUFFaGJnd1FJRUNBQUlGTUJZUjRwb1hUYkFJRUNCQWdJTVNOQVFJRUNCQWdrS21BRU0rMGNKcE5nQUFCQWdTRXVERkFnQUFCQWdReUZSRGltUlpPc3drUUlFQ0FnQkEzQmdnUUlFQ0FRS1lDUWp6VHdtazJBUUlFQ0JBUTRzWUFBUUlFQ0JESVZFQ0laMW80elNaQWdBQUJBa0xjR0NCQWdBQUJBcGtLQ1BGTUM2ZlpCQWdRSUVCQWlCc0RCQWdRSUVBZ1V3RWhubW5oTkpzQUFRSUVDQWh4WTRBQUFRSUVDR1FxSU1RekxaeG1FeUJBZ0FBQklXNE1FQ0JBZ0FDQlRBV0VlS2FGMDJ3Q0JBZ1FJQ0RFalFFQ0JBZ1FJSkNwZ0JEUHRIQ2FUWUFBQVFJRWhMZ3hRSUFBQVFJRU1oVVE0cGtXVHJNSkVDQkFnSUFRTndZSUVDQkFnRUNtQWtJODA4SnBOZ0VDQkFnUUVPTEdBQUVDQkFnUXlGUkFpR2RhT00wbVFJQUFBUUpDM0JnZ1FJQUFBUUtaQ2dqeFRBdW4yUVFJRUNCQVFJZ2JBd1FJRUNCQUlGTUJJWjVwNFRTYkFBRUNCQWdJY1dPQUFBRUNCQWhrS2lERU15MmNaaE1nUUlBQUFTRnVEQkFnUUlBQWdVd0ZoSGltaGROc0FnUUlFQ0FneEkwQkFnUUlFQ0NRcVlBUXo3UndtazJBQUFFQ0JJUzRNVUNBQUFFQ0JESVZFT0taRms2ekNSQWdRSUNBRURjR0NCQWdRSUJBcGdKQ1BOUENhVFlCQWdRSUVCRGl4Z0FCQWdRSUVNaFVRSWhuV2pqTkprQ0FBQUVDUXR3WUlFQ0FBQUVDbVFvSThVd0xwOWtFQ0JBZ1FFQ0lHd01FQ0JBZ1FDQlRBU0dlYWVFMG13QUJBZ1FJQ0hGamdBQUJBZ1FJWkNvZ3hETXRuR1lUSUVDQUFJRStBZ0lFOGhGWVgrK2xqZjU2ODlYdjkxSnZyWmQ2dmJYbWEyMXRMZlhXMWhib3pHMmFURzdUNVBiTDk1dWJTUnFQYjlMMStPNTdQR2NpUUtEN0FrSzgrelhTd2tvRit1dTl0TDIxbWJhM050SmdzNS82L2ZVRlEvcmxZQkhxMXpjMzZmTHlPbDBNUjJsNGRmM3ltWm9EQVFLdEN3angxa25Oa01EekJHS05lbnV3a2JZaXVBZDN3ZjI4T2IzOFhiSEdIMTlibXh2cHpjRk9zK1orY1RscVFqMENmWFE5ZnZsQ3pJRUFnUmNMQ1BFWEU1b0JnWmNKN0d4dnB0M3RRZHJkMlV6cnZXNGVwaElmTVBaMkJzMVg5UFp5ZExlR0htdnBWeU9CL3JJUjROMEVuaThneEo5djU1MEVYaVFRbThvUDk3YlM3czdnUmZOWnhadGpEVDIramc1MzAvSHBNSjJjRGRQbyttWVZUYkZNQWxVTENQR3F5Ni96cXhEWUdteWt3NzN0dExlYlgzalA4anJjMzA3N2UxdnA1SFNZanM4dW13UGtacjNPWXdRSXRDOGd4TnMzTlVjQ013WGl3TFMzQnp2cFlHOXI1dk01UHhoSHhjZSs4LzNkclhSOE5rd2ZqeTl5N282MkU4aEdRSWhuVXlvTnpWa2d3dTNvY0tjNXdqem5manpXOWpnWUxqYXh4NEY1SDQ0djBxV2oyaDhqOHp5QkZ3a0k4UmZ4ZVRPQmh3Vmk3VHZDTzBLOHBpbjI5Lzg1Z3Z6VGVmcDBPcXlwNi9wS1lLa0NRbnlwM0JaV2swQXRhOS96YWhvWG4zbjNkaS9GTVFBZmpzOGQrRFlQeXVNRVhpQWd4RitBNTYwRVpnblV1dlk5eXlJZWk2UHZwMEYrY25ZNTcyVWVKMERnR1FKQy9CbG8za0pnbmtDY1MvM3V6Vzd4Kzc3bjlYL2U0N0d2L09lai9TYk0vLzNoTE4yNnJPczhLbzhUZUpLQUVIOFNseGNUbUM4UXAxbjljclEvL3dXZWFZNE5pR3UvLy9QOVNZcEx1NW9JRUhpWlFEY3ZEL1d5UG5rM2dhVUx4TG5TQW53eDl0aTAvcmZmM3FiTmpmWEYzdUJWQkFqTUZSRGljMms4UVdBeGdWL2U3YWVmM3U0dDltS3ZhZ1JpOC9yZi9uVFViRjVIUW9EQTh3V0UrUFB0dkpOQSt2TXZiNm83ZmF6TnN2L2wxemZOSGRyYW5LZDVFYWhKUUlqWFZHMTliVlhnVDc4Y05yY0piWFdtRmM3c3I3KzlUUnMyclZkWWVWMXVRMENJdDZGb0h0VUovUHJ1SU8xc2JWYlg3OWZxOEgvODZhaTU5ZWxyemQ5OENaUXFJTVJMcmF4K3ZacEE3UDh1NWVZbHI0YjBqQm4vejcrOFMzR0JHQk1CQW9zTENQSEZyYnlTUUhPVGp6Z1MzZlE2QXYvcmJ6Kzl6b3pObFVDaEFrSzgwTUxxVnZzQ3U5dDNGM0pwZjg3bStMVkFIT3htSWtCZ01RRWh2cGlUVjFVdUVPYzAvL1IydDNLRjVYUS96aU9QcTk2WkNCQjRYRUNJUDI3a0ZRU2FHM25FTmRGTnl4R0llNVBiYmJFY2EwdkpXMENJNTEwL3JWK0NRQnpJNWtqMEpVQi90NGh3ciswV3J0OFIrQytCUndXRStLTkVYbEN6UUlTSU5jTFZqWUIzYjNlZFE3NDZma3ZPUUVDSVoxQWtUVnlOUUgrOWw5NGU3cXhtNFpiYUNLejNldW50Z1JvWURnVG1DUWp4ZVRJZXIxN2c3ZUZ1aWp0dW1WWXJFRnRENHA3a0pnSUVmaFFRNGorYWVJUkFpdnVDSCt4dGtlaUlRS3lOdXc1TVI0cWhHWjBTRU9LZEtvZkdkRUdnMllSck0zb1hTbkhmaHNGbXY3blF6djBEZmlCQW9CRVE0Z1lDZ2U4RVlqLzQ1a2IvdTBmOWQ5VUNjZHBaaExtSkFJRXZBa0w4aTRXZkNEVGhmZUN5cXAwY0NiMjF0WFN3NTVLM25TeU9ScTFNUUlpdmpONkN1eWl3dnp0SWJzSFJ4Y3JjdFNtT1U3QTIzdDM2YU5ueUJZVDQ4czB0c2FNQ2NVclp2b1BaT2xxZEw4MnlOdjdGd2s4RWhMZ3hRT0N6UUFSNEhOUm02cmFBdGZGdTEwZnJsaXZnTDlaeXZTMnRvd0t4djlVbFBqdGFuQm5Oc2pZK0E4VkRWUW9JOFNyTHJ0UGZDOFJhdUF1N2ZLL1MzZjliRys5dWJiUnN1UUpDZkxuZWx0WlJBV3ZoSFMzTUE4MVNzd2R3UEZXTmdCQ3ZwdFE2T2s4ZzdsL3RpT2Q1T3QxOWZHZDcwNWtFM1MyUGxpMUpRSWd2Q2RwaXVpc1FZV0RLVHlCMmYreHN1Nlo2ZnBYVDRqWUZoSGlibXVhVnBjRHVsaERQc25BcEpSL0FjcTJjZHJjbElNVGJralNmTEFXMnR6YlNwa3Q1WmxtN2FIU0VlSnhaWUNKUXE0QVFyN1h5K3QwSTJCeWI5MENJQy9SWUc4KzdobHIvTWdFaC9qSS83ODVjd0tiMHpBdG9rM3IrQmRTREZ3a0k4UmZ4ZVhQT0FqdGJtMmxqWXozbkxtajdkSk42enlaMWc2Rk9BU0ZlWjkzMU9pWDd3Z3NaQlhHcDNJRmJ4eFpTVGQxNHFvQVFmNnFZMXhjajRBOS9NYVYwbm44NXBkU1RKd29JOFNlQ2VYazVBb05CdjV6T1ZONFRaeGhVUGdBcTdyNFFyN2o0TlhjOWptcDJyZlJ5UnNCZ2M2T2N6dWdKZ1NjSUNQRW5ZSGxwT1FMVzNNcXBaZlJrYzJQZGJXVExLcW5lTENnZ3hCZUU4ckt5Qkxhc3VaVlYwSlNTM1NQRmxWU0hGaEFRNGdzZ2VVbDVBdjdnRjFoVFY5NHJyNmg2OUtpQUVIK1V5QXRLRkxBbVhsNVY3UmN2cjZaNjlMaUFFSC9jeUNzS0U0anppbnN1RGxKWVZWT0tneFZOQkdvVE1PcHJxN2orcG43ZnNDOXhHS3dMOFJMTHFrK1BDUGhyOWdpUXA4c1Q4TWUrdkpwR2o5WnRYU216c0hyMW9JQVFmNURIa3lVSzJPeGFZbFZUV2x0YnM1dWt6TkxxMVFNQ1F2d0JIRStWS2RCZmQ5T1RNaXNiYStQK3BKVmFXLzJhTFdERXozYnhhTUVDOW9tWFcxd2hYbTV0OVd5MmdCQ2Y3ZUxSZ2dWc1RpKzN1TDExdHlRdHQ3cDZOa3RBaU05UzhWalJBdXMycHhkYlgydml4WlpXeCtZSUNQRTVNQjR1VjhDYWVMbTFkZVpCdWJYVnM5a0NRbnkyaTBjTEZvaWptRTFsQ2lodG1YWFZxL2tDUW55K2pXY0tGWmpjVGdydG1XN2QzaklnVUplQUVLK3IzbnFiVXJxZCtFdGY2a0M0bGVLbGxsYS81Z2dJOFRrd0hpNVhZT0lQZmJuRjlmbXMzTnJxMlV3QklUNlR4WU1sQzB5c2lSZGIzdHNreFlzdHJvN05GQkRpTTFrOFdMS0FUYTdsVnRkR2xuSnJxMmV6QllUNGJCZVBGaXhnVGJ6YzR2cUFWbTV0OVd5MmdCQ2Y3ZUxSZ2dYc0V5KzN1RGFtbDF0YlBac3RJTVJudTNpMFlBRWhYbTV4clltWFcxczlteTBneEdlN2VMUmdBYWVZbFZ0Yys4VExyYTJlelJZUTRyTmRQRnF3d0kyajA0dXRyZzlveFpaV3grWUlDUEU1TUI0dVYyQTh2aW0zYzVYM2JEUWVWeTZnKzdVSkNQSGFLcTYvYVh6anNxc2xEb1BZSCs3TWd4SXJxMDhQQ1FqeGgzUThWNlRBYUdSdHJjVENqcTV0WVNteHJ2cjBzSUFRZjlqSHN3VUt4Tkhwam1JdXI3RFhkcE9VVjFROWVsUkFpRDlLNUFVbENvekhOcW1YVnRmUnRTMHNwZFZVZng0WEVPS1BHM2xGZ1FMWE56YTlsbFpXYStLbFZWUi9GaEVRNG9zb2VVMXhBbzVRTDY2a2FYeHQ2MHA1VmRXanh3U0UrR05Dbmk5UzRHcGtUYnkwd2pxOXJMU0s2czhpQWtKOEVTV3ZLVTdBL3RQaVN1cjBzdkpLcWtjTENBanhCWkM4cER3QnA1bVZWVk5uRzVSVlQ3MVpYS0MvK0V1OWtrQytBaHY5OWZUdTdWN2EzZDVzT25GNmZwbHZaN1Q4QjRFNFIveXZ2NzFOZzgyN1AybkhwOFAwL3VQWkQ2L3pBSUhTQklSNGFSWFZueDhFMXRiVzBzOUgrMmw3YStQK3VmM2RyZnVmL1pDL3dEUzhwejA1M045T2NiUjZoTG1KUU1rQ05xZVhYRjE5YXdSaTdmdnJBTWRTaDhEaDNuWWRIZFhMcWdXRWVOWGxyNlB6R3h2cmRYUlVMNzhSaUxwL3Y0Yit6UXY4aDBBQkFrSzhnQ0xxd3NNQ2c4MHZtOUVmZnFWblN4UFlHcWg5YVRYVm4yOEZoUGkzSHY1WG9FQnZiYTNBWHVuU0lnSzludG92NHVRMStRb0k4WHhycCtVRUNCQWdVTG1BRUs5OEFPZytBUUlFQ09RcklNVHpyWjJXRXlCQWdFRGxBa0s4OGdGUVEvZmovdUdtT2dWY3lhM091dGZVYXlGZVU3VXI3ZXZWNkxyU251djIxY2c5eG8yQ3NnV0VlTm4xMWJ1VWtqL2s5UTREdGErMzlyWDBYSWpYVXVtSysra1BlWjNGajh1dVRpWjJwZFJaL1hwNkxjVHJxWFcxUGIyNW1hVHh6YVRhL3RmYWNSL2VhcTE4WGYwVzRuWFZ1OXJlK29OZVgra2RDMUZmeld2c3NSQ3ZzZW9WOXRrZjlQcUs3b05iZlRXdnNjZEN2TWFxVjlqbjRhVWoxR3NxZTV4YXB1WTFWYnpldmdyeGVtdGZWYzh2cjY0ZDVGUlJ4YVBlSmdJMUNBanhHcXFzajQzQXhlV0lSQ1VDRjdhOFZGSnAzUlRpeGtBMUFqYXZWbFBxTlBTQnJaNWlWOTVUSVY3NUFLaXAremF4MWxIdE9EL2NRVzExMUZvdlV4TGlSa0UxQXFQcmNScGQzMVRUMzFvNzZzTmFyWld2czk5Q3ZNNjZWOXZyaStGVnRYMnZwZVBuRjQ1OXFLWFcrbWxOM0Jpb1RPRDBRb2lYWFBMWUYzN3VnMXJKSmRhMzd3U3NpWDhINHI5bEM0eEc0M1FteUlzdDhzblpaYkY5MHpFQ3N3U0UrQ3dWanhVdGNIWnViYnpFQXNlK2NCL1FTcXlzUGowa0lNUWYwdkZja1FLeHVkWFJ5K1dWOXZUY1duaDVWZFdqeHdTRStHTkNuaTlTNE1QeGVaSDlxclZUY1NFZm05SnJyWDdkL1JiaWRkZS8ydDVmREVmcDA4bEZ0ZjB2cmVOcVdWcEY5V2RSQVNHK3FKVFhGU2Z3NGRONUdyckdkdloxalFCM05iN3N5NmdEenhRUTRzK0U4N2I4Qlc1VFNuOThQRXZqbTBuK25hbTBCN0ZGSlQ2TW1RalVLaURFYTYyOGZqY0NjWURiNzMrY3B0dElkRk5XQWszdFBwd21wY3VxYkJyYnNvQVFieG5VN1BJVGlBdUUvT3Y5c1UyeUdaVXVEbUw3L1krVGRHTXJTa1pWMDlUWEVPaS94a3pOazBCdUF1ZkR1TkxYS08zdmJxWGQ3YzIwdWRGUC9YNHZyYTJ0NWRhVkl0czdtZHltdVBaOUhJVWVtOUNkSWxoa21YWHFHUUpDL0JsbzNsS3VRSnhyUEQzZitMZWZENXRBTDdlMytmVHM0OGw1K25ReXpLZkJXa3BnU1FJMnB5OEoybUx5RTVoTUhQRFdsYXFOeDJyUmxWcG9SN2NFaEhpMzZxRTFIUkt3djdVN3hiaVpPSHl0TzlYUWtpNEpDUEV1VlVOYk9pWGdIUEx1bE9OcWROMmR4bWdKZ1E0SkNQRU9GVU5UdWlVUU45UXdyVjRnNmhBSHRwa0lFUGhSUUlqL2FPSVJBbzFBQkljZ1gvMWdzRVZrOVRYUWd1NEtDUEh1MWtiTE9pQVFwelNaVmlzUTUvR2JDQkNZTFNERVo3dDRsRUFqY0hVMUpyRkNnVGk0MEhYUlYxZ0FpKzY4Z0JEdmZJazBjSlVDbDZQcmRPdWFyQ3NyZ1UzcEs2TzM0RXdFaEhnbWhkTE0xUWpFZm5GQnNocjdXS3ExOE5YWlczSWVBa0k4anpwcDVRb0ZITnkyR3Z6WUFHSi8rR3JzTFRVZkFTR2VUNjIwZEVVQzV4ZFhLMXB5M1lzOUgxNmw2L0ZOM1FoNlQrQVJBU0grQ0pDbkNZeXViOUtaSUYvNlFJZ1FOeEVnOExDQUVIL1l4N01FR2dGcjQ4c2RDT09iU2JxNGNHclpjdFV0TFVjQklaNWoxYlI1NlFLeEpoNjN3alF0UnlBK05FMmNGYkFjYkV2SldrQ0laMTAralYrbWdFM3F5OU9lM2c1MmVVdTBKQUo1Q2dqeFBPdW0xU3NRT0QyN1REZHVUL3JxOGlkbmwrbHFaS3ZIcTBOYlFCRUNRcnlJTXVyRU1nUmlQMjBFdWVsMUJVN09ocSs3QUhNblVKQ0FFQytvbUxyeStnS3hsbWhmN2VzNVd3dC9QVnR6TGxOQWlKZFpWNzE2SllFNGI5bmErQ3ZocHBTc2hiK2VyVG1YS1NERXk2eXJYcjJpd01tNVRlcXZ3WHQ4T3JRdi9EVmd6Yk5vQVNGZWRIbDE3alVFUnFOeGlzQXh0U2NRZHl0ajJwNm5PZFVqSU1UcnFiV2V0aWp3OGZnaXhaWGNUTzBJZkRxOWNJblZkaWpOcFRJQklWNVp3WFczSFlFNDFlemp5WGs3TTZ0OExuR0RtVThudG14VVBneDAvNWtDUXZ5WmNONUc0T3o4S3Jrb3ljdkh3U2U3Smw2T2FBN1ZDZ2p4YWt1djQyMEl4R2IxMko5cmVwNUFmQkJ5WGZybjJYa1hnUkFRNHNZQmdSY0l4Q2xuRWVTbXB3dkVoNStQcCt5ZUx1Y2RCTDRJQ1BFdkZuNGk4Q3lCNDdPaHRjbG55UDN4NlR6RmtmNG1BZ1NlTHlERW4yL25uUVR1QmQ1L09rL1hqbGEvOTNqc2h6aWR6UEVFanlsNW5zRGpBa0w4Y1NPdklQQ293SGg4azk1L09udjBkVjZRVWh5Ti91R1RJL3VOQlFKdENBanhOaFROZzBCSzZXSTRTckdKMkRSZjRQYjJ0akZ5L2ZuNVJwNGg4QlFCSWY0VUxhOGw4SWpBcDVNTG00a2ZNSW9QT2JFbWJpSkFvQjBCSWQ2T283a1F1QmQ0Ly9ITU5jRHZOYjc4RVB2QVhWcjFpNGVmQ0xRaElNVGJVRFFQQWw4SlRDYTNLWUxjSnVNdktHY1hWK24zUDA2L1BPQW5BZ1JhRVJEaXJUQ2FDWUZ2QldLVHNiWE9PNVB6NFNqOTYvM0p0MEQrUjRCQUt3SkN2QlZHTXlId28wQWNnWDF5VnZkdFM2OUc0L1RQZngvL2lPTVJBZ1JhRVJEaXJUQ2FDWUhaQXYvK2NGcnRoV0RpSmpILy9jK1BzMkU4U29CQUt3SkN2QlZHTXlFd1grQ2Y3MC9TOExLdUk3TGpWTEwvL2Q5L3pFZnhEQUVDclFnSThWWVl6WVRBd3dKLy8vMVRHbDZPSG41UkljL0c5ZVQvNi8rOUw2UTN1a0dnMndKQ3ZOdjEwYnFDQlA3KyszSHhCN3ZGQjVYLysvY1BCVlZOVndoMFc2RGY3ZVpwSFlHeUJPTFVzOWhYZkhTNFcxYkhVa3FuWjVmcDl3OU9JeXV1c0RyVWFRRWgzdW55YUZ5SkF0TmJsNVlVNUIrT3o5MlN0Y1RCcWsrZEZ4RGluUytSQnBZb0VFRWUrNDUvZnJ1ZmVyMjFiTHM0dnBta1B6NmVwYmlZaTRrQWdlVUxDUEhsbTFzaWdVYmc3UHdxamE4bjZkM1JidHJhM01oT0pZNjRqOTBEbzJ2M0JNK3VlQnBjaklBRDI0b3BwWTdrS0hBNXVrNy8rUDA0dTV1bXhOWG8vdkh2VHdJOHgwR256VVVKV0JNdnFwdzZrNk5BWEdzOXJpc2VhN1pIaHp1cDMxL3ZiRGRpclR0MkJkaDgzdGtTYVZobEFrSzhzb0xyYm5jRjRpNWZ3NnU3SU4vZjNlcGNReitlWEtTNDFXcDg2REFSSU5BTkFTSGVqVHBvQllGR1lEeSt1VjhyUDlqYlNsdUQxZThyUHg5ZXBVOG5RL2NCTjBZSmRGQkFpSGV3S0pwRUlOYks0MnQzWjVCaXJYeDNlM1BwS0hIM3NaT3pZYm9ZMW5HbHVhVURXeUNCRmdTRWVBdUlaa0hndFFUT0w2NmFHNmhzRHpiUzN1NVcydHNadlBvcGFjTDd0YXBwdmdUYUZ4RGk3WnVhSTRGR0lNNytibXZ2Y2V3cmo2KzR2ZW51em1iYTNSNmtuUmJYenVPV29mR0I0ZUp5bE9Mbk5xZllMWEI2ZnBYaXBpZ21BZ1RhRlJEaTdYcWFHNEY3Z2JXMXRmVFQyNzBVdHlOdGE0cEx0c1k5eXVOcnZkZHIxc3hqay90R3YvZWtvOXJqS1BQUjlVMXppbGlFZC96YzloUWZZbjQ2Mms4UjRuRTB1d3h2VzlqOENLUWt4STBDQXE4a0VDRWVBVGJZN0RkQjN2WWFiZ1Q2OGRtdytacDJJVTVQYXdKOWZUMzErNzEwTzdsTms5dmI1b2p5T0twOGZCUEIzWDVnVDVjLy9SNEg1TVVIbU9oN1RHSFIzbmFKNlZKOEowQkFpQnNEQkY1Sm9NbXRsSm9nKzh1dmI5UDdqNmZOR3ZRckxhNlpiUnpkSGw4cHJlNys1WWY3MituZG03MDA3WDgwN0M3RVg3UG41azJnVGdFaFhtZmQ5WG9KQWw4SFZ3VGF6MGY3YVhPam4vNzRkRjdrL3VIb2I2eDl4OWFINzZldkEvMzc1L3lmQUlIbkN3ang1OXQ1SjRFSEJXWUZWNnlseGlibXVPcFpIRVJXeWhTYno5KzkyWjE3WHZ2WEgyaEs2Yk4rRU9pQ2dCRHZRaFcwb1VpQmVjRVZnZmVuWHc2Yjg4RGpHdVJ0N3l0ZkptYmNnUzArbUx3NTJFbTlXWjlhUGpjbTMvdTBMVlBUc2dnOFhVQ0lQOTNNT3dnc0pEQXZ4S2R2am91NHhOZW4wMkdLTUwvYmx6MTl0dHZmSTY4UDk3YlR3ZjUyMmxqZ1d1OFBCWHkzZTZwMUJMb3RJTVM3WFIrdHkxaGdrWENMN3IzWjMwNzdPNE1teUNQUXUzNCtkZXp6UHRpNzJ5MndhSGsyTnRhYjg5d1hmYjNYRVNDd21JQVFYOHpKcXdnOFdXQXdXUHpYYTMyOWw0N2U3S2E5M1FqenkzUjJjZG01RzQwOEo3eW5hSnVmVHpXYi90OTNBZ1RhRVZqOHIwdzd5ek1YQXRVSUREYWUvdXNWUjYvL2ZMU1gzaHhzcDdQenEzUjZjWm11bDNCZTk3eWl4SG5uY2QzMjJPdy9QZWQ3M21zZmV2dzVGZy9OejNNRUNOd0pQUDJ2RERrQ0JCNFZpUDNoTDFuN2pFM3hidzkzbWpDUGE1bGZYbDZuODh2UlV2YWJ4NVhnNHBLdUVkNDcyNE52enZkK3RPTnpYaEFmQU1LazY3c0s1alRmd3dRNkt5REVPMXNhRGN0WklFS3JqWU81SXZqaXBpZng5Vk5LelpIc2NXdlFxNnR4dWh4ZHQ3TEpQWTR3MzlyY1NMSDVQNDZjajUvanNUYW42RWVZWEY2dDdpSTBiZmJIdkFoMFJVQ0lkNlVTMmxHVVFLeEZ2OFlVUVJoZjB5bXVnWDU1TlU3WDErTTB2cGswWHplZnYwL1hldVBEUklUeTNWY3Y5ZGQ3elVWbk5qZlhtKytMSG9BM1hlWnp2eDhkN3FTLy8zNzgzTGQ3SHdFQ013UysvRFdZOGFTSENCQjR1a0JjOUdSbmF6bjMvNDU5NlBHVnc3Uzl0ZGxjMGUzOXg3TWNtcXVOQkxJUTZHWFJTbzBra0lsQWJQYU9DNStZWmd2RWhXSGlDSHdUQVFMdENPVHhFYjZkdnBvTGdWY1RpUDNJaHdmYnpiN3JWMXRJSVRQKzlkMUIydDIrU3NjbncyYS9maUhkMGcwQ0t4RVE0aXRodDlCU0JHSi9jcXhkeHBkcGNZSHB3WHB4cGJyNHVtN3V2TGI0KzcyU0FJRTdBU0Z1SkJCNGhrQ2NQMzJ3RzFjdTIwcHhvUmJUOHdTYXplczdnK1lXclNmbmwwczVoZTU1TGZVdUF0MFVFT0xkckl0V2RWUWdMaDk2Rjk3YnJaK0cxZEV1djNxejRrTlFITTBmZ1g1eU5rd1I1cXU4d00ycmQ5Z0NDTFFvSU1SYnhEU3JjZ1hpdEs2NGFsbXNlY2M1ejZiMkJlSVV1RGdvOEM3TUw1dTd2T1Y4aDdmMmhjeVJ3SThDUXZ4SEU0OFFhQVJpRFhGNjFiTDRibHFPUUh4SWlpQ1ByN2hhM2NYd3F2a2U1NytiQ0JENFZrQ0lmK3ZoZjVVTFJJRGNCWGRjZG5SZ2svbUt4MFBVSXI3ZVRXNVRYS251WWpocUFuMTZJWnNWTjgvaUNheGNRSWl2dkFRYXNHcUIySXk3UGRoTTIxc2JUV0RFUVd1bWJnbEVqYWIzWDQvN3JzY2ErdkR5T2cydlJxMWNlclpidmRVYUFvc0xDUEhGcmJ5eUlJRTRRRzBhM0hGMXRRZ0pVeDRDOFNGcnVybDlNcmx0N2xOK2VUbEtGNWZYS1M1RGF5SlFrNEFRcjZuYWxmYzFMdnNaYTlzUjJsOWZmN3h5bHF5N0h4Kys3amU1cDlTRWVMT0dmbm0zMlQzcnptazhnUVVFaFBnQ1NGNlNwMERjNkdNN2JxZTVkZmRsYlR2UE9qNmwxZE5yeWNlYSt1VDJ0cm1GNjBXc3BROUhMaWp6RkVpdnpVWkFpR2RUS2cxOVRDQkNPdGF3cDZIOWt2dDVQN1lzejNkZklPN2VGdmRGajYvME50YlNiNW9qM1NQVTQ5UzEyQlJ2SXBDN2dCRFB2WUtWdG44YTJCSGFnN2dYOW1ZL0xldVdtcFdTWjkvdHpZMjQ5ZXJPL1ExcTRsS3ZFZVpYbyt2UDN3Vjc5a1d1c0FOQ3ZNS2k1OVpsZ1oxYnhmSm9iM3pvaTYrNGp2dDBFdXhUQ2Q5ekVSRGl1VlNxb25adURlN1dyT1BPWUlPQk5leUtTci95cnM0TjlxdHhjOGUxV0hPL3ZMcGVlVHMxZ01CVVFJaFBKWHhmbVVEc3c0Nmp4dVBvY1VlTnI2d01GanhINEQ3WXY3b1Blb1Q1OUlBNW9UNEh6c05MRVJEaVMyRzJrSzhGK3YxZTJ0a2FwSjNQd2UybzhhOTEvSnlEd04yeEdQMzA5bUFuM1V3bXpkSHZFZVlSN09PeHk4UG1VTU5TMmlqRVM2bGtCdjJJemVUN3U0TzB0N1BsNGlvWjFFc1RGeE5ZNy9YdXJ5WVhSN3lmWGNUTlc2NXNkbCtNejZ0ZUtDREVYd2pvN1k4TDdPME8wdjdPMXQycFBvKy8zQ3NJWkNzUVc1VU85cmFicnpnMy9mVGlNcDJkWDJYYkh3M3Z2b0FRNzM2TnNtMWg3T09PKzBSdkR6YXk3WU9HRTNpdXdQUWM5WU85Ni9UeCtDSU5MMGZQblpYM0VaZ3JJTVRuMG5qaXVRSnhKN0Nqd3kvbjR6NTNQdDVIb0FTQitCQzcvY3RoK25SeWtUNGNYeVIzWUN1aHF0M3BneER2VGkyS2FFbGN4L3J0NGE2anpJdW9wazYwS2ZEbVlLYzVBK1BqOFhsekY3WTI1MjFlOVFyMDZ1MjZucmN0Y1BSbU4vMzI4NkVBYnh2Vy9Jb1JpS1BhNDNmazNadmRZdnFrSTZzVkVPS3I5UzlpNld0cktmM3licjg1M2FhSURvRUVlam9BQUF3clNVUkJWT2tFZ1ZjV2lMWHlQLzE4bUdMWGs0bkFTd1NFK0V2MHZEZkZ2WjMvK3R2YjVoUWJIQVFJTEM0UUI3Nzl4NStQYkxsYW5Nd3Jad2dJOFJrb0hscE1JTTc3L2g5L1BrcHgrMGNUQVFKUEY0amI1Y2FINEsrdjMvNzB1WGhIelFKQ3ZPYnF2NkR2Y2VyWVgzNTk4NEk1ZUNzQkFsT0JYMzg2YUU3SG5QN2Zkd0tMQ2dqeFJhVzg3bDdnWUc4ckhSMDZNT2NleEE4RVdoQ0kzNm5tM3VjdHpNc3M2aEVRNHZYVXVwV2U3bTRQMHM5SCs2M015MHdJRVBoV0lBNTJjeE9nYjAzODcyRUJJZjZ3ajJlL0VvaGJnLzcyODhGWGovaVJBSUcyQldJZmVkdzV6VVJnRVFFaHZvaVMxNlM0ODlndlAxa0ROeFFJTEVNZzlwSEhqVlZNQkI0VE1Fb2VFL0o4YXM0RFB6cXdkbUFzRUZpU1FHeFNqMnN2T0l0OFNlQVpMMGFJWjF5OFpUVTlEcmpaM25JVGsyVjVXdzZCRUlpRDNPSXFpQ1lDRHdrSThZZDBQTmVFZDF4ZHlrU0F3UElGN3E2MzdnUDA4dVh6V2FJUXo2ZFdLMm5wV3dHK0VuY0xKVEFWOERzNGxmQjlsb0FRbjZYaXNVYmdjSCs3dWVzU0RnSUVWaWV3dmJXWjRuZlJSR0NXZ0JDZnBlS3h0TEd4N29ZbXhnR0JqZ2pFMm5qOFRwb0lmQzhneEw4WDhmOUdJUDVvcks4YkhvWURnUzRJeE8raXplcGRxRVQzMnVDdmRQZHFzdklXeGMwWTluZTNWdDRPRFNCQTRJdEEvRTY2VWNvWER6L2RDUWh4SStFSEFVZWovMERpQVFLZEVQQzcyWWt5ZEtvUlFyeFQ1Vmg5WStLVHZtczNyNzRPV2tCZ2xrRDhidTd0RG1ZOTViRktCWVI0cFlXZjEyMmI2K2JKZUp4QU53VHM2dXBHSGJyU0NpSGVsVXAwb0IxeGc1UGRIWi95TzFBS1RTQXdWMkJuYTlPcG4zTjE2bnRDaU5kWDg3azl0cGx1TG8wbkNIUks0R0RQZ2FlZEtzZ0tHeVBFVjRqZnBVWDMxM3N1S05HbGdtZ0xnUWNFWXJlWDI1VStBRlRSVTBLOG9tSS8xTlY5bit3ZjR2RWNnYzRKdUlwYjUwcXlrZ1lKOFpXd2QyK2hiL2JkNUtSN1ZkRWlBdk1GYkZLZmIxUFRNMEs4cG1yUDZldkIzbmJxOWR5NWVBNlBod2wwVW1CdGJjMHVzRTVXWnJtTkV1TEw5ZTdrMHVLK3hTWUNCUElUaUp1am1Pb1dFT0oxMTcvcC9lWFZOUVVDQkRJVXVCcjUzYzJ3YkswMldZaTN5cG5uekVhamNaNE4xMm9DbFF0YytkMnRmQVNrSk1Tckh3SXBYVjBMY2NPQVFJNENRanpIcXJYYlppSGVybWVXYzd1NW1hU1JJTSt5ZGhwZHIwRDh6c2J2cnFsdUFTRmVkLzN2ZS8rdjk2ZnBZamk2Lzc4ZkNCRG9yc0R3Y3BUKzljZHBkeHVvWlVzVDZDOXRTUmJVYVlINFZQK1BmeDgzMTA1L2Q3aWJOamJXTzkxZWpTTlFvOEI0ZkpNK0hGK2swL1BMR3J1dnp6TUVoUGdNbEpvZk9yKzRTaGZEcXhUM0xUNDYzSzJaUXQ4SmRFcmcwOGxGK25COG5tNXZPOVVzalZteGdCQmZjUUc2dVBqNEkvSHgrQ0tkRDBmcDdjRk9jbnZTTGxaSm0yb1JPTHU0U3NjbnczVHBkTEphU3Y2a2ZncnhKM0hWOWVJNDlleGY3MC9TcDgxK2MrdkR1MXNnYnRTRm9MY0VWaUFReDZjTXI2NVQ3UHQyQlBvS0NwRFJJb1Y0UnNWYVZWUGpqMGg4eGVhOGZyK1hkcllHYVdkckkyMXZiNmJlbXN1MXJxb3VsbHVPd0dSeW04NkhWMmw0ZVJmY1kwZWRsMVBjVis2SkVIOWw0TkptUHg1UDBzblpzUG1LdnUxdUQxSmN0alcrNG5hbUpnSUVGaE9JZzlSaWwxVjh4UnEzaWNCekJJVDRjOVM4NTE0ZzFoN2lLNmJCWnI4Sjg4RkdQMjF1OXQzditGN0pEd1JTdWg3ZnBOaEZGUmRYaXMzbE5wTWJGVzBJQ1BFMkZNMmpFWmh1ZHA5eXhGMldJdGczTjliVDVrYi84ODk5ZDB5YkF2bGVwRUJzR285VE51UDNJYjZQcm0rYW4yOGRWbDVrdlZmZEtTRys2Z29VdlB6NG94VTNWL24rQml2OS9ub2FmQTcyV0dPUE5YZm5wUmM4RUFydTJuVUVkQVQxNThDK3VyNUpzWm5jUkdCWkFrSjhXZEtXY3k4UWYrU20rd09uRDhieGNiRzJ2dEZmVHhIeWNRRGR4dnJkOS83NnVyWDNLWlR2U3hXSXRlcnhUWXpYU2JyKy9EM0dick5wL0hyc25PMmxWc1BDWmdrSThWa3FIbHU2UUd4cC9INXovTmVONlBYVzdnSitHdXo5OWJTeDNyc1AvUFdlZytxKzl2THpZZ0kzazBrVDBFMHczOFRQZDRFZHdSMUJIU0Z1SXRCbEFTSGU1ZXBvMjcxQS9ERnRRajdOdnVOYW5Pb1dhKy9OV3Z6bm9GL3ZyYVZlcjVlKy94NGZDRXpsQ3NSWW1Vd202V2JHOTFpanZsdXp2Z3ZyaWYzVTVRNkVTbm9teENzcGRPbmRqRC9HY1FCUmZEMDJ4YWI3V2VIZWhQMzZqNkUvL1JBZy9CK1RiZmY1aDhKNGNqTTdwQ084NVhLN2RUQzNiZ3NJOFc3WFIrdGVRU0QreU1jdEhHK2F2SDg4OUw5dVFnUjVISFVmYS81Zi83d1dhLzFybjU5clhwTVdlTTJYK1h5OWpOeCtic0wyOWpiRmdZenhjL005L2g5cndsODlmdmR6V3VBMWQvUEp6VUY3Q2F4Q1FJaXZRdDB5c3hXSWtFcnBOajB0K2hmcjd0M0Y3KzQyOVg5OUlieTF0SlkrLzd1YlVYeFltTTZ5ZVdyR2V6N1BZUHE2Nlo3ZHIwOXptcTZ4M3FiYjZGSXpOZDgrUDNIM2N6dzFmWGY2YWkwM3ducmFDTjhKRUZpVmdCQmZsYnpsRXZoTzRDNFU3NUx4MjRDVWx0OVIrUzhCQXA4RkhOSnJLQkFnUUlBQWdVd0ZoSGltaGROc0FnUUlFQ0FneEkwQkFnUUlFQ0NRcVlBUXo3UndtazJBQUFFQ0JJUzRNVUNBQUFFQ0JESVZFT0taRms2ekNSQWdRSUNBRURjR0NCQWdRSUJBcGdKQ1BOUENhVFlCQWdRSUVCRGl4Z0FCQWdRSUVNaFVRSWhuV2pqTkprQ0FBQUVDUXR3WUlFQ0FBQUVDbVFvSThVd0xwOWtFQ0JBZ1FFQ0lHd01FQ0JBZ1FDQlRBU0dlYWVFMG13QUJBZ1FJQ0hGamdBQUJBZ1FJWkNvZ3hETXRuR1lUSUVDQUFBRWhiZ3dRSUVDQUFJRk1CWVI0cG9YVGJBSUVDQkFnSU1TTkFRSUVDQkFna0ttQUVNKzBjSnBOZ0FBQkFnU0V1REZBZ0FBQkFnUXlGUkRpbVJaT3N3a1FJRUNBZ0JBM0JnZ1FJRUNBUUtZQ1FqelR3bWsyQVFJRUNCQVE0c1lBQVFJRUNCRElWRUNJWjFvNHpTWkFnQUFCQWtMY0dDQkFnQUFCQXBrS0NQRk1DNmZaQkFnUUlFQkFpQnNEQkFnUUlFQWdVd0Vobm1uaE5Kc0FBUUlFQ0FoeFk0QUFBUUlFQ0dRcUlNUXpMWnhtRXlCQWdBQUJJVzRNRUNCQWdBQ0JUQVdFZUthRjAyd0NCQWdRSUNERWpRRUNCQWdRSUpDcGdCRFB0SENhVFlBQUFRSUVoTGd4UUlBQUFRSUVNaFVRNHBrV1RyTUpFQ0JBZ0lBUU53WUlFQ0JBZ0VDbUFrSTgwOEpwTmdFQ0JBZ1FFT0xHQUFFQ0JBZ1F5RlJBaUdkYU9NMG1RSUFBQVFKQzNCZ2dRSUFBQVFLWkNnanhUQXVuMlFRSUVDQkFRSWdiQXdRSUVDQkFJRk1CSVo1cDRUU2JBQUVDQkFnSWNXT0FBQUVDQkFoa0tpREVNeTJjWmhNZ1FJQUFBU0Z1REJBZ1FJQUFnVXdGaEhpbWhkTnNBZ1FJRUNBZ3hJMEJBZ1FJRUNDUXFZQVF6N1J3bWsyQUFBRUNCSVM0TVVDQUFBRUNCRElWRU9LWkZrNnpDUkFnUUlDQUVEY0dDQkFnUUlCQXBnSkNQTlBDYVRZQkFnUUlFQkRpeGdBQkFnUUlFTWhVUUlobldqak5Ka0NBQUFFQ1F0d1lJRUNBQUFFQ21Rb0k4VXdMcDlrRUNCQWdRRUNJR3dNRUNCQWdRQ0JUQVNHZWFlRTBtd0FCQWdRSUNIRmpnQUFCQWdRSVpDb2d4RE10bkdZVElFQ0FBQUVoYmd3UUlFQ0FBSUZNQllSNHBvWFRiQUlFQ0JBZ0lNU05BUUlFQ0JBZ2tLbUFFTSswY0pwTmdBQUJBZ1NFdURGQWdBQUJBZ1F5RlJEaW1SWk9zd2tRSUVDQWdCQTNCZ2dRSUVDQVFLWUNRanpUd21rMkFRSUVDQkFRNHNZQUFRSUVDQkRJVkVDSVoxbzR6U1pBZ0FBQkFrTGNHQ0JBZ0FBQkFwa0tDUEZNQzZmWkJBZ1FJRUJBaUJzREJBZ1FJRUFnVXdFaG5tbmhOSnNBQVFJRUNBaHhZNEFBQVFJRUNHUXFJTVF6TFp4bUV5QkFnQUFCSVc0TUVDQkFnQUNCVEFXRWVLYUYwMndDQkFnUUlDREVqUUVDQkFnUUlKQ3BnQkRQdEhDYVRZQUFBUUlFaExneFFJQUFBUUlFTWhVUTRwa1dUck1KRUNCQWdJQVFOd1lJRUNCQWdFQ21Ba0k4MDhKcE5nRUNCQWdRRU9MR0FBRUNCQWdReUZSQWlHZGFPTTBtUUlBQUFRSkMzQmdnUUlBQUFRS1pDZ2p4VEF1bjJRUUlFQ0JBUUlnYkF3UUlFQ0JBSUZNQklaNXA0VFNiQUFFQ0JBZ0ljV09BQUFFQ0JBaGtLaURFTXkyY1poTWdRSUFBQVNGdURCQWdRSUFBZ1V3RmhIaW1oZE5zQWdRSUVDQWd4STBCQWdRSUVDQ1FxWUFRejdSd21rMkFBQUVDQklTNE1VQ0FBQUVDQkRJVkVPS1pGazZ6Q1JBZ1FJQ0FFRGNHQ0JBZ1FJQkFwZ0pDUE5QQ2FUWUJBZ1FJRUJEaXhnQUJBZ1FJRU1oVVFJaG5XampOSmtDQUFBRUNRdHdZSUVDQUFBRUNtUW9JOFV3THA5a0VDQkFnUUVDSUd3TUVDQkFnUUNCVEFTR2VhZUUwbXdBQkFnUUlDSEZqZ0FBQkFnUUlaQ29neERNdG5HWVRJRUNBQUFFaGJnd1FJRUNBQUlGTUJZUjRwb1hUYkFJRUNCQWdJTVNOQVFJRUNCQWdrS21BRU0rMGNKcE5nQUFCQWdUK1B3QzRJQXAyUEhvL0FBQUFBRWxGVGtTdVFtQ0MiLz4KPC9kZWZzPgo8L3N2Zz4K) center center no-repeat;
      background-size: contain;
      margin: 0; }
      header .agent-link .avatar, header .agent-link .avatar img, header .agent-link .avatar svg {
        height: 34px;
        width: 34px;
        display: inline-block; }
  header .logo-wrapper {
    width: 50%;
    display: flex;
    justify-content: flex-end;
    min-width: 157px; }
  [dir=ltr] header .logo-wrapper {
  -webkit-transform: translateX(68px);
          transform: translateX(68px); }
  [dir=rtl] header .logo-wrapper {
    -webkit-transform: translateX(-68px);
            transform: translateX(-68px); }
    header .logo-wrapper .logo {
      width: 137px;
      height: 74px;
      position: relative; }
      header .logo-wrapper .logo .image, header .logo-wrapper .logo .underline {
        width: 137px;
        height: 74px;
        display: block;
        position: relative;
        z-index: 2; }
      [dir] header .logo-wrapper .logo .image, [dir] header .logo-wrapper .logo .underline {
        background: url("/images/logo/logo.png") no-repeat;
        background-size: contain; }
      header .logo-wrapper .logo .underline {
        height: 130px;
        width: 130px;
        position: absolute;
        top: -61px;
        z-index: 1; }
      [dir] header .logo-wrapper .logo .underline {
        border-radius: 100px; }
      [dir=ltr] header .logo-wrapper .logo .underline {
  background: -webkit-gradient(linear, left top, right top, from(#F9A51B), to(#FECE08));
  background: -webkit-linear-gradient(left, #F9A51B 0%, #FECE08 100%);
  background: linear-gradient(90deg, #F9A51B 0%, #FECE08 100%);
  left: 3px; }
      [dir=rtl] header .logo-wrapper .logo .underline {
        background: -webkit-gradient(linear, right top, left top, from(#F9A51B), to(#FECE08));
        background: -webkit-linear-gradient(right, #F9A51B 0%, #FECE08 100%);
        background: linear-gradient(-90deg, #F9A51B 0%, #FECE08 100%);
        right: 3px; }

@media (max-width: 1139.5px) {
  .with-search .notifications {
    top: 170px; }
  .with-search header {
    height: 160px; }
    .with-search header .search-wrapper {
      display: block;
      position: absolute;
      top: 80px;
      width: 750px;
      max-width: 100%; }
    [dir] .with-search header .search-wrapper {
      padding: 0 20px; }
    [dir=ltr] .with-search header .search-wrapper {
    left: 50%;
    -webkit-transform: translateX(-50%);
            transform: translateX(-50%); }
    [dir=rtl] .with-search header .search-wrapper {
      right: 50%;
      -webkit-transform: translateX(50%);
              transform: translateX(50%); }
      .with-search header .search-wrapper section {
        justify-content: space-around; } }

@media (max-width: 767.5px) {
  .with-search header .search-wrapper {
    width: 100%; }
  [dir] .with-search header .search-wrapper {
    -webkit-transform: none;
            transform: none;
    padding: 0 10px; }
  [dir=ltr] .with-search header .search-wrapper {
    left: 0; }
  [dir=rtl] .with-search header .search-wrapper {
    right: 0; }
  header .logo-wrapper {
    width: 240px;
    justify-content: flex-start; }
  [dir] header .logo-wrapper {
    -webkit-transform: none;
            transform: none; } }

@media (min-width: 1140px) {
  .with-search header .logo-wrapper {
    width: 240px;
    justify-content: flex-start; }
  [dir] .with-search header .logo-wrapper {
    -webkit-transform: none;
            transform: none; } }

@-webkit-keyframes Loading {
  0%, 50%, 100% {
    -webkit-transform: scale(0);
            transform: scale(0); }
  20% {
    -webkit-transform: scale(3.5);
            transform: scale(3.5); } }

@keyframes Loading {
  0%, 50%, 100% {
    -webkit-transform: scale(0);
            transform: scale(0); }
  20% {
    -webkit-transform: scale(3.5);
            transform: scale(3.5); } }

.loader {
  height: 300px;
  position: relative; }
  .loader.full-page {
    position: fixed;
    top: 0;
    bottom: 0;
    z-index: 999;
    height: auto; }
  [dir] .loader.full-page {
    background-color: #F5F5F588; }
  [dir=ltr] .loader.full-page {
  left: 0;
  right: 0; }
  [dir=rtl] .loader.full-page {
    right: 0;
    left: 0; }
    [dir] .loader.full-page.white {
      background: #FFFFFF; }
  .loader.segment {
    position: absolute !important; }
    .loader.segment .wrp {
      top: 350px; }
  .loader .inside {
    position: absolute;
    top: 45%;
    display: flex;
    width: 100%;
    justify-content: center; }
  [dir] .loader .inside {
    margin: 0 auto;
    text-align: center; }
  [dir=ltr] .loader .inside {
  left: 0;
  right: 0; }
  [dir=rtl] .loader .inside {
    right: 0;
    left: 0; }
    .loader .inside span {
      width: 6px;
      height: 6px;
      -webkit-transform-style: preserve-3d;
              transform-style: preserve-3d;
      position: relative;
      display: block; }
    [dir] .loader .inside span {
      background-color: #f9a51a;
      box-shadow: 0 0 3px #f9a51a55;
      border-radius: 50%;
      margin: 15px;
      -webkit-transform-origin: center;
              transform-origin: center;
      -webkit-transform: scale(0);
              transform: scale(0); }
    [dir=ltr] .loader .inside span {
  -webkit-animation: Loading 2.4s ease-in-out infinite;
          animation: Loading 2.4s ease-in-out infinite; }
    [dir=rtl] .loader .inside span {
      -webkit-animation: Loading 2.4s ease-in-out infinite;
              animation: Loading 2.4s ease-in-out infinite; }
      [dir=ltr] .loader .inside span:first-of-type {
  -webkit-animation-delay: 0s;
          animation-delay: 0s; }
      [dir=rtl] .loader .inside span:first-of-type {
        -webkit-animation-delay: 0s;
                animation-delay: 0s; }
      [dir=ltr] .loader .inside span:nth-of-type(2) {
  -webkit-animation-delay: .3s;
          animation-delay: .3s; }
      [dir=rtl] .loader .inside span:nth-of-type(2) {
        -webkit-animation-delay: .3s;
                animation-delay: .3s; }
      [dir=ltr] .loader .inside span:nth-of-type(3) {
  -webkit-animation-delay: .6s;
          animation-delay: .6s; }
      [dir=rtl] .loader .inside span:nth-of-type(3) {
        -webkit-animation-delay: .6s;
                animation-delay: .6s; }
      [dir=ltr] .loader .inside span:nth-of-type(4) {
  -webkit-animation-delay: .9s;
          animation-delay: .9s; }
      [dir=rtl] .loader .inside span:nth-of-type(4) {
        -webkit-animation-delay: .9s;
                animation-delay: .9s; }
      [dir=ltr] .loader .inside span:nth-of-type(5) {
  -webkit-animation-delay: 1.2s;
          animation-delay: 1.2s; }
      [dir=rtl] .loader .inside span:nth-of-type(5) {
        -webkit-animation-delay: 1.2s;
                animation-delay: 1.2s; }
      [dir=ltr] .loader .inside span:nth-of-type(6) {
  -webkit-animation-delay: 1.5s;
          animation-delay: 1.5s; }
      [dir=rtl] .loader .inside span:nth-of-type(6) {
        -webkit-animation-delay: 1.5s;
                animation-delay: 1.5s; }
      [dir=ltr] .loader .inside span:last-of-type {
  -webkit-animation-delay: 1.8s;
          animation-delay: 1.8s; }
      [dir=rtl] .loader .inside span:last-of-type {
        -webkit-animation-delay: 1.8s;
                animation-delay: 1.8s; }

.search-big-loader {
  width: 100%;
  height: 100%; }

[dir] .search-big-loader {
  padding: 40px 90px 0; }
  .search-big-loader h1 {
    font-size: 30px;
    line-height: 38px;
    font-weight: 100; }
  [dir] .search-big-loader h1 {
    text-align: center;
    margin-bottom: 7px; }
  .search-big-loader h2 {
    font-weight: 300;
    font-size: 15px;
    line-height: 18px;
    color: #777777; }
  [dir] .search-big-loader h2 {
    text-align: center;
    margin-bottom: 20px; }
  .search-big-loader canvas {
    width: 100%;
    height: 100%;
    max-width: 1300px; }
  @media (max-width: 767.5px) {
    [dir] .search-big-loader {
      padding: 20px 20px 0 20px; }
      [dir] .search-big-loader h2 {
        margin-bottom: 40px; } }

.modal-wrapper {
  width: 100%;
  height: 100%;
  position: fixed;
  z-index: 1000;
  top: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow-y: scroll; }[dir=ltr] .modal-wrapper {
  left: 0; }[dir=rtl] .modal-wrapper {
  right: 0; }
  .modal-wrapper .hide-close-button .close-button {
    display: none; }

.modal-scroll {
  max-height: 100%;
  overflow: auto;
  width: 100%; }

[dir] .modal-scroll {
  text-align: center;
  margin: auto;
  padding: 20px 0 35px; }

.overlay {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  opacity: .4; }

[dir] .overlay {
  background: #f9f7f4; }

[dir=ltr] .overlay {
  left: 0; }

[dir=rtl] .overlay {
  right: 0; }

.modal {
  width: 600px;
  position: relative; }

[dir] .modal {
  margin: 0 auto;
  background: #ffffff;
  border-radius: 25px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2); }

[dir=ltr] .modal {
  text-align: left; }

[dir=rtl] .modal {
  text-align: right; }
  @media (max-width: 767.5px) {
    .modal {
      max-width: 90% !important; } }
  .modal h2 {
    line-height: 24px;
    font-size: 18px;
    font-weight: 500; }
  [dir] .modal .title {
    padding: 18px 26px;
    border-bottom: 1px solid #e5e5e5; }
  [dir] .modal .content {
    padding: 30px 25px 35px; }

.body-wrapper {
  min-width: 480px; }

body.modal-open {
  overflow: hidden; }
  body.modal-open .body-wrapper {
    -webkit-filter: blur(2px);
            filter: blur(2px);
    overflow: hidden; }

.modal.confirm {
  max-width: 480px;
  width: auto;
  display: inline-block; }

[dir] .modal.confirm {
  padding: 40px; }
  .modal.confirm .danger {
    color: #E25141; }
  [dir] .modal.confirm h2 {
    margin-bottom: 24px; }
  [dir] .modal.confirm p {
    margin-bottom: 12px; }
  [dir=ltr] .modal.confirm button {
  padding-left: 40px;
  padding-right: 40px;
  margin-right: 20px; }
  [dir=rtl] .modal.confirm button {
    padding-right: 40px;
    padding-left: 40px;
    margin-left: 20px; }
  [dir] .modal.confirm .bottom {
    margin-top: 30px;
    text-align: center; }
    .modal.confirm .bottom .button {
      width: 100%; }
  [dir] .modal.confirm .row + .bottom {
    margin-top: 0; }
  [dir] .modal.confirm.duplicate {
    margin-bottom: 250px; }
    [dir] .modal.confirm.duplicate p {
      margin-bottom: 20px; }
    [dir] .modal.confirm.duplicate .button {
      margin-top: 20px; }

.close-button {
  z-index: 1;
  position: absolute;
  top: 5px; }

[dir] .close-button {
  cursor: pointer; }

[dir=ltr] .close-button {
  right: 5px;
  padding: 10px 10px 5px 5px; }

[dir=rtl] .close-button {
  left: 5px;
  padding: 10px 5px 5px 10px; }
  .close-button .icon {
    display: block;
    -webkit-filter: brightness(0);
            filter: brightness(0);
    opacity: .4; }
  .close-button:hover .icon {
    -webkit-filter: none;
            filter: none;
    opacity: 1; }

.navigation {
  position: relative;
  height: 54px;
  line-height: 54px;
  z-index: 150; }[dir] .navigation {
  background: #F7F8FC; }
  @media (min-width: 1140px) {
    .navigation {
      position: -webkit-sticky;
      position: sticky;
      top: 80px; }
    [dir] .navigation {
      box-shadow: 0 0.5px 0 #B9B9B9; } }
  .navigation nav {
    display: flex;
    flex-grow: 1; }
  [dir=ltr] .navigation nav {
  margin-left: -20px; }
  [dir=rtl] .navigation nav {
    margin-right: -20px; }
    .navigation nav a, .navigation nav .item {
      display: block;
      position: relative;
      font-size: 15px; }
    [dir] .navigation nav a, [dir] .navigation nav .item {
      padding: 0 10px;
      margin: 0 12px;
      cursor: pointer; }
      .navigation nav a:hover, .navigation nav .item:hover {
        color: #f9a51a; }
      [dir] .navigation nav a.active, [dir] .navigation nav .item.active {
        cursor: default; }
        .navigation nav a.active:hover, .navigation nav .item.active:hover {
          color: #231f20; }
        .navigation nav a.active:after, .navigation nav .item.active:after {
          content: ' ';
          height: 4px;
          bottom: 0;
          width: auto;
          position: absolute; }
        [dir] .navigation nav a.active:after, [dir] .navigation nav .item.active:after {
          background: #f9a51a;
          border-radius: 25px; }
        [dir=ltr] .navigation nav a.active:after, [dir=ltr] .navigation nav .item.active:after {
  left: 8px;
  right: 8px;
  background: -webkit-gradient(linear, left top, left bottom, from(#FFCD0B), to(#FBAC19));
  background: -webkit-linear-gradient(top, #FFCD0B 0%, #FBAC19 100%);
  background: linear-gradient(180deg, #FFCD0B 0%, #FBAC19 100%); }
        [dir=rtl] .navigation nav a.active:after, [dir=rtl] .navigation nav .item.active:after {
          right: 8px;
          left: 8px;
          background: -webkit-gradient(linear, left top, left bottom, from(#FFCD0B), to(#FBAC19));
          background: -webkit-linear-gradient(top, #FFCD0B 0%, #FBAC19 100%);
          background: linear-gradient(-180deg, #FFCD0B 0%, #FBAC19 100%); }

.notifications {
  position: fixed;
  top: 90px;
  width: 100%;
  z-index: 1001; }
  .notifications .wrapper {
    display: block;
    position: -webkit-sticky;
    position: sticky;
    top: 20px;
    max-width: 430px; }
  [dir=ltr] .notifications .wrapper {
  left: 100%; }
  [dir=rtl] .notifications .wrapper {
    right: 100%; }
    @media (max-width: 600px) {
      .notifications .wrapper {
        max-width: 100%; } }
    .notifications .wrapper .list {
      position: absolute;
      width: 100%; }
    [dir] .notifications .wrapper .list {
      padding: 10px 20px; }
    .notifications .wrapper .item {
      position: relative;
      overflow: hidden; }
    [dir] .notifications .wrapper .item {
      background: #FFFFFF;
      box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
      border-radius: 25px;
      padding: 14px 0 0;
      margin-bottom: 20px; }
      .notifications .wrapper .item .holder {
        overflow-y: auto;
        max-height: 150px; }
      [dir=ltr] .notifications .wrapper .item .holder {
  padding-right: 14px; }
      [dir=rtl] .notifications .wrapper .item .holder {
        padding-left: 14px; }
      .notifications .wrapper .item .content {
        display: flex; }
      [dir] .notifications .wrapper .item .content {
        padding-bottom: 14px; }
      .notifications .wrapper .item .text {
        display: flex;
        flex-direction: column;
        justify-content: center;
        line-height: 16px;
        min-height: 58px;
        max-width: 250px;
        word-wrap: break-word; }
      [dir=ltr] .notifications .wrapper .item .text {
  padding-left: 22px; }
      [dir=rtl] .notifications .wrapper .item .text {
        padding-right: 22px; }
      .notifications .wrapper .item h2 {
        font-size: 14px;
        font-weight: 600; }
      [dir] .notifications .wrapper .item h2 {
        padding-bottom: 4px; }
      .notifications .wrapper .item .style {
        width: 90px;
        min-width: 90px; }
      [dir] .notifications .wrapper .item .style {
        background-position: center center;
        background-repeat: no-repeat; }
      [dir=ltr] .notifications .wrapper .item .style {
  box-shadow: 0.5px 0 0 #B9B9B9; }
      [dir=rtl] .notifications .wrapper .item .style {
        box-shadow: -0.5px 0 0 #B9B9B9; }
      .notifications .wrapper .item .close-button {
        top: 23px; }
    @media print {
      .notifications .wrapper {
        display: none; } }
  .notifications .progress-timer {
    width: 100%;
    height: 3px; }
  [dir] .notifications .progress-timer {
    margin-top: -3px;
    border-radius: 2px; }
  .notifications .bar {
    height: 100%;
    width: 200%; }
  [dir] .notifications .bar {
    border-radius: 3px;
    background: #aaa; }
  [dir=ltr] .notifications .bar {
  -webkit-animation: progress 15s linear;
          animation: progress 15s linear;
  margin-left: -100%; }
  [dir=rtl] .notifications .bar {
    -webkit-animation: progress 15s linear;
            animation: progress 15s linear;
    margin-right: -100%; }
  [dir] .notifications .warning .style {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjciIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNyAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJNMTYuODQyMiAyLjIwMjYzTDI2LjI2NDcgMTguNTU1NkMyNi40NzIgMTkuMDQzNyAyNi41NjI4IDE5LjQ0MDYgMjYuNTg4NyAxOS44NTI5QzI2LjY0MDUgMjAuODE2MyAyNi4zMDM2IDIxLjc1MjcgMjUuNjQyNiAyMi40NzMzQzI0Ljk4MTYgMjMuMTkxMyAyNC4wODczIDIzLjYxNTIgMjMuMTE1MiAyMy42NjY2SDQuMTQwNjJDMy43Mzg4NCAyMy42NDIyIDMuMzM3MDUgMjMuNTUxIDIuOTYxMTkgMjMuNDA5N0MxLjA4MTg4IDIyLjY1MTggMC4xNzQ2MjQgMjAuNTE5NiAwLjkzOTMxIDE4LjY3MTJMMTAuNDI2NiAyLjE5MTA3QzEwLjc1MDYgMS42MTE3NiAxMS4yNDMxIDEuMTEyMDkgMTEuODUyMyAwLjc5MDk2NkMxMy42MTUgLTAuMTg2NTM3IDE1Ljg1NzIgMC40NTU3MTIgMTYuODQyMiAyLjIwMjYzWk0xNC43NTc1IDEyLjk3OThDMTQuNzU3NSAxMy41OTYzIDE0LjI1MiAxNC4xMTE0IDEzLjYyOTkgMTQuMTExNEMxMy4wMDc4IDE0LjExMTQgMTIuNDg5NCAxMy41OTYzIDEyLjQ4OTQgMTIuOTc5OFY5LjM0NTkxQzEyLjQ4OTQgOC43MjgwNiAxMy4wMDc4IDguMjI4NCAxMy42Mjk5IDguMjI4NEMxNC4yNTIgOC4yMjg0IDE0Ljc1NzUgOC43MjgwNiAxNC43NTc1IDkuMzQ1OTFWMTIuOTc5OFpNMTMuNjI5OSAxOC41MDQyQzEzLjAwNzggMTguNTA0MiAxMi40ODk0IDE3Ljk4OTEgMTIuNDg5NCAxNy4zNzM4QzEyLjQ4OTQgMTYuNzU2IDEzLjAwNzggMTYuMjQyMiAxMy42Mjk5IDE2LjI0MjJDMTQuMjUyIDE2LjI0MjIgMTQuNzU3NSAxNi43NDQ0IDE0Ljc1NzUgMTcuMzU5N0MxNC43NTc1IDE3Ljk4OTEgMTQuMjUyIDE4LjUwNDIgMTMuNjI5OSAxOC41MDQyWiIgZmlsbD0idXJsKCNwYWludDBfbGluZWFyKSIvPgogICAgPGRlZnM+CiAgICAgICAgPGxpbmVhckdyYWRpZW50IGlkPSJwYWludDBfbGluZWFyIiB4MT0iMS4xODY0OSIgeTE9IjExLjk5OTkiIHgyPSIyNy4xMTI0IiB5Mj0iMTEuOTk5OSIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPgogICAgICAgICAgICA8c3RvcCBzdG9wLWNvbG9yPSIjRkFBQjE5Ii8+CiAgICAgICAgICAgIDxzdG9wIG9mZnNldD0iMSIgc3RvcC1jb2xvcj0iI0ZFQ0QwQSIvPgogICAgICAgIDwvbGluZWFyR3JhZGllbnQ+CiAgICA8L2RlZnM+Cjwvc3ZnPgo=); }
  [dir] .notifications .warning .progress-timer .bar {
    background: #f9a51a; }
  [dir] .notifications .information .style {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxjaXJjbGUgY3g9IjIwIiBjeT0iMjAiIHI9IjEzIiBmaWxsPSIjM0U5MEYyIi8+CiAgICA8cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTE5LjAwNjkgMTUuOTkzN0MxOS4wMDY5IDE2LjU0MTEgMTkuNDUxIDE2Ljk4NzQgMTkuOTk1IDE2Ljk4NzRDMjAuNTU0OSAxNi45ODc0IDIxIDE2LjU0MTEgMjEgMTUuOTkzN0MyMSAxNS40NDYzIDIwLjU1NDkgMTUgMjAuMDA2MyAxNUMxOS40NTU1IDE1IDE5LjAwNjkgMTUuNDQ2MyAxOS4wMDY5IDE1Ljk5MzdaTTIwLjk4NzQgMTkuNTk3QzIwLjk4NzQgMTkuMDQ5NiAyMC41NDExIDE4LjYwMzMgMTkuOTkzNyAxOC42MDMzQzE5LjQ0NjMgMTguNjAzMyAxOSAxOS4wNDk2IDE5IDE5LjU5N1YyNC42MTY3QzE5IDI1LjE2NDEgMTkuNDQ2MyAyNS42MTA0IDE5Ljk5MzcgMjUuNjEwNEMyMC41NDExIDI1LjYxMDQgMjAuOTg3NCAyNS4xNjQxIDIwLjk4NzQgMjQuNjE2N1YxOS41OTdaIiBmaWxsPSJ3aGl0ZSIvPgo8L3N2Zz4K); }
  [dir] .notifications .information .progress-timer .bar {
    background: #3E90F2; }
  [dir] .notifications .success .style {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxjaXJjbGUgY3g9IjIwIiBjeT0iMjAiIHI9IjEzIiBmaWxsPSIjNUNDODczIi8+CiAgICA8cGF0aCBkPSJNMTkuMDAwMyAyNEMxOC43MjQ1IDI0IDE4LjQ0ODYgMjMuODk1MyAxOC4yMzgxIDIzLjY4NDhMMTUuMzE1OSAyMC43NjI2QzE0Ljg5NDcgMjAuMzQxNCAxNC44OTQ3IDE5LjY1OTIgMTUuMzE1OSAxOS4yMzkzQzE1LjczNyAxOC44MTgxIDE2LjQxOCAxOC44MTY5IDE2LjgzOTEgMTkuMjM4MUwxOS4wMDAzIDIxLjM5OTJMMjQuMDgzNyAxNi4zMTU5QzI0LjUwNDggMTUuODk0NyAyNS4xODU4IDE1Ljg5NDcgMjUuNjA2OSAxNi4zMTU5QzI2LjAyODEgMTYuNzM3IDI2LjAyODEgMTcuNDE5MiAyNS42MDY5IDE3Ljg0MDRMMTkuNzYyNiAyMy42ODQ4QzE5LjU1MiAyMy44OTUzIDE5LjI3NjEgMjQgMTkuMDAwMyAyNFoiIGZpbGw9IndoaXRlIi8+Cjwvc3ZnPgo=); }
  [dir] .notifications .success .progress-timer .bar {
    background: #5CC873; }

@-webkit-keyframes progress {
  from {
    -webkit-transform: scaleX(1);
            transform: scaleX(1); }
  to {
    -webkit-transform: scaleX(0);
            transform: scaleX(0); } }

@keyframes progress {
  from {
    -webkit-transform: scaleX(1);
            transform: scaleX(1); }
  to {
    -webkit-transform: scaleX(0);
            transform: scaleX(0); } }

.stars {
  position: relative;
  top: 1px; }
  .stars i {
    display: inline-block;
    vertical-align: baseline;
    width: 12px;
    height: 12px; }
  [dir] .stars i {
    background-repeat: no-repeat;
    background-position: center center;
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNiAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGQ9Ik0xMi40MzkyIDkuNzQwMTJDMTIuMjQ0OSA5LjkyODM3IDEyLjE1NTcgMTAuMjAwNiAxMi4xOTk5IDEwLjQ2NzZMMTIuODY2NyAxNC4xNTc2QzEyLjkyMjkgMTQuNDcwNCAxMi43OTA5IDE0Ljc4NjkgMTIuNTI5MiAxNC45Njc2QzEyLjI3MjcgMTUuMTU1MSAxMS45MzE0IDE1LjE3NzYgMTEuNjUxNyAxNS4wMjc2TDguMzI5OTEgMTMuMjk1MUM4LjIxNDQxIDEzLjIzMzYgOC4wODYxNiAxMy4yMDA2IDcuOTU0OTEgMTMuMTk2OUg3Ljc1MTY2QzcuNjgxMTYgMTMuMjA3NCA3LjYxMjE2IDEzLjIyOTkgNy41NDkxNiAxMy4yNjQ0TDQuMjI2NjYgMTUuMDA1MUM0LjA2MjQxIDE1LjA4NzYgMy44NzY0MSAxNS4xMTY5IDMuNjk0MTYgMTUuMDg3NkMzLjI1MDE2IDE1LjAwMzYgMi45NTM5MSAxNC41ODA2IDMuMDI2NjYgMTQuMTM0NEwzLjY5NDE2IDEwLjQ0NDRDMy43Mzg0MSAxMC4xNzUxIDMuNjQ5MTYgOS45MDEzNyAzLjQ1NDkxIDkuNzEwMTJMMC43NDY2NTcgNy4wODUxMkMwLjUyMDE1NyA2Ljg2NTM3IDAuNDQxNDA3IDYuNTM1MzcgMC41NDQ5MDcgNi4yMzc2MkMwLjY0NTQwNyA1Ljk0MDYyIDAuOTAxOTA3IDUuNzIzODcgMS4yMTE2NiA1LjY3NTEyTDQuOTM5MTYgNS4xMzQzN0M1LjIyMjY2IDUuMTA1MTIgNS40NzE2NiA0LjkzMjYyIDUuNTk5MTYgNC42Nzc2Mkw3LjI0MTY2IDEuMzEwMTJDNy4yODA2NiAxLjIzNTEyIDcuMzMwOTEgMS4xNjYxMiA3LjM5MTY2IDEuMTA3NjJMNy40NTkxNiAxLjA1NTEyQzcuNDk0NDEgMS4wMTYxMiA3LjUzNDkxIDAuOTgzODcyIDcuNTc5OTEgMC45NTc2MjJMNy42NjE2NiAwLjkyNzYyMkw3Ljc4OTE2IDAuODc1MTIySDguMTA0OTFDOC4zODY5MSAwLjkwNDM3MiA4LjYzNTE2IDEuMDczMTIgOC43NjQ5MSAxLjMyNTEyTDEwLjQyOTIgNC42Nzc2MkMxMC41NDkyIDQuOTIyODcgMTAuNzgyNCA1LjA5MzEyIDExLjA1MTcgNS4xMzQzN0wxNC43NzkyIDUuNjc1MTJDMTUuMDk0MiA1LjcyMDEyIDE1LjM1NzQgNS45Mzc2MiAxNS40NjE3IDYuMjM3NjJDMTUuNTU5OSA2LjUzODM3IDE1LjQ3NTIgNi44NjgzNyAxNS4yNDQyIDcuMDg1MTJMMTIuNDM5MiA5Ljc0MDEyWiIgZmlsbD0iI0ZCQUMxQSIvPgo8L3N2Zz4K);
    background-size: cover; }

.status.Success, .status.Confirmed {
  color: #1CBE69; }

.status.Cancelled {
  color: #777777; }

.status.Error {
  color: #E25141; }

.switcher {
  position: relative;
  display: flex; }[dir] .switcher {
  cursor: pointer; }
  .switcher:hover, .switcher.open {
    color: #231f20; }
    .switcher:hover .icon, .switcher:hover .switch-arrow, .switcher.open .icon, .switcher.open .switch-arrow {
      -webkit-filter: brightness(0);
              filter: brightness(0); }
  [dir=ltr] .switcher .icon {
  margin-right: 10px; }
  [dir=rtl] .switcher .icon {
    margin-left: 10px; }
  [dir] .switcher.currency-switcher {
    cursor: default; }
    .switcher.currency-switcher:hover, .switcher.currency-switcher.open {
      color: inherit; }
      .switcher.currency-switcher:hover .icon, .switcher.currency-switcher:hover .switch-arrow, .switcher.currency-switcher.open .icon, .switcher.currency-switcher.open .switch-arrow {
        -webkit-filter: none;
                filter: none; }
    .switcher.currency-switcher .currency span {
      color: #777777; }
  .switcher .switch-arrow {
    width: 9px;
    height: 10px;
    -webkit-filter: grayscale(1);
            filter: grayscale(1);
    display: inline-block; }
  [dir] .switcher .switch-arrow {
    background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iOCIgaGVpZ2h0PSI2IiB2aWV3Qm94PSIwIDAgOCA2IiBmaWxsPSJub25lIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPHBhdGggZD0iTTMuMzQwMjUgNS43MDEzQzMuMzA2NDIgNS42NjgzNCAzLjE2MTc1IDUuNTQzODkgMy4wNDI3NSA1LjQyNzk2QzIuMjk0MzMgNC43NDgzMSAxLjA2OTMzIDIuOTc1MjkgMC42OTU0MTcgMi4wNDczQzAuNjM1MzMzIDEuOTA2MzcgMC41MDgxNjcgMS41NTAwNiAwLjUgMS4zNTk2OUMwLjUgMS4xNzcyNyAwLjU0MiAxLjAwMzM4IDAuNjI3MTY3IDAuODM3NDQzQzAuNzQ2MTY3IDAuNjMwNTkxIDAuOTMzNDE3IDAuNDY0NjU1IDEuMTU0NSAwLjM3MzczMkMxLjMwNzkyIDAuMzE1MTk5IDEuNzY3IDAuMjI0Mjc1IDEuNzc1MTcgMC4yMjQyNzVDMi4yNzc0MiAwLjEzMzM1MiAzLjA5MzUgMC4wODMzNDM1IDMuOTk1MzMgMC4wODMzNDM1QzQuODU0NTggMC4wODMzNDM1IDUuNjM3NDIgMC4xMzMzNTIgNi4xNDcyNSAwLjIwNzc5NkM2LjE1NTQyIDAuMjE2MzIgNi43MjU5MiAwLjMwNzI0MyA2LjkyMTMzIDAuNDA2NjkxQzcuMjc4MzMgMC41ODkxMDcgNy41IDAuOTQ1NDE1IDcuNSAxLjMyNjczVjEuMzU5NjlDNy40OTEyNSAxLjYwODAyIDcuMjY5NTggMi4xMzAyNyA3LjI2MTQyIDIuMTMwMjdDNi44ODY5MiAzLjAwODI1IDUuNzIyIDQuNzQwMzUgNC45NDc5MiA1LjQzNjQ4QzQuOTQ3OTIgNS40MzY0OCA0Ljc0OSA1LjYzMjU0IDQuNjI0NzUgNS43MTc3OEM0LjQ0NjI1IDUuODUwNzYgNC4yMjUxNyA1LjkxNjY4IDQuMDA0MDggNS45MTY2OEMzLjc1NzMzIDUuOTE2NjggMy41Mjc1IDUuODQyMjMgMy4zNDAyNSA1LjcwMTNaIiBmaWxsPSIjZjlhNTFhIi8+Cjwvc3ZnPgo=) no-repeat; }
  [dir=ltr] .switcher .switch-arrow {
  margin: 7px 0 0 10px; }
  [dir=rtl] .switcher .switch-arrow {
    margin: 7px 10px 0 0; }

.table .the-table {
  border-collapse: collapse;
  width: 100%;
  font-size: 13px;
  line-height: 16px; }
  [dir] .table .the-table tr {
    border-top: 1px solid #e5e5e5; }
    [dir] .table .the-table tr:last-child {
      border-bottom: 1px solid #e5e5e5; }
    [dir] .table .the-table tr.clickable {
      cursor: pointer; }
      [dir] .table .the-table tr.clickable:hover td {
        background: #F3F4F6; }
    .table .the-table tr td {
      vertical-align: top; }
    [dir=ltr] .table .the-table tr td {
  padding: 35px 30px 35px 0; }
    [dir=rtl] .table .the-table tr td {
      padding: 35px 0 35px 30px; }
      [dir=ltr] .table .the-table tr td:first-child {
  padding-left: 40px; }
      [dir=rtl] .table .the-table tr td:first-child {
        padding-right: 40px; }
      [dir=ltr] .table .the-table tr td:last-child {
  padding-right: 25px; }
      [dir=rtl] .table .the-table tr td:last-child {
        padding-left: 25px; }
  .table .the-table strong {
    font-size: 11px;
    line-height: 13px;
    color: #777777;
    display: block;
    font-weight: 400; }
  [dir] .table .the-table strong {
    margin-bottom: 12px; }

.table .empty {
  font-size: 17px;
  font-weight: 300; }

[dir] .table .empty {
  border: solid #e5e5e5;
  border-width: 1px 0;
  text-align: center;
  padding: 120px 0;
  background: #fcfcfc; }

[dir=ltr] .table .icon-search {
  background-position: 0 3px; }

[dir=rtl] .table .icon-search {
  background-position: 100% 3px; }

.table .controls {
  display: flex;
  justify-content: space-between; }
  .table .controls .form {
    width: 260px; }
  [dir] .table .controls .form {
    padding: 30px 0; }
  [dir=ltr] .table .controls .form {
  margin-right: 16px; }
  [dir=rtl] .table .controls .form {
    margin-left: 16px; }
  [dir=ltr] .table .controls .search-wrap {
  margin-right: 0; }
  [dir=rtl] .table .controls .search-wrap {
    margin-left: 0; }
  .table .controls .left {
    display: flex; }
    @media (max-width: 767.5px) {
      .table .controls .left {
        flex-direction: column; }
        [dir] .table .controls .left .form + .form {
          margin-top: -40px; } }

.dropdown.room-details {
  width: 360px;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
  min-height: 0;
  max-height: none; }[dir] .dropdown.room-details {
  padding: 28px 36px; }[dir=ltr] .dropdown.room-details {
  right: 0;
  left: auto; }[dir=rtl] .dropdown.room-details {
  left: 0;
  right: auto; }
  .dropdown.room-details h3 {
    font-size: 16px;
    line-height: 19px; }
  [dir] .dropdown.room-details h3 {
    padding: 35px 0 20px; }
  [dir] .dropdown.room-details h4 {
    padding: 5px 0; }
  .dropdown.room-details .input {
    overflow: hidden; }
  .dropdown.room-details .row {
    display: flex;
    line-height: 40px; }
  [dir] .dropdown.room-details .row {
    margin-bottom: 15px; }
    .dropdown.room-details .row.children {
      flex-wrap: wrap;
      width: 310px; }
      .dropdown.room-details .row.children .part {
        display: flex;
        width: 135px;
        flex-grow: 0; }
      [dir] .dropdown.room-details .row.children .part {
        margin-top: 10px; }
      [dir=ltr] .dropdown.room-details .row.children .part {
  margin-right: 18px; }
      [dir=rtl] .dropdown.room-details .row.children .part {
        margin-left: 18px; }
        .dropdown.room-details .row.children .part .field {
          width: 52px;
          min-width: 0; }
        [dir] .dropdown.room-details .row.children .part .field {
          margin: 0; }
          .dropdown.room-details .row.children .part .field .input {
            height: 40px;
            line-height: 38px; }
          [dir] .dropdown.room-details .row.children .part .field .input {
            padding: 0 5px; }
            [dir] .dropdown.room-details .row.children .part .field .input input {
              text-align: center; }
        [dir=ltr] .dropdown.room-details .row.children .part .btn {
  margin-left: 3px; }
        [dir=rtl] .dropdown.room-details .row.children .part .btn {
          margin-right: 3px; }
    [dir] .dropdown.room-details .row:last-child {
      margin-bottom: 0; }
    .dropdown.room-details .row .caption {
      flex-grow: 1; }
    .dropdown.room-details .row .btn {
      display: block;
      width: 40px;
      height: 40px;
      line-height: 37px;
      opacity: .5; }
    [dir] .dropdown.room-details .row .btn {
      border: 1px solid #e5e5e5;
      border-radius: 25px;
      text-align: center; }
      .dropdown.room-details .row .btn.enabled {
        opacity: 1; }
      [dir] .dropdown.room-details .row .btn.enabled, [dir] .dropdown.room-details .row .btn.active {
        cursor: pointer; }
        .dropdown.room-details .row .btn.enabled:hover, .dropdown.room-details .row .btn.active:hover {
          color: #f9a51a; }
        [dir] .dropdown.room-details .row .btn.enabled:hover, [dir] .dropdown.room-details .row .btn.active:hover {
          border-color: #f9a51a; }
    .dropdown.room-details .row .value {
      font-size: 14px;
      width: 53px; }
    [dir] .dropdown.room-details .row .value {
      text-align: center; }

.dropdown.date {
  width: 644px;
  min-height: 0;
  max-height: none; }

[dir] .dropdown.date {
  padding: 25px 25px; }

[dir=ltr] .dropdown.date {
  -webkit-transform: translateX(-50%);
          transform: translateX(-50%);
  left: 50%; }

[dir=rtl] .dropdown.date {
  -webkit-transform: translateX(50%);
          transform: translateX(50%);
  right: 50%; }
  @media (max-width: 767.5px) {
    .dropdown.date {
      width: 330px; }
    [dir] .dropdown.date {
      -webkit-transform: none;
              transform: none; }
    [dir=ltr] .dropdown.date {
    left: 0;
    margin-left: 0; }
    [dir=rtl] .dropdown.date {
      right: 0;
      margin-right: 0; } }

.dropdown.residency {
  width: 350px;
  overflow: visible;
  min-height: 0;
  max-height: none; }

[dir] .dropdown.residency {
  padding: 20px 20px 10px; }

[dir=ltr] .dropdown.residency {
  right: 0;
  left: auto; }

[dir=rtl] .dropdown.residency {
  left: 0;
  right: auto; }
  [dir] .dropdown.residency .field {
    margin-bottom: 15px; }
  [dir=ltr] .dropdown.residency .field {
  margin-right: 0; }
  [dir=rtl] .dropdown.residency .field {
    margin-left: 0; }
  [dir] .dropdown.residency .checkbox {
    margin: 25px 5px 15px; }

.dropdown.filters {
  width: 300px;
  min-height: 0;
  max-height: none; }

[dir] .dropdown.filters {
  padding: 0 30px 15px; }

[dir=ltr] .dropdown.filters {
  left: 0;
  right: auto; }

[dir=rtl] .dropdown.filters {
  right: 0;
  left: auto; }
  .dropdown.filters .close-button {
    top: 10px; }
  [dir=ltr] .dropdown.filters .close-button {
  right: 10px; }
  [dir=rtl] .dropdown.filters .close-button {
    left: 10px; }

.dropdown {
  position: absolute;
  color: #231f20;
  z-index: 100;
  overflow: hidden;
  width: 100%;
  max-height: 280px;
  min-width: 120px;
  font-size: 14px; }[dir] .dropdown {
  background: #FFFFFF;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  border-radius: 25px;
  margin-top: 10px; }[dir=ltr] .dropdown {
  left: 0;
  -webkit-animation: Appear .15s ease-in 1;
          animation: Appear .15s ease-in 1; }[dir=rtl] .dropdown {
  right: 0;
  -webkit-animation: Appear .15s ease-in 1;
          animation: Appear .15s ease-in 1; }
  .dropdown .scroll {
    max-height: inherit;
    overflow-y: auto; }
  [dir] .dropdown .scroll {
    padding: 8px 0; }
    .dropdown .scroll::-webkit-scrollbar {
      width: 7px; }
    [dir] .dropdown .scroll::-webkit-scrollbar-track {
      background: transparent; }
    [dir] .dropdown .scroll::-webkit-scrollbar-thumb {
      background-color: #e5e5e5; }
  .dropdown .line {
    position: relative;
    z-index: 1;
    line-height: 22px; }
  [dir] .dropdown .line {
    cursor: pointer; }
  [dir=ltr] .dropdown .line {
  padding: 9px 15px 9px 27px; }
  [dir=rtl] .dropdown .line {
    padding: 9px 27px 9px 15px; }
    [dir=ltr] .dropdown .line .flag {
  margin-right: 12px; }
    [dir=rtl] .dropdown .line .flag {
      margin-left: 12px; }
    [dir] .dropdown .line:hover, [dir] .dropdown .line.focused {
      background: #F3F4F6; }
    .dropdown .line em {
      font-style: normal;
      color: #777777; }
  .dropdown .subtitle {
    height: 24px;
    line-height: 24px;
    font-size: 12px;
    color: #777777; }
  [dir] .dropdown .subtitle {
    margin-top: 8px;
    background: #F3F4F6; }
  [dir=ltr] .dropdown .subtitle {
  padding: 0 15px 0 27px; }
  [dir=rtl] .dropdown .subtitle {
    padding: 0 27px 0 15px; }
    [dir] .dropdown .subtitle:first-child {
      margin-top: 0; }
  .dropdown.locale {
    width: 160px;
    line-height: 0;
    top: 20px; }
  [dir] .dropdown.locale {
    padding: 8px 0; }
  [dir=ltr] .dropdown.locale {
  left: -26px; }
  [dir=rtl] .dropdown.locale {
    right: -26px; }
  .dropdown .summary {
    display: flex; }
  [dir] .dropdown .summary {
    margin-bottom: 0; }
  [dir=ltr] .dropdown .summary {
  padding-left: 17px; }
  [dir=rtl] .dropdown .summary {
    padding-right: 17px; }
    .dropdown .summary .photo {
      width: 80px;
      height: 60px; }
    [dir] .dropdown .summary .photo {
      background: #e8eaec center center no-repeat;
      background-size: cover;
      border-radius: 6px; }
    [dir=ltr] .dropdown .summary .photo {
  margin-right: 20px; }
    [dir=rtl] .dropdown .summary .photo {
      margin-left: 20px; }
    .dropdown .summary .title {
      max-width: 260px;
      width: 260px; }
    [dir] .dropdown .summary .title {
      padding: 0;
      border: 0; }
    .dropdown .summary h2 {
      font-weight: 400;
      font-size: 14px;
      line-height: 17px; }
    [dir] .dropdown .summary h2 {
      margin-bottom: 3px; }
      [dir=ltr] .dropdown .summary h2 span:first-child {
  margin-right: 8px; }
      [dir=rtl] .dropdown .summary h2 span:first-child {
        margin-left: 8px; }
    .dropdown .summary h2, .dropdown .summary h2 span {
      color: #000; }

.form .field {
  min-width: 88px;
  position: relative;
  flex-grow: 1; }[dir=ltr] .form .field {
  margin-right: 18px; }[dir=rtl] .form .field {
  margin-left: 18px; }
  [dir=ltr] .form .field:last-child {
  margin-right: 0; }
  [dir=rtl] .form .field:last-child {
    margin-left: 0; }
  .form .field .label {
    text-transform: uppercase;
    line-height: 16px;
    font-size: 11px;
    color: #777777;
    height: 24px;
    z-index: 1;
    position: relative;
    display: block;
    font-weight: 600; }
  [dir] .form .field .label {
    margin-bottom: -13px;
    border-radius: 10px; }
  [dir=ltr] .form .field .label {
  margin-left: 24px; }
  [dir=rtl] .form .field .label {
    margin-right: 24px; }
    .form .field .label span {
      text-overflow: ellipsis;
      white-space: nowrap;
      max-width: 85%;
      overflow: hidden;
      display: inline-block; }
    [dir] .form .field .label span {
      cursor: pointer;
      padding: 4px;
      background: #ffffff; }
  .form .field.error .label {
    color: #E25141; }
  [dir] .form .field.focus > label > .input {
    border-color: #f9a51a; }
  .form .field.focus > label > .label {
    color: #f9a51a; }
  .form .field.focus .icon-arrow-expand {
    -webkit-filter: none;
            filter: none; }
  [dir] .form .field.disabled .input {
    background: #f5f9fA; }
    .form .field.disabled .input input {
      color: #231f20; }
  [dir] .form .field.disabled .label span {
    cursor: default;
    background: -webkit-gradient(linear, left top, left bottom, from(0), color-stop(25%, rgba(245, 248, 250, 0)), color-stop(50%, #fff), color-stop(57%, rgba(255, 255, 255, 0)));
    background: -webkit-linear-gradient(0, rgba(245, 248, 250, 0) 25%, #fff 50%, rgba(255, 255, 255, 0) 57%);
    background: linear-gradient(0, rgba(245, 248, 250, 0) 25%, #fff 50%, rgba(255, 255, 255, 0) 57%);
    border-radius: 28px; }
  [dir] .form .field.no-input {
    cursor: pointer; }
    [dir] .form .field.no-input label {
      cursor: pointer; }
      .form .field.no-input label .inner {
        white-space: nowrap; }
        .form .field.no-input label .inner .placeholder {
          color: #777777; }
  .form .field input, .form .field .button {
    width: 100%; }
  .form .field.size-medium {
    max-width: 273px; }
  .form .field.size-half {
    width: 50%; }
  .form .field .error-holder {
    color: #E25141; }
  [dir=ltr] .form .field .error-holder {
  margin: 5px 0 0 17px; }
  [dir=rtl] .form .field .error-holder {
    margin: 5px 17px 0 0; }
  .form .field.select input {
    position: absolute !important; }
  [dir] .form .field.select, [dir] .form .field.select label, [dir] .form .field.select input {
    cursor: pointer; }
  .form .field .value-object {
    width: 100%;
    height: 100%;
    overflow-y: hidden; }
    .form .field .value-object.placeholder {
      color: #777777; }
  .form .field .input {
    color: #231f20;
    line-height: 52px;
    position: relative;
    display: flex;
    font-size: 14px; }
  [dir] .form .field .input {
    border-radius: 30px;
    background: #ffffff;
    border: 1px solid #e5e5e5;
    padding: 0 27px; }
    .form .field .input em {
      font-style: normal;
      color: #777777; }
    .form .field .input .inner {
      position: relative;
      display: inline-block;
      flex-grow: 2; }
    .form .field .input .suggestion {
      position: absolute;
      white-space: nowrap;
      top: 0;
      z-index: 1;
      font-size: 14px;
      color: #777777; }
    [dir=ltr] .form .field .input .suggestion {
  left: 0; }
    [dir=rtl] .form .field .input .suggestion {
      right: 0; }
      .form .field .input .suggestion.solid {
        color: #231f20; }
      .form .field .input .suggestion span {
        color: #ffffff; }
    .form .field .input input[type="text"], .form .field .input input[type="password"], .form .field .input textarea {
      width: 100%;
      font-size: 14px;
      position: relative;
      z-index: 3; }
    [dir] .form .field .input input[type="text"], [dir] .form .field .input input[type="password"], [dir] .form .field .input textarea {
      border: 0;
      background: transparent; }
      .form .field .input input[type="text"]::-webkit-input-placeholder, .form .field .input input[type="password"]::-webkit-input-placeholder, .form .field .input textarea::-webkit-input-placeholder {
        color: #777777;
        opacity: 1; }
      .form .field .input input[type="text"]::-moz-placeholder, .form .field .input input[type="password"]::-moz-placeholder, .form .field .input textarea::-moz-placeholder {
        color: #777777;
        opacity: 1; }
      .form .field .input input[type="text"]:-ms-input-placeholder, .form .field .input input[type="password"]:-ms-input-placeholder, .form .field .input textarea:-ms-input-placeholder {
        color: #777777;
        opacity: 1; }
      .form .field .input input[type="text"]::-ms-input-placeholder, .form .field .input input[type="password"]::-ms-input-placeholder, .form .field .input textarea::-ms-input-placeholder {
        color: #777777;
        opacity: 1; }
      .form .field .input input[type="text"]::placeholder, .form .field .input input[type="password"]::placeholder, .form .field .input textarea::placeholder {
        color: #777777;
        opacity: 1; }
    .form .field .input.textarea {
      height: auto;
      min-height: 80px; }
      .form .field .input.textarea .inner {
        overflow: hidden; }
      [dir] .form .field .input.textarea .inner {
        padding-top: 12px;
        padding-bottom: 6px; }
        .form .field .input.textarea .inner textarea {
          width: 100% !important;
          resize: none;
          overflow: hidden;
          min-height: 40px;
          height: 60px;
          max-height: 200px;
          display: block;
          line-height: 20px; }
      .form .field .input.textarea .placeholder {
        color: #777777; }
    .form .field .input .clear {
      width: 8px;
      height: 8px;
      display: inline-block; }
    [dir] .form .field .input .clear {
      background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iOCIgaGVpZ2h0PSI4IiB2aWV3Qm94PSIwIDAgOCA4IiBmaWxsPSJub25lIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgo8cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTQuOTUwNzkgNC4wMDAwMkw3LjgwMzA5IDEuMTQ3NzJDOC4wNjU2NSAwLjg4NTE2NCA4LjA2NTY1IDAuNDU5NDc1IDcuODAzMDkgMC4xOTY5NTZDNy41NDA1MyAtMC4wNjU2MDE4IDcuMTE0ODggLTAuMDY1NjAxOCA2Ljg1MjMzIDAuMTk2OTU2TDMuOTk5OTkgMy4wNDkyOUwxLjE0NzY2IDAuMTk2OTE4QzAuODg1MDk4IC0wLjA2NTYzOTMgMC40NTk0NDcgLTAuMDY1NjM5MyAwLjE5Njg5IDAuMTk2OTE4Qy0wLjA2NTYzIDAuNDU5NDc1IC0wLjA2NTYzIDAuODg1MTY0IDAuMTk2ODkgMS4xNDc2OEwzLjA0OTIyIDMuOTk5OThMMC4xOTY4OSA2Ljg1MjMyQy0wLjA2NTYzIDcuMTE0ODcgLTAuMDY1NjMgNy41NDA1NiAwLjE5Njg5IDcuODAzMDhDMC40NTk0NDcgOC4wNjU2NCAwLjg4NTA5OCA4LjA2NTY0IDEuMTQ3NjYgNy44MDMwOEwzLjk5OTk5IDQuOTUwNzVMNi44NTIzMyA3LjgwMzA4QzcuMTE0ODUgOC4wNjU2NCA3LjU0MDUzIDguMDY1NjQgNy44MDMwOSA3LjgwMzA4QzguMDY1NjUgNy41NDA1MiA4LjA2NTY1IDcuMTE0ODcgNy44MDMwOSA2Ljg1MjMyTDQuOTUwNzkgNC4wMDAwMloiIGZpbGw9IiNBNEFBQjMiLz4KPC9zdmc+Cg==) no-repeat;
      cursor: pointer; }
    [dir=ltr] .form .field .input .clear {
  margin-left: 14px; }
    [dir=rtl] .form .field .input .clear {
      margin-right: 14px; }
    [dir=ltr] .form .field .input .icon-wrap {
  margin-right: 8px; }
    [dir=rtl] .form .field .input .icon-wrap {
      margin-left: 8px; }
    [dir] .form .field .input .after-icon-wrap {
      cursor: pointer; }
    [dir=ltr] .form .field .input .after-icon-wrap {
  padding-left: 14px; }
    [dir=rtl] .form .field .input .after-icon-wrap {
      padding-right: 14px; }

.never-submitted .possible-hide {
  display: none; }

span.required:after {
  content: ' *';
  color: red; }

.form {
  display: flex;
  flex-direction: column;
  width: 100%; }
  .form .row {
    display: flex;
    width: 100%; }
  [dir] .form .row {
    margin-bottom: 20px; }
  .form .vertical-label {
    width: auto;
    display: flex;
    justify-content: center;
    flex-direction: column; }
  [dir] .form .vertical-label {
    cursor: pointer;
    margin-top: -3px; }
  [dir=ltr] .form .vertical-label {
  padding-left: 14px; }
  [dir=rtl] .form .vertical-label {
    padding-right: 14px; }

.checkbox {
  line-height: 22px;
  display: inline-flex;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none; }

[dir] .checkbox {
  cursor: pointer; }

[dir=ltr] .checkbox {
  background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAiIGhlaWdodD0iMjAiIHZpZXdCb3g9IjAgMCAyMCAyMCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHJlY3QgeD0iMC41IiB5PSIwLjUiIHdpZHRoPSIxOSIgaGVpZ2h0PSIxOSIgcng9IjYiIGZpbGw9IndoaXRlIiBzdHJva2U9IiNEOERBREMiLz4KPC9zdmc+Cg==) no-repeat left center;
  padding-left: 27px; }

[dir=rtl] .checkbox {
  background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAiIGhlaWdodD0iMjAiIHZpZXdCb3g9IjAgMCAyMCAyMCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHJlY3QgeD0iMC41IiB5PSIwLjUiIHdpZHRoPSIxOSIgaGVpZ2h0PSIxOSIgcng9IjYiIGZpbGw9IndoaXRlIiBzdHJva2U9IiNEOERBREMiLz4KPC9zdmc+Cg==) no-repeat right center;
  padding-right: 27px; }
  .checkbox:hover {
    color: #f9a51a; }
  [dir] .checkbox.on {
    background-image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAiIGhlaWdodD0iMjAiIHZpZXdCb3g9IjAgMCAyMCAyMCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICAgIDxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJNNS42NyAtMC4wMDAxMDY4MTJIMTQuMzRDMTcuNzMgLTAuMDAwMTA2ODEyIDIwIDIuMzc5ODkgMjAgNS45MTk4OVYxNC4wOTA5QzIwIDE3LjYxOTkgMTcuNzMgMTkuOTk5OSAxNC4zNCAxOS45OTk5SDUuNjdDMi4yOCAxOS45OTk5IDAgMTcuNjE5OSAwIDE0LjA5MDlWNS45MTk4OUMwIDIuMzc5ODkgMi4yOCAtMC4wMDAxMDY4MTIgNS42NyAtMC4wMDAxMDY4MTJaTTkuNDI5MDIgMTIuOTg5OUwxNC4xNzkgOC4yMzk4OUMxNC41MTkgNy44OTk4OSAxNC41MTkgNy4zNDk4OSAxNC4xNzkgNi45OTk4OUMxMy44MzkgNi42NTk4OSAxMy4yNzkgNi42NTk4OSAxMi45MzkgNi45OTk4OUw4LjgwOTAyIDExLjEyOTlMNy4wNTkwMiA5LjM3OTg5QzYuNzE5MDIgOS4wMzk4OSA2LjE1OTAyIDkuMDM5ODkgNS44MTkwMiA5LjM3OTg5QzUuNDc5MDIgOS43MTk4OSA1LjQ3OTAyIDEwLjI2OTkgNS44MTkwMiAxMC42MTk5TDguMTk5MDIgMTIuOTg5OUM4LjM2OTAyIDEzLjE1OTkgOC41ODkwMiAxMy4yMzk5IDguODA5MDIgMTMuMjM5OUM5LjAzOTAyIDEzLjIzOTkgOS4yNTkwMiAxMy4xNTk5IDkuNDI5MDIgMTIuOTg5OVoiIGZpbGw9InVybCgjcGFpbnQwX2xpbmVhcikiLz4KICAgIDxkZWZzPgogICAgICAgIDxsaW5lYXJHcmFkaWVudCBpZD0icGFpbnQwX2xpbmVhciIgeDE9IjAuNjY2NjY2IiB5MT0iOS45OTk4OSIgeDI9IjE5LjMzMzMiIHkyPSI5Ljk5OTg5IiBncmFkaWVudFVuaXRzPSJ1c2VyU3BhY2VPblVzZSI+CiAgICAgICAgICAgIDxzdG9wIHN0b3AtY29sb3I9IiNGQkE4MUMiLz4KICAgICAgICAgICAgPHN0b3Agb2Zmc2V0PSIxIiBzdG9wLWNvbG9yPSIjRkZDRDBCIi8+CiAgICAgICAgPC9saW5lYXJHcmFkaWVudD4KICAgIDwvZGVmcz4KPC9zdmc+Cg==); }

.input-range__slider {
  -webkit-appearance: none;
     -moz-appearance: none;
          appearance: none;
  display: block;
  outline: none;
  position: absolute;
  top: 50%;
  -webkit-transition: box-shadow 0.3s ease-out, -webkit-transform 0.3s ease-out;
  transition: box-shadow 0.3s ease-out, -webkit-transform 0.3s ease-out;
  transition: transform 0.3s ease-out, box-shadow 0.3s ease-out;
  transition: transform 0.3s ease-out, box-shadow 0.3s ease-out, -webkit-transform 0.3s ease-out;
  width: 19px;
  height: 19px; }[dir] .input-range__slider {
  background: #FFFFFF;
  border: 1px solid #e5e5e5;
  border-radius: 100%;
  cursor: pointer;
  margin-top: -13px; }[dir=ltr] .input-range__slider {
  margin-left: -10px; }[dir=rtl] .input-range__slider {
  margin-right: -10px; }

[dir] .input-range__slider:active {
  -webkit-transform: scale(1.1);
          transform: scale(1.1); }

[dir] .input-range__slider:focus {
  box-shadow: 0 0 0 5px rgba(63, 81, 181, 0.2); }

[dir] .input-range--disabled .input-range__slider {
  background: #cccccc;
  border: 1px solid #cccccc;
  box-shadow: none;
  -webkit-transform: none;
          transform: none; }

[dir=ltr] .input-range__slider-container {
  -webkit-transition: left 0.3s ease-out;
  transition: left 0.3s ease-out; }

[dir=rtl] .input-range__slider-container {
  -webkit-transition: right 0.3s ease-out;
  transition: right 0.3s ease-out; }

.input-range__label {
  color: #aaaaaa;
  font-size: 0.8rem;
  white-space: nowrap; }

[dir] .input-range__label {
  -webkit-transform: translateZ(0);
          transform: translateZ(0); }

.input-range__label--min,
.input-range__label--max {
  bottom: -1.4rem;
  position: absolute; }

[dir=ltr] .input-range__label--min {
  left: 0; }

[dir=rtl] .input-range__label--min {
  right: 0; }

[dir=ltr] .input-range__label--max {
  right: 0; }

[dir=rtl] .input-range__label--max {
  left: 0; }

.input-range__label--value {
  position: absolute;
  top: -1.8rem; }
  .input-range__label--value .input-range__label-container {
    display: none; }

.input-range__label-container {
  font-size: 12px;
  line-height: 14px;
  color: #1267fb; }

.input-range__track {
  display: block;
  height: 8px;
  position: relative; }

[dir] .input-range__track {
  background: #F4F4F4;
  border-radius: 8px;
  cursor: pointer; }

[dir=ltr] .input-range__track {
  -webkit-transition: left 0.3s ease-out, width 0.3s ease-out;
  transition: left 0.3s ease-out, width 0.3s ease-out; }

[dir=rtl] .input-range__track {
  -webkit-transition: right 0.3s ease-out, width 0.3s ease-out;
  transition: right 0.3s ease-out, width 0.3s ease-out; }

[dir] .input-range--disabled .input-range__track {
  background: #eeeeee; }

.input-range__track--background {
  position: absolute;
  top: 50%; }

[dir] .input-range__track--background {
  margin-top: -4px; }

[dir=ltr] .input-range__track--background {
  left: 8px;
  right: 8px; }

[dir=rtl] .input-range__track--background {
  right: 8px;
  left: 8px; }

[dir] .input-range__track--active {
  background: #f9a51a; }

.input-range {
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
  height: 33px;
  position: relative;
  width: 100%; }

[dir] .input-range {
  margin-bottom: 15px; }

.switch-control {
  width: 51px;
  height: 31px;
  display: inline-block;
  position: relative;
  -webkit-transition: background .15s ease;
  transition: background .15s ease; }[dir] .switch-control {
  background: #e5e5e5;
  border-radius: 50px;
  cursor: pointer; }
  [dir] .switch-control.active {
    background: #FAA61D; }
    [dir=ltr] .switch-control.active:after {
  -webkit-transform: translateX(20px);
          transform: translateX(20px); }
    [dir=rtl] .switch-control.active:after {
      -webkit-transform: translateX(-20px);
              transform: translateX(-20px); }
  .switch-control:after {
    content: ' ';
    width: 27px;
    height: 27px;
    top: 2px;
    position: absolute;
    -webkit-transition: -webkit-transform .3s ease-out;
    transition: -webkit-transform .3s ease-out;
    transition: transform .3s ease-out;
    transition: transform .3s ease-out, -webkit-transform .3s ease-out; }
  [dir] .switch-control:after {
    background: #FFFFFF;
    border-radius: 50px;
    box-shadow: 0 3px 1px rgba(0, 0, 0, 0.05); }
  [dir=ltr] .switch-control:after {
  left: 2px; }
  [dir=rtl] .switch-control:after {
    right: 2px; }

.account.block {
  width: 100%;
  height: 100%;
  min-height: 100vh;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center; }[dir] .account.block {
  margin-top: -80px;
  background: #74B1F5 center bottom no-repeat fixed;
  background-size: cover; }
  .account.block .small {
    max-width: 550px; }
    [dir=ltr] .account.block .small .section {
  -webkit-animation: Appear .33s ease-in-out 1;
          animation: Appear .33s ease-in-out 1; }
    [dir=rtl] .account.block .small .section {
      -webkit-animation: Appear .33s ease-in-out 1;
              animation: Appear .33s ease-in-out 1; }
  .account.block header {
    height: 300px;
    position: absolute;
    top: 0;
    z-index: 1; }
  [dir] .account.block header {
    box-shadow: none !important;
    margin-top: 0; }
  [dir=ltr] .account.block header {
  background: -webkit-gradient(linear, left top, left bottom, from(rgba(255, 255, 255, 0.7)), color-stop(36%, rgba(255, 255, 255, 0.55)), color-stop(61%, rgba(255, 255, 255, 0.38)), to(rgba(255, 255, 255, 0)));
  background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 0.55) 36%, rgba(255, 255, 255, 0.38) 61%, rgba(255, 255, 255, 0));
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 0.55) 36%, rgba(255, 255, 255, 0.38) 61%, rgba(255, 255, 255, 0));
  left: 0;
  right: 0; }
  [dir=rtl] .account.block header {
    background: -webkit-gradient(linear, left top, left bottom, from(rgba(255, 255, 255, 0.7)), color-stop(36%, rgba(255, 255, 255, 0.55)), color-stop(61%, rgba(255, 255, 255, 0.38)), to(rgba(255, 255, 255, 0)));
    background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 0.55) 36%, rgba(255, 255, 255, 0.38) 61%, rgba(255, 255, 255, 0));
    background: linear-gradient(-180deg, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 0.55) 36%, rgba(255, 255, 255, 0.38) 61%, rgba(255, 255, 255, 0));
    right: 0;
    left: 0; }
    .account.block header .logo-wrapper {
      width: 100%;
      min-width: 0;
      max-width: none;
      justify-content: space-around; }
    [dir] .account.block header .logo-wrapper {
      -webkit-transform: none;
              transform: none; }
  .account.block section.section {
    position: relative;
    z-index: 2 !important; }
  [dir] .account.block section.section {
    margin: 100px 0; }
    [dir] .account.block section.section .link {
      border-bottom: 1px solid; }
  .account.block .section {
    max-width: 700px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center; }
  [dir] .account.block .section {
    padding: 20px; }
  [dir=ltr] .account.block .section {
  -webkit-animation: Appear .15s ease-in-out 1;
          animation: Appear .15s ease-in-out 1; }
  [dir=rtl] .account.block .section {
    -webkit-animation: Appear .15s ease-in-out 1;
            animation: Appear .15s ease-in-out 1; }
    .account.block .section > div {
      min-height: 480px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      flex-grow: 1; }
    [dir] .account.block .section > div {
      background: #ffffff;
      border-radius: 50px;
      padding: 50px;
      box-shadow: 0 0 5px 1px rgba(0, 0, 0, 0.2); }
  [dir=ltr] .account.block .label span {
  background: -webkit-gradient(linear, left top, left bottom, color-stop(43%, rgba(255, 255, 255, 0)), color-stop(46%, white), color-stop(51%, white), color-stop(54%, rgba(255, 255, 255, 0))) !important;
  background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0) 43%, white 46%, white 51%, rgba(255, 255, 255, 0) 54%) !important;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0) 43%, white 46%, white 51%, rgba(255, 255, 255, 0) 54%) !important; }
  [dir=rtl] .account.block .label span {
    background: -webkit-gradient(linear, left top, left bottom, color-stop(43%, rgba(255, 255, 255, 0)), color-stop(46%, white), color-stop(51%, white), color-stop(54%, rgba(255, 255, 255, 0))) !important;
    background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0) 43%, white 46%, white 51%, rgba(255, 255, 255, 0) 54%) !important;
    background: linear-gradient(-180deg, rgba(255, 255, 255, 0) 43%, white 46%, white 51%, rgba(255, 255, 255, 0) 54%) !important; }
  .account.block .button {
    font-size: 16px; }
  [dir] .account.block .button {
    margin-top: 15px; }
  [dir=ltr] .account.block .breadcrumbs, [dir=ltr] .account.block h2, [dir=ltr] .account.block .paragraph {
  padding-left: 27px; }
  [dir=rtl] .account.block .breadcrumbs, [dir=rtl] .account.block h2, [dir=rtl] .account.block .paragraph {
    padding-right: 27px; }
  [dir] .account.block .breadcrumbs {
    margin: 9px 0 7px; }
  [dir] .account.block .form .row:last-child {
    margin-bottom: 0; }
  .account.block h2 {
    line-height: 34px; }
  [dir] .account.block h2, [dir] .account.block .accent-frame {
    margin-bottom: 12px; }
  .account.block .paragraph {
    line-height: 22px;
    color: #777777; }
  [dir] .account.block .paragraph {
    margin: 0 0 30px; }
    .account.block .paragraph .link {
      color: #231f20; }
      .account.block .paragraph .link:hover {
        color: #f9a51a; }
    [dir=ltr] .account.block .paragraph.action {
  margin: 0 27px 13px 0;
  text-align: right; }
    [dir=rtl] .account.block .paragraph.action {
      margin: 0 0 13px 27px;
      text-align: left; }
      [dir] .account.block .paragraph.action .link {
        border-bottom-color: #fff; }
        [dir] .account.block .paragraph.action .link:hover {
          border-bottom-color: inherit; }
    [dir] .account.block .paragraph + .accent-frame {
      margin-top: -10px; }
    [dir] .account.block .paragraph + .form {
      margin-top: 50px; }

.password-rules {
  color: #777777;
  font-weight: 300; }

[dir] .password-rules {
  padding: 0 27px; }
  .password-rules .ok-lowercase .rule-lowercase,
  .password-rules .ok-uppercase .rule-uppercase,
  .password-rules .ok-number .rule-number,
  .password-rules .ok-length .rule-length,
  .password-rules .ok-symbol .rule-symbol {
    font-weight: 400 !important; }
    .password-rules .ok-lowercase .rule-lowercase span,
    .password-rules .ok-uppercase .rule-uppercase span,
    .password-rules .ok-number .rule-number span,
    .password-rules .ok-length .rule-length span,
    .password-rules .ok-symbol .rule-symbol span {
      color: #231f20;
      font-size: 0; }
      .password-rules .ok-lowercase .rule-lowercase span:after,
      .password-rules .ok-uppercase .rule-uppercase span:after,
      .password-rules .ok-number .rule-number span:after,
      .password-rules .ok-length .rule-length span:after,
      .password-rules .ok-symbol .rule-symbol span:after {
        content: "\2713";
        font-size: 14px;
        color: #1CBE69; }
  .password-rules span {
    width: 20px;
    display: inline-block; }
  [dir] .password-rules span {
    text-align: center; }
  .password-rules .columns {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between; }
    .password-rules .columns div {
      width: 230px;
      height: 20px; }
  .password-rules .title {
    color: #797F89; }
  [dir] .password-rules .title {
    margin-bottom: 18px; }
  .password-rules .caption-wrapper {
    overflow: hidden;
    position: relative;
    height: 5px;
    width: 100%; }
  [dir] .password-rules .caption-wrapper {
    background: #D8DADC;
    margin: 17px 0 15px; }
  .password-rules .caption {
    height: 5px;
    box-sizing: content-box;
    width: 0;
    -webkit-transition: width .3s linear, background-color .15s ease-in-out;
    transition: width .3s linear, background-color .15s ease-in-out; }
  [dir] .password-rules .caption {
    background: #FE3824; }
  [dir=ltr] .password-rules .caption {
  border-right: 2px solid #f5fafb; }
  [dir=rtl] .password-rules .caption {
    border-left: 2px solid #f5fafb; }
  .password-rules .label span {
    width: 12px; }
  [dir] .password-rules .label span {
    cursor: default;
    text-align: center; }
  .password-rules .title:after {
    content: " None"; }
  .password-rules .total-1 .title:after, .password-rules .total-2 .title:after, .password-rules .total-3 .title:after, .password-rules .total-4 .title:after {
    content: " Weak"; }
  .password-rules .total-1 .caption {
    width: 20%; }
  [dir] .password-rules .total-1 .caption {
    background: #FE3824; }
  .password-rules .total-2 .caption {
    width: 40%; }
  [dir] .password-rules .total-2 .caption {
    background: #FE3824; }
  .password-rules .total-3 .caption {
    width: 60%; }
  [dir] .password-rules .total-3 .caption {
    background: #FF9F00; }
  .password-rules .total-4 .caption {
    width: 80%; }
  [dir] .password-rules .total-4 .caption {
    background: #FF9F00; }
  .password-rules .total-5 .title:after {
    content: " Strong";
    color: #1CBE69;
    font-weight: 500; }
  .password-rules .total-5 .caption {
    width: 100%; }
  [dir] .password-rules .total-5 .caption {
    background: #23D578; }

.account.block .label.with-actions {
  display: flex;
  justify-content: space-between; }

[dir=ltr] .account.block .label.with-actions {
  padding-right: 27px; }

[dir=rtl] .account.block .label.with-actions {
  padding-left: 27px; }
  .account.block .label.with-actions .link {
    color: #1267fb; }
  [dir] .account.block .label.with-actions .link {
    border-bottom: 0; }
  @media (max-width: 767.5px) {
    .account.block .label.with-actions span:last-child {
      display: none; } }

.account.block .error-field {
  color: #E25141;
  line-height: 19px; }
  [dir=ltr] .account.block .error-field ul {
  padding-left: 24px; }
  [dir=rtl] .account.block .error-field ul {
    padding-right: 24px; }
    .account.block .error-field ul li {
      list-style-type: disc; }
    [dir] .account.block .error-field ul li {
      margin-bottom: 4px; }
      [dir] .account.block .error-field ul li:first-child {
        padding-top: 9px; }
      [dir] .account.block .error-field ul li:last-child {
        padding-bottom: 9px; }

.account.block .auth-input input, .account.block .auth-input textarea {
  color: #231f20;
  height: 54px;
  line-height: 16px;
  position: relative;
  display: flex; }

[dir] .account.block .auth-input input, [dir] .account.block .auth-input textarea {
  border-radius: 25px;
  background: #ffffff;
  border: 1px solid #e5e5e5;
  padding: 0 27px; }
  [dir] .account.block .auth-input input:focus, [dir] .account.block .auth-input textarea:focus {
    border-color: #f9a51a; }

.account.block .auth-input textarea {
  width: 100%;
  height: 120px;
  resize: none;
  font-size: 11px; }

[dir] .account.block .auth-input textarea {
  padding-top: 20px; }
[dir] .account.block .auth-input textarea::placeholder {
    font-size: 14px; }


/*# sourceMappingURL=main.e49168f3.chunk.css.map*/
```

### HappyTravel.Odawara.Api/Pages/Account/ForgotPassword.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.ForgotPassword

<div class="small">
    <div class="breadcrumbs">
        <div class="links">
            <a href="@Model.BackToLoginUrl">@Localizer["Sign In"]</a>
            <span class="small-arrow-right"></span>
            @Localizer["Reset Password"]
            <span class="small-arrow-right"></span>
            @Localizer["Address Enter"]
        </div>
    </div>
    <h2>
        @Localizer["Reset Password"]
    </h2>
    <div class="paragraph">
        @Localizer["Please enter the email address connected with your Happytravel.com account in the field below to begin password restoration."]
    </div>
    <div class="form">
        <form method="post">
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span class="required">
                            @Localizer["Email"]
                        </span>
                    </div>
                    <div class="auth-input">
                        <input asp-for="ForgotPasswordInput.Email" placeholder="@Localizer["Please enter your Email"]" />
                    </div>
                </div>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">@Localizer["Reset"]</button>
                    </div>
                </div>
            </div>
        </form>
    </div>
</div>

```

### HappyTravel.Odawara.Api/Pages/Account/Confirm.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.ConfirmEmailModel

<div class="small">
    <div class="breadcrumbs">
        <a asp-page="./Login">@Localizer["Sign In"]</a>
        @Localizer["Registration"]<span class="small-arrow-right"></span>
        @Localizer["Confirmation"]
    </div>
    <h2>
        @Localizer["Account Confirmation"]
    </h2>
    <div class="accent-frame">
        <div class="data only">
            @Localizer["We have sent a message with a confirmation link and a confirmation code to your email address. The message will arrive within 5&ndash;10 minutes."]<br />
            @Localizer["To confirm the address, manually enter the code from the message into the field below:"] <br />
        </div>
    </div>
    <form method="post">
        <div class="form">
            <div class="row">
                <div class="field">
                    <div class="label">
                        <span class="required">
                            @Localizer["Confirmation Code"]
                        </span>
                    </div>
                    <div class="auth-input">
                        <textarea class="confirmation-code" placeholder="@Localizer["Enter Code here"]" rows="4" asp-for="Input.Code"></textarea>
                        <span class="error-field" asp-validation-for="Input.Code"></span>
                    </div>
                </div>
            </div>
            <div class="error-field">
                <div asp-validation-summary="All"></div>
            </div>
            <div class="row">
                <div class="field">
                    <div class="inner">
                        <button type="submit" class="button main">
                            @Localizer["Confirm The Email Address"]
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>

```

### HappyTravel.Odawara.Api/Pages/Account/ResetPasswordInfo.cshtml
```json
@page
@using Microsoft.AspNetCore.Mvc.Localization
@inject IViewLocalizer Localizer
@model Account.ResetPasswordInfo

<div class="small">
    <div class="breadcrumbs">
        <div class="links">
            <a href="@Model.BackToLoginUrl">@Localizer["Sign In"]</a>
            <span class="small-arrow-right"></span>
            <a href="@Model.BackToForgotPasswordUrl"> @Localizer["Address Enter"]</a>
            <span class="small-arrow-right"></span>
            @Localizer["Reset Password"]
        </div>
    </div>
    <h2>
        @Localizer["Reset Password"]
    </h2>
    <div class="accent-frame">
        <div class="data only">
            @Localizer["We have sent a message with a password recovery link to your email. The message usually arrives within 5&ndash;10 minutes. The link will be available for 24 hours."]
        </div>
    </div>
</div>

```

### HappyTravel.Odawara.Api/wwwroot/images/other/mc-sec.png
```json
�PNG

   IHDR   �   �   g�
�    IDATx��}�%E��[�'�ss�p'G�a�3"� `b��VL調��ꮺ��.�\WW]s� 
����Hf�&09�;7����?���t�W�g���3���9s�鮪�������q�ôZ�T@����/;`0:
p �T�@{(���]�@�T*@[(W��K��Q a;v�6s� �0d3@:G��3[���� ��d� ,��&���]~z�
֖9�}��Ux��%�7k���e����7e��H{'lTK��G�d9:�<��[eK�y1?������s�Z�i[��܃�3�x�7�8n%��I �f� 
Š~˗קP:ہ��`����A��{��s��;�]{�L�	,YLLwK�������s,��5�8y��i�"`��d{�ݿH�mn	��@e��
[��� ����y �8%�%�r�X�`_�R`r#��k����̫�w޻���X���{a��v��;g�fN�B�	Z OvzH�V��[F�#�T�,{7��7c�IO��9�"�� �]9	���a �$;����h_$s@y�\�?�� ���@uh��{|]p=�3��<���+ł2��z��T�@i'�����۱ Y�3h�� ]ˁ�Y��>8KL/JV4v�`=��÷o	~)yb%۳��mۑ��0�Q	�����Dk�/��$X�P/
C۹1�'��2�?l]ǲ_0�e��;Q�7�����aG��'��g���e^����l �
�����)�N@u�Yo��_����Ǐ���y,�5�*��%0�N����e�2�L`�Q�Q#2 ��1�(����g@�<�����It.]ϓ=� ��՚���"�Q�&��<��>P��ӂ�lx��Y���7�G}x���u��Ai�?鰯c~{���7^�k��,I�'m��;2kdW�����o�d�C�gJ7�j���3f�8�`�!�+��$���f%B�h�t�.�Ć�b��0��lkrCU�v�" �/f���:VO`lS7��}�pRmW�M�V�w�zޱ���GV��?j���s۞�g`�@{��{���[��6lx|�	l�A�̦C)L�
A�����Ȃ�2X�e�H����Z�L�[��ì��v�.�ʬܩ�_^5_�x;����<�{�c������VK���{��c�Ț���x�7/a#��T�Y�'y(�$u5s
`��Rػ�Hf�H�;��w�5�ܼ�x�»ټ�~�%۷�NN�Tr���T���*�>��{��f��B�ǳ��!8l��`ʬ����,�_��L)���{�r�D�Q0_����L��[ `~z��#W|��|�����dj��a+ *ب2����������v�3��ZOV�̇�1I(	�N����g1g�u?�[oƩ��/�76������C�� �`O�w��P���+�������rk�t���5hr&�	Z�j�j���L�%�����T��R����v<|�%��Ƌ�g��(;��3G�}��b��g�d;����c�=�a;~�rktm��/'`�˘ڲYa������rÏx��_����5x����()�\���5y�̖�⭷�6~���ms�CI�� ��W�5# S����\�Bx��do�{�X�) gR���f���w1�?>��t��J'����K�N�8�khk���a���_f��~�<����ʆ��6����@� ����j��s�v���v��N��Wo�����f��i��5~4�0���"����͕��׳��|��eK
�~�n��
������Q
�ae��@G��ɱi}��j�ji�eU]X��ُ���ly���=�vv�1Wf_�̴��?l���'��߿����lV�>��sLbZ6�T#O�O�]��^����k��o�f|��,��?�f����/}�k�[��&k�é����L�IT 2"YU�F�p?��#���R>�5@ʝCVH�MA� u[�j󦌽y�|��ű��e?�����G����n���_��m�巬���c��S�����c;�Y��>|/�zK.�u����=|X������ֆҍ�z+~��/[����>]Nj��� J��Q@�p����k�9�7��M�Z�
���T�A��f�_��{K�>z.����\zJ V1��� ���%|�?�6^{2ʅ�=�4Y�e�I���8;︟/}�O��w]�vK�[bS������ý��s�������o_�"oA�LV���Ad��b��:c��W�]�2�8@:FV�U�`J���[w�փ{��J��ߟ|˛/�f�z �M0�,�i�k��=l�嫬�V(TZV���Z��{��w�.����� +��3��E{v�*�Z8�K��Ow[?�ɩ�4�GpkA"@�2U��)��t�&ba���,nW������������\���;��j���.��_�L��O�4\̈j���ƿa��z�����_�l�]�����y_|�֌��|T�P�:;Q���g?��w��T�*��eHԛk\��E+��jԗ+ �(�#ep�Oe{�L���TS� �F�G�7&&�}�_\z���˿��<�t볜D���������z�S_`Ï�� }��bl��j���s/{��?z��Ez֪s�L��Tp�t��_����w�n�c�mD�z��ebL�73r���4��aFf�R9������r�|��&:��?ߛ��l���g��e.���L�~t<�z�t���֖?�� }���n�	��[챯|��o����O�!ٵ�PW����b�J<�1:>P~�U���;خ�,��Lc:�b� ��L�LǡG���h�0��6��(s�����G�ʓ0j�r�v�`�?~����^���^,"����������ٶ?��1��`��s�� )l������ZϷ��m"(w(/�jWJ7����[޾��w�l�hJ�0���i���{���Lyes�ϪF�#Sc��V�c*�!��3��5T���Z�T|���*W����w�u�}�"k��4Ylb�[R�~�{����i���I���F���Q�������Ea���U�5F��<;�K�H�'!�D�)����NQ��@�E�^�&���C�|k.�.�SmH�x8�n�,`�v8����&p�a����xq�zv�?;��1��2��h%P�׫o����{Y!_̏cA��X&��~t1k���7Ηe�G�	�s���c�7-E�wM�2�,�t.5��s$^Z[�`�C ��m����e�����+�Aix�?��=b�*VWH�R�w��A|�_.���1��$�O&5R^����b-'�����c@�|$;�(q�������E{e1����0VK�Ua�[��Hkj�i k�'�Z6�&���f���=�7���Hu5�7g)��A��Q<����{������M0��[XEi�*7�	Q��y�>.b���C�L�-O+4��\���tN���dڙԙt^�Y���C�ms��~�X��&܋l7��?�!+����C#Ӽ��3���0:�U��u����0������7�C�~�X=�.�UA����p��	r�(=4�nJ�A�����^R�}6)#���Bl�Hؠ���<�&�qi�#W��v����S�l�T1�.>��ݥ�^����_w�*�E-���,�`��> :ɲW�h���� a�cu���S��)	$HҺ�B�Tb ���e�g���\���C��~ѠZ�b�������k�����OٜF��[&$�ޑ��e��Z}��I!yu\�F�C�Dr.������s��o&?�K�Cb3F�SK���*'�o|�+:����^^�u���4Xĥ�,)D�?�Lh����l�����{~�a�f�lz�*�t�S����o`��<�Ƥ�Q}:��JG���l���U��2[�!5e;��t�L��EeQz}�����+H�ɩ_52��V���f�K�0��k��u�t���I���ҫ._c]��#Շ��8e�!�>S����o���̾j'����r���j4�Fhi9r����jz���*��*��) M�%�É�s� ���:ZF�x��*{я���I����vp����7�� ��"�����^���ҐD\4Uf+��Љ�_��)�����!���)�L�-^�Rr%/6.��`�r��n�DwMY8)�k���Z����g��t���P�9����-�?��/^}y�xZ�8�!S��^�l����t��a( �L�Ϊ���ɭ3��2�3H�����CP���t��>E�kbFZ��_ ^L-��z�i�������k��-���9��a\�"bM0�e�z�W�7���(�����ڗԕ�	�鼧��������R;�.A�v(��$�A:!]p��|$���Z�=vJ	l�����ꎾ��,>�>�{���-�>� ފ7�����)�}@�&�����E�yrY��s��ذx�i0�+��CIO=JSZHe��Q�PV��qu���ES'��
�i�d��}Q^ qb�*'X��)��qȧ'W37\/�$��5!�9V�y��4[��OH�Lb�������W��Ӎ��a��21h#�R;�-������t�>�E��|�Nt�?��0���hb��I��Ȣ��z��j����ܠ�^q�bt,�⿳g?l�}T�P���~���W�֤��m���h���/�A�1���R�L�j��Q�������d������)@����Yɵ��+��ɕ����f8�)�췊1��a���X\s������oò�k���<S�0�w����'>�çi��	0�C}?$����R�����Wu\�I0��ԥRd��5U�E��.��}L9[�B�,�z�}B%���8 �nv��>S��~q;7��"Z���x'����������c�f\D���A0�~|2��o�gc{���e^����&s��hik���eԾ��K�Wۥ^'�4�#�ҷsi���R�f���CU�x֢*�|/��&�^�8�%5j�b8s)Οm����y�+�b��ߺ?��񉩥�fP�����]���joFCL ��"h'#��>-��OR߷Q����D�F �ܪ�IK6u8�q�;�!��b��t�)�d�C�}��v�E�[6�&����v��=H�
�U�2ˏ���Ӗ����U��:/���ܦ���~t��|?�cfC��FG��L�'��V.E�#w̰�TcU�#&]BH�vi�4��M�^L9eb��6!{ׇ�5y�9&�սR�ߺ�[�x���H���#��"tt&��ۮ|?�Ί�2*P !�*���l4uO��lEe��(��xѺ��>��e�ݡ�uMa6{i�����Q�U�j���T.YU̥9�A�����C����.E������R�>�k���\���~���^tߩ�e����2u.�����@;z^5�c�Ld�1����/�a�Xj�	$�
qr��[<�(�4����G��֔�����P+��4j�0��_��&?�����>Ą���x�U�b������X���P�����*��S�>F onW�;�9�*h!���rX6jQ��䧪�g$OTOk�t���A&� ��K+T����bӦ��b���+��|��*De��?	���XK�Q�ħ���t{N<�F6�Q?�������4d�pi#t$T.[��Èx�eU�*��h,S���������\��\*-�ߒ�`��lJ�`�����QyT���������I��iڲ}���.���b��@�7��G���u��ٕ�O܊�V�VoH ԙ�Z�hgԄ9�W=LI}Ě����*��V�=X��b9r+�J�/��ؒ8��ǟf��Z���B.Nͳ�ߴ\�j^���)!p[�������q���%ݺ5��U��__���� �������Qer���i-M�Ǎ�A%nT�^&=.W�T�*��sx��_�aI	i��j�T8S.T�)G�511e����pN�˸�f,5�
�����KFǠ}���������F���1�L���rj:!��Gjz�f�u�pe	,���T�X��ꋛ�XT������$��!M���2.�z3�M��f3"�o��������ח�/�6��l\6�ʷ��!�;r��dw����c؈���?'�F�i���M-RL��'?*{�&�;���r"4�W��RZ9��0��b '�����CM�%`��|��ƪ[Zঙ�ޕ2�ؗ��8�+/P�>�%(�R��W��DS�IL�S�NjQG.�Ɖ��tLU�eE}�f��#��<�L���@�2>�J1a�ZZր3�L��dTðM?ݟe���}�Uw��Wm�	V�������s�U����4�OG;��ﺘ�}{��V�F>�P�iQ1j��]0�>�^"m�&Y,ب%�؛
���LY[�;*�U��қ�����}D�fbX*(���
���T&����Ok�RsL\����_���5���ɤ����_����OA�3��&��h��i��U.C�U�l"5�<�I��hK�&�h�o�����B���Kq�5����8yW6=�J�tK��31�������W�NEke3F��f2�|�(vӯ����L@�*���ۘ�[n��LO�@IK��l����u0���_�䈎i:+�����[�$Bٛ" �� )K�FL��R9L��C�tG��M#?�en��ǁM��}�����>#�u�pn��?���i<^��Q�mJu��69�+�T��"�q)$$�1����r�"	/KY]ˁ!�b�;��K��ZI]C�f"�����3��R
vN�����b���VP��&`����QؕFe�L��_ᾶK�����t��o�*���G���<ԥ3ݧ�tP|��HQzl��s+s�D:�X>Bjzrǥ�t�U��5�O�x�MS�Q�O��{]������4��;[�*5�ē5#O����_�t�7�';`������;ｂ=�P�I����8�-NmQ���j��c�2�咩o�3"�tQB�QL� {�\)G���qL˔¬%��0��v�ѶF��+��}��r��d��*�5�����
�� ���'i
!߸�E��K)0�JӌD6:n)�a}��'ӎ��KV�'C�#df�V���4D�v�:��b�P�Eb�6l�D��3N
S��Ja��T�I��t'|���5\�T]{�ϦV@�����LL�>.�@T�̩56��*u��������i�ۨ�=�R�I�
i�n�&���\)Ygi9�$��Ԏ�&�k���?���f��m*�M}%=�h,�M���H�gB��i&F���W+�*��f��o�އ��\^[6=���W��is��j��pLi���$����|\ʍ:�d�;ٿ5u2��Twh,K����q��P��wU?��%n[T�x��: ��V�X<����l��PMs��c|�=�L�ꄅ��x�����[؎g,TF�%�A�rcWe�2UF@�:v�;Ѝ+�(��e���'5E{u��J�J�����${M�i ���d�KK%/W.����׬��-�6�v�u�"��a���9ކ��-�݌ f
��u�����B5�pu��T2�D�}�C@4:L�v\0h_H�Z��Ӹ��>7X��� ���``Z&�w�.c#֦yLl]�B�[v�&���C�������D*	8��y�X��M���&���e�:xSiT�Ejĥzk�_��+B*S=w�)5�ϕ\u���S�0F��V+�0j����HӀl��m�m��0MSM�Rjr�ͷ�q�xF��kQ���#�C�t�^�0��/#�>0ܪ�G.d�[3����]���͂����#����PՀ�_a������nb�F �@3�b\�aJH�������ܲ�7�v��� ���wæW"�7��8/T��~#Uۍ�"�cм�PV|^=��Iԡ=�Ĕ|t�	S��R���d��}D�ߌ���?��bq�?U�����[���w�d,���U*g������A��`���<��7�7�ʊK#3u�D��p2�D��I=(~�R+3�r�z�誗����=�]2�ɦ�3R� �y�Q>5�q�X��8���6�|�~ �є�J��GW��������;��B�,    IDATT2C�#S	#�y�U��/S+E�>:M�[������s���/x�4N�F�S�� T�K�����Oc���1�в�1��I����,�)k����US�l�B�2F�)L��}��	U&���^����AD{��~)`��hR�<_��.z���=��@����@cnc��\6-w�>�i�dk�CS�w1��#k.��O=����jO˨�Y4)^
�rC�ƵT\b��,�)���~��L���P�F��Qa�Ȫ&�%w��8W�T��tHǞ�?��m����Kg:F�R���k�2�edZ����&�m;O��G��'f���&��z�\JK�T����֡�Z�Hqdb�y�v�z��u�s5�M��R�)�k&~�dm���6RV�t���X�Y8�<�_�޲����c�X|���WQ�9�B%�|�#�Q��Qy<�^2)��[;��Zz���AC#�r�Z���>�j�N�v]1ʤ�~ƃ@R�L}e�8���o�֪a>v�:Q�v����6�ĵ-n�i��%�Xg6��+4����i�}( �\'V�H��tz����&c�jȹh�W9�X�*������E�N�� 5����
`M����μ��h��w1�3�$g�O0�����է3�T
�M~��K���lE�IU"˱Y^�����2���|�j��,����=��0 �����r:�Ɓ.�L`<�,���Z6��+�ņG�A�����@�|w�nӀl:�4��Rm�]�ʵ��,�OI;��rĴ~z���bY�GY1N�°m_�4n�	��du��j��Ҷ���w1�v��P��t�O���ٚʀ�Z�E�2��d����m�z����,�� mw�h�d�nQ�֑H�G6o,ki%V�n���40�,-+�W�W��[ɛkP-����u5�FG9JS��osB�Se/S�ZU	kn%�	�m��i�\�RK�IN&��
�Nr���JNr��&&3p*�Ieo#F�;�mͷU7��[6=�晅��<�W��t��%@�ێ�
eV2s��OeR=�T�*nUg�A���) �P���B9"ݣ��㧹���e)o ljfb>�Ly ��m���M��e}y�Y�;f˫Fa�U��I%&���Z

f�1)W|N����j�B��|Yʫ�A�F�]�̢���m�X�RF����k&楌i*���I�_�P�o��bn����Ɣ�nYU%g�K�?F  �č]���d&7��d���C+�+u��S�%�2x$�9)�ܭ�˶��:k�$&�NF��1d\!��|9O����G��WI�6�FXUNћ3���h���-Ʀm�,�5�)�$w \:3*���W�Q>�i:%-+=&��b@��D�m��q>(MW�[����ő�`$���h ��}�=B�/u�N��Ɗ������)��m�����S�Z�gȔ\�"�뀸���
��hH�n�r(s��`
�F���Խee˵qN��!zi�W�����?<P�Ӝ ��$�	|2wF�I@s���v<�$���eM�#�(�)�Ҟd��_k�ªbW�S)l�)o\ږM�	�w���CIg��6s�7�yhF���aU��lriz{4�8�V:417�TTN�|j�(�|����aj�و��K�ֈ�L�y��{8��1��;P!g�< M*��k�U�+��n��ɏ����t����q���U�#'�U8��,OAB�KeJ?ߓ�#-��5�i;U�4��=-�N]Yin�%j�NB�1	|���M���5�Jy=���yղ��Rp���*NA��W�B��Ԏ����Q-�x�yh:Ӿ����8P�}`���P�n�nq�ǡLFP��QQgKy���ڷ���(0k��t3�O�/���5�s��:��]=�?�Ϗ) �Ui%Fvr��8������cL%�	�q NƔѲ�1q씗��c��E�4@����4e9��V��PmO�Ut�0�)��Q�[��T��x.L+���ҡ^b߽��j�|	T`6�F�'����:S٩�,�e�g�Z'�����V�h������Mĥ�ޘ�48Ն��m
��ށTz����Y���SK�e�M�	�`R���2�
f���{����!�8���e�O&���M��@�p,oF�z�k�
̨�5�`+V=Ĩ\��Q^҇L�2���_�Mc��ȝ�,}�:���ů��ϳx�� ��6ԡ��ʢ���f䳯��0⛉9V˦ϬdѲ-|�%۵�`��Q��X5����>��r�|����rt��̄c�7�O�A��&-��Z����|tD�a�o\�B�Bj&Y�䉓ɍʦLN�#|�}�Ԧ���KvYl�?z==
P��&�0ͷ3u�2�E\j��*�3p�����F��$�U?8�����j*�U��gf�kd&�� �QF��]�7�L*�S������̧���?���/מ 65��1����ڐU�Sڨ����f3IfԷ��5N��������Q}�:����K����O�MC����͵�_ϵ��tK0�c���'�g5�ÚI/������Wm�&�K�_e���Jhn�O�-�ت��K��J�1���MX�����O�U5� f�1&��t,ZN����5��-;x�a��g�,��;��G!5jڔա�Q�-��4d`RS�ִ"�
�8SU��oR��ᥨ�`J�(?2b0�����ӧܙ����	�M`5���t;�>�)0���PϖM��YI�yUֹ�~O=��~�9���\i�*�jۘ��u&�[���j���$R�d� �'�Q�d��ׁn��C�����@�qp���Y*P)@�o_�h>��|Щ��B�P�v��Gm�����WmC�ϳ0{&0k�/�����TY$�����/
�$��_�ŲЅ򝦠Ҙ2r�{�(�^7��:��t=�>�\�H��jR`f���X*A1�"H�M�%H90�obN�«l�M5q���Q�]�Ƚ�M����D8i��$�>"I�K�v�h2�\v�n����QY���UY_�W�x�M��YԻ�	e�����c>� ,[\ZS:�t����6�5�Z6�&�q*��K�͂��\ _�l�|G̜ ���Ȱ	Ѿ8�H����i:�,��ΟZ���ѥ�9����|�-��Hr�N;l�A�a-����%�Nth����L���E�5�kQ��*�/�/]�c���Yf�H�ٗ�w�2���2��t_��}��|걔��/�������ՒijZN�ֆ24��Ǥ��	� ξ�P�ӈIMr��8=�i_���M�	���ܫ�2mY ��?�G��\��@&E� ��?
��>��%�����ɝMg�	u)n��/�E[��ty$��h��^��N��d���3�T�Tts*�61��q�l����T�y�� 3 ���B�u<{����ԓ��]}CB�]#�3a�;-��Ҏ����&�
1eQ��c��]d�/W��r��^��|��������8�3�'5�LǤ���A�n�i�
�;�^��`b5�$,�@�L���̹[>��<�R]v�����R�R�(�A"�G�CL������Y�}W�m�]C�4	t�ە �b�S)�$(�Lܾ��qy�0��6��6Մ�y�k�K��[�*,����t�[0�Vt��6yt6������&���I���jΨ� ��QK2��f8��0��<t�և^�����o����_8����R<M@����}h���!�>�A��5�:�g�N��?�c��sLI~�)������,�g���|�F��*��D���QU���W��&��q%�
r^���ay�����2 �4��8��;�T�_�� m=1�<�5��]��/���?�j�̩'s��_ֺO&q����$ɾ���&*0�n/w�v�6π��[�N��2U���^='���W�Yo�b��`q��a��C�ɿ)�@MY��?��[��m��y� |ऒ��=��)�9�c��P�LN��0_r���K
 �G�r��^�<T2�����G�G�<��3#P�R5�&��t��\��Ҙ�r$��[�~3R �Q�l���	��ߵm^8$Ӓ��5��ӯ� ���c!�@�c�hۛv8g��tM��c�f�i+�~5�US�ydb�͝7hM�W�E��L�9�����Eә�T�]�tjy}?u��(�'ǌqLK��cʂ��rj��`kߦ����,���Hv ف��B�V�N=�#h�Em	�4��Tv�A�KA��B��~��A�\�.�MiM�.N�Fx�Z�@��m{%��`� �2�Z��8��7Η���i�p&�@�M�j��=�1k��[P����?z�X���MW�P����U�5H��f�T����r/t����Y�@(�R��* U���eѲ�6?�o梅��$u)�L�LF�2�^`M�� jw+��T��\�&�T0�X")�Xh�A���J����s��Ҕ�to�zn��ǹ\*��rT-Z����^h�Mp�#0��� .L�F�c��	��L]m>��(��
�8`����[t��c�i���`�Y�n�f����ۋ��k�ϖ�h{��?p�;��*��������'}c�
9�qY�����K꓂���}0�J�ebz�iP�L5���:�8g��$չ�T��1b��.߾RTJ�i���I-�����<;�=���>	V��,���~�;o�0hc�AL�('mO��v�J
58��*�s�ǀF���!��T�̴vO� ���Z;�`3y�8p#9���KF}�쥠��`8��D�[S�c��z�}�go�+�B2�ZAi箷�'���M]SU�?���,�ՠH����c$GTB�hV�"L���cP;���몸V>�+���D}��T �8����Μ����qLJ�QFw�H� ���[6]&�s:�y������h�޶6L|�S_I_���1�ן��5{�Pj��28�ı���-l*�se�Z/*�M4D��v��7�pt��<���<���UY�V�T�������A-q�O	1k�is�x/�[/��ռz�YhoG�ǲ���O��9��e���kR��ߨ���u����A�%En.O=6�D��y�C=�7��fz5�i芚�!�%�%.5Y暘����!���Ej���� |����{E��������%��8௸�u|�bs�UcR01�45j��S�><�n��+�^H{���i ����O�?wk�� ��z�_/w��ɊގF�%�����.��t� ��ŵ�HM5��ŗ?`����uB	c���3�����&SM�A��w\_����&@�~�l4��((C�6���P}C5�*Q�Zշ�j%�	&���@�"�7����A_HG�M�w#6�cQ�m�~- �����M5A�3��H�8N)����ޗ��Y�ă���x��'�ś�E����k���j)A�eBST��Q��A��40 �\.'����Y����r��
^�=^ނ�H���QS����|TNǥwC�.h��f<�����O����9��zi}��R��L�}�������;s�d���(X5}�Ѫ�K��B��s��U��Y�/��Ȕ�<5G�i��Gue_�+9T�]��=��f/`��Қs�]� ���+��V :~˦��p̌�=�܇���5�,�o��K��/,_r�P�7�yRv��܆��J���ᾌ��Z�
>m�DJ��y��i�	$��0�����~�B;w��<���x?�V�Z\yB����6���ι�^�r6�B�gRfn�j����T>bl�u��8�\g�q��V�+��I�qD�Č_E�8i�\ۮ�0�"�y�r:���a,O��ST�y�V08�%0�M�H���2�p�<�ߑӲ�7L$��\x�}ڿ��g��Y�e��O�~ )(�gR������zx|x�ˉ͛�B� ��۠L5dH P%%�m4H��1#��ye����b9-��#�Z �`e�9�Y.�}2	6���/I3F�j���\�_�/��[vp&-]t����o��~�*�S ߰ �p�Ì��`C����+�/��'�����Jf����2�A�~��[��$��^���s��G5O��xTW@�zz�"P���}V}[�s�q�w42�S�X�ީ��M���J�]$^\���5���b�~��Q�|dkRSMLl�{�Ö���0��c����^y��+��n��u�kP9��<u4߻�����`�7���ʺuc�?����= De�P�7��PF!e?��WB� S�#�=��mj>�#��w��P�����ޕF�%��� ����L �\Ǵ@�s��X��ZG��I�bf�j���Df�w�c��{a%'=�A�TFv�>�f���I�c�`���71����zC��Z� C;�t2�xe�Ge.Su���H�6scҠ7�I;�e� [eV��u�;:?==Sνv^b}%���3+��H+���}E�e�:a�h��iƖM���r�l����_�%��m`�ܨU����p�C�[���"y���u��p�=qҿ���v��F'l���=_���������7���-���"F
C"�2���D���[�*�ն�O.C��Oc�G�٬�N��ɐ<����UF@g�Ծ�Q'��{Rk1���D���C�ŷ��zC1$È�����
��?�_T�e����������|�x���g�pa�Ȝ��Ѱ�T7��"*`
�d���!�D m����:��ح\u��k���~�fi8L�o���}2Cʕ����_����G�^��Ts�^���=2'��å��å"��9,��c���{�����`�x���=�d�z��~mԬu�3�,�0��LKT��>qL�Kv�WnQ:��|��Y��*���N���;wg�w�`��ߙ�[kG������XY<cz��������S����g?W���5���g�x�h0Y�!����6�Ѩ-H�V�
]5B�	Fc�&�U�.ȩ�7MnP�D�^N�Z���U�4�^��V0�ρ�O�;�Ҧ����X	,����׌����3���jH����݅jg���s/��?jΔ�,�꿙�p�o	��4�
�u����f��}Sj��1@�Au'[��/Ӹ-[xɂs{&xM"6�y�`�UᤆH�cb���q�zN��u���΃��[�5�D���'��7.(���'��t0�)�V�2�P�))��Y:&i��:���G\t�˃Dr�̪�v6�O�b����y@�0a��}g��iR��o/�b�tNk�Q�L�t�)p��W&N��������5d�7^�O�쥫�瞟6/7݄�5�*�U�Qk�Tf��b�YM�\j�2��Q^5$�W}U��D�U�^69���e��m8�ˀ�H2XGv�X�+m��޷^�����d��6�����R��չ�p�yü[��d&!�5&<@��*%��L��^��R�bʪ$�;R�h$9'��T�r�lUsx�YX�;W�Z�
y|l뱵�Ԋ7�\x}�$q�Ǧ�h�
���v8]�K������|p���b���>���Du4v��&A�x�,���&3����Q}��~F$|���^2/Xp�������	�`��Ti�t�Ϳ!���e>��ڧ���r�M���I���>:�����\:���ys�`�L��b5�KTS@I�G��4f�!K�R֍[O��TV:#e�54�ZW��l-���X�č�����}-��)��������.�W��Z��b6�wlt����c�݅�w^�x�!�E&�>����+M�JH5�z�}M�>�tyJ����A څ�R�VO�J�E7e~9p�9,_����a;�/
G��6��5Mwu�ft.;*����8�5]�|��L�"��������N��2^-�o�����ߴ����B(`��*]>�)ۡ 3*�F�k{`�D�+������)�R�wH�ab�Y��ؕH����<c^֥ef5t�)p��_&O����gF��M����%z�
�?���+W]x�K��⍸�:0)?��*[�4j�5
71��G��r�W�&s)E3Ғ��I5�3Rw��Qiyx�!���83z�O���yތ��)T����᪁��.{���<�3��_�����*���moy_:�\��    IDAT��~=����|�~���h��TUDl��Y�6x�@4*���%������G�c�2�SZN�/�-�z�(�<����ߏ};6�	�G�����7��3}�����b�C{�������?e=23^�8�=��k"{v^K�ϣ�o?������J��aw�L0�����iT.��Fl���\�i��TO�=M��ҥ�� CF�|s�,Zz�+����.����?�B1��U�<��B��%����z��&>�lj�τ���9G,���	�4q��6)�������������?���qꂱ���-�y��jG�^ң��oy�%�ΐ�o(��L�%��is�� �ɵ��A]o�0�}Q�k|ɲ�qک����_�B3�a/�l�>��ǻ,�~g��
o�k��x�B�^
���1�g�9�ɟ�
N�@K��O��Kex��¡�������x���W����ŋ_����u���֤z}:�1�>��FpA|�,yP�K<����Һ�"�ꀏZ���;��-Y��%�x�Y�����֫:��l�>�w�H���kX׊���ǛuZ�[a�B�+^"�λp��~��е|A�ث��^�B��J%�ٳCCCpݩ3���8�>�����o��e|)�أi����6좳�\jd�����G��ݴE$b�BJ	�x��)�� u��tđ���y������;J%����G�u��[����U�>�k��'��G�
Х��7;�T^�Q���msޑ8�y��;���Ϟ###(���V��b�e���^�B�o~��G���u��]�b�CW�=�em~�"j�ԛ8�1E���P�����kR=bH��N�h��D��4A��[p� ˗~���{���I��#�C^�c߷��Or���k�r�iy�^k�Q�q�w�j���8<3�~�y�<����">=cp�*&&&}�2F�8��r���߷E�j�d��Q�����7�\~��/]s�g_N.Yvqv��MA����Ws,����P��,*��-b�f\�HD��b�8)�h�x�����(�.���_&�Y�����ز���М<�^u����nv'��8�C�3�o�}(���g��ղ���������:~0q�?݄���en�|�:������L$����8,_�����5q�ݻv�N$������h3 ��}�6�Z�d��?XRX�����N?��lk��ᕶ�I� PY�RQ��̈jY$��h�F��-��zU�s�#V��`�7w�g��Y?���g����2wjƑ8�8�q���jϛ��s���#/�&���(��zғD�N�=�;�s�ϐ�&�������ni��l"��
���=n)���`q,١��oFu#���ON������H&��+����w�ǂy�����?��﵏{�U�����{`?�d�_��=���yҶ�3# �?0 _�`���
�T	띂�x�3f��x����������7��Ag;�/���5��/�";x3ٛ��>~��}ԇ��3��ʨe���BpB�s��	<��[���龟�D�ث�#3(���������B�2����ó�@�	�� �~U��e�R�b׮]�����]3�]�7���'�
�u���=��y�;Y���SN5�ܾ����u�&��W�Dx�~��5yT�ҭj�'��	[�銿���(>0�e�PN$&y:}�5k�����Dk-�2�gb��"��OHd_�<��y^��n����W^jU��Mn|Y�L�I��x=��S=�*��Yn�!;p]b��\��8��k]����z	+`��8tP��T+?�U.��?0������5ɇ/��y��CW�}C�?+N�/��u���义����m�`m�
��D	�+�GLK'@��i;$�U��u��G�W�����wޜN��'�\�%�tɍ=��z�g �v ��&�N�޾m��Bu�S�{?~6�:�R��x+n?ު�f�ؓ@����^סo-��I7 �+�����c���ËC��J.��!$�� �jF��s�1�^
	��Z�Q�δնR�`Æ(�˘=g�9d3��Seq������6Nz��qϟ1rۭ+����:�y�]*��ض-k��_!�y�L�I��㩑���v��h�����>��g��;8ɻ��8��W������E:	�::����\�i��y�ǽ�N�'�;�>���{/E��ױD�4V�9���*{?O�=�S����q�x��Z��H�'ځ�,p��X�wK��=~�յ����;�N/b����kd1p�B>�K$Hu�6�
�M$�-��I�N8�$�{�`v�<�!�q���w_��6���Y_���\8�Z����gcbb\g�U�ډr	V�k26>	T�@���z�J�B�V"G[�N��t�ko����mo�S��,�����'�������.y�ʇ:_si;v�'�����V�fM�E$ҭ��w���C5:����'�sd^Ⱥsg�+�Y��b�x+�uu�#������"E���~3��Ɍ'2@j \��7�.TU"7���5�s��s�@������#��*^�9ҖZ�B����-;��7�r��c�!=̴�(�b'���Mq���`V�2��״�h�
:/�8�#Wރ9���@��P(a�׿>���ҦM+�d~1��Z�}���Y>��f�lc�D��8����i����j�������5�?�]g#\o�76�L�E�g�6>����w`pf�u�`�L�i���k�
c^ ��\�r�L A�,}�Z0��o���y�mH����{��"d���`�f��H/Kd;����K*N�::E]�V��K{�pʓ�c�.���s6�s�a�ǈ�<Ͳ37�v������nѯ˴�}N�ĵM��y�n�9]��r=����o��V�}��51�I��|+��hL����$nv�W�@��޷�u2����w�� �l^z.p�m��w���_�c��>�o_L;;v$��N��;�%�>����U�r�P�����p����J ��_���6n����7���q���crpx[���� &����Ž��
�`��X���j�vm�ޕ@a;�̍p��Y.���� �=��$�3�Fu̵�zw�Z�ꢷ��v:Y��*�@iP�����L��O`6�3B-!�=�@������a̝�E*�l�e>$�PDs�[����ғNO&PƳi��Ύ`�������������\�}�8����~ع�x@L-����q��L�Ǟ�v�9�N�v����g�� ��9;;��0��m�)�;}H�z0�s-��]��ř�!�Ň���8��y{��`�.�P��%�N:�^x���z�C^_�H��ϵ�������U�����Ni
���!�0fq`�RFiF?ʝ�ؾm;��s(��q�b��
�V�gk<o-��sۆX��������=P��u���X�GG��hCOo7���zH�b��mO>���؅U˲�Y���L�����&�lރm�*�|+�\���"~�Att呴�!s���N1E��`1�o�(ܗm�Ӗ�v������Lm��/��η��93Q�O%�2���v�s� �G���v�BB�.�{��a'<�ރ�ٳ�kij26�n��[��l�)�tӊ�Sּp��	����G�9�U{���=��:+6\�������΀[xb��L{��a3۟�&X�sL8x_��q��]H�σ��lޒE��{Q���gM�������#`S(b��m�5��ͺA�x�"���C�N���V��������h˰@R�(��˱:��l��(�&xΙ/�m��8IT�.*�4��۰sh[wq���oTH�{�����Y	V&��+��e1�r$y�9�~cc�ȟ����� ^_/��yA����^��4��.P��~2�z?Նp��U���jC����[)X�&'�سw�	&�Š-Ջ^��7ɹ�	��<T&'.��qh�:�ΐ�g@�ӕ����C9=�w���6<��<�\`3/捽Ϯ�5	�C<�¹��,1i��8k t]x��R��Ҍ>$�2�hC*�FW.�Ձd{�.�a�Q ���~�>�H' "L����u���GaÙI}}�f�o�E�u�~MM�Һ�#n���pxvfV�c�Z���fZ�,p�K�a�Mch�������-�by6ڲ���8g���x.d�*1��&�a��<Cy,�J֧���U�:Qv8�F��AGn )� ����X t,�W����nA�j`�=<qx�a��!7
^p4�����Du�49���։1<�n-:��0�-�M�Ytf�(��:	fa��X�ۍ����ƎX�D:�=^.#m۰Ő
�x�p�㺨
H�u
&S6ܱQt����=h��D>�����2�f3�����@~����|��e`x�!���|���Ts�O�W�B�Ąۋ�r���V�3�X��N�RM�T���1��hC6�Q����9�r�bI���p�	mR��_5�P�
�ޱ$r�	�+`��ҩ��=?p��̦����ϱE(Y�m)8<�j����}s�6��*ڙO��sw�p��T�QϝǄ=<���`��> ����tK��[��ح�	�q���P�V0�M�`[�dF:s��`��ك�#���-�\���O9�_<�ˠ:o{R	L�]��S��g�����PEj�B�[�g#;9��û�h�",�?�Tj��}���XP�d�}������_>��{v���F�I��8�FǱcW�sY�g=OX�5x�=�8�ŭh�e�߾��8R�a�R<|96��x5�{�&�q�.tuu�'������$�\E�j�X,��u<��u����a�u�Rt ��N��p���=��Ϟ����ںu�i�!�,S�D���κ+��T�:������Աxo���j�� �z&�`��aU� _x!�\�Ǭ���0:ItΚ��Uxj�d��Q�S��-�+�D� ��z�����J&a�Ў3�:��%���d�%�b�{�𓞳f%�u,��Med3	vdm�(is<�2�Bbze_r��%�eV���,�ر3�KX�s�ksP��ɾ%ڃ�1tww!aR�xB�O���?�i�ˣh~ L<���Ca�����ߚ��d��%/�����a���sn��[�e2�M��5~�̦��2��LȰJ07t�\B��$���l@X���٥�&��£=|����˶=�q�UG�؂�7�I�COk�����3�3 ���p|�x��J�1�0��'{�П&&W���s�,��~�_`�6�v����x�w���D6��b����$���'�>c�&n������[�E���}��_�����6;��bbb��я�s�+Չ����"�l�È��p�sm(X����/9�y�����O��+�5o��<w��P��2̘9]]]>��ލ|�,b�f͞=#�N#�I��x��}�嚈��GGG�\a��&��s]����Q���u�V����A���V�$��	��v�A�d*0u(�]�)�W4�Rǭ��d����֤j�s�\��t�e~G��C���i���?�,�M�bC�jy�ij�R���\�D臘��A%�"N��%/)e�!@z0Lz��{�'ש��jf����N���	�E(��˗��*��@��~K_����������߶\N�I��%���{$��Lf��Je��|_�st阜���\תT���}}c����������\ٶ61��u�f��mmm�x�r�\l��jBD]�H�k��(��s,����(�Jv��M,�?��d�����[XH7TY�7�L�H�����z>6Ω��8�j���/����HL���d*ilp48&�����[��r*�G���](�����[����\nw*��}2[\/�'�͊��6<4�;:2��	����U5��u���d���J�r�h}���i˵�D� �>ڶ�؞����3� )����qږ�o��*�J6���K%�q��T*U����#���������N��`(�L:'�t���ϲ8����c ^T�n|�5kV�R)^*��%�x�-g�y�m7n\��o}��֯������XOO������e~��SN�eM�!.����[~s˛֭[{���x������_�p�_^xދ�;�ē~�J�
S�[*��+��t㍗���[_�}�֓����'����9�lٲ{.y�+�u'�Zl��~��;v�0�D]��,��_�3��mo.,c~c۲e���u�e�?��K���V�OL�q�Y&��tuu����{z�QG�7o޼_,Y��AL�D��f,�n�?�"�Ns᧮y��T"4ё���;�n��_��SG�?���?{_^u������:Xl�G��H��
**� JHH�tH"*݌�V����l����9��p�������{s����O���9~�f�u�NJ#7�?j����C/�=7(--�MnNN�Mja!�����M����7�K*������9b1y��Q��G���j4�����޽{�QZV����{���鰂�;>�����O��ȍ�ǎ]����Ȟ���q����BCB����m�h�|�H�vtt�puu�ئ���o�uR,�TJ�?}�԰kW�����(+*B-M���CV����m�nO�-��rs�ػw��LA3Db��C���BA&
�hْ>35%����۾��k4Z���4壑#�(e�w�����a��Yu����*{�'ojР�
c	�JjfF

����ۨ���w�����d2�H(�88:��������z����OYQs�
���PH���۔��v�[XZ���"���ῈN�~��y����|�SH�R��8d��.]�~�����:/Z�p�٠3����#�a\�TQ�ض]��E˖�mڬ% cd`Ȋ��������Z��w�_����c��Dx&�q�0����'N�f��.ZH�$��/Ə������d22��!d��O=&������999�\6^-���RŖ�ml��;v�<���Wt���"R�ީ�&�lߺ���� �����q=I0FC�D��ܯR)��{�����d<@^n.9w�l�?��r�nc���yCBc�6j�1��ϧZYY�ٹKgN�Q�  $ѵ�W7�7p�x�����|3i½��Z�۴n����G0�l�����l��:9;� �əS�?߱m���d+��I+����l��k����ɓFi��Ge
��N�:�wRZ��*`��O�oٺ"6�}����c͑h �?����=z��QXX@x<>�ZHɺM�FED��[�.�����Ȩ�:�n��]�T��=�����5+W}�䉓�Č�),,"���ӱs��Flmm��K��ݹ}����tІ�S���c��������	u��M�<��`d�`���Cǎ�����������
�\8<44���g����� ��bP-�������BQ�ַ�;/Z��Ա�G[�"��UǢ3�1��]�E��]زcG;;���*)C�7n_�p���b	���u�S�U�;v��Т���<>_��Ɔ`�===�ͥ�T7y�óϟ;���͛炰�l<F�)�̩���G�������ܵK=�>F��qv���e!�0"c�x&�@"��T��c���̬��p��͡���[TT$����׃�	�~�q���.3��ǘ/>7��N�_�\���FemeeoemE�
{[&���ܶ�Ttt�#�N��&���K:u�r�2PkkJ�<�lՊ��طo�H("`�cl��3�ō�כ�KJ.oز��R�̵��Hh$�v�7o��-X����dm|(�z������E"����;��i��ٙ�%b-�477��^`G�:˵�W������8�~���v��I�f���� ��B�ի��z��=l��0C"�4++�r��������	���5%9�^��N�� ����_�o@@�&t2�QQ$##�0'/�2����#?�02��3g	&/�I�fM����%G9�p�����/8�z���B��� ʒj�gO�m�I�.Z�e� ��3�c��    IDAT�GGR\TD"�=������͛��_$4H�2����˓4nܘ.������_��g !3`c��ۗ�hق�ȉ�	$,,�DEFR)�����&K�/O���j��)���`�,�Z�׾�Q����@M��B|�ן~1�i[���zH�׶};ңG�@��#�/����k��۷=�u`��w�=�����Z�)e��}��1W���?��FFD��C��R�"����\XXh�|ɒkg��4QC�ǰ�^��8 ��n'��ʜ$���M"??�b̕f�d��5~^�b�X,!B� 9�{ܨQ ���%���=��~h���"n]ّN�V�r��j�Ƽ>|(�j�Hk���7mD�9&:�ԯ_Lb+����l޴���s�Z�}�|�oܘ�I�$%5�$'&�Lǎ�7N�j�^�����s��K���?���nkg���JUJe�k׮5���q���89c��r9�h�H�Ì�v�vy<�L��������r�Xz~�AX�h��{�viּ�U�j� ҧ���Y�j5�3�`<x�ѣ��'h���.���,�g������ۥ۷n��Vd\�����I���ˋA|~�F�u� iԸ�o<�	x!�C�Tfef��w�մ漣�.P[ZZ�����C����>�ꛯ;��(S�����ɕ˗)���6nԾս[�F�1�>Κh@�bLIN��r��MZ-���V�^^d��_@��b�}�]��Ȩ�|��������i��]m۷�lee�_�����@*�BM,ZD�~0<���2�
�L��hk4-���m�W�<~������q���d�A�R���V�-#:�WrJJ�C�֮\E���I�7h�U�P(��p%$������"�U(�Ȅo��%����.!$_�V{ffdt޾uo�Ν�����ZP^�$?-��8���
8�+))iUZZ���h���>uj\Щ�M!i�M�ͥd�ʕ��[]�mmm��x<X���<y�z�Y���'/�����%%%d�ɄI�f�x��Ο�%'���]:���3d﾿��.O�}��7������7����A�O�B2�jH�t���б��zw�!d�5����*��r/kkH��yyy|f%�X���N���㭼s�v����Ȑ��2�ϙ7oR�N��~:r��
k`�99�¹s�atT�?[`�y�%e�ڇ��=lڬ)E���X��-[Z{xz��ֽ�3���B�67�[�A6��R�����!I��R��yyy�qtta�L���悉@S�x�"#Ǥ$%��m��v�v���)u��p���{��s���i�f�>u�[�.�`6�����/_�����/����W�W�y���d�_L!���������CK�X�1��YY����S ���#9v�d����B�׮\�ݿw��~������3>����l�V{���n�����f�s�;0����Iz��9������
h
M�6%}���Z�tɔv��M�j�x"�G�������|9~���gDD�کN��e��_XP :{��T�辩T�7���?��֫�ձ#GdfdP-(0��عs��ǎm����mΝ9K�ú�3x-G�~�d�7�l..*^v��-��x��RK�Pkߡ�p 3Z��R����#77�۸�8z�T(Tb%&$\o�o����$��I�mA�Py����䚟?w~8��(++�H5hȐQiii�#�>Ka�Ͽ|)/9)駞�{\�q��g@X<�{�9�<�F#f^Op��䤸���G�����j:UY���������������ɴ� ?� -55-������\�Z�3��P!E�����������̌�̌��t�F����L�\��������k�ꕲ:u�4{䁱#4$�deg!H<��ݽ��իTh�����ŋ����Jc$1~7772������$��k5�Y�^X��yLL��������E>��`�qT�#'��yzyֿy����ϟ�����I�_X�]�nSw��T����ZM=z�1��i���0���6�H=y�d���p��nߺ����1yА!�6��}�
�gg2�O��JINQ��j5�Tz7$Z����1C�\���p8�u����տu��d�0B^�<v��B�h�q����[h�6.�Z*�	 �a�?�ˊ��.-�S}^�Pг+��O�<!1�1���};��SFF��{�(�ˉ��=c���W77bkg��^x�����<4����3���5����t��u;15V�(r���{볳s���4FVb�U�����DE��7`�D_�4��888�����������h�bc�Z�y���O�r#,4쳈g����<��@4gg:���hexh�g%�D�ҚA���h`>�T�_��Ӌa�@�-6v3��G��A�	q	n���A-��R�����Ç�h܅H(\f�A��gO�f�խ�|d� Ɂh�o�qz��a��d�܆� �E�w�=4���0��8���PH"����{S��L��ջ�λ�wc�N�ݴY3b���
T���8>����Z\TL�22?b_XHa/��d2�Iooobl,ı�N')O�m�w��Y��E���C@�&Z�F3������n�t}`پ/�{�yv�֕�7j4x�~���9㔖j[���S�J��-]��(��U-�����c\Aa!��	���w}J��yL�F�q&�/�%.n���wo���6o5���-��|���>H���Œ��t��eNH45�����o�i�h���@sG���`�T������1�I�5		�B��E�"��AXl��}���u�׶ggl�PDBCB:���t���ڔ�������=nѢŝ�;�sus���$�����I~^�ޒXQ�K9mȜ;#CbI��(a����u������D]��kִ��t�i����I'�l%3�`�\�V�==�<�R�I9��wus-�a�*8�����9*�ż��x&�a1m޲ENzzz!�[�W�����Q��8��jaJ������r��7!P��O����f�`o��rrv^���s<99Eo�1
��L��	��Ą-֔��$b3:&���Z�	/.6�KG�:8�����6�߼eA���ՕzFz:�O��N�N���MQ~��ԊP�혐��B�Di�b�7�A���J�Ba�5�����W.[N%�@@��4kѼ���t4�����-Z��U�$���u�5
�2�j�8��S	E8�������k�Zk�qa,�i�$m8\]������y���q��~Wn8Ȍ1�����j�>�]lm�mq��w����<B�}k���W���^,@Bh,�g�`Xy��)ց!��ȡC}�ZE{bg�����8�3D�
��E�i�K���
MHŽ_ST,��Ζ|�:-����utt�8�\�4iJ��>G(�w�3�"3,,��<�N.�nݺ�A��c��O\\\I�F��葜�d�577� _??Mbb�����bBS(,,�mX�V�1H}�܃a2M@(��RS��u�wzyy=�����>o�%04׭[�p/Ml�h�|||��2k#�;���@�t_��`�\]!���^�N���h;`.�BP8s��*�-�q~4.:����]�����I�Ν��h��IPk����Yo����AgT6B�����ԿQ��~^��w��%Jp�8P�Q�Z��8vl̽�𡋖.ѧo��@0^	����2���P���l�t]��_���*��rFvB:5Z�5�KWNW��_y$w@bJ�l���?B��䈘+��pus%�?��<}��w���(�R��Ba����`�z�[#��P��=ɝ۷�A"<}#�y����)u:]�,JԵ(���B/-}��X �4hzjt�J�ʆ���s��� \��K:J�2���iP͹��I(�yA�aB�dB-�)X3x14���j۴*�݌sߣG������Y�� ��$Q]+�|@A}���$';�O8�.,,��XW,��3g���%U��h���榧�~������IdT$INL"Y�Y��Lo��ʎ3bJJ���e��h޲����Ucz{�iYTT����%�|�Z\���"c��w�?�3x�ŗ��!&R.�X��R��J�[�Aip=A��K����V��R�����##r2!�t�u�R���
����$oB|B)S�q]Td�]�ƍ��鳌���ɉ$[s����OY��6))Y,`Qcգ�aMxFX
�E���;T\>�H$R�Y�J�
$�+�D��9C$�X�{Rb�!�L++;�))9��W�+
pg��k%Q_'���̑��H�Yƅv���HMt���ۤ��T�J���D�*n������t��t�B��d�ϣ���_}�Q-�,[Z�x��G����K?��������V�~��h4�J��'##�AtT�d�ҥ$�y,������S���o��Ŏ:��r�ϰ}��"L��Y���x�T�/v.�6v�v;;��\�>S:܉��T�Z?��1����QbG�3_���+�h44�!��"���3�2I�.,��TK�4�}�NC57��g3�6��7o�L�4�\bfV
���4hbz�%�c������%��Y5�$6g�i�Q`}E\�=.I$��4ju�cͮ]���ݡ��R���=�HStvq�L�'O�IOM��^ވY�6�l�}c��l�j$Qb��vqu=ض%HLq�g��&�Ô����XM�v�>r��gϞ
+�t�e����'u���߁<Po��=}�$'11�pTdԌ�11C��b�����x���Ï�:������ �٥�e�`��Ce�B!�l%%2>$�6��s����Ɋe8�^���;��?���ݿ�T�M
�b�L&[�T*�k4��*�j}ii��������sss�����:�n=!dUNvv��;�������ꝛ�;qsw��\}��M8�j��7�c�c���zG/Y�� �o&SD�>�8\��b���D�޽{f���T|���J222��W��Đ��*�N�<��Z�6*�V�1�]hLІH#K+�!a�`3�ޣG�_�+.ۆ����<�
%�;2����d6����{A�6�F�e+U4��K��/T��2�!�Ǎ���f@�k֎���/S��b�6}�VU����Q�,\$1r��k����mߞ��:uj�F�1��^��")t�޽��g�*��&���j�!���gY��C�ш*++ں�_�N�ֈ�'%%9�b���K��,��ݍ�@ ���Hv��޽s��#�uQ$&& ���4޸v����>11����7�q�z/���s��¬x��rTx��Ɔq�7j�s����ƚ��ß>�����7��2��X��Ĥ�w�������������?p ]c��e��,�s������Ş�;"6��0������<�*Q�L�gc컍�5�k�ƍ�u����$A�kV�����;'n��,5/%9�r�W_�r钛7����#��2���Vx�H��q.jl"�.�A���q�=ׯ_�a��_����1k6/??�oǶm~�6c7!̢�oJ�3AA��0e���K�޾<\&�9P�2_�O����NOKw\���Ma��B&�~����Â���M7>!>����CM'�����q~=|�`�7�w���W(vO?~w��e332\�`9����iggO��7??_�d�3G��Z���PIeffHb�/�op����3��t�Y���'�.
_>��^����h��:��Q�?���E8⊈x�8��/n�8v��Ҳ2'F@������|˦_~�7g�ݭ���g��b��ѽ���p�q��#?N��k�=k�%rw��U ���d7۴~ï�oXm�!���H�`MRRR�V�rs9���S��*1�S�c�ÔI��9th��D���T$	���6m��w�^��a�//^�u݋�z�kR?���I���Ku����*).&"{{�n���ףF~L�&$=q����׮}ҴY�'�r��1�ٳ�YYYfXH��IC��� Nپ}{��d���lٸi+,��n�l~���������[�����3WW��y��yL�OtTt���R�}�#%�b2`�@�lg�!�B*S�QI�I$?�,�>2"��ۯ��ޤiӧfff��)��b���l���7;v괖ZGy|bck�qϟ{��ןO\,��I2�M��ϩ�����YTT$�HOk�&7;[ĩ���+V�\�d�b����R��z�f�Ќh�d����c�'޹y�Y�222��/Y��^�z�|��=
�e��Ϟ5,�/�cnݼ�)ߵ�[�ԡ{hgo���??�2i2�h���VK80��͛c}��"-�����|����F���|�Oq_{�- ?7���Ё
�����v��u���w(�a-RSSm�Κ��^�zK}��a�999>O�<��f�k��G*`�[?%P�߽��E"Q��ﵟ�kZ�j�6�BХee�A�qR�~����ٽ㛯�&.7SZZ*�~�Z�Vۂ�Qh��9iۮ-EΧO��Ĺ��#��+W` �{��;s���9xRR�}\\\/�F�KåA	8�%R�T
�H�no��6 p���	DBb'�3����we�>}�?���V*��;�o7a��9����G��m�v�<�F�,�o׾��.���{T�f�����.�#�^5ιlMG�������&R�0|G%T����1b얐H$��?t���E�[o޸���X�_jj�]|||W�X/�ŀ�xl�V���������alqrrz�ɨQ��]\:B�������l�����̪o�W�����r����T����jX�qF��,�3��}��=��V{v�&�/���0O�"Tf�y>��r��1j�7ƨ9">�����mii�XV\|�k�n'5\���L&C�����BQ��Q�T����(�=�z��84T����K���~d%�2|-W��)77�>;!>~g�޻x��
҈���qin����{C�'���޾�M��UA E������{d��?�BS�0 [T�`T�&��q<L`�Ȯ=�]������)��Sb�0>|T*���K�D{�����]��[��^�R"5%շ� �1���"!������z�	-�/�gg��ĭ/IcnȹD�s��DN�>]��'gZ�ZG�S%F�T*/�po�zy� �6s������h�;֛EB�S������~��Ma!M��e�6s'gg"�^�k.���۟��}��,\F=���@%]�/Ǐ'�O�vx�p�W�T�Hg�5�j����Ƹ�B`�_��Rqt�G��4p��忮ݰ��5����l���=g�?��c!L��3/7���(�Xhyp�~��~~���k׾]G�B1���D�3������ߏfP�� �&M�������πF�)g���:��{h�	K73�HH�ց��PI[*�fzc6�i��<,������>x���Æ58u���7n�8r��<��1X\;t�H��ڶo�M�Վ�޻��-�#	�n~����^�q����2`���-;�ϊx���˗��N}�0J�e��g#�G�^�߀�02Ϳ{�μv�A�<����{xt�p���S�~۵��1 �.����i�����Y��H����]��������s`�>$��@l��ܜ�XYa�I�>�I�=K=<=�$'��2��&����q����_��]+�#T�7 ��lb���	�z��U��'N�UE	��	l�q�]��7$	����KJJ��o�~���v�x�tP��Ǐ�����i�h�n�ȧ�?�g��������)�������
*G0���W̜=��=����� ';[W�%����\�eX�Ǳ��~4r��zm�s�^���/ ��20[gggһO�Մo��]\�>�l�3f�q��O���ԭ����յ��K�-}���%��prpp�P�be-�n>$� ��KɺL�T�=oBz�L�R�=����=0��Bv��(..���F H�n=z��tC���P:�C`��rpp����v<|����FX�I�30I����4�y)8���Z���d�����Z���\.���u����k��ͳl�����|Ndgg?�ŵs�./�Ĥ"�#    IDAT�	�� �>��Sƫ�
�⃌����JQ�P�;�'ZYY]#�M������q 332�
)������!�����999�lll��yy*;;�+k�0>��=:�Id�ajFЂ��>�7{���a�m�ӻ��ԟ���r*8�j�����ڔ���I�RAA~~���K�T*�M��*0���[ߏ>�j�ҹ�{//�t:ݠ���N|>�﶐J�l���# ?,4�~�6m`v�E�轀�(4Vn�>��Q\ʡ����!"���I7��ڬys�Ч��_vV���eee
{{�x�Tzis����666.�����X�`�ހ��6��1�I����M���%j�ڢi�f���Ҡ�UM	�^o(bJ]�>B1��c�8*�XgL��"��F�$%Am��v{v�.�ٻ�o�{!�A©`���ԗ	BE�QQQ!��w��ԩ��c��{~/�������\HH@u���}�L����нGP""-��ϝ�ÕckoOD!z�TJ�T]��A@ju|1>թ'J�&�墶�%��`����+�<�|��U	��PAx�T2gfe�!�����J�JI%"�PWE�Ȑ�������Ç�W�|�mݟ{����!د6m۾�KU�r5TF�<�b�Pi���?�Qv������=�K�1ʈ*?6��hА������j�\�����["7�k��>���*���*f�l^X�uk�Bu/:|�Pɛ|���?jmX�^n��g���q�z:�U?�,��sj��Ռ	��Ǐ#����3���1�I]t�ԩbc����*11����;�����g��?�1&:��<�C���J��¿� �%Կ	�Ouf�^����c�D�����2o�9�YX�����k�<�o�?�Pq�F�&ҝjJ��tI��Pv?��M�Q�_�gCy�#�,��7A�*<Ԃ8*���>��Z������ jA��7��S��RE��Ian�d�5�Ȟ^�����@���R}-[�c��X�1���X<��<��G�����=Xh߄�T�1�bp��@�6�D����T4I (�ڴ��BA��$B� �,PZ����BC���셆A0a#X����h�Id��������W����HC��Y̡Ok+��L��J��)���X�>W��*u�!gE��>�����2���TO�%�J�:A|lа!��˧����5�,�a��	>��SRg�����p� �/\0������dnY�&߹� ځ����'��t(���,�V/�=�zI˺U�����T�j���~&X�N�;Q� i��r��h$f�����5�+�6X"6�Oԙ+����Ra2��VS@���H;���Ǻ<��-��y�D5����ݎ��)�^��#x㞓�<_�T6#i��-$�t� �s5{QMA�{���@�eW�S�
�5�7)�S}Y@>�{���a�H�8g*,� 4he�]�
�q��B�{a*�zė��}���������<�م�.+U��T<V4ͱ�#QS�(��2_��˴c-��Ĭb����P[�$�G/�6��}�7�s�s~Z4K.������Z�&�X�@A�f͛�n4��G�/��ci�nx�#E�Kh9���F�7%a�g)Le�����?��r�T[`!z"�~ِ��C�k�&���ތI��Ⱥֱ8b�%���<��LLE�r�{�W�K����7>�%����� �P�C�x�RL�q�*���P�䡎¢��a�b�oM%.�V������I�-�����F\�Wj�y��gWT4v��(Ȇ�.��|S$0�����x&�^8��R9�!���i("j2fƄi�5.�鈨[\�A}}��<���N4l@������7lؐ>Vwx4�\;��A�p6S���f?c��(QU��j�Ӵ��&ήδ�<�{�^�� 3�y,��[Hi�֊2?^2�����8a`A�c ���:�f����3\u��TN����I�C`}���̍��m����3���E��I		Ć���oxΒ&w�/7���MMI�M�h��9�T�� @p\ÚT�"�ŠM?��*)�D��:��;���U�gƔ�Y0Q�~.�f�}�6��І�M �z���}P�P�D��]�?�5:���M���K%O_I�.�YD+t~���Owww�՚�����KǤ��;Zy>M��j��m��ҝƌ�,S��:zW����9�F��|!1�~�2�3h�}�\��	�ͿZ��G+IZ��t�����IIM�wqqI�[�^dM"�t�W(�"�z��J��)֘Pu:��T.())�t:�N�C�!euW�T�%%%^j��X!���T`R����kk�"�RYbae%�J�6@r ��@E�u{m`>N��hP���랖��鞟�g�r6662��"���!���7���<��v��(�ø`y 2$'&<`�o@�Q�G���g��nG�ӯ�VK�}}��k����$Y��m�L[�^=Co���@��b�#����OW\p*���'��(�"�~�� �(ii�c�7~	�c����S�O�e���֜�u�t�:
!Ck��V��Why������bbbl������R(=3�I�9�D���1cǶ�y��JO]�t�P-���I��D����߲U�K͚7?���(�����U"ak4���C��;�ϟ�$-5�E^^n��l3�Ă!X�>���u�ZZZ�4i��z�=vxxyޮ��c�T����`������k�T��	������(�w3F�#�� (yC�/Z�%�q2�i���wZH@O@�ߏ�'g��̮������w�Pk�Z]c(V��u�6tN�y)��ԗ2���B�@`���j�D"�3�{e���ή���M��J�'�c���ܟ>y����S�\��
�u���}������?!+��u%KJLl���?\�vm 
��[@�H0���13��-p���a���Տ���t���N�;�7��B''�p�v;s��F[�/R#�t��x �R��)�XY�҃�
�P�T�x]�1 �/�)�QB�<�߫5j9c����XU�-S��X�U����Ժ\(���
�|>�l���h�T}�hi�Ԟ�z��U:pX�JKi 
G��b}uD�*�dg�����ϝ9�y�޽v�?l�T�T�_	�շ$a��A�N�:u��b}�7}�<�2o�{�Z4�:���1����`t�r�2�~�*�
8"�xqp�ݻ��?n��F��/��:�!�͐P�Pc:��ہ�iߔƣ?:��&+{j��F+&M��������8L���շ���HR./!#?�8m�C�s5t��~	Wd�V�V{�U�F999�?ż	�ȡC�>�\.ط\RbR�/�?�������:�I�[ټ�LPPpD���Ý��	�է� 7��̓!�Qd�yq���7j ���ݻ�\.��<&&�LPٱu�ȨP(D֮ۓ��8m挟lll
�4�ɓdFD�S��i�<��@+1��ktn�M�_����=�da*�k ����1���}�?s�����}wO��������	�|�Ǟ�ɒ���0<,���_�;7�ϧ��|��ی�=f�����G��a�����ɓ�w�O�H�ҍ�rx����P����]�
P�ytJ�>���=!�Z�عk��͚7����>�i�r��	�
m۲eR|\\�M[����+&F�&�\������5U�<�- Z�Tt� ����&sYM\yS��?��h5�@�c}<*� ���bFEF�޾ykhH���j��k�o�C���/����7�8t����8'���D&��i�ܓO=����{Ӗ���9��J�R�ׯONMM}l���5gN=�v�ʧ�n���l��ӂ��u���sg�v_�z�B������qE D���ѻ��?&T��H,!:�֋}j
��uSxPkB}�׳	��k\7�&��ɓ�������n�����Y���͛W'&&z����]�|X8���K�߸v�#�����>|\bbb�𰰄ڮ���7����9~����%��
����GbbbHZZ��뇫�*%��7UO�X:;�7�]�v�@�7x�z�3� V J� :â\^��������;�鈑~�aa@^��3�o޶��D"Q��	 4� �*�8�������;�m� �:N�;KZ�j5 /'���\����=*�,�;^�J����_�����(e1�&Uoнh����[c�_7�k�(�˵6�/�"��d��k�Qkŏ��Ĉ��0�)�Y��������v�ޕܮU��8t��c�F����-�����ִF���gBʁ�ѧ�E˖ST*�i�DL��Y��� ���D��B>^�dI�@�ؼ�
����T�.EBcx+i*��3 ���RZ���>Ϛ�h��R���M֨5�֚WTp�1��g��cU��[YY���<,h���K��ݚ2�;��yp���-��L����&��C���|�B]X|A��V�������,�[���HcPk]�3q�+��bc�踎4>R.��0Zk'�z�D��buY�R���������Y�}�]�>��A�N����YS��	&���'���V�U5y�xԀ���ג�or��Bm����[9��Yz"a!rI
�큃��e�I ����:���>�ֽ�nE�at���fV'qx\��gϮ a�3����3�� S�|�Vm���J+."�����*me�pF0���|��̬z7o�p�����cc����R���2�t���2X�3��x$'����B�y�2???���.��Ң̔�� �ТR���HK�Q������J�JU�D"IA�&�S���CU��h���h}\9S}M@t}Њ���)^����H$�� �֌�=c2B���M�"X�"��@݃��"��5��Xެ�9�L��o�7N�g^�reZ��v� �j,`�TCT�2J��op��ɖL��p�G�rrrb�a��y jsN��QW���>0EG�R������C�x��qׂ���%2���y����7h�0��[oj߱��DR��%s��������'G]�v���4�LfŒ�����V�n�7	����ū&��*0��Q޺y�ߍ��>	�&�#Ɯ�6��ja!wvvy����C���uqq�7�d�d���A5L:A�����>�knn���˗�����P*$>ګO��<Oc*���HT�3�$+�|j����#E�ک`�yyy�-[�·����Ө��v��U"���ʆr��U1F�GFD����A�E*''��P+Y��� �jgoOcG��٥*w���h��
��ݿ�h�����"��o8�K�%Ȕ�KKM�s)�R���F�=�u�6�u�{�,���A���mzZj�;[wc�RVV&���G��:u�|;h����9_ V�X�3���իo�]�z�����"UkN	�k��.-�;��9��=|����yZ��}�6t�H,�:+�x��@}�<N��n�^����j���Κ13(,$��5?L�����Ɉ�����3�)��t��͸�k��0, ru��\9)����4i7>^���hgo�:_fY���meA�x��6NB8�ѳW�����[�jB�<IaQM{`��8�>~UG,--�B��b��e'�9�ȃ���M�����h7uh%�����[4E���1���/]zz���5i����gI0F�Ba�j��N�8�~m���7l@�[7w7���C���l���;�m�����b�ĉ��"�I6}�6�ﯿ�._�d�'Su���x��{>~��<z������,����q���6m�����9�D�մ�T�}�<};�1!h	鞟�_g��7�̘=�L���|>E�\�33:"U[am��v��i���j��p��=�KKM�����٥��켪*��NG��P��԰����<o�j�����-���U��`nfN����j��
����ᚴ�t�į�
:�|W0�-;��.]�K�$�A(���qZ�P�J?�O��+Xw��f�>lck3b�!�˯]�s%��֓&L<|�bWP� ���[��X,>N���+5�Թs��_~1(#=���3�ŋɅs����d�����[[��ME�B(�w�ر���cc�">�r�82j�g��+13C��x���F�iYVV�3,$�i��$!.�\�|����[v�;��R�Ա6�$�0K/�H��%`<.sg�:{���Ƹ�����/�Xs���)�:ѥk�j���<kt���@L������YP۠a�t{{{]jJ*ϰ��qlۮ�J�aA�������Aq��,#-ݒj'�'83�Z9 �1z�������ꄉ��!=�<�["���}�/]"^�^K	!+���
 1��OZ�nM�еn����o�]3w�l�=�v�=�5?��p���*�%6c�`G0.J�;k֦sg�vŻ�u�N6oߦ�������'�C�6j܈�������#��;v�k��Λ�}�VN�?�[�z��Q��C%V�}ٸ~�O�c!������H��\{�7.��1�A�f��y��ݺI���퇳.̛������h2c֏��� ��T��bg7�\��CPXhhs��%�ɘ�??|��)�%�W�u?�) ��^h��Hw���i�lllPE�i�BQf�LY!�@
V_�*���f%�j��x���⁀5���֔!��gR���eD�Ъ�%4�0�R�xx¹�� �B�@!o���ļq����]�-\������ N�����|��g��
Axn���ͤRr�칁�����q� ��ם�R��KJJJ<��b�f�;�`c��O�o���O�_�}��ꕫ������`�o:v4��A<y���)��=j�i���8������K�� >366v��+�w���W�՚Em۵ݿnӦ����D"y�� ����*����3x�E�q��S��m��/_F>3�`�ݻ�+�'6^+��V��Z�ĊRC*؛����S�|jU�*H��\.hU���}�~�,�Z-O�35�W^�
��
��m>�J����t\u��|�(K�`5�Ry)Y�a=T��:�� �<�Ȍ�1�g�Je�P(l��ĉ�@B��O�<�@ۘp�pp��sE�񿼤�b�Ν�hĖTJ~ۻD�D.�ǃ���\�\)��6&fZ7�AXc322e|>������5}ͪ=�v�/��Ab�����ӵA��٠3�
�X�o&N��.+-]F� ��_�#
�Z�Xfff&��`(2#CRu@u`8�]��̝=�|Xhh+�U��t��{���n'��_���hH����5I�r�!�Ų�kqoff�����-g�J��'��q\����z�Mv��\#��GR�	*cF@`~�J��fb�I���R����m>�wà����T�����+Ex��KX$-�Rbkg�W�T���mZ�����@qVf�[qq1�k���Vgr�����/]��0 ���h��kH�f}����A���2_XP�����U�^��^�L._
x��q=���J��Tg cHKKs�~�Z7��Q/i���!�a�����0���@Ր�������WMV�ʢ�_�����SXP���wS�BCC�b�+׬&}<r��G��J��hZ	Q�Wc}ML��=v�����G��dc�V[���\i:�擒�J�%�*Kgԯ_?������Z��$%%����LI���J#��BQ�^��՝Y1X���ܪ%TH����K$���Ґ�knNs�R�E>��?Y�1)�����h]o�8�����:,
�J<��O=Ҷm;��� {p��ׄV@А>}�Ε�d1LJ����j>�(�0K~Wrn��#J.�?gQZJ�����̢����\g �������c�c�����h�5}9�Hyi�,��Hb��*�����#x�������SN�����wk֯�_���g�&W�GL[�uH�����?ژDC�Bz^`܋gK�U��:�Υ��HO����[VeL¦��ԣe:*�`M�B������i�g����F�>��Ҙ�O��!{E^*��nk�&RZVj������ˇ�����n0���4��Ẽ!�99;����ye���bY���ӧ�Y ������&JOO ���צm�ӵ)�*������  YIDAT�6=#
�§���ֽE��*J�x�-H^^�j���H�>��j5�TDT1�t
h҄�Ԧ����s����奩�T��Z{	%YYK.:r�nK�Ð�8t��]1��cL!R\�X+��G�Q	'A�@$�������f��R�LրrXZx۞4�omnf�G��@��R���U�����S5ZM}H�;�n��_�*�9`��i�t�"�iF��@ŵQ�x<�L*�2��+Z�U7*�Q�}�&��:�N���(�j+�b���C�䪔���BK|�;�n+�����V�k�&B�P[TX����]Y)��x���X&#M����Q	���ؒ�*�H${<������|�43�V_н)+���
ה�pq׎��ǎ��(5''�3))���ƚ�۸����z\\�
3^�G��g�����h�пer���?ҏ����#_��Ұx�ƪL�/q��n����p���Ψ��ւ�:�G�Sp��n���k��JĒ��LW._�����tt��\�jͲk
��i(��f� ,�I	S���"�2|/��Z;�~b���6:��+����@��J��JJh-H���WO�ҁ{NM�I:���<�E�B���A�¾��r��p�i��{,|�յ"4�!)1����[Kǒ�9�?7;�2���A�INN�DEF�`!�0�a<�ܣGO�V�~`0Dr	Z�w�o`�(#aK~u%�8�W���l۲��E���Vk�k
C�q�Y��7��<�%�Fq~^~�U�~���1"W�}�[����pwAȝ���F��-�ςq�� ��Xo
�X�1+
:}	����3H֊�K�38J��cU���ZH���4�?��V�,	�6ʗ��ژQ5Z����=w��S���RW�c�kF�7l@�ZwOO� 7����T�I_<��X��H,@ G�-�*�HD��P�+!>���6��H�R��ę���U}ߴT�!���;Ill,q�`��*�99���������/�p�|=�XD6�{�8��Ղ�
���Тޕ��V_�3���&+>��#	���~w�v;;�\�,�q�nDD=g���8���+��>�ʁ�Y�!5�6oNS�x<����J��n�F�>�+��k_��:��/�H.=�O3S<�<�GGGM++,B�ZГ�����R�F}c�ܙD���<7W���H��2����W��ƹ65eee�N��r���vޜ9����	.��e�$l�|RGLM.��HT��h("���#����Ԕ$����A�Zrsr�wl۶��AT����PA�UI6�|�z�Z��{���5o�ctT4		�[���KZ��K�^�P5�u�6�!g�h��5z����ɺ��6Tę����5���������;�YY�$7'����o�M�TP��vB���̤��W�>�6ԿZV֮m�vg��<.�ej���kVC�w0 ��'�����|_��,���\@��%�7m
Q��5:Dvn�161!�旭[>����ԚD5��V�� _1��Um�d�$X� T�HH�-�-Y�:<,��~D�7th�V�K55T���49ss�ţǎ�>i�|����k�{C��k٪�S�a �L<G��,�����
����_���g͚7��.g瞪�Q�j[��g+���P��F=�2�V'�z��ɓ��6<���D�ߣ�V�ȋ��KYV�H�@U�� 
�Sg�aai��§�_�8�q�P ����۷�w���<�LV?�ٓ��yW׶�pZ�{��1�%���E�"2�`�,..F�d�M[�F8l�g��r�ҥ�?/[�������r�_��Zq3X| 4��*K�t<}�'�bޜ������X��p!l޾�4�o�=�|�z���P5�B!0h�97w֪��jŊ]�EE���Cu���Ƃ���ϝ9�����6oΜK�n���(��n��/���W8�;GGҫO�D}���\�v}����={�'��#l,�Y�0�@	m+���%��]���ɥJ�5n��a����s�����F��9��ѣ-�=}Z��閖� ���hj�6�I�c��w��a~\��2�D�����'�*;�o�r��3�K��!ךP�D�8J����uk֒��8�\S��%G^q���eT:���'��]�L��tɐv5��ث��]VVV��_P�r'��M�|0t蝇��i�oy �a,���{Κ1�0�((���,��x&�*V�L{���+ϥ��B���e��@�F��Acg��W.]�	�f[���qv����m����!D�5m֌x{�%�'tv����a#d��[]H@��T�&'%�>�����}4r�J�T�-ش~�[[[>a�
E�I=t���A�"��J?�a}_�������>�fX�V�\�qݺo�D���z��HU�IAP���E%B�L~/�#3	������ֲ�K.�������C�{�L�1)\�	�5����i�?��ϝ��{�:E-�`�<j8᫯/_�z�X,�[�Ml4"��9���������xbW�^�$�i#8zEZ�e)P᧶�WK�xzyRC�H$�=o��"�TJKKy˖,��yL�k��gt] 5!1�GZ�i��V�<�ӓ'OD<{֝p��Y"�3tE"tx�8�k~6�L��G���s%QL��5�h4VYYY��	�G���d����q%H�V����֙��<�*��5�:<�5���k������D���ۨ$E�2�g�a�v���R�"�E�[�zͮe��\y��I�P\L��0�l��K�H$�m�ΚԮ\m�͡cGi�)�E��b�ys���}��M�b ɂ9TT`��1���=9))p��g~^�b�\.C
� u��u���!(�z�V��o����xa CYV��GC����~����QDQ��n?|7�TffF=k�h�Ƭ<ˆ�u7�]�f��3�M;����9�_�A�p'g���~�IJd%T;�3��Ϳ��=�k]Y�U;��HO�]4~Ц�.gee�c�s�(P�uUٚ�Ý[��\8��0"���a��#�6V��m�Ԅ�����#��AG���
��ZgL�P=�r9��(�!��ͳ�h5���cG<}�9����_�#$N�E˖���z���x}R���=<=��yEEF�ҡSG�3ϯ�6�{�_��y�C~}����۴>�8  ���+��O=�����
}Al�����12"��\.�	�r�����|���[���F�IS�#�.΃	!�콯�`jsC+�H����q�?'O�����N���M���[�.ͽ���%͚7?"�JS�� ����@09�9VO�>�r/<����b�333mssr����.`	���_'J�ܜ�z��z�ʟ�̝5�"X�d�;��ݡC������r!�����YVfV�����v��Ĵ�T{0���,�8y�Ge��:8�A}⦬#�!�ͬN����ٽw���?N��������}�\!F�5S�o�L�z��»	Km�������`�sdB��;��6m�tҷ_?����Ԝ��8��נ�ś �5<4tu`�6YGO_{��a��?���H*,��?{�C|����v2s�y����SRR"�;���ZR'8��Eb	Y�b���|>�����P��._'}�W����;�p�bci)��ÆB�Ba��K���ݐ����i��u��nZ�hq���%L $��bmvV����O�޹�391�..�ѹsg�y�vk"^��6�������"Y�i��S>:�&\p���nn��u�:;;'��ڨ�R�������5��!=/���cdߘ)
Z�5�wը�I�i$�j��n=�����՟����͚1󨧗WW�������D(��Ze�ţ��1&�C�����°�hL��{�X��Q ��;�#G��]��,--wB�ܺy3�9�[�q�������8���#W����ح������8����ͱ���,���fD��+Vg'2@�Pu�=�y>.99��j��pmyU��N˧k��5~eR���Uf���e�k���J�����#�sgϒ�=z\H��ʄ����.�ݣg����/�i��6�II|� �z����8��J�b3g�"�ǎ������+��V>�߱u����}����- 脇q��$�$''� �`���r�g�V�Ƴ��!���
�YuD݀A�Yj�Y>|�%�A�VUx\4���E��5�������U���n
���OG]��ha�.]�>�*\�_`WPP�vI�L&�H��]�Ȋ_cB���.o��C;A�D"���'+0�:u����K�`�1�Ĉ�u�~�y����c+K+�մy3Z{����9��?Dٚ����N-���	�v�jA�:u>oԸ��̞3q�^���>M���IvV&u)8i!�%�����"]��J:v����u��5���,��^�{ӿ7h�P#�ipHy�LOjA4���]�N��83�{x�<==5I	�/���Z�a�~��D�s�C�=�t��eڍ��-�=J�_q��p��ME-���(sH��D�tj(�k�����+�j��6�@˩�fJ��p�����	AjA7���{�{�������]�I>h "�Ľ��oY�Y(Rx�X���O�>_�ܹ��F��݃�M��t�)%T<[Wغ��� D���X��䄼�zio�V����Q��0�U��2��n��-�"���Q�|���m�ˉd��ݮ�e��Z{�i�dc,����<�X��E��v���J��W����3�|� TZ� V$_m�W7�T2n��l�Q�K�?�! D UKw�w_��ALx1��|@�P�� �c�����/��RZ�$�6�h�� KFNz%;7SĔ�������6�3�K�k>� ɒ擼^ ��ڷ0ޔnlf)��h:4�h�H~ϕ�:������ �'�_�a�    IEND�B`�
```

### HappyTravel.Odawara.Api/wwwroot/images/logo/logo.png
```json
�PNG

   IHDR     �   R��    IDATx��	�fGU.����ӧ�t��L�����)AADd��� �u��G����u��'fD@�_���2�	C� !	�;�y>��Z�SUk�ZU�xN���_�|}�a��5�zk�E�ҿ���� ��� "��&Q���S�"~F��*��t�=:�`w=��Ё�s�Z����Z+��Y���L!�4�$#�݉c��!s�9����\��$��2��^���C��)،��K���>�2`^hd��u�X�os@�.�������}�W�I�u�o�L��>�O���X���ԛ���j�'`r���K[8?�h }���	�ό}��Р<��k��Z����@����b1#����[N�q��^,��
��ŋ@X�"����pL�����N�2hY��28��zbR˹�,���o���k���L��Z�JK|�P�F�RΠQ��ɚ�-��9��u��-�ja���a���"4�힉J���BE�:��h�À2�Tw�Ӆ:[�_0��Y�D�&�s��I崐8����M�( @�kx Y�r?Ӡ\�@Yv�f�v�����c[:���:ފ���|�)@P��7y����)(P�}`�4��d�����G I�҃l����qC/{G���ɡl(}��v
����q�C��C=FF�������P(�I��"d H:P@�������	2���nϹ=�D��R�n-Mb� �6��L�&z���{"p� eC�j
����*?��
8��j� 礛S p'�����A�"�F�/lĕ9&qiSȵ͏��_�G�yD���@�3Q�K�H�ɵ��y��j���h��J�C�86y�9�Y撸���,�m��#6N6/S�8Ʋ�$�d 󻔟��G�,Fč-Z�
�aK�A���>�ﹹ�j�0�t����־}��R3��D�U�&vkLi�k��U��d�E��{���&�K�EΑ���ǵ��>w,s)bD6�H���B�[�"���j�w2�Aޒ���fP�׌o}�����5V��YL�B8·�d�;�.�9���Dħ4����n�1+��i7�eδk	Hz�,s!�.��e
aMM���"�P'��^���[�5㴙@��C�
�8�rD�"���j00V�� .���Q��`��"ʌ�� �u����m_��q�=
�* �
�35��2~*^v��ts+�,������
in�2��3D!�3з$���Ŗ(�+��p-�_�̍������֏��AWt(�׉9��Q-n��'�M~���!�8���s"�i�`k�E�0 �9	��ߣ.(�Ϻ>��
D�p�I�Ƣ��]=V���E���H~�2�+�Oh�aw�t�M(�0iE�"��{��;PĘ\�v�y.�n�X����DDO40���lx1	�9^��rf��3"�g�n�8����k�[�^�����M^i������8_�vu4"3�x[�[~w���3r�\�u���M��"�*@Iך���©QYt��2������2o	D��Q��, 9��U�rxY�y�H����$T�8<����@�M ��ׄڰ�$N�UV��Z�Edb#����fg�S�&ʥ��u��s���{p���`�\�LƍH�n!�����c%��.�|��J��i�8PR�u�,�S0�
� ���������v�<��
0$��T�I�7Tuԫ��������C�V]/�Šp
��҇�w"��(����I��Kq}��Aݏ���N�E���g9;�f�(Y"�����[���R�Y�&��^�d������rW`lF�Y��A7��N'e�RQ�׍�ih��G�3��Sq�F�*Q�\e��y��UI1�=�j+�7�M�2��0X��W4<��ҵe���Ƣ܈�N����;�b�a�Nș��I�,N�*u��<��8�3���w�}��9�y��Լ=¤-�����Nq���
P�C	L�!�gCM�:JRO�Ѫ}M5��h���D�9��~ߨ���s�śY�f������E�^9�p;g��S�	5�����S�����@�8�ۓ�Q�:��L�5X0��cB�ZFM:r+]e�
���<�v��؍A�Dp��Ӄ+�٬�A�/��{��K>0b� <��ǆ��zؘ%��]7(�O�S�K���r��Xĭ$������c8�f��DH��+�ɃҶ�|�[��e��y7u'e����i�M
�8��m��Ow�����zFŧ��g�2�b���6J�<L�;[�)�vW��L}\�˜��|y}°�;mXҦ�/yK^�*�Q��x.�u��CU( �������Q�����`T.���ǍWu��m(�¡4b�ߋ�i���ݏA��0(��D�K��\�Q�{S�s̯�{�@���ȋ_�-�@�Հ�Q.�Z�	(��E������؏��M���4�>�Z����:��͘�N,�Ɔ�^ǣ���j��

*N�ilM=M 1��P����%FS9%�P-��Օv��J(JA=�P�8v��h�#Չ�"{����(.��<�W�vS
��uJ�&���HÉ�!�s�K�"wm������@��Pě�\�L�+�A}�H��l�:�I��r�yG��m�	j���b���ש���_wn��z�Q����g����Q�Еe5���.�Ny^B����x��c�E6Gy�E�h ��湈OA�B�&v`b�d�.d�K��=��<GRL�܌�ն��Z�6�1��g`,$���Ԫc\��PR��x�V>3�D�,3��D�dcn�d���:H����r�u���䜇l�&�}�8��@U��q(�~��������;EC��A�����Ul���T&}�H��S�횵�
��;�0W<bL*Q���N��x��߫�3��
�w�%	�@E�Xl4�}H��L�*�(w %p�N�i�bO(v�W�z�k�,�u�p@Jo�,�J��ցw��Ps/�W�ѡ��s8�cv�Q~�C��"��q VJ*��e�6�Fs~<�Z����n�]���e�	˼dvvn�5��T�#XBt�e~ӕt���IF��EM�-��۰��9|�i��TP�	���y��)�-k�)S�oDŻ6�P�2MC/��I:Y��9O%�� >A<>�xr��b�^�h9�O_N�a(������t*��oAW �����ٻ��r�K�:ѥҹ���
@��C�s��HE"��݊��n�7�}��-LAus]ȶ�\���w2bo�+;\Ũ�Af���4����f��+����ۮL�W�C�3�x1�̂�&��hɢ�7��&8��{&��u;8�����n(qY��� vm�{q/n�P�BP�<I���r.��'8%*�]�a^��������O6ޅ�"�����V��5�`�Y�~�<����Y��� �B����5�9�9׉7��Yr��\�h=׉�bC��XR�'�z1��*��P��U�S.%��e��:%o}~���e�=���r�Ġ�;�x˄���x9�P�n���[}j�ě\%p�,Xpx�)��6�����D�PG�"�7c��8Å�©���{�ǲp-N4J:����ӈ>�>咼��e�WP8?��)�O������tY��ou���kۤzź�U��nǵ4
iF�G&��y���E�۸�V&�GY!.����^���{����[ �"d�˕U�_�D�P�d�B(��v�����ʯ�f�>�eQ��"Gh~�!8�����W��N�����{	�F���.���1g�:M�ݵ�]	��f��pkj9��z�V�#�0;�g{=D3��¥-�HB�T�,�(
���yn|+q�����F �G�������%2�v�?d�	�ʴ��쳱`��x��	k�:��%��!��Ή�!�hQ��O,��N,�i��P���p�Z��1P�f5�P�)�5�ܢVQ��S��+4�ۃI9���� @翫ڒ�~X@�ĸ��$��)Ъ&���*Rg�����%�M<'�O��usUP�;=�Z*���S��_M�e	5to"��Z__�զ�Űu�������i�7].N��C���en���q�gl�g8�1��Z�����j��AW\�+oN�I�c�vo8�FRn�
�/sڱ#*]�Vgp"N'w�8���yP|B*��ݙoPE�����_���D�g�ü��G͆����G�s�a�F��B��Kz-FVCWoI���1�S�8T�����̒e�sQ�-��*γl�j�zp�3߷b���W��l�u�w#�ъ���Ԃ����ㄊ�ވ;��?n�Z�(�c���0D�y�JT	N,�{�y���(8.û�K�����ω>CXLM�;m�ٸ�9i��d���e�U�n�Q�hPڠ
JF	��z!&�N�))c�
�*��B��R�\�`hb�z�ZP�@\��jcQ������r���r5%�U�m��Ht�)cs)�B�W�h���8��zg!{�C��\��$��(�1�l��x�ŞՔ	rR��c@���)%"U����E)�#�e�X�� G�ٴ����%�s�X+�[�AIC�7/�+�"0�-�s�N��d�,eI����$1oʙ�:@MH��%0�%rWĂށIk�ӫ��!�%8�o�0sNޤ=�ˬ��<���h���˵y���ns}�'��G?@	��V����X��Q#�Y�X9�M{{W_���GT�����B9��BF;9CZ��Qv����Fea���b����F@Ȕ^F�7��1���R�;	�[V��)�$bj�H�t��C�?�X��a��m�>t�,��o��B�.ǔ�^\���Xkܘ
���x����3�`X�j�5�I-�q�!iB�bk�OG�Ij�>0� �0�pRam�@��n����b��Q��h��	�(*Qɐ�s���Et�l�'F�rB��n�ް#�G-�u�R���WY��~N�*W���p�x���71�Ď�¯N�O�g)�d�7�C\���>K8])�Lf"Ɯ%�`�6{�ܐ��������_BI�]��>��ʥ���aRڗ��e9��n\Y�J�ț�����Z��]�j����g�u�1�*�U�ЪX����#*��w��hXt�h#����d5��1H��X*-�a�m:
�۷�s�F���X<:�"�Q�r���+�G�z�,X�,_��|z-7��Yߢ�O������LA�;�TŬ����/���8��5'M�~���G6����gȩ�IK��.�z��������;�Xac�Ϫ�Sn�6W�\���ʥ�U++a�.W�b(�eo�P��%��s��v3wQRC��������3��`F�sv�,q�#��<��|��Xf�ݢo�T	�%���a���x�vX2N��K�]��9��4��ŉ����{ �@����� .���7��0�L*vywC7��\�v���!T���q�˜~L^y���z.�9BR\(���%Sڶ��/�nT"#~yKihg�����K��H��GwO/�FtRª�zߑ��G�m��ʻ~7�U�V��>�f�Q��htLAE�`3T[(�b�2���XsH=�0k��0���q���q��_l]��]����<��\P�/ᆁ��%� X7���]\��VU����#]�/p����};*"��� �@�s�z��`�p>�5.�o�^=�]�1�IB��[ ����:�� vV��J�N��~E�U
�J�-x��ϸů¬On�6�(�B=
��徭|�Z����J�Y���&7vU�py����T�84�iK��,�v���;P.́�[	_�bY��Vy�:Q��P��U�4̥of�k���r64��ů&�`5�#/�d��T,h)�t��6`b�88S��´�����Nl�})__�_쳓 B�.�4�P�	Q��6��G���3���=��:�knn��<o��ḱ��|�XDC�v�;Φ}�J�
1�-���*�#�٫Ռ;��čG�Tk���8y����?��;�yX�v�;u�����)�ZT鹈A�wz�Fa�uN?B"F�n����b��X\�dz�k�C`{^ok"�8E�W��79��p�Z����"Ңs�uhDkwj�����$Y�:�̉����-k��6�&�F���J��	N<L����r�U�WP9��cH�Ľ!��vW��H��
��ղ
.�.[ھ�� ʮ���\40�8��.Ԍ��-9�3b�z$�&��\6�L"�y��h�� �*�\�y;�s��ڂg/1&.󻚝��ƕ8F���9��@!^�ԏ���'?�N慽�T����_?���5�=.���p�R�d( Yʉ�	�����z)�lm�w�\���y�ǹR����}��@��͆ա�%&�;n�J=�J��l���(�UE�>����<d��f,�6�;��l�0��7��wY��G3�U���,
Ώ��晚�L��k�Vg5���wުN��]��6�k`�@�y�T
~�Vrr��M��57��FEd��w��1�h&��`��K���u���s>]��̸.p�]�n��y@0_�̅��jࢫh����R(����w|�}�]�T۬�v�q���es�lrپ�2n�L�!�J�F�J�^��j34���C�ہK�c��>j'@�+����@���HjR͢��J�&���sb����.��V'%v��½XRw��=��M�hc���b�p�
�+�n��n�#����� ����h�B��:�Z�N\������yf�4�F\��ĳ7��)ekQ�6�7�,�jfַP�D4V!�>|�R6�\�v0�P�)fȽ� 8�,K����g!��~�(bS�q�j=u������n��8�!��3a2��RM��:�q��zFo�2�9�$�;��
��H������<ǆ�D�.� j8}Ʋ���k��~)����r����>Fg/�7K�V��s��L�A�����du2e`	��W �^Ƨ�2�1Z��-���=H[�/;� %�����z_�+`����~7ydl�]�����u��8�����
�nP�aDF��R���Y5cgAr�I��*�ʨ{Wȩ���mLp���~�Z8L�Vv�<;��J�T�jZ��RE�z.�+�{�k��M��r��C�
��YG���V��D9�x�f�7C�m�s�4�3 h�taA�<�(}1/�B#�H�,v:�sX6P�mZD���O
��kd�(� �%�`u)1�8���+�K��>S�������nܾ>r�74���$��e����a����)�R�ݵ��͙��4����2X>��
��jgU�P�Al��g�E��Q�6SF��Y0�\G"�B�XL���j�2��{�F���v�p��wF�EŅ�+�Zs�i>���bݒ����a���9�����b�pQ�b��[���2=�����B�~��r"��5�b����qK�J�:��%�.|:��F�q��885
����+��lX���k��c��(��f�U�Zw
�t�g�Z�Cހ0D�UĞs*�S�i�^]�J�%���x����q+E��31�c�YZ��N��[���Qn�dK+ ��2�7%��'}J�DHi㑃��*6ݎ�~Wtm���N�T�ؘ���t.��ɹ�(Ѿ]Z�A����9�rY�LGA�37���H�����	.9k��rs \�e���x�#��r^�!q3�hV�-��C�� ��͉X�{�u��.^*;��&�+2���p!3*}��1�ln����16Ns

˅�b}�`���Ve%�b�U{E�i�@���E�1"��No�nM����x<��[t>��܍5K�K�œ��AU̪�� $�-2��*�y����aQ��$�am�n4�^4-�>9�2����%�i%���?�X�����Ojt�͚��K]�?o�;ߑ�#s ��[r�K��7��9.��R<�\��э������+Wt �M~���*�u�s����x.EgW&�p:N.�p��fh�i"��Hˮ�������W�O��J�aՠr{K˅iC|Z�B�,�B"�#a�F|��BF����J7n5#�u��&Fr�o'�������mR}&I�	QN��=��y�X���ڊg��o�پ�p6��ҏ���cw�K�    IDAT��BJ����.�N�N��lc���;ν�| �[�r'^�k@��2։���b����]=���WY�\���ξ��Uģ
�RǾ4ja��]`�:?�(�ز���y�|vf��W�!��P��n�.X&��}��`�H��ւo}��c*^�j	��@k5�R�P�����}��y�x�̹���Ǧ@������H��p�5��ri�,�NX��\ל�~f�=z 0�RQ9��O���領�M��j�S.Ã�&eBr�ezg�ǁN[��ч��	ym�D�����n�ݽK��Xbr,�j.꽋,>���܉�!��H�H`eP��r��V�f*˗X�*�Y�e�� �ܗF��RV^7T�p �[�c.�CaGT�_��\�S�z�kB�q�-��Lb�[�^k�\��]����*�]u�؝�n���]ԛ�E,�-4Q1;�!� Ew���~T��@�Ӷ?[9��t�,,��V�m�fY�kQn���nqzS�}�M�n����n��~�y�\�}(�r,r�����;�����;���D�Ō����-�܉�K�DZ��Љ�o.���ܳ<��E*x(:�;����υ��	 �W+ˎ�����^�!�}��S@;���|�8e�)����Hf(|�<��v��
������ު��w�J�Uq����}5}#f�+�S�c����5Oq�T�@�-D����;
;�z��vb*�#����S��_��$�]����G��v�Y�y6�G�,F.��hHyY���
�IX�!��CпK�"�ޫ�U�[���
��N� u��ycl�ԟǍ_+
����]��ֱ�����?��8�OM��'�����D��ca�Lyj2�cѽ�x�uނ���T����{}���#��<��ʪõ8b� ;cZu��
7_�ԋ�r๕�� ��ΟG�e�٧��Ҵ$Nf/�`Gs��#zm�Cy>;q�jr��������)�Ld��ި�8�-e�:������}��)�=Uy~�.j/��Hb����s
ȘU׸�\G�����s?>ٔ�+ ��z�:@��𳜜ܒ��s��
T�� @1�Qչ�U4�e���9��7�P��6��m5����P���7���}Ԩ=o1p��E��v�"�����ծ��YLI��U`��� �i�H5ӆ�uk���&�#t�I�D�,��l:T'��g���A�u���`e�/��Pћ�A>aӔ�
 �T8h�٬K�d{�j�r�)~��I�2�u$�
 ��=��a�J�q����q+�1�Z~�T��}W�}�ϱ�O�O��V�TD}#��2���l�G�;	pT�wa��Y�:��E�������iޝ9��ƍs׌ڼ[���*����g�-8pY��?��ݚR/r�g��Ɛ�8�؟���LW�%��s�S�C3��f(�/9��]����y�+m���
K>��}_��`�gW��a>{�.�D����t��� s����4A���#\�[�����WW�~B�~�h�:�#9���,���_U]dk)�.�<v��D��� o�1�������L[��0���q�3�U��-gvF�ghxA�b����"|����0o�`e6����xi�{�\ǒ�l���L*�ޣ���t6��)��Z{0�!�������	�sE~�:��FcYi�Qi�I�N{*h�#@��� /�����_�{>��4˚���[�Lʛ�`߻��H��.����㣨�8�	��U��xP캵��Om~4h���[ Z ��ढ़��O"��g���s���*�O�t���-ɼV�c	R� xy{����ƻ�Nzh�#���$�~�;,]>�!�wGn�����A�/���u�&2ִ >�1`�Z`n�3�dV&w��bV@n~Jj;�c%#q^k�Ǿ,� t�
M
p&k�.r㈹�8�m2������%�=�f��#Et��+y��u`ӿ� �V'��^���P���}�k�+��5�u��/Y���qK�` �b觧 �NV��:�t
.�<�:םʥn���;s��-h0���6�|��x��z���ƅ��{�6�����Ny,h0����`���o�*�Ώ��#��ծ�R[��ap�'=xDN����y7~
�%�Dw�s�^���o��m\r�f���|��/̒�=�t�s���ۑ������w�7�S�0�/��&�8��i�������f�W<t깠{����4]��p�n�u�����0�����_ ֜>��E��Q��u	���nd"��:�9�s_3�K7 \��̶u�e���gr���)u������fQ�J�W�?��n>�Ф��$������M�T�p������dq4�p���Ca�=������$"���Rf���z�zQ)8���(�eq���� �D����}���	��?�s�17�6�ɱ���5��{d����P�L���n�����w}	t�㧂I�������#�.��vV��#Wt>�=u�-z/��S�$?�;�]�It<��Z7��ٚ�>8�iw�x?��
�ݞ?�Ĳ�^�����u�`�Ŝ�W{�c O�\G8�nۥ<�*t�� &țm�>��%�Y�CD�H�;��7O�u�#A�,�]ݏ���h�ؓjb]|�:p�K@��1~d���;M�a^Y[Tg�/��P<e��F���kqEL��ŏ�̚����B_qE�R����z���5��
ؔ�VE�<'�q�81�������P�v �1#v� �޻c1���JLv�`e�w�c&ї�硻���cv�	��ן�;������Q"���Ȧ���a��+�?��~��������R�W�]�N`ӃV�:��]�����j�N�פ{O~���Э[ճ���t΋Ĳn{�����ُ��&!!������mz��zx�ˀ(IYU����[C���B��s
uB�Mщ�,^���	�yZy�|�f@��h��|X�
����^��J�j�6Ƶ8݌��|d�O�`���Zg�#�b�`�����c
`�f�t����,�}�
t���F!{,L��w[�����c������_Z8��^�oKZ�߃�ޯK�Q)1�LÌ���[o׳��ݶ_�k�WGGZK[O�1���2pВ��@�җ3����ß�3h��w�X�P�=A#��a�>�Μ����d=TV���	.Q���[��Xnř�͡M9������L����q;�r�+]{W��m �A�a���/��)4�{'��f���%ߋ��Y�2�Xr���'��;�w��3�1`p�W�N����ӑ����L�ٿ���D<;x�ҝ�A��<����SJ����-V���&d����G'ױ������.Nf#�9�����ǵ��Ӌ9-��8��q@~�wiD$!�J�R�f���8ҤI��L��>�,[=�;�����*s����,h�� �ڎ�/)��^R�5���!>B��F�)j���^�T}y�KQ�)�fOP�ӟ:�b�1�?*`�~
蜟;�Ͼ�Jw��֟���;��ytg�Q2C�m� ��8�r:姀��K��fD$��>�@׊1������;�8�
8�����DX�LV�V�:.Du^&���e����&�R�Vނ��(J���u�N���xn#�5�8.$���f��n���!�ϵ��EI���\��R� �����d\%�-;����f���3�ls'�_=:4I+(qPw���syw�pж��M��H��_����.�Ù��Go|��u�6M�U�����\o�|��d��5��� h�E�kX؆��_C����<|%�K��<.\ :�G�؆)�N~2��"l?�Dw�a�٠�� GnfNL��>`as��L,�_:v=�po��Y�ES-�P���G��3�}tu��5�Hf���r�F�码�H{�X�!���w����ş��p�dXE.���Ň�a�ҩ��
��Pӽz��?����~��f3��"4{=PrJ��L����~ �tr���g�\9t�+�
쾢R��և�{����M��_�-��;�>���G��u�o<|�e���E:]!��_:�Y36~�%�����~|���s�$q���/%�|�S����Gn�K�7��zs� �ϭ�5t�t���� u��~3p��LV�o�! J8�ʹ�h� �����1݆�ʟq��IN�$���c	�x�6�|	�]Zfg*��F��o���.���%Q	ȹ�&^�Ȩu0j�Q�+���
(��K�I���|�LЉ3�x�PĜ4>H3���&+dQ�P�q�(`n9ݰ��n�ǣ	S]"�̌'ˇ��I��+�趰��g�7�+ϟZ�/Fô·��#����l�_#Vɸ��_�p�O���+��� ���W=��_M�N����^�}�����"|��98lR�� ����aFG��[�_���^��@�E$�y|��a�_y.���3�*���@�����4�ړ�l�.`�PI�}��&�������P-��z+7�ީ�hY7� ��&��2�H^8'{1C>���&s1sA^&��Ed��Uq��5Zo�\&���?Y�u(���ȸ8"?vY)�^|k�F7��{��{�t��K0��k�7.	�����q�8�p>�(b�nl4�\����/�-�3KR�E$�=��L�D���o�����.�D��'�� l�*�s�J����� |�E�u��J�������}��&��1�c}Ǉ���ɕ��6>rf@	_{6xϕ�~�|f 9@k���k������b`�ř6w�U�wv���������t�OLn쮗�����XYt�lخ���.2�GZk�X��g���0N�->�7�'P��jT�e�ۢ|^�HK���� S�.5:��/����ӫ�`°�|�Dg�)��cV�o�w���ˬ��?C���T��Ib�|V�@����-���%̰k����Er��U�jy/8*�&��\w�0Y�})���%qq��É���5p�[S�0��oz��Ej����ϻ�i:����3<5��� ����1R ���<w.o
�c��*����9��W������O~:���l���c�]�}��]��ɵu�PZ<��0��¨k��%�]�9G��P}n���9xŪ.����sbs���� fI�}gQ��bͩ� I�.�V�Q�/�Z�
ʉ�o%��cY7.��� �+;4��EH��p�e�ׁKM}�%=6r9J�<����gp5��L*��H?�ϝra�Q/�>Rc�N�i~qx��&VI:V�ڸ�K))y~�clgh��g�z:��p7��l
��pA�Lef�쉏%q���S��5��Tv���͠-sA[��9�Szr[w�X:t$�|k�q:�=8�Ē���	q0篒GD8�P�T���ZE����>��RE����E(�s�_�JV����(��6�;��#���c�q�[�+��ؘ��?3x�d���J�
��ng5�|gj��$d���^D�Ѐ�N����Da�8K.=�D�i���\3�y��5�cW���I�_���Ёj����I�<r�ERW�N�ɥ@�k8����;U"}'x�Ɵ]U��%�ks3�x��]���&��U�|	�s2���O��l�?[�}�=���,;�Ro���Z�E�$��pE#ބ�^�C�IJLM����Xc�LϢ֙�_��X�"K�T~�P�ĕ,:���O�'4�l�z7��8���Jh����������_ $7E;���i��p�@�`o�#��PM�壑�s*:�Q�ܬ�N�������,X�V-��^4�	A���}�+�q��;��*��i�d��`�m��������=��@��s1���?`b��jt�l�Jrk�2HA,;Nj�g�"��D��__r����]j_;�3mn1�ٌ���u�M�cSv�n���!��@9(QW��\|�W�z1
$�O�rF�}���$v�k `�h�ex��-��>�+�XF7M�b�0"����%�	tƵLnH���d7bش���;�sAs��ݷk�����ל9��3��D��΁��
�߫����'�Vc�K�u~�uk�r��z���x6]qJ�OM�w/"Q��5=�����? ������8�)�D��Z���@D���2TTDE�+�I�hX�no^. ���E�|�A�5��boB(����\�\v��wbs���:�9.z_���ˮg���p�ӊ��G'7����h�ܩ�H�d�%�qqgָ
��fh��]{�����r��@�[��М���d>B��C6NS|r���}�%���gζ�
De��^{h�#'?:#c�#^����K!M��3e�6~/�����^e�zPG�>�B�/��{����W�2�U�5���p&�s�3��|���Ϯ���[e�j�k�w��+���-9ޛU��B�K�L��D�����%�־�q:>5�Xe��+q�t�ƃ�/tobw[����kEMJ�>cY w�ԝU^��Y���Co&W�~xrX}*��d��>�a�,��g��?0��:�g�du�3�:"�
�ſ�梃�iO���}u6�=���6_���3��X
O�y������E߅�l�!d%6�~+��gghЈr�+��n]�?��;wq%��eo��lNa�F�B��WuR��a��h8P�S�ZzA�O|PL��>�=�	��K{�!�����8�1�^�C�����,�﯂Ǿr��~��?J�os��:�>ͫ(a�&T�����r��Lh�dj���fX5�;>�����O;�i���$Kt_�1W8�M�6�{૧FC7;���.x!����І*�� :���.��ɕE����?�	Ͼ�kA'�j���-���=����t��+�#WZ��T�Y��g�Ӗ�ׂw�	4��|��n��-(r^e�Ǒp�[;��Ջ��(�� J&^5/ɽ��(N!/���.@QE�Zm�W�:���r㣈���8�r+�$��ճ��H@B�1/�}�`����!h���^PyӢt�+���������v�Œ�+q(��>6���!� :Wv�#�᐀��tw�#>���Y���/�)���yj�c�ȵ���F����)y�+ɿd��~)���%��N�8=�#v4SF ��{� t�_M�*�yo�񹆬�D�����<7j|_v�8v,�Ig���)M�nh���S�4�\��wj�#'A�֘b�-fb6c��&p�^vf]�i�����|O�Ts?��~D�z���ϑ`�?Uqb��s������Jg�q#��0�|avI�+�1�z��?����$�z+O�:��Yp�m�����M���Y?2�ڹu��|/��7"�|p�[��. m�N����Lɂ�ZV�����5���y�m����(�PLo��A[�:�s���fo����u�@��+@��8�Yp[^X��@�?�顳ճ�ߚ@^����w�t������(x�k˩�rD��lP���)Y��*!{[*�F�j���`�:�u/�~V��Q�(~��ɕ������[=�K�sQp 8�
��
����ӫ��=����kvB�e����ƌ�bт��6�'��M�kdW+��k	%O�*J�r�f�F5E}��ˀ#������������U�W��j.W�<_��5FH�M��&{��h����֧��ٷ���#��7��E���w�뀳X3=9��W��7����ʙ2�B�J��xs�sN}ʪ-.WO��+<W�O	M�po�Q J�\��ˁwÖ�G6T��\v��i�Z�D4��	^��� ��x)��w��_�T��V���o�ɵF�6><��V� ���N��#
<B�1����H+������~<�^������pt�����U�3�/.;�齺JL�÷�w�r����'�D:H����.�M�QL�Bp>%@Y�K&&;���"I0���n�l]6'4��tX.�����`��e��у��"�\+Zى2v���@�ԇ� ÁP�5n-�G���j�B�F^+�\�@�g�D���I��{�Vù%��p)�ԙg/�8k��W=Cvm�@��?N�nж\�H���_��������_��}�`$_�e�}WIL���Q��z7������'��:a�`    IDAT͸�� �y{�TfU �~��y�#O��L�Q�����ڲЋ�0���Y�����5�8���y�Y�	e-�ۃ�іH�c���Y�̨Xm4ƈ�E��l%=����}������eu.��K9�'�����3�}:.vPR�eo����(`�E���?ӊ]G��%��6Q����՛8Z-3������9e�,�y��ޏ!|���a�ĕp��o|u��k�O��`9b���;^����V�ƄOv�B��"��0��U�''�AiG�l�dQ�D�Q��@1�� �s8Tl��2��w�j�`�Hg��5}��� � W�~�eU�b��@�GPUg~�z�����Px�����6�y�;,6�P����$�0�'��02� q/J��\�9�S��v9��G�?�k���߁�&��I[�	�5ጢ)�w���♧�I���+��T��DN�|QΡR�����I@�;l��e /�+K���J:�Le�$����Rv񔡌Dw�f�~��r6$F,�C�_�h�%��:�&h��b��c)x/�K�M�C����,������A�f�YjD�^�=ɝ�ǒX�����r�;3��y���ҲBI�.��3O����(�E���V��T&�l�刉����'�"R�V���:���u�&l ��M�7���OoK[��/A�����'_ǎ��T'!|�'���|�ެei/�U?�p����i��`J9v�g�qe�%>p�����ߗs�D�D�uqL6cq��,L�����|p0��S�(D�c�Έ��x�MT��r�x��)`���R���c1{& Z`�(_]��Rto����cXct0�	���hF���e�+��ΛB�#��=3�>��9���/񁎦?BmZ��w%x���x���R؁�q��#�C�x�RG�5(�!fΉ�H��α4�'ᣠ��/Жx��~xID�����U�����K1�t_����xJNv�;�-�I�^�w:9�[ǀ�Q>��ʗ��=輟���ٿd0"A��k�;���/�[���u��?BL�P=;f#:�t4%�&��h�,�����^tw{��Gs�ʮ� ��/��F���Gf�� Ev^���./��ү���o@��&p�����}�Z���?�;^�qY��z��>OP oh2K��ޢ���?�LԓJ����C$�Fcs���l9���K�Heq�:sej:�E�ߊ�.�\8�H�Q��^��U�Jh/��n��
C�Г��l�v.B-��:`����U��<�w������h��K�˩2�4��7�e�U�A��
�*�N}����D�3a)��N����DU�yE�������糏{�rӮ�rT�~��Z5krڌ�+^��к��N�bY8zMv;,z�u҆��N��ε�����=�+���L����	�I����n<CN�㲴�)o��_k�8�,��Bw�	�|���c/����1&_�0�mw���/�����C��6�l�~P�Gq�~om{����� |鱒lʙo%u)+�#� �8���I�@*G��)�r��EH�,�����&L�k��Z��_&�{�J.����`��Z����O}I��>+��5���>.H@����9�|3��8澛@9c���rB3m��B�Zf~.1=�t��ĝX:�
.і��UAs�(��m���� �$��w��S)��$�e�?��kd��y:��o{�?TZ�#���Ur[ ЂS{�T��r�}ԣ�!��&ǫ��M��7"%Y�#˛T�_��S���_F_.��:��R���wel˯�Ħ�'�Ij��Wf��a:0.Yu�L����%��p�Y鸜����<ɪ�����:�wP:�j�Q�%ɘ��,V<�:m���m��[�qt�]�x�̕�a��
`��[8�İil�O��G1�%���۫�/�ش�x'����ȳN��ҝ�����Y1���~����$tV<�!�-�s�HM�`'��6&o����N��U|3�v*�H������^��J/�]�'���7.^?6c�i�9q?��gOl8���93�D��e��P8]�qK(4�}U<w u�$Ǻ���Tv
�;Q��'k���,�(��F�޲�xˋJp��si�R
�|d�F���#���h�j
����5@xyʤ;-��l|��y'���B���ٽ��p�(�+�)� 9Ґ�6�=���� ��h�[l��5���(����w���v���N�֛^nn߳�QBBQ>P!��\&���ￓ�ҁ�����g�Μr��-������N�4��ƏM�XB��[I�h�������˥&U���!Rt��U9n7�sfGI�� &�S�ģ��<�b�r/m�L�ƪ��V����gjU)�O�e��_���$!�\�����;�F��"pI�ʪc�b���4����m�>'�@��{p��<�;6��m]8�x��E���I8=��?be��#ޕ�C�4��K�jc[8��#9�1��A[� �]��gA/�K֝����?�z<p�����|źi�)��t1���-r��*V�(\�K!�>��к@ĝ��˄oozL�̿�p��^����.��2zP�ue��p�di����sUL�G����٪z<�?���)y`���Ru��'�����I�Fx��  WY��T#5���̖�[Σ�z������ɗ�R�Ɣ�f&���g�)�g���c���W *�Am���>��)Ϸ�7����̟�|��7��L�	��*E�h��^��(�z��j��S��#5���1�[��(kmS���a  ?����V�5����A�^
g�M�ZE������W�����ӡWw|���Fv^�@O�\�p�b�({v( �ޛUxǺ8Δ�h�AA��|K���T
p�D	5jV�Y"|JŦ�ɏ��䇤�hG�I�&��v�:`N3�u��g�������5f����sD��ds1���N�[�jwC��[�����m/��%`1�������ۅ���n�V�)�� �`����E�a�RtS�壠5w�Os��Th0�o��������,V��>�Ms_�8+���̚C򴾊:��K�qn���Y�J��|��f[�������V~&��r�Cž����ߐU]q��$Ϗ�8�N&��,^�5�A�˚��J2A!��㬁��v����MԲ,�B�:�a��S�{��K�${? �+���5)mo6?�dYK��{���0F\1	Xҳ���K��g�`���z�V�����9ϛ���)`�Ge��,N�q��8�-�B6�v�,JLV�%�a�h��~�'��8b����ŧ����hTbgDԡ.%e7�t*ݐ�T���Rpx�x�㕝���I�J�$����a?2�f~{���@Yy,ϸO��)o�����z���J��͡�o������L�)�U'�ʩL��(���ս\؂8O�M���s�5g G�/��(ɣXw�3|n�)@���=�����jV5��yZ�yŔ{ޅ��gs����2�����g&�U�T��0���T���`3�M�Tnx9(:�n�,���վ92����5]�c{��c�E�-����o��LH!�+��O��P_K�C$�]ƫX������ם8@P1K�5���YX�=GV闒>�o �2����<e������Uf�Ê�<rpu�C�%��=kJW��xT��X��8��+��"T)^\A�\�Mr��T�U��)c.:}��5w|�郖p,�nω�Wf�^�
�E��Ү2��O\Ln������n6��\Rˍ��4܀u���� ���ek�a�ǣH~>���YG���t�|�h��:������Z�ǔ�p4{![�n/�WG,��������ؙ����@�8e��3Z���� ���]�� �����Օ(�Q��j �FO��`��&�
K���{�6��t����_�p.�t#�E_�Uh\v;����nҕ�1$���×�=��4I�V����˒ԕ�+�ߊ��6.̼� �k8�ɔ*�)��mG��1z��#���><�ݦ~hNL$��eɍC�4���vU���%՝jq�~ ��s�ތ�Pv(�z![
F,�cdG��F](`Nn$�U4Ϻ1{��I��a��xl�mS:��r"��� MR�@0,��y<�՜ѩ���XP��^t�SF�\)��Y]l�20���s*�M�T�c�3�����3��󼕵%�a0k'��$�e�Y�^G��bR^���l�w���A-;j2��EY��:D?4(��`�z.��7� @E!ݻ���Fr:2���g���]lK��ֶ�S�t��L|��x �;��[��q��B�PJ��8�3-Q�h�\ ,W��EL��Z�y}=�ɔ^����H@�����oA��@Å�wl�����Is��������)c�<a�-�\%��Q/���z�"3*X��?�⮾ �DG�80����y.���?��t.h�����>z5���y��{��gQ4�1%�;�.$�ґb][��3�h��;u�;�s�b�`F�J���"��	u�:I!@�>l��Gl_)s��� a�n� b�d�)�/�3)��X�i��:yJYĦ�^����:
s�w�*�+��w;�0$�Ns�X��>�������@���_�^}�g�sr���A�ؾ�x�"��i��T�
�*�T;�J�VB��d��]�ț�g�Z�/�FY)1
�nOm{��M�3گ9����a`�>G4��<0߁.�%`�}��Hׁ�y������?�����z.(�����ɋҢ�A#�ŵqH�#N{p�Ss��5g��ꨟ�yU~|�ہ�����3��B�)?�#5F Z���&`q1)&�-��z�JQfZ���f��4܀N�y�u�s���l��(��{?:x9h��I?qDX���N}�����{��R�����c7�|��q�w���� �'C�&�"[�lyf�9�ueQ��}0�_�.�W�pF�&H�5ӕ��dܦo�!�n� �ð��h�����ǿ�%�B�����i֨�/�����ЁL����9�l1.��$�����L��iW�K�r�pkA�+`�Y����f�=�g��v+�C���0��z�-�7����g=�%���U׬�G>�3�Z���n��й��t��v�?/'�>��=A����;�������)c��WlG��?0x��&&�_=-�oX߈���(s^�y��w�X���^��]ٱ>3[������ѐ�Ճ���Y�_������ �J��΍m�P����x�8u,���e�S�n�;��F� ��#�s��{�n��}��Qgt��)����̎3�ە`i�r��.UQC��XQIǟ��BH�Cv�fL��5rS�-Yv5G���f���"�)������`�T49�� ���#>2&���ʹ��#�sr�����t�;ǃI[N~�K?��gI>�	eͶ,~ى���W�׫E�:QB�3��"�v�/M~f<�G�������oC��Og���e���{��G}����VGZ�&�P8� ֜A%N���@��7����	����aa�#�	�TQ���h�s������x��/��y�!��J�h��ΫHN�G��Y�ώo:a6p�9�ɘ�dl�eJ�S���ك����L!���A�zn�*۽��S���ls��+�|&}��ᗃN{ªG�y9p��M�H��7L�k�=A����aJ�):�v�S�M����o���I�#��m{ڊ�V����]�~��?�I[#.Q�&��C�Pgj��]v8ɤ��09��D���*ĖK�@凢:�����2��"�� �J���rGp(�Hf�qE-ؤѐAC�]��.���PieI�G�ٝd�ʩ�2OޞB�Vp���m��CvJ�&��}��)n�K��-S/K�P���瓓�;�+=le���p�s&?lǿ{?��sù��X?�!n5���U�s�	>�*�����.�si#�mv���s�Y8b�i�SQ Q����{qg���ے���_?.�8C1@YYJڕ� |��Cl��_�9W���8�X�:��Gs*z=`Ѣ�Cʽ0sk�]u�y1����-L��,Ȕ.z�����N�I7�lr��z&h�}AǊ�_K���/m��M�/����~�8���񢗂6����q�Q���K�U��r��E%��aGЖ7��)�W��+�)@b֌_=a��p(�,x�^/ ����K>�N�L�T���oT��|�����sEl(ޒ�.��R�%���d�Yu�����cE��z�s@۞t|ꝡ����o�nBc���D�r�Ŵ�>k���^��YM{��_�4�������p:��K���$���X��a�D���"��++�����v��b��Q���Q���X�b>^����)P�W��c ���⏦���zN�8��t-��x�bM3��PԒ�G��ox'p� ��)���Y����N��.���+�!����8r3�Ƙ��f��5�.������W�c��_���X^y�4���yx|�+&_{�6���[t��~#�Nm���m��t;��:�L����Q��������W��9��� >墜X~���8�f$bz(I��c���k�ѣ�d��D�c���Yx�8qo�3��;�`�X������¬uq��'��+�D�b��j�dO��a����AN~�	�?|�W�C@w*�G�}|{��F��c�1�A�l�ᬃXA	��3��^L�!;�u��%`��S�l��~
�k��~�r�����]���z��K��[��������_8����׼4�pP_�����t��A����Ey��$���f]�|�_�����\�|%��W�.�0G1���B�������B\Tyj�5�9M)�p����y�0	��a�"�@����0�s�[	��rG)e]I\�e`~U�n'�dѲ/�fS�R͆����>-AH�!;��f$���ct�Z> ��)�Z�3:""������F����7R����i��Ry��O���G��w�~&��Z�C������s?��_��=�zfQ��p�&��wN�gۏX��	��g���6o��& &'���\�W�Ͻ��v*�L���+���:h���X�ɥp�B栬iʢ'O;p4�H�L*%N���yO0oW:_gb�����W.�C��7R�;@ ��7�dW�z�����K~�?�SaS�/�ʥ�MNd���q�u�Ž%.�nJ2��F:�DzF�l,��������GR�Ŗz*��GL�c�'�xv^l^?$��3�q�[����]��)��&�q��&��	Y�N}4�䇁w:�{ۓ���WfP�� �K&_��c��.֞�r�h{;�iہ�k�Y�#���b���\9��Zإ\}4�c��c4k�ꏖ��]Wa�(ek�a��_��+澎Sq��w8���� >�nC�'�a�*f)g�/�7�&��Δb�d��JW5�Y��Tɖ�b���>����2���-oMr}�&�%a�`�t/�O�XG�����\��dfpJ�ȭ\�:���Ld�t*.ޘ����p�������y�el<$mקr�QM�(�����:���&+J.�n��TH�X������ٍJ�#8p�|ҟ�`�u�!.�r%��� �mGuǖ;A������h�U���-�͉7���f��y�F��s��E�J	�!�W�U�*����n��Ŝ-��ٱ�wΟ\��a���-"��j��)&�NL�t۔s��)�#����i�ٳ�f��̩Ni��W&s�&�GߚZh:iϟ-�p��dL���zƂr"���]4�/�9��>�=]ӹuМ�D&>���gqм�_Z�l@�D%ˡ���΍�r**�8�X���½���z�Gܻ]���w�;Q���rz"J�c�D���đo��]k�ˌ��+*Kn�2�Jn߱d{��u��	&��Im{h˓&r����ˊ�zG��Ux���%�:[*�Di�I�d�occm\�	�Э8���- ��0lS�]Pby�?Z��+��>��?��{Qf�Xݩ]�F�R��X��c�yVJ
�Cvqg
eɳ\^ܱu���һW�i�L��P�Ή	���M����o�.���U�:sbd  �IDATx;x�(|��;����34�IZ�Տ��{�`8���N[lb���6#<��4�1�(HyS�o�z3B�֝�ʹ�!'�L�� A��ZV�|vd�k+S��
���+����`t��=��6	���<�yk�L`p��_3f�%���0"q\?���1_�M���Ǖ�6N��Y>��U�h��:�s�q(�1��M�Uz3fl�x�(�0��6An���"��|�as��9�:}����q�'�8Ĝ��at�8�.�O�B.�r9r��%։^���=g%���C�%�,ٽ�X3��MG�(���٬^��L��T���oy���>�����w�����S&K��r9Q9������5��;���Z��ZI@-m[�y�M��������.{�p���o�7wZIn�̓���oyx.C���F5����}H����{�Yo���=C�w��@:�.S�ǶɅ�I�=�¡���-�&���Ʋ���+죷�H��2,���I��=>8z|�cW�h�W��rY�釯���l�o���Ηa��H�`�j}�qJ<��^ژL��#�Z@��Ŵ.��CWM������!º��|d�֯��w1�i�KڜhB��`y����<�4�m�Ы���@�׷�R�g�ߥ�]I����u^S ��~���,R�r�7��k�V}D�R�l����1Ҳ���U��̝�a������y�����~<x�~�Y�y����@����a�3���{��CEx����i.������p�c�{|_����t�`'����">�RtO>�����{��mZ��G+���h���m�b��*���H�p$/�_3�d՝T�v O��씪>�Ǚ�R	N��&΢��o�ѝ��(F�R�1i�)~��!��}gi�>`��cT�K/����~IqT^����{dLp��3B�Tpt�̜�_���%:�����Q�������]�:ж{�t��-o٘��w�T��{��)eݽ^��ׂ���PεR��2Ʃy����I`���h�S���ڦeK%��$K�K��*���y ��S�m���'�u%ԓk��^�〜�Y�,KOnVg֡|��J����������M�h0��a���O�9O 6=t҃Aw{
��|��ɹ3tę�-߮��Ʉ���8s9|38���q���gq�o~�����Q_@���m<D�<Dg�{��}���� �nE��w`���
�|w9'|z��΍�!]GU��ٜ���}�8�%Kw�$�z/�6�\���
/-w=J]�v�� }<�K���QQ\�k�0�L�$�<���^�D�\����N(�3�JvA_Lfw��Qzݫ@�:����U��?�^��xUTp��������_���g�Z��-��ج�Ns�.x>hӽ�W2�t�?��Ǡ�;�c���lYL>�g>)e��'?d�=���ݧ�vu�߼�eŭ�՗��2�7�︼�,wAeh�/����G�+х���-��?@�W4׊���L7����+���&I��̷U\WsLB|E8ݷ��?ZACngaH��D�Z{�]Qqӛg{NTn��:�~u�g�H'V��K��b��b=�[.�ǹNm��w�(��s�3��F��Nܮ�{�	���2�NOU %}�)0ݎ<�w\�+�<����LMΚJ�J��
8������\vU�g����jC�64Ph�T�TĨA�J���54�����#��&%~�b�����A�P5��N����:��@g,t��v���N��e�9{���}�}��0����g���s���}�~���ς�!sV�q��_27M� �5�t0�u9�L5v�@��s7�?�V�}��+�SV�{5=GN�? Ym��û��vp��g���d,_Rf}x�&�V�>�)��_@���x���m mE	PCa�L���SA��\������鲀vE�sC�
�d�$��4ۆ�M���
{[I����^�ǸS7UP�:a�h��`M=%�
QZ�&O���+�_�͵.��!a��	w���!���8x���������'*�ӆ���3�C��=�,���A��lI �� 2�6�m��:�j�lm�I%����&��G_�`k�V�+����q�"��,�s���C˔M��ԾZ�ۊ�9��r4Q�U+z���!S�Q��=^<�O�`O�x������s��u�K,jFHp�Z�7Ytt/���dY���,�3�^�3�Ae���?#����H��0������b���~��RJ�EKD����7�OZMd鲎���3�n���V���8�zґ�8j�
 2��3���⦬�|l�;{�+��'j��i=O>��2p�k}u����2���_�T�(N<�t�wC��x�P;�U��߃��-�K���t���s�+��R���Ѓ�W!����ye�H�"�ʎ�ž�G���5B�#�}����Wbq�{�6�|�����8��$��� ����t�S�W��V4��%Y6�q4�n�G�o��?EZ^�����<�5���u�K�����Z��V�L�4��p՛�K�^4o���G��X����Yq�<n��!`׵�t������ȏ���+�ߕH�W��X.�+���]χ���+�
\�*����#�d���Cz�O��G����o ��I��Ѳp��a��������N.�J�E����^=����T� ��>t˰ԡ[���,w�W��{�k�+r���SJ����A�>�A���BO\�7#����.�X��;��r%z�#�/�\��F_�{-��ShhcH��2�ɻ��Nj��9*罭F����z��%/Ҹ�rJ�̀Eʠ�
 ��=ءe=�i�5)���[�h�^f� �6�f�:�K���8���n��@_�=9	T0�O�RNx�.��/+���
��'_{d8���+?ᆲ�����:Zw����(���4�u!���6l*��$*���.�%Z/��E�$g�x )�=YR�/���&�4ɉ�/�\+�qIq�Bcn�D�b�Qr�4�����k8�� ��/f@9{�PLv�U��%������.~�� 667�1�iY��ӄ C����4ޮЈ��K�F�C�^Ka��z���n`�$���4jW�X��n�e�0���h���Ȉ��o�*N�e�r�
?�f�F��T����E{*�f��>K�J��?ெy�ʹ�ض�h����E�V�h����9]��{��%g��V1s�v�])�Q���
8����TVL̾�KJ��J;8�5
|-R4��l�(��h���O�BnUp-@Ьo�A���尗2�/�j$)"�#����z<6m�����9��y=�O����m��4��dɓ�n�����֓m(E�]*̿���o*�Y�̞������(e�Rӵ�ѿ0!i��`�������i]�����I�ʀ���K�`"(���5�[43iG�$m��g����@I�r� ֶ������4;�3�\�f��I>�G��W���� �w�j�@���8.�|�s����ͫ�o!�N����� |4�c��;)���h)���9+��`%+�x95�N}h��Tm�Rڭ���}�ٵ#�>L�U�P�f~�Ad�-�ٌ��A� �ܦC��zٟGy��`�4�]���k���׳cj��~��U�4*T���`'�e�a{ �m�$��+[1l�J����o
<���h��,��<%�G^��	N)�&0-eJ�=�BiG��͒�h��?�Y8<Q�(�k�9Z��RG�7�H&$N|�K:�3������f���~e���S)Ԟ�����3N��j�H_�S���4T��6�{��������E3�V`�m����9UTn����T���D��M)��Ә��3U=�'L����ƴoW�S9�sQ�\V�}u��#u 6���얘�[)�4 +{��Z�
�ș��-?aY���ט<\R�9֮-�V��su%��	��H<�[����5#�iAAA��x�4D��-��֖�
((��;{�<�[-ak�q��e�_�Ri��s��Hhx2��F9J�C'�!_�Hu�Lh�aQ�մ�-����.4�2]YOK{�Pˈ<Er��/�0)��(�?��:��q�NajG���ڀ`Gk�R8,|���X{����w�ӷ�������$��|e�W�ĳ��0��
=d��h�}xށ��i���J�#�G��̞�m�R[�Q�A$��'�����3y���j�8Bo�W2� hR(!x�ϡ&/��&]�Q	<��Y*H8���`w����T�ԏkY{Fеę�JЬ�����@N!��MZ�6� %K���B�?`\ڿ�e�JV���]�9��EH*BMMMi,�A$X5J
��;Zu�n��<�Y�.��|�}���i�n#�R�+���� � SP���Ø�]�x��n�sm�{���j �^X���fU��p�ỡg:�H_rW�o�)�����k:D�>R�l�\�j��Ԍ�P'�&5��B�Dr���b�Z��3}��(mې3����>���^Ӑ�@�_���K{��V�/�A�m[�AJ�$(�{��u�-ߕiCL5>���]�#�}��Y�USR��'�{@�z3�����@�G'Oa�N�j)���~zJ�]���Y�Z���%��G��:uX�Y�2������h��Q�iop�
Y��6�y�.�RD� ��F<�^�`���5ZM��Q)�L_R�8����o״r����
�V�1:�ؤ
6�ӱ�$��e@o�~�)�����S}�!C���d��DÑ��T
IS~D�j� ��=�k��qb�V�:�k@��(�%=ԕhO0�G:yG$t
mS������n��5iY���w���@�PH�Bxyj������~�����Ў_�^ϑCb)\>��M�3}��=P��n9K��9/;QCaɎ�L��9e�����~�T����g11V.w%T���Y=�`W�m�{*]����k+�@ۃ�	|�Y��&����P��L���V���v����nf�p�*]���>� ?7ᇻ��8��h��<�递_�SH������ }9(�SY��.g�ZĖ�7���M65Z3�}�y�y8��1Q�d�	�VC� �AA)��s��}.㾠�����が�4��4���~8���u��b�G�	�8�PL㓚��f�+���2n�e��<S�V~W�d����34��I��I�k^\k
0���j-�*��6f��f�ܮ�p��~�V�I=��]��{��5��Mi����5�lٮ�z�4'�!�kj�$T]C���\�+��l���8��L��9��*ߥ��1TA��P&%W��Л�j���0�C]��W|	Z��S�� l�@^�hDPHm*������)�	X��PI��+������	ph�Ə5�a0�h�a��U��Wi/�O��o������� �@���ו�Q��̀2-�1���B��Z�	:8�q,
a���S��F�j.aPO��ѱ�-Z�&K��CS�������B���U �cb��-e��������|N)�i��dA��]�Ӹ�e���GK�x�?����e`����mq�}(���������)�^=v�q�Ä���I쫒�f�(��D�
g�AM���L������#�k�o��pK���O�&�+�+էA��ZSp��_(����F�X������M�c��>V�Q����� +ȁg�e=��k ����1��4 cBAA��:;�3W��ݺn�j��f�h ����i��- �KM��g�S���A����5Cq��m�Lj�?��I��6���OJ�U#�-��R�gg YOf��Iΰ������^�AZp�@ᦐ筰�uPه]]05��#�6�[k���֏B�����ԘW�8I����	��q��
�}�ž��Q��/�z-����#����-��{��@��ѳ!3�<s�C�f�~'T������r�Nd�'i� �yh.�Z^� �^Sͩ��w����u����/��%���&���ݙ<q��Q�.�TQ�Z	���:�ݡ��>�8hTS��XN�,�@�RhO)��4ߏ-3�|]"�t����5�R���Ĭ�h�vQ巬O-�~Q* ���<�@��h�`4ӥ�0�B�- �H���Tx3n\guH�Ҧ��L��)�&`�o"iO�GPy� 7 x�}ݶ��>�3#��7�N��K�)��˔�UzJ3]���UW3R��P�u- Z j���F4�q��B\��A�2��`2�eG��%U3zz��p��ϊ����ʆ~(����r�2ʙ��]��^@~�����6a�B�]�{~G	�����N_����xp�u�����I��q� �qVhjXӚ��#]�����~�	�-m�ȏ�a#��A�?��3|�3&3��y���G��[}���3g����T�i������L�i�i��VL������A"�׬���)(m��6��YU��֤DO�8@��u҇ � ����t4h�3!3��}�D�2�J�W{��W�J�&S$B��M����v�,���)[��(�֤/)I^��%�IuK��܏}v,! ����)6�uT
=�å�g�B���m��ٓ� n��V����Es���[��Y�\&L��1�
k;2���<i��%�-��a�`���k)=C8��/h+��Z������Vv��ْP��D�z�7
�R o�M��)ՇW���+���5Z���Y�1�L���o�6�tN�62���T�F_�Â6~��ؽwK�c��p'��,�#r��٢y�d��^2����R^#��a�^����l�Hc�ߞC���2�؀�Jԫ��Z�z4i�ɼ�kwm�k2��~�N׾d��!�O�*���3E�fy�d�sK���y��B����e��ś�m0�F��	�.�p7j�q�ʄ���Kg�j�RR��~��2����>�ޥ�;Dq7�;7�Yf@9�����y����e]�K
��������e����<�P�Yٖ8Q�h�,(��a�DZ�N�"k�S��o���^���|�圔P���/������ߡ�y$/Wȵ2��.%��di��.���$RC�aA�X$��'^��L�s���\�Z�`?D���G�hf�*2��->��Ctw���.�^#*X�R��^%��(�+Tqy��K1ZU7�L<�l_J���9Urq�@p��� ��L
._ �`��Jf@ٞ�)	�͛v%弗@�M�\����7��e<_�op� ��F������T(�ߜq�;B3����a�ǁt�c�>�\=
�Q�*��2�,�̲� � �?�$=8�    IEND�B`�
```

## matsumoto

### src/core/api.js
```json
import settings from "settings";
import fetch from "./misc/fetch";
import getLocale from "./misc/get-locale";

const v1 = settings.edo(getLocale()),
      osaka = settings.osaka,

API_METHODS = {

    BASE_REGIONS          : v1 + "/locations/regions",
    BASE_CURRENCIES       : v1 + "/payments/currencies",

    COUNTRIES_PREDICTION  : v1 + "/locations/countries",
    LOCATION_PREDICTION   : osaka + "/predictions",
    EDO_LOCATION_PREDICTION : v1 + "/locations/predictions",

    CARDS_SAVED           : v1 + "/cards",
    CARDS_SETTINGS        : v1 + "/cards/settings",
    CARDS_SIGN            : v1 + "/cards/signatures",
    PAYMENTS_CARD_NEW     : v1 + "/payments/bookings/card/new",
    PAYMENTS_CARD_SAVED   : v1 + "/payments/bookings/card/saved",
    PAYMENTS_CALLBACK     : v1 + "/payments/callback",
    BOOK_BY_ACCOUNT       : v1 + "/accommodations/bookings/book-by-account",
    CARDS_REMOVE          : cardId =>
                            v1 + `/cards/${cardId}`,

    ACCOUNT_BALANCE       : currencyCode =>
                            v1 + `/payments/accounts/balance/${currencyCode}`,

    AGENT                 : v1 + "/agent",
    AGENT_REGISTER        : v1 + "/agent/register",
    AGENCY_REGISTER       : v1 + "/agency/register",
    AGENT_REGISTER_MASTER : v1 + "/agent/register-master",
    AGENT_PROPERTIES      : v1 + "/agent/properties",
    INVITATION_DATA       : invitationCode =>
                            v1 + `/invitations/${invitationCode}`,
    AGENT_INVITE_SEND     : v1 + "/agent/invitations/send",
    AGENT_INVITE_GENERATE : v1 + "/agent/invitations/generate",
    AGENT_INVITE_RESEND   : invitationId =>
                            v1 + `/agent/invitations/${invitationId}/resend`,
    AGENT_INVITE_DISABLE  : invitationId =>
                            v1 + `/agent/invitations/${invitationId}/disable`,
    AGENT_INVITATIONS     : v1 + "/agent/invitations",
    AGENT_ACCEPTED_INVITES: v1 + "/agent/invitations/accepted",
    AGENCY_INVITATIONS    : v1 + "/agency/invitations",
    AGENCY_ACCEPTED_INVITES: v1 + "/agency/invitations/accepted", // todo: some misunderstandings possible. This methods should be renamed in API
    AGENCY_ACCOUNTS       : v1 + `/agency-accounts`,

    CHILD_AGENCY          : agencyId =>
                            v1 + `/agency/child-agencies/${agencyId}`,
    CHILD_AGENCIES        : v1 + "/agency/child-agencies",
    CHILD_AGENCY_INVITE_SEND: v1 + "/agency/invitations/send",
    CHILD_AGENCY_INVITE_GENERATE: v1 + "/agency/invitations/generate",
    CHILD_AGENCY_MARKUPS  : agencyId =>
                            v1 + `/agency/child-agencies/${agencyId}/markups`,
    CHILD_AGENCY_MARKUP   : (agencyId, policyId) =>
                            v1 + `/agency/child-agencies/${agencyId}/markups/${policyId}`,
    CHILD_AGENCY_ACTIVATE : agencyId =>
                            v1 + `/agency/child-agencies/${agencyId}/activate`,
    CHILD_AGENCY_DEACTIVATE: agencyId =>
                            v1 + `/agency/child-agencies/${agencyId}/deactivate`,
    CHILD_AGENCY_TRANSFER_ACCOUNT_FUNDS: (payerAccountId, recipientAccountId) =>
                            v1 + `/agency-accounts/${payerAccountId}/transfer/${recipientAccountId}`,


    ACCOMMODATION_BOOKING : v1 + "/accommodations/bookings",
    A_BOOKING_FINALIZE    : referenceCode =>
                            v1 + `/accommodations/bookings/${referenceCode}/finalize`,

    BOOKING_LIST          : v1 + "/accommodations/bookings/agent",
    BOOKING_CANCEL        : bookingId =>
                            v1 + `/accommodations/bookings/${bookingId}/cancel`,
    BOOKING_PENALTY       : bookingId =>
                            v1 + `/accommodations/bookings/${bookingId}/cancellation-penalty`,
    BOOKING_STATUS        : bookingId =>
                            v1 + `/accommodations/bookings/${bookingId}/refresh-status`,
    BOOKING_INVOICE       : bookingId =>
                            v1 + `/accommodations/supporting-documentation/${bookingId}/invoice`,
    BOOKING_INVOICE_SEND  : bookingId =>
                            v1 + `/accommodations/supporting-documentation/${bookingId}/invoice/send`,
    BOOKING_VOUCHER       : bookingId =>
                            v1 + `/accommodations/supporting-documentation/${bookingId}/voucher`,
    BOOKING_VOUCHER_SEND  : bookingId =>
                            v1 + `/accommodations/supporting-documentation/${bookingId}/voucher/send`,
    BOOKING_GET_BY_ID     : bookingId =>
                            v1 + `/accommodations/bookings/${bookingId}`,
    BOOKING_GET_BY_CODE   : referenceCode =>
                            v1 + `/accommodations/bookings/refcode/${referenceCode}`,
    BOOKING_PAY_WITH_CARD : referenceCode =>
                            v1 + `/accommodations/bookings/refcode/${referenceCode}/pay-with-credit-card`,

    AGENCY_BOOKINGS_LIST  : v1 + "/accommodations/bookings/agency",

    ACCOMMODATION_DETAILS : (searchId, resultId) =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/results/${resultId}/accommodation `,
    A_SEARCH_ONE_CREATE   : v1 + "/accommodations/availabilities/searches",
    A_SEARCH_ONE_CHECK    : searchId =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/state`,
    A_SEARCH_ONE_RESULT   : searchId =>
                            v1 + `/accommodations/availabilities/searches/${searchId}`,
    A_SEARCH_TWO_CHECK    : (searchId, resultId) =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/results/${resultId}/state`,
    A_SEARCH_TWO_RESULT   : (searchId, resultId) =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/results/${resultId}`,
    A_SEARCH_STEP_THREE   : (searchId, resultId, roomContractSetId) =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/results/${resultId}/room-contract-sets/${roomContractSetId}`,
    REQUEST_DEADLINE      : (searchId, resultId, roomContractSetId) =>
                            v1 + `/accommodations/availabilities/searches/${searchId}/results/${resultId}/room-contract-sets/${roomContractSetId}/deadline`,

    PAYMENTS_HISTORY      : v1 + `/agent/payments-history`,

    BASE_VERSION          : v1 + "/versions",

    DIRECT_LINK_PAY : {
        SETTINGS     :         v1 + "/external/payment-links/tokenization-settings",
        GET_INFO     : code => v1 + "/external/payment-links/" + code,
        SIGN         : code => v1 + "/external/payment-links/" + code + "/sign",
        PAY          : code => v1 + "/external/payment-links/" + code + "/pay",
        PAY_CALLBACK : code => v1 + "/external/payment-links/" + code + "/pay/callback"
    },

    AGENCY               : v1 + `/agency`,
    AGENCY_AGENTS        : v1 + `/agency/agents`,
    AGENCY_AGENT         : agentId =>
                           v1 + `/agency/agents/${agentId}`,
    AGENT_ENABLE         : agentId =>
                           v1 + `/agency/agents/${agentId}/enable`,
    AGENT_DISABLE        : agentId =>
                           v1 + `/agency/agents/${agentId}/disable`,
    AGENCY_BANNER        : v1 + "/agency/images/banner",
    AGENCY_LOGO          : v1 + "/agency/images/logo",
    COUNTERPARTY_INFO    : v1 + "/counterparty",
    COUNTERPARTY_FILE    : v1 + "/counterparty/contract-file",
    COMPANY_INFO         : v1 + "/company",
    AGENT_SETTINGS       : v1 + "/agent/settings/application",
    ALL_PERMISSIONS      : v1 + "/all-permissions-list",
    AGENT_PERMISSIONS    : agentId =>
                           v1 + `/agency/agents/${agentId}/permissions`,
    AGENCY_APR_SETTINGS  : v1 + `/agency/system-settings/apr-settings`,
    AGENCY_PAYMENT_OPTION: v1 + `/agency/system-settings/displayed-payment-options`,
    MARKUP_TEMPLATES     : v1 + "/markup-templates",
    AGENT_MARKUPS        : agentId =>
                           v1 + `/agency/agents/${agentId}/markups`,
    AGENT_MARKUP         : (agentId, policyId) =>
                           v1 + `/agency/agents/${agentId}/markups/${policyId}`,

    REPORT_DUPLICATE     : v1 + "/accommodations-mapping/duplicate-reports",

    OUR_COMPANY          : v1 + "/company"
};

API_METHODS.methods_dont_show_error = [
    API_METHODS.AGENT_SETTINGS,
    API_METHODS.BASE_VERSION,
    API_METHODS.BASE_REGIONS,
    API_METHODS.BASE_CURRENCIES,
    API_METHODS.OUR_COMPANY
];

export default fetch(API_METHODS);

```

### src/components/form/field-text.js
```json
import React, { useState, useCallback } from "react";
import { getIn } from "formik";
import { observer } from "mobx-react";
import { getLocale } from "core";
import { decorate } from "simple";
import { $ui, $view } from "stores";

const FieldText = observer(({
    label,
    placeholder,
    Icon,
    AfterIcon,
    clearable,
    password,
    className,
    id,
    Dropdown,
    value,
    disabled,
    required,
    readOnly,
    options,
    ValueObject,
    maxLength,
    numeric,
    suggestion,
    formik,
    setValue,
    additionalFieldForValidation,
    autoComplete,
    onChange,
    onClear,
    dataDropdown,
    noInput
}) => {
    const [focused, setFocused] = useState(false);
    const [optionIndex, setOptionIndex] = useState(null);
    const [everTouched, setEverTouched] = useState(false);
    const [everChanged, setEverChanged] = useState(false);

    const onFocus = useCallback(() => {
        if (Dropdown) {
            $view.setOpenDropdown(id);
            setOptionIndex(null);
        }
        setFocused(true);
    }, []);

    const blur = useCallback((event) => {
        setFocused(false);
        if (formik)
            formik.handleBlur(event);
        if (!everTouched)
            setEverTouched(true);
    }, []);

    const onKeyDown = (event) => {
        if (!Dropdown || !options?.length) return;

        const scroll = document.querySelectorAll(`#${id} .scroll > div:not(.subtitle)`),
              scrollElem = document.querySelector(`#${id} .scroll`);

        if (!scroll?.length)
            return;

        const suggestion = $ui.suggestions[id]?.suggestionObject;

        switch (event.key) {
            case "Enter":
            case "ArrowRight":
                if (optionIndex !== null || suggestion) {
                    event.preventDefault();
                    setValue(formik, id, optionIndex !== null ? options[optionIndex] : suggestion, false);
                    setFocused(false);
                }
                break;
            case "ArrowUp":
            case "ArrowDown":
                let possible = optionIndex > 0,
                    newIndex = optionIndex - 1,
                    correction = -100;
                if ("ArrowDown" === event.key) {
                    possible = optionIndex === null || options.length > optionIndex + 1;
                    newIndex = optionIndex === null ? 0 : optionIndex + 1;
                    correction = -20;
                }
                if (possible) {
                    setOptionIndex(newIndex);
                    scrollElem.scrollTo(0, scroll[optionIndex]?.offsetTop + correction);
                } else {
                    scrollElem.scrollTo(0, 0);
                }
                if (setValue)
                    setValue(formik, id, options[optionIndex], true);
                break;
            default:
                return;
        }
    };

    const clear = useCallback(() => {
        if (formik) {
            formik.setFieldValue(id, "");
            formik.setFieldTouched(id, false);
            if (Dropdown)
                $view.setOpenDropdown(null);
            if (additionalFieldForValidation)
                formik.setFieldValue(additionalFieldForValidation, false);
        }
        if (onClear)
            onClear();
    },[]);

    const changing = useCallback((event) => {
        if (Dropdown) {
            $view.setOpenDropdown(id);
            setOptionIndex(null);
        }
        if (numeric)
            if ("/" == numeric)
                event.target.value = event.target.value.replace(/[^0-9.\/ ]/g, "");
            else
                event.target.value = event.target.value.replace(/[^0-9.]/g, "");
        if (onChange)
            onChange(event, formik, id);
        if (formik) {
            formik.setFieldTouched(id, true);
            formik.handleChange(event);
        }
        if (!everChanged)
            setEverChanged(true);
    },[]);

    const errorText = getIn(formik?.errors, id);
    const isFieldTouched = getIn(formik?.touched, id);
    const fieldValue = getIn(formik?.values, id);

    if (suggestion)
        suggestion = decorate.cutFirstPart(suggestion, fieldValue);

    /* todo: Remove this workaround when server rtl suggestions works correct */
    var isSuggestionVisible = getLocale() != "ar";

    if (ValueObject !== undefined) {
        if (ValueObject)
            ValueObject = <div className="value-object">{ValueObject}</div>;
        else
            ValueObject = <div className="value-object placeholder">{placeholder}</div>;
    }

    if (formik && !suggestion)
        suggestion = $ui.getSuggestion(id, fieldValue);

    var finalValue = "";
    if (!ValueObject) {
        finalValue = value || (formik?.values ? fieldValue : "");
        if (finalValue === 0)
            finalValue = "0";
        if (!finalValue)
            finalValue = "";
    }

    return (
        <div
            className={
                "field" +
                __class(className) +
                __class(focused || $view.isDropdownOpen(id), "focus") +
                __class(disabled, "disabled") +
                __class(noInput, "no-input") +
                __class((
                (errorText ||
                (additionalFieldForValidation && getIn(formik?.errors, additionalFieldForValidation))) &&
                isFieldTouched),
                    "error") +
                __class(!errorText && finalValue, "valid")
            }
            data-dropdown={dataDropdown || id}
        >
            <label onClick={noInput ? onFocus : null}>
                { label &&
                    <div className="label">
                        <span className={__class(required, "required")}>{label}</span>
                    </div>
                }
                <div className="input">
                    { !!Icon &&
                        <div className="icon-wrap">
                            {Icon}
                        </div>
                    }
                    { !noInput ?
                        <div className="inner">
                            <input
                                name={id}
                                type={ password ? "password" : "text" }
                                placeholder={ !ValueObject ? placeholder : "" }
                                onFocus={onFocus}
                                onChange={changing}
                                onBlur={blur}
                                value={finalValue}
                                onKeyDown={onKeyDown}
                                disabled={!!disabled}
                                maxLength={maxLength}
                                autoComplete={autoComplete ? autoComplete : "off"}
                                {...(readOnly ? {readOnly: "readonly"} : {})}
                            />
                            {ValueObject}
                            { isSuggestionVisible && suggestion &&
                                <div className={"suggestion" + __class(numeric, "solid")}>
                                    <span>{ fieldValue || value }</span>{ suggestion }
                                </div>
                            }
                        </div> :
                        <div className="inner">
                            { (value || finalValue) ?
                                (ValueObject ? ValueObject : finalValue) :
                                <span className="placeholder">{placeholder}</span>
                            }
                        </div>
                    }
                    { AfterIcon &&
                        <div className="after-icon-wrap">
                            { AfterIcon }
                        </div>
                    }
                    { clearable && !!finalValue &&
                        <div>
                            <div className="clear" onClick={clear} />
                        </div>
                    }
                </div>
                { (errorText?.length > 1 && isFieldTouched && !$view.isDropdownOpen(id)) &&
                    <div className={
                        "error-holder" +
                        __class(!everTouched || !everChanged || focused, "possible-hide")
                    }>
                        {errorText}
                    </div>
                }
            </label>
            { !!Dropdown &&
                <div className={__class(!$view.isDropdownOpen(id), "hide")}>
                    <Dropdown
                        formik={formik}
                        connected={id}
                        setValue={
                            (...params) => {
                                if (setValue)
                                    setValue(...params);
                                setFocused(false);
                            }}
                        value={fieldValue}
                        options={options}
                        focusIndex={optionIndex}
                    />
                </div>
            }
        </div>
    );
});

export default FieldText;

```

### src/index.js
```json
import React from "react";
import ReactDOM from "react-dom";
import * as Sentry from "@sentry/browser";
import { getLocale } from "core"
import { initApplication } from "core/init";
import App from "core/app";
import settings from "settings";
import tracker from "core/misc/tracker";

if (settings.sentry_dsn)
    Sentry.init({
        dsn: settings.sentry_dsn,
        environment: settings.sentry_environment
    });

tracker();

window.setPageDirectionFromLS = () => {
    var dir = ("ar" == getLocale() ? "rtl" : "ltr");
    document.getElementsByTagName("html")[0].setAttribute("dir", dir);
};
window.setPageDirectionFromLS();

window._renderTheApp = () => ReactDOM.render(<App />, document.getElementById("app"));
window._renderTheApp();

initApplication();

```

### src/simple/logic/date.js
```json
import { default as basicFormat } from "date-format";
import localeUtils from "tasks/utils/date-locale-utils";
import { getLocale } from "core";

//const parse = (date) => {
//   return basicFormat.parse("format", date);
//};

const shortMonth = (val) => {
    if ("ar" === getLocale() || (val?.length < 5))
        return val;
    return val.substr(0, 3);
};

const dateFormat = (template, rawDate) => {
    let date = new Date(rawDate);
    return (
        basicFormat(template, date)
            .replace('DAY', localeUtils.formatWeekdayLong(date.getDay()))
            .replace('MTH', localeUtils.getMonths()[date.getMonth()])
            .replace('mth', shortMonth(localeUtils.getMonths()[date.getMonth()]))
            .replace('DD', date.getDate())
    );
};

const format = {
    api: date => dateFormat("yyyy-MM-ddT00:00:00Z", date),
    a: date => !date ? '' : dateFormat("DAY, dd MTH yyyy", date),
    c: date => !date ? '' : dateFormat("dd-MM-yyyy", date),
    e: date => !date ? '' : dateFormat("dd-MTH-yyyy", date),
    day: date => !date ? '' : dateFormat("MTH DD", date),
    shortDay: date => !date ? '' : dateFormat("mth DD", date),
};

const addMonth = (date, amount) => {
    let result = new Date(date),
        previous = result.getMonth();
    result.setMonth(previous+amount);
    return result;
};

const addDay = (date, amount) => {
    let result = new Date(date),
        previous = result.getDate();
    result.setDate(previous+amount);
    return result;
};

const passed = (date) => {
    if (!date)
        return true;
    let result = new Date() > new Date(date);
    if (format.api(new Date()) == format.api(date))
        result = false;
    return result;
};

const parseDateRangeFromString = (text = "") => {
    const res = text
        .split(/[.,\/\- –]/)
        .filter(x => x);

    if (res.map(v => v.length).join(',') != "2,2,4,2,2,4")
        return null;

    const range = [
        new Date(+res[2], +res[1]-1, +res[0]),
        new Date(+res[5], +res[4]-1, +res[3])
    ];

    const isDateCorrect = (date) => {
        if (passed(date))
            return false;
        if (date.getFullYear() > new Date().getFullYear() + 5)
            return false;
        return true;
    };

    if (!isDateCorrect(range[0]) || !isDateCorrect(range[1]))
        return null;

    return range;
};

export default {
    format,
    addDay,
    addMonth,
    passed,
    parseDateRangeFromString
};
```

### src/tasks/utils/date-locale-utils.js
```json
import React from "react";
import { getLocale } from "core";

const WEEKDAYS_LONG = (locale) => {
    const list = {
        en: [
            "Sunday",
            "Monday",
            "Tuesday",
            "Wednesday",
            "Thursday",
            "Friday",
            "Saturday"
        ],
        ar: [
            "الأحد",
            "الإثنين",
            "الثلاثاء",
            "الأربعاء",
            "الخميس",
            "الجمعة",
            "السبت"
        ],
    };
    return list[locale] || list['en'];
};

const WEEKDAYS_SHORT = (locale) => {
    const list = {
        en: ["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa"],
        ar: [
            "الأحد",
            "الإثنين",
            "الثلاثاء",
            "الأربعاء",
            "الخميس",
            "الجمعة",
            "السبت",
        ],
    };
    return list[locale] || list['en'];
};

const MONTHS = (locale) => {
    const list = {
        en: [
            "January",
            "February",
            "March",
            "April",
            "May",
            "June",
            "July",
            "August",
            "September",
            "October",
            "November",
            "December"
        ],
        ar: [
            "يناير",
            "فبراير",
            "مارس",
            "أبريل",
            "مايو",
            "يونيو",
            "يوليو",
            "أغسطس",
            "سبتمبر",
            "أكتوبر",
            "نوفمبر",
            "ديسمبر"
        ],
    };
    return list[locale] || list['en'];
};

const FIRST_DAY = (locale) => ({
    en: 1,
    ar: 0,
}[locale] || 1);

const formatDay = (d) => {
    const locale = getLocale();
    return `${WEEKDAYS_LONG(locale)[d.getDay()]}, ${d.getDate()} ${
        MONTHS(locale)[d.getMonth()]
    } ${d.getFullYear()}`;
};

const formatMonthTitle = (d) => {
    const locale = getLocale();

    return (
        MONTHS(locale)[d.getMonth()] +
        (d.getFullYear() !== new Date().getFullYear() ?
            " " + d.getFullYear() :
            ""
        )
    );
};

const formatWeekdayShort = (i) => {
    const locale = getLocale();
    return WEEKDAYS_SHORT(locale)[i];
};

const formatWeekdayLong = (i) => {
    const locale = getLocale();
    return WEEKDAYS_LONG(locale)[i];
};

const getFirstDayOfWeek = () => {
    const locale = getLocale();
    return FIRST_DAY(locale);
};

const getMonths = () => {
    const locale = getLocale();
    return MONTHS(locale);
};

const localeUtils = {
    formatDay,
    formatMonthTitle,
    formatWeekdayShort,
    formatWeekdayLong,
    getFirstDayOfWeek,
    getMonths
};

export default localeUtils;

```

### src/core/misc/get-locale.js
```json
import { windowLocalStorage } from "core/misc/window-storage";
import settings from "settings";

const getLocale = () => windowLocalStorage.get("locale") || settings.default_culture;

export default getLocale;

```

### src/simple/logic/agent-settings.js
```json
import { API, getLocale } from "core";
import { switchLocale } from "core/misc/switch-locale";
import { $personal } from "stores";

const settingsCleaner = values => ({
    nationality: values.nationality,
    residency: values.residency,
    nationalityCode: values.nationalityCode,
    residencyCode: values.residencyCode,
    weekStarts: values.weekStarts,
    preferredLanguage: values.preferredLanguage,
    availableCredit: values.availableCredit,
    experimentalFeatures: values.experimentalFeatures
});

export const loadCounterpartyInfo = (callback = () => {}) => {
    API.get({
        url: API.COUNTERPARTY_INFO,
        success: information => {
            $personal.setCounterpartyInfo(information);
            callback(information);
        }
    });
};

export const fillEmptyAgentSettings = () => {
    loadCounterpartyInfo(information => {
        if (!information?.countryName || !information?.countryCode)
            return;
        saveAgentSettings({
            residency: information.countryName,
            residencyCode: information.countryCode,
            nationality: information.countryName,
            nationalityCode: information.countryCode
        });
    });
};

export const loadAgentSettings = () => {
    API.get({
        url: API.AGENT_SETTINGS,
        success: (result) => {
            $personal.setSettings(result);
            if (!Object.keys(result || {}).length)
                fillEmptyAgentSettings();
            if ("ar" == result.preferredLanguage && !getLocale())
                switchLocale("ar");
        }
    });
};

export const saveAgentSettings = (values, after) => {
    API.put({
        url: API.AGENT_SETTINGS,
        body: settingsCleaner(values),
        success: () => {
            $personal.setSettings(values);
            if ("ar" == result.preferredLanguage && "ar" != getLocale())
                switchLocale("ar");
        },
        after
    });
};

```

### src/core/index.js
```json
import { session, localStorage } from "./storage";
import { getParams } from "./misc/get-params";
import API from "./api";
import getLocale from "./misc/get-locale";
import { redirect } from "./misc/history";

export {
    session,
    localStorage,
    getParams,
    API,
    getLocale,
    redirect
};

```

### src/core/internationalization.js
```json
import i18n from "i18next";
import { getLocale } from "core";
import settings from "settings";

import english from "translation/english";
import arabic from "translation/arabic";

i18n.init({
    lng: getLocale(),
    resources: {
        en: english,
        ar: arabic
    },
    fallbackLng: "en",
    debug: __localhost,

    ns: ["translations"],
    defaultNS: "translations",

    keySeparator: true,

    interpolation: {
        escapeValue: false,
        formatSeparator: ','
    },

    react: {
        wait: true
    }
});

export default i18n;

```

### src/pages/settings/agent-permissions-management.js
```json
import React from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { Formik } from "formik";
import { API, redirect } from "core";
import { PassengerName } from "simple";
import { Loader } from "components/simple";
import Breadcrumbs from "components/breadcrumbs";
import { FieldSwitch } from "components/form";
import SettingsHeader from "./parts/settings-header";
import Markups from "parts/markups/markups";
import { $personal } from "stores";

const generateLabel = str => {
    if (!str)
        return "Unknown";
    var split = str.split(/([A-Z])/);
    for (var i = 0; i < split.length; i++)
        if (split[i].match(/([A-Z])/))
            split[i]= " " + split[i];
    return split.join("");
};

@observer
export default class AgentPermissionsManagement extends React.Component {
    state = {
        loading: true,
        agent: {},
        permissionsList: []
    };

    enable = () => {
        const { agentId } = this.props.match.params;
        API.post({
            url: API.AGENT_ENABLE(agentId),
            success: () => this.setState({
                agent: {
                    ...this.state.agent,
                    isActive: true
                }
            })
        });
    };

    disable = () => {
        const { agentId } = this.props.match.params;
        API.post({
            url: API.AGENT_DISABLE(agentId),
            success: () => this.setState({
                agent: {
                    ...this.state.agent,
                    isActive: false
                }
            })
        });
    };

    componentDidMount() {
        if (this.props.match?.params) {
            const { agentId } = this.props.match.params;

            API.get({
                url: API.AGENCY_AGENT(agentId),
                success: (agent) => this.setState({
                    agent,
                    loading: false
                })
            });
            API.get({
                url: API.ALL_PERMISSIONS,
                success: permissionsList => this.setState({ permissionsList })
            });
        }
    }

    submit = (values) => {
        this.setState({ loading: true });

        var { agentId } = this.props.match.params,
            url = API.AGENT_PERMISSIONS(agentId),
            body = Object.keys(values).map((key) => values[key] ? key : false).filter(item => item);

        API.put({
            url,
            body,
            success: () => redirect("/settings/agents"),
            after: () => this.setState({ loading: false })
        });
    };

    render() {
        var { t } = useTranslation(),
            { agent, permissionsList, loading } = this.state,
            { inAgencyPermissions } = agent;

        return (
        <div className="settings block">
            <SettingsHeader />

            { loading ?
                <Loader /> :
            <section>
                <Breadcrumbs items={[
                    {
                        text: t("Agent Management"),
                        link: "/settings/agents"
                    }, {
                        text: loading ? t("Agent Permissions") : PassengerName({ passenger: agent })
                    }
                ]}/>
                <h2>{t("Information")}</h2>
                <div className="row">
                    <b>{t("Agent")}</b>:{" "}
                    {PassengerName({ passenger: agent })}
                </div>
                <div className="row">
                    <b>{t("Position")}</b>:{" "}
                    {agent.position}
                </div>
                <div className="row">
                    <b>{t("Status")}</b>:{" "}
                    {agent.isActive ? "Active" : "Inactive"}
                </div>
                { agent.isMaster ? <div className="row">
                    <b>{t("Main agent")}</b>
                </div> : "" }

                { $personal.permitted("AgentStatusManagement") &&
                  $personal.information.id != agent.agentId &&
                <div>
                    {!agent.isActive ? <button
                        className="button"
                        onClick={this.enable}
                        style={{ paddingLeft: "20px", paddingRight: "20px", marginRight: "20px" }}
                    >
                        {t("Activate agent")}
                    </button> :
                    <button
                        className="button"
                        onClick={this.disable}
                        style={{ paddingLeft: "20px", paddingRight: "20px", marginRight: "20px" }}
                    >
                        {t("Deactivate agent")}
                    </button>}
                </div> }

                { $personal.permitted("PermissionManagement") && <>
                    <h2>{t("Permissions")}</h2>
                    <div>
                        <Formik
                            onSubmit={this.submit}
                            initialValues={{
                                ...permissionsList.reduce((obj, key) => (
                                    {...obj, [key]: inAgencyPermissions.includes(key)}),
                                {})
                            }}
                            enableReinitialize
                        >
                            {formik => (
                                <form onSubmit={formik.handleSubmit}>
                                    <div className="form">
                                        <div className="permissions">
                                            {permissionsList.map(key => (
                                                <div className="row" key={key}>
                                                    <FieldSwitch formik={formik}
                                                                 id={key}
                                                                 label={generateLabel(key)}
                                                    />
                                                </div>
                                            ))}
                                        </div>

                                        <div className="row controls">
                                            <div className="field">
                                                <div className="label"/>
                                                <div className="inner">
                                                    <button type="submit" className={"button button-controls" +
                                                    __class(!formik.dirty, "disabled")}>
                                                        {t("Save changes")}
                                                    </button>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </form>
                            )}
                        </Formik>
                    </div>
                </> }


                { $personal.permitted("MarkupManagement") &&
                    <Markups
                        id={agent.agentId}
                        emptyText={"Agent has no markups"}
                        markupsRoute={API.AGENT_MARKUPS}
                        markupRoute={API.AGENT_MARKUP}
                    />
                }
            </section>
            }
        </div>
        );
    }
}

```

### src/pages/settings/child-agencies/parts/transfer-balance.js
```json
import React from "react";
import { useTranslation } from "react-i18next";
import { API } from "core";
import { price } from "simple";
import { CachedForm, FieldSelect, FieldText } from "components/form";
import { transferBalanceValidator } from "components/form/validation";
import { $notifications, $personal } from "stores";

const generateOptions = (accounts) =>
    accounts.map(item => (
        {
            text: `Account #${item.id}: ${price(item.balance)}`,
            value: item.id
        }
    ));

const ChildAgencyTransferBalancePart = ({
    payer,
    recipient,
    onUpdate
}) => {
    const { t } = useTranslation();

    const submit = (values, formik) => {
        API.post({
            url: API.CHILD_AGENCY_TRANSFER_ACCOUNT_FUNDS(values.payerId, values.recipientId),
            body: values,
            success: () => {
                onUpdate();
                formik.resetForm();
                $notifications.addNotification(`${price(values)} transferred!`, "Completed!", "success");
            }
        });
    };

    if (!$personal.permitted("AgencyToChildTransfer"))
        return null;

    return (
        <div>
            <h2>{t("Transfer Balance")}</h2>
            <CachedForm
                onSubmit={submit}
                initialValues={{
                    payerId: payer?.length == 1 ? payer[0].id : "",
                    recipientId: recipient?.length == 1 ? recipient[0].id : "",
                    amount: "",
                    currency: "USD"
                }}
                validationSchema={transferBalanceValidator}
                render={formik => (
                    <div className="form" style={{ width: 400 }}>
                        <div className="row">
                            <FieldSelect
                                formik={formik}
                                id="payerId"
                                label={t("From Account")}
                                placeholder="Please Select"
                                required
                                options={generateOptions(payer)}
                            />
                        </div>
                        <div className="row">
                            <FieldSelect
                                formik={formik}
                                id="recipientId"
                                label={t("To Account")}
                                placeholder="Please Select"
                                required
                                options={generateOptions(recipient)}
                            />
                        </div>
                        <div className="row">
                            <FieldText
                                formik={formik}
                                id="amount"
                                label="Amount"
                                placeholder="Amount"
                                numeric
                            />
                        </div>
                        <div className="row">
                            <FieldSelect
                                formik={formik}
                                id="currency"
                                label={t("Currency")}
                                required
                                options={[
                                    { value: "USD", text: "USD"},
                                    { value: "EUR", text: "EUR"},
                                    { value: "AED", text: "AED"},
                                    { value: "SAR", text: "SAR"}
                                ]}
                            />
                        </div>
                        <div className="row">
                            <button type="submit" className="button" style={{ width: "100%" }}>
                                {t("Transfer")}
                            </button>
                        </div>
                    </div>
                )}
            />
        </div>
    );
};

export default ChildAgencyTransferBalancePart;

```

### src/core/misc/fetch.js
```json
import Authorize from "core/auth/authorize";
import { isPageAvailableAuthorizedOnly } from "core/auth";
import { $notifications } from "stores";

export default api => {
    const showError = (err, url = "") => {
        if (api.methods_dont_show_error.includes(url) || url?.includes("/state"))
            return;
        if (typeof err.detail == "string")
            return $notifications.addNotification(err.detail);
        if (typeof err.title == "string")
            return $notifications.addNotification(err.title, "Server error");
        if (typeof err == "string")
            return $notifications.addNotification(err, "Server message");
        $notifications.addNotification("Server Request Error");
    };

    api.request = ({
        url, external_url,
        body = {}, formDataBody,
        method = "GET",
        response, // function(response)                - Fires first
        success,  // function(result)                  - Fires second on success
        error,    // function(error)                   - Fires second on error,
        after     // function(result, error, response) - Fires the last
    }) => new Promise((resolve, reject) => {
        Authorize.getUser().then(user => {
            if (!external_url && !user?.access_token) {
                if (isPageAvailableAuthorizedOnly())
                    Authorize.signinRedirect();
                reject();
                return;
            }

            var finalUrl = url || external_url,
                request = {
                    method: method,
                    headers: new Headers({
                        ...(external_url ? {} : {
                            'Authorization': `Bearer ${user.access_token}`
                        }),
                        ...(formDataBody ? {} : {
                            'Content-Type': 'application/json'
                        })
                    })
                };

            if (["POST", "PUT", "DELETE"].includes(method))
                request.body = JSON.stringify(body);
            else {
                var getBody = Object.keys(body).map(key =>
                    [key, body[key]].map(encodeURIComponent).join("=")
                ).join("&");
                finalUrl += (getBody ? "?" + getBody : "");
            }

            if (formDataBody)
                request.body = formDataBody;

            var rawResponse = null,
                failed = false;
            fetch(finalUrl, request)
                .then(
                    res => {
                        rawResponse = res;
                        failed = !res || (res && res.status >= 300);
                        if (response) {
                            response(res);
                            return;
                        }
                        return res.text().then(text => {
                            var value = null;
                            if (text) {
                                try {
                                    value = JSON.parse(text);
                                } catch (e) {
                                    value = text;
                                }
                            }
                            return value;
                        });
                    },
                    error => {
                        showError(error, url);
                        reject(error);
                    }
                )
                .then(
                    (result) => {
                        if ((rawResponse.status == 401) && isPageAvailableAuthorizedOnly()) {
                            Authorize.signinRedirect();
                            reject(null);
                            return;
                        }
                        if (rawResponse.status == 403) {
                            showError("Sorry, you don`t have enough permissions", url);
                            if (error)
                                error(result);
                            if (after)
                                after(null, null, rawResponse);
                            reject(result);
                            return;
                        }
                        if (failed) {
                            if (result && result.status >= 400)
                                showError(result, url);
                            if (error)
                                error(result);
                        } else {
                            if (success)
                                success(result);
                        }
                        if (after)
                            after(
                                failed ? null : result,
                                failed ? result : null,
                                rawResponse
                            );
                        if (!failed)
                            resolve(result);
                        else
                            reject(result);
                    },
                    (err) => {
                        if (error)
                            error(err);
                        else
                            showError(err);
                        if (after)
                            after(null, err, rawResponse);
                        reject(err);
                    }
                );
            }
        );
    });

    api.get = (params) =>
        api.request({
            method: "GET",
            ...params
        });

    api.post = (params) =>
        api.request({
            method: "POST",
            ...params
        });

    api.put = (params) =>
        api.request({
            method: "PUT",
            ...params
        });

    api.delete = (params) =>
        api.request({
            method: "DELETE",
            ...params
        });

    return api;
};

```

### src/pages/agent/bookings-management/bookings-management.js
```json
import React, { useState, useEffect } from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { API, redirect } from "core";
import { date } from "simple";
import Table from "components/table";
import { FieldCheckbox } from "components/form";
import { Columns, Sorters, Searches } from "./table-data";
import { $personal } from "stores";

const AgencyBookingsManagementPage = observer(() => {
    const [filterTab, setFilterTab] = useState(null);
    const [agentIdFilter, setAgentIdFilter] = useState(null);

    useEffect(() => {
        $personal.setBookingList(null);

        if (permittedAgency)
            API.get({
                url: API.AGENCY_BOOKINGS_LIST,
                after: list => {
                    if (list?.length)
                        $personal.setBookingList(list.map(item => ({
                            ...item.data,
                            agent: item.agent
                        })));
                    else
                        $personal.setBookingList([]);
                }
            });
        else
            API.get({
                url: API.BOOKING_LIST,
                success: list => {
                    $personal.setBookingList(list);
                }
            });
    }, []);
    
    const Tab = ({ text, value }) => (
        <li>
            <div
                className={"item" + __class(value == filterTab, "active")}
                onClick={() => setFilterTab(value)}
            >
                {text}
            </div>
        </li>
    );

    const filter = list => {
        let result;

        if ("Cancelled" == filterTab)
            result = list.filter(item => "Cancelled" == item.status);
        else {
            result = list.filter(item => "Cancelled" != item.status);

            if ("Future" == filterTab || "Complete" == filterTab)
                result = result.filter(item => {
                    var isFuture = !date.passed(item.checkInDate);
                    return ("Future" == filterTab) ? isFuture : !isFuture;
                });
        }

        if (agentIdFilter !== null)
            result = result.filter(item => item.agent.id == agentIdFilter);

        return result;
    };

    const permittedAgency = $personal.permitted("AgencyBookingsManagement");
    
    const { t } = useTranslation();

    return (
        <div className="management block">
            <section className="sticky-header">
                <h2>
                    { permittedAgency ? t("Agency Bookings") : t("Bookings") }
                </h2>
            </section>
            <div className="sticky-header-before" />
            <div className="navigation">
                <section>
                    <nav>
                        <Tab text={t("All Bookings")} value={null} />
                        <Tab text={t("Future")} value="Future" />
                        <Tab text={t("Complete")} value="Complete" />
                        <Tab text={t("Cancelled")} value="Cancelled" />
                    </nav>
                </section>
            </div>
            <section className="content agency">
                <Table
                    columns={Columns(permittedAgency)(t, setAgentIdFilter)}
                    list={$personal.bookingList}
                    textEmptyResult={t("No reservations found")}
                    textEmptyList={t("You don`t have any reservations")}
                    onRowClick={item => redirect(`/booking/${item.referenceCode}`)}
                    filter={filter}
                    sorters={Sorters(permittedAgency)(t)}
                    searches={Searches(permittedAgency)}
                    CustomFilter={
                        permittedAgency &&
                            <div className="agent-filter">
                                <FieldCheckbox
                                    label={t("Show Only My Bookings")}
                                    onChange={(value) => setAgentIdFilter(value ? $personal.information.id : null)}
                                />
                            </div>
                    }
                />
            </section>
        </div>
    );
});

export default AgencyBookingsManagementPage;

```

### src/parts/header/agent-menu.js
```json
import React from "react";
import { useTranslation } from "react-i18next";
import { observer } from "mobx-react";
import { Link } from "react-router-dom";
import { $personal } from "stores";

const AgentMenu = observer(() => {
    const agentName = ($personal.information?.firstName || "") + " " + ($personal.information?.lastName || "");

    const { t } = useTranslation();
    return (
        <div className="agent-menu">
            { $personal.permitted("AccommodationBooking") &&
                <Link className="button" to="/bookings">
                    {t("Bookings")}
                </Link>
            }
            <Link to="/settings" className="button agent-link" title={agentName}>
                <span className="icon icon-burger" />
                <span className="avatar" />
            </Link>
        </div>
    );
});

export default AgentMenu;

```

### styles/block/management.sass
```json
.management.block
    padding-bottom: 150px
    @include except-phone
        .sticky-header-before
            height: 120px
        .sticky-header
            position: sticky
            top: 25px
            z-index: 1002
            white-space: nowrap
            height: 0
            h2
                overflow: visible
    h2
        font-size: 24px
        line-height: 30px
        margin: 25px 0 45px
        font-weight: 300

    .controls
        .form
            @include phone
                display: flex
                flex-direction: column-reverse
                justify-content: flex-end

    .content
        min-height: 370px

    .gray
        color: $color-text-light

    .payment-amount
        font-size: 90%

    .the-table
        tr
            $cols: (1: 130px, 2: 170px, 3: 110px, 4: 130px, 5: 90px, 6: 80px, 7: 80px, 8: 110px)
            @each $id, $col in $cols
                td:nth-child(#{$id})
                    min-width: $col !important
                    max-width: $col !important

    .agency
        .the-table
            tr
                $cols: (1: 150px, 2: 140px, 3: 150px, 4: 100px, 5: 100px, 6: 110px, 7: 80px, 8: 80px, 9: 110px)
                @each $id, $col in $cols
                    td:nth-child(#{$id})
                        min-width: $col !important
                        max-width: $col !important
        .agent-filter
            div
                margin-top: 52px
                margin-left: 15px
                cursor: pointer
                @include phone
                    margin: 0 0 30px 10px

    &.payments-history
        .the-table
            tr
                $cols: (1: 90px, 2: 90px, 3: 80px, 4: 120px, 5: 140px, 6: 140px)
                @each $id, $col in $cols
                    td:nth-child(#{$id})
                        min-width: $col !important
                        max-width: $col !important

```

### src/pages/settings/parts/agent-application-settings.js
```json
import React from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import i18n from 'i18next';
import { Loader, Flag } from "components/simple";
import { CachedForm, FieldSelect, FieldSwitch, FORM_NAMES } from "components/form";
import FieldCountry from "components/complex/field-country";
import { loadAgentSettings, saveAgentSettings } from "simple/logic";
import { switchLocale } from "core/misc/switch-locale";
import { $ui, $personal } from "stores";

@observer
class AgentApplicationSettings extends React.Component {
    state = {
        loading: false
    };

    submitAgentSettings = (values) => {
        var shouldDropSearchCache =
            values.nationality != $personal.settings.nationality ||
            values.residency != $personal.settings.residency;

        this.setState({ loading: true });
        saveAgentSettings(
            values,
            () => {
                if (values.preferredLanguage != i18n.language)
                    switchLocale(values.preferredLanguage);{
                    if (shouldDropSearchCache)
                        $ui.dropFormCache(FORM_NAMES.SearchForm);
                }
                this.setState({ loading: false });
            }
        );
    };

    componentDidMount() {
        loadAgentSettings();
    }

    render() {
        const { t } = useTranslation();
        if (this.state.loading)
            return <Loader page />;

        return (
            <>
                <h2>{t("Application Settings")}</h2>

                <CachedForm
                    initialValues={$personal.settings}
                    enableReinitialize
                    onSubmit={this.submitAgentSettings}
                    render={formik => (
                        <div className="form app-settings">
                            { $personal.permitted("ObserveBalance") &&
                                <div className="row">
                                    <FieldSwitch formik={formik}
                                                 id="availableCredit"
                                                 label={t("Show Available Credit")}
                                    />
                                </div>
                            }
                            <div className="row">
                                <FieldSwitch formik={formik}
                                             id="experimentalFeatures"
                                             label={t("Enable experimental features (may be unstable)")}
                                />
                            </div>
                            <div className="row double">
                                <FieldSelect formik={formik}
                                             id="preferredLanguage"
                                             label={t("Preferred language")}
                                             placeholder={t("Preferred language")}
                                             options={[
                                                 { value: "en", text: "English", flag: "gb"},
                                                 { value: "ar", text: "اللغة الحالية", flag: "ae"}
                                             ]}
                                             Icon={<Flag language={formik.values.preferredLanguage} />}
                                />
                                <FieldSelect formik={formik}
                                             id="weekStarts"
                                             label={t("Week starts on")}
                                             placeholder={t("Automatic")}
                                             options={[
                                                 { value: 0, text: "Automatic"},
                                                 { value: 7, text: "Sunday"},
                                                 { value: 1, text: "Monday"},
                                                 { value: 2, text: "Tuesday"},
                                                 { value: 3, text: "Wednesday"},
                                                 { value: 4, text: "Thursday"},
                                                 { value: 5, text: "Friday"},
                                                 { value: 6, text: "Saturday"},
                                             ]}
                                />
                            </div>
                            <div className="row double">
                                <FieldCountry formik={formik}
                                              id="nationality"
                                              anotherField="residency"
                                              label={t("Nationality")}
                                              placeholder={t("Choose your nationality")}
                                              clearable
                                />
                                <FieldCountry formik={formik}
                                              id="residency"
                                              anotherField="nationality"
                                              label={t("Residency")}
                                              placeholder={t("Choose your residency")}
                                              clearable
                                />
                            </div>
                            <div className="row controls">
                                <div className="field">
                                    <div className="inner">
                                        <button type="submit" className={"button" +
                                                                __class(!formik.isValid || !formik.dirty, "disabled")}>
                                            {t("Save changes")}
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}
                />
            </>
        );
    }
}

export default AgentApplicationSettings;

```

### styles/components/notifications.sass
```json
.notifications
    position: fixed
    top: 90px
    width: 100%
    z-index: 1001
    .wrapper
        display: block
        position: sticky
        top: 20px
        left: 100%
        max-width: 430px
        @media (max-width: 600px)
            max-width: 100%
        .list
            position: absolute
            padding: 10px 20px
            width: 100%
        .item
            background: #FFFFFF
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2)
            border-radius: 25px
            padding: 14px 0 0
            margin-bottom: 20px
            position: relative
            overflow: hidden
            .holder
                overflow-y: auto
                max-height: 150px
                padding-right: 14px
            .content
                display: flex
                padding-bottom: 14px
            .text
                display: flex
                flex-direction: column
                justify-content: center
                padding-left: 22px
                line-height: 16px
                min-height: 58px
                max-width: 250px
                word-wrap: break-word
                div
                    margin-bottom: 10px
            h2
                font-size: 14px
                padding-bottom: 4px
                font-weight: 600
            .style
                width: 90px
                min-width: 90px
                box-shadow: .5px 0 0 $color-shadow-border
                background-position: center center
                background-repeat: no-repeat
            .close-button
                top: 23px
        @media print
            display: none

    .progress-timer
        width: 100%
        height: 3px
        margin-top: -3px
        border-radius: 2px

    .bar
        height: 100%
        border-radius: 3px
        background: #aaa
        animation: progress 15s linear
        width: 200%
        margin-left: -100%

    .warning
        .style
            background-image: url("../icons/notification/warning.svg")
        .progress-timer .bar
            background: $color-brand

    .information
        .style
            background-image: url("../icons/notification/information.svg")
        .progress-timer .bar
            background: #3E90F2

    .success
        .style
            background-image: url("../icons/notification/success.svg")
        .progress-timer .bar
            background: #5CC873

@keyframes progress
    from
        transform: scaleX(1)
    to
        transform: scaleX(0)

```

### src/core/routes.js
```json
import React from "react";
import { Switch } from "react-router-dom";
import Route from "./misc/route";

import accommodationTitle                 from "pages/accommodation/title";
import accommodationSearchResults         from "pages/accommodation/search-results/search-results";
import accommodationBooking               from "pages/accommodation/booking/booking";
import accommodationContractsSets         from "pages/accommodation/room-contract-sets/room-contract-sets";

import accommodationConfirmation          from "pages/accommodation/booking-confirmation";
import accommodationViewBooking           from "pages/accommodation/view-booking-page";
import accommodationConfirmationInvoice   from "pages/accommodation/confirmation/invoice";
import accommodationConfirmationVoucher   from "pages/accommodation/confirmation/voucher";

import paymentPage                        from "pages/payment/payment-page";
import paymentResultFirst                 from "pages/payment/result-first";
import paymentResultSecond                from "pages/payment/result-second";
import paymentDirectLink                  from "pages/payment/external/direct-link-page";
import paymentDirectLinkConfirm           from "pages/payment/external/direct-link-confirmation";

import registrationAgent                  from "pages/account/registration-agent";
import registrationCounterparty           from "pages/account/registration-counterparty";
import acceptInvite                       from "pages/account/accept-invite";

import bookingsManagement                 from "pages/agent/bookings-management/bookings-management";
import accountStatement                   from "pages/agent/account-statement/account-statement";
import invitationsManagement              from "pages/settings/invitations";
import invitationSend                     from "pages/settings/invitation-send";
import invitationResend                   from "pages/settings/invitation-resend";

import childAgencyInvitation              from "pages/settings/child-agencies/invitation";
import childAgencyObserve                 from "pages/settings/child-agencies/observe";
import childAgencyItem                    from "pages/settings/child-agencies/child-agency";

import agentsManagement                   from "pages/settings/agents-management";
import agentPermissionsManagement         from "pages/settings/agent-permissions-management";

import personalSettings                   from "pages/settings/personal-settings";
import counterpartySettings               from "pages/settings/counterparty-settings";

import contactUsPage                      from "pages/common/contact";
import termsPage                          from "pages/common/terms";
import privacyPage                        from "pages/common/privacy";
import aboutUsPage                        from "pages/common/about";

import NotFoundPage                       from "pages/common/not-found-page";

export const routesWithHeaderAndFooter = [
    "/",
    "/search",
    "/search/contract",
    "/accommodation/booking",
    "/booking*",
    "/accommodation/confirmation",
    "/contact", "/terms", "/privacy", "/about",
    "/payment/form",
    "/settings*",
];

export const routesWithFooter = [
    ...routesWithHeaderAndFooter,
    "/pay/*",
    "/signup/*"
];

export const routesWithSearch = [
    "/search",
    "/search/contract"
];


const Routes = () => (
<Switch>
    <Route exact path="/"                           component={accommodationTitle} />
    <Route exact path="/search"                     component={accommodationSearchResults} title="Search Results" />
    <Route exact path="/search/contract"            component={accommodationContractsSets} title="Select An Accommodation" />
    <Route path="/accommodation/booking"            component={accommodationBooking} title="Accommodation Booking" />
    <Route path="/booking/:id/invoice"              component={accommodationConfirmationInvoice} title="Invoice" />
    <Route path="/booking/:id/voucher"              component={accommodationConfirmationVoucher} title="Voucher" />
    <Route path="/accommodation/confirmation"       component={accommodationConfirmation} title="Your Booking Confirmation" />
    <Route path="/booking/:code"                    component={accommodationViewBooking} title="Booking" />

    <Route path="/payment/form"                     component={paymentPage} title="Payment" />
    <Route path={[
                "/payment/result/:ref",
                "/payment/result"
                ]}                                  component={paymentResultFirst} title="Processing" />
    <Route path="/payments/callback"                component={paymentResultSecond} title="Processing" />

    <Route path="/pay/confirmation"                 component={paymentDirectLinkConfirm} title="Confirmation" />
    <Route path="/pay/:code"                        component={paymentDirectLink} />

    <Route path="/signup/agent"                     component={registrationAgent} title="Sign Up" />
    <Route path="/signup/counterparty"              component={registrationCounterparty} title="Sign Up" />
    <Route path="/signup/invite/:email/:code"       component={acceptInvite} title="Sign Up" />

    <Route path="/bookings"                         component={bookingsManagement} title="Bookings" />
    <Route path="/settings/account"                 component={accountStatement} title="Account Statement" />
    <Route path="/settings/invitations/send"        component={invitationSend} title="Invite an Agent" />
    <Route path="/settings/invitations/:id"         component={invitationResend} title="Invitation" />
    <Route path="/settings/invitations"             component={invitationsManagement} title="Invitations" />
    <Route exact path="/settings/child-agencies"    component={childAgencyObserve} title="Observe Child Agency" />
    <Route path="/settings/child-agencies/invite"   component={childAgencyInvitation} title="Invite Child Agency" />
    <Route path="/settings/child-agencies/:id"      component={childAgencyItem} title="Child Agency" />
    <Route path="/settings/agents/:agentId/"        component={agentPermissionsManagement} title="Agent Permissions" />
    <Route path="/settings/agents"                  component={agentsManagement} title="Agent Management" />
    <Route path="/settings/counterparty"            component={counterpartySettings} title="Counterparty Settings" />
    <Route exact path="/settings"                   component={personalSettings} title="Personal Settings" />

    <Route path="/contact"                          component={contactUsPage} title="Contacts" />
    <Route path="/terms"                            component={termsPage} title="Terms & Conditions" />
    <Route path="/privacy"                          component={privacyPage} title="Privacy Policy" />
    <Route path="/about"                            component={aboutUsPage} title="About Us" />

    <Route component={NotFoundPage} />
</Switch>
);

export default Routes;

```

### src/pages/account/registration-agent.js
```json
import React from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { API, redirect } from "core";
import NotFoundPage from "pages/common/not-found-page";
import BasicHeader from "parts/header/basic-header";
import { authSetToStorage } from "core/auth";
import { getInvite, forgetInvite } from "core/auth/invite";
import { Loader } from "components/simple";
import Breadcrumbs from "components/breadcrumbs";
import { CachedForm, FieldText, FORM_NAMES } from "components/form";
import { registrationAgentValidator, registrationAgentValidatorWithEmailAndAgencyName } from "components/form/validation";
import { fillEmptyAgentSettings } from "simple/logic";
import FormAgentData from "parts/form-agent-data";
import { $personal, $notifications, $ui } from "stores";

export const finishAgentRegistration = (after) => {
    API.get({
        url: API.AGENT,
        success: (agent) => {
            authSetToStorage(agent);
            if (agent?.email) {
                $personal.setInformation(agent);
                $notifications.addNotification("Registration Completed!", "Great!", "success");
            }
            fillEmptyAgentSettings();
            forgetInvite();
            $ui.dropFormCache(FORM_NAMES.RegistrationAgentForm);
            $ui.dropFormCache(FORM_NAMES.RegistrationCounterpartyForm);
            $personal.$setRegistrationAgentForm({});
            $personal.setRegistrationCounterpartyForm({});
        },
        after
    });
};

@observer
class RegistrationAgent extends React.Component {
    state = {
        initialValues: {
            "title": "",
            "firstName": "",
            "lastName": "",
            "position": "",
            "agencyName": ""
        },
        loading: false,
        childAgencyRegistrationInfo: {},
        invitationCode: null
    };

    submit = (values) => {
        $personal.$setRegistrationAgentForm(values);
        if (this.state.invitationCode) {
            this.setState({ loading: true });
            let url = API.AGENT_REGISTER;
            let body = {
                registrationInfo: {
                    ...values,
                    email: this.state.initialValues.email
                },
                invitationCode: this.state.invitationCode
            };
            if (this.state.childAgencyRegistrationInfo.name) {
                url = API.AGENCY_REGISTER;
                body = {
                    registrationInfo: {
                        ...values,
                        email: this.state.initialValues.email
                    },
                    childAgencyRegistrationInfo: {
                        ...this.state.childAgencyRegistrationInfo,
                        name: values.agencyName
                    },
                    invitationCode: this.state.invitationCode
                }
            }
            API.post({
                url,
                body,
                success: () => {
                    finishAgentRegistration(() => this.setState({ loading: false}));
                    redirect("/");
                },
                error: error => {
                    $notifications.addNotification(error?.title || error?.detail);
                    this.setState({ loading: false});
                }
            });
        } else
            redirect("/signup/counterparty");
    };

    componentDidMount() {
        var invitationCode = getInvite();
        if (invitationCode)
            API.get({
                url: API.INVITATION_DATA(invitationCode),
                success: data => {
                    let initialValues = data?.userRegistrationInfo;
                    if (data?.childAgencyRegistrationInfo?.name)
                        initialValues = {
                            ...data.userRegistrationInfo,
                            agencyName: data.childAgencyRegistrationInfo.name
                        };
                    this.setState({
                        initialValues,
                        invitationCode: invitationCode,
                        childAgencyRegistrationInfo: data?.childAgencyRegistrationInfo
                    });
                }
            });
    }

    render() {
        let { t } = useTranslation();

        if ($personal.information?.email)
            return <NotFoundPage />;

        const { childAgencyRegistrationInfo, invitationCode, initialValues } = this.state;

        return (

<div className="account block" style={{ backgroundImage: `url(/images/bg04.svg)`}}>
    <BasicHeader />
    { this.state.loading && <Loader page /> }
    <section className="section">
        <div>
            <Breadcrumbs
                items={[
                    {
                        text: t("Sign In"),
                        link: "/logout"
                    }, {
                        text: t("Registration"),
                        link: "/logout"
                    }, {
                        text: t("Agent Information")
                    }
                ]}
                 noBackButton
            />
            <h2>
                Agent Information
            </h2>
            <div className="paragraph">
                Create a new Happytravel.com account and start booking.
            </div>

            <CachedForm
                id={FORM_NAMES.RegistrationAgentForm}
                initialValues={initialValues}
                enableReinitialize
                validationSchema={
                    childAgencyRegistrationInfo.name ?
                        registrationAgentValidatorWithEmailAndAgencyName :
                        registrationAgentValidator
                }
                onSubmit={this.submit}
                render={formik => (
                    <div className="form">
                        { childAgencyRegistrationInfo.name ? <div className="row">
                            <FieldText
                                formik={formik}
                                id="agencyName"
                                label={t("Agency Name")}
                                placeholder={t("Agency Name")}
                                required
                            />
                        </div> : null }
                        <FormAgentData formik={formik} />
                        <div className="row">
                            <div className="field">
                                <div className="inner">
                                    <button type="submit" className="button main">
                                        { invitationCode ?
                                            t("Finish Registration") :
                                            t("Continue Registration")}
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                )}
            />
        </div>
    </section>
</div>
        );
    }
}

export default RegistrationAgent;

```

### src/pages/account/registration-counterparty.js
```json
import React, { useState } from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { Link } from "react-router-dom";
import BasicHeader from "parts/header/basic-header";
import { API, redirect } from "core";
import NotFoundPage from "pages/common/not-found-page";
import { PAYMENT_METHODS } from "enum";
import { finishAgentRegistration } from "./registration-agent";
import { Loader } from "components/simple";
import Breadcrumbs from "components/breadcrumbs";
import { CachedForm, FORM_NAMES, FieldText, FieldTextarea, FieldSelect } from "components/form";
import FieldCountry from "components/complex/field-country";
import { registrationCounterpartyValidator } from "components/form/validation";
import { $personal, $notifications, $ui } from "stores";

const RegistrationCounterparty = observer(() => {
    const [loading, setLoading] = useState(false);

    const submit = (values) => {
        setLoading(true);
        $personal.setRegistrationCounterpartyForm(values);
        API.post({
            url: API.AGENT_REGISTER_MASTER,
            body: $personal.registration,
            success: () => {
                finishAgentRegistration(() => setLoading(false));
                redirect("/");
            },
            error: () => setLoading(false)
        });
    };

    const { t } = useTranslation();

    if ($personal.information?.email)
        return <NotFoundPage />;

    return (
        <div className="account block" style={{ backgroundImage: `url(/images/bg04.svg)`}}>
            <BasicHeader />
            { loading && <Loader page /> }
            <section className="section">
                <div>
                    <Breadcrumbs
                        items={[
                            {
                                text: t("Sign In"),
                                link: "/logout"
                            }, {
                                text: t("Registration"),
                                link: "/logout"
                            }, {
                                text: t("Company Information")
                            }
                        ]}
                        noBackButton
                    />
                    <h2>
                        Company Information
                    </h2>
                    <div className="paragraph">
                        Create a free Happytravel.com account and start booking today<br/>
                        Already have an account? <Link to="/logout" className="link">Log In Here</Link>
                    </div>

                    <CachedForm
                        id={ FORM_NAMES.RegistrationCounterpartyForm }
                        initialValues={{
                            "name": "",
                            "address": "",
                            "legalAddress": "",
                            "country": "",
                            "countryCode": "",
                            "city": "",
                            "phone": "",
                            "fax": "",
                            "preferredPaymentMethod": "",
                            "website": "",
                            "postalCode": ""
                        }}
                        validationSchema={registrationCounterpartyValidator}
                        onSubmit={submit}
                        render={formik => (
                            <div className="form">
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id="name"
                                        label={t("Company Name")}
                                        placeholder={t("Company Name")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldTextarea
                                        formik={formik}
                                        id="address"
                                        label={t("Company Address")}
                                        placeholder={t("Company Address")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldTextarea
                                        formik={formik}
                                        id="legalAddress"
                                        label={t("Legal Address")}
                                        placeholder={t("Legal Address")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id={"postalCode"}
                                        label={t("Zip/Postal Code")}
                                        placeholder={t("Zip/Postal Code")}
                                    />
                                </div>
                                <div className="row">
                                    <FieldCountry
                                        formik={formik}
                                        id="country"
                                        label={t("Country")}
                                        placeholder={t("Country")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id="city"
                                        label={t("City")}
                                        placeholder={t("City")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldSelect
                                        formik={formik}
                                        id="preferredPaymentMethod"
                                        label={t("Preferred Payment Method")}
                                        required
                                        placeholder={t("Preferred Payment Method")}
                                        options={[
                                            { value: PAYMENT_METHODS.ACCOUNT, text: "Virtual Credit"},
                                            { value: PAYMENT_METHODS.CARD, text: "Credit Card"},
                                            { value: PAYMENT_METHODS.OFFLINE, text: <>Offline <em>(Bank transfer or cash)</em></> }
                                        ]}
                                    />
                                </div>
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id="phone"
                                        label={t("Phone")}
                                        placeholder={t("Phone")}
                                        required
                                    />
                                </div>
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id="fax"
                                        label={t("Fax")}
                                        placeholder={t("Fax")}
                                    />
                                </div>
                                <div className="row">
                                    <FieldText
                                        formik={formik}
                                        id="website"
                                        label={t("Website")}
                                        placeholder={t("Website")}
                                    />
                                </div>
                                <div className="row">
                                    <div className="field">
                                        <div className="inner">
                                            <button type="submit" className={"main button" + __class(!formik.isValid, "disabled")}>
                                                {t("Get started")}
                                            </button>
                                        </div>
                                    </div>
                                </div>
                                <div className="paragraph" style={{ paddingLeft: 0 }}>
                                    By clicking this button, you agree with <Link to="/terms" className="link">HappyTravel’s Terms of Use</Link>
                                </div>
                            </div>
                        )}
                    />
                </div>
            </section>
        </div>
    );
});

export default RegistrationCounterparty;

```

### src/pages/accommodation/title.js
```json
import React from "react";
import Tiles from 'components/tiles';
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { Loader } from "components/simple";
import Search from "parts/search-form/search-form";
import { $personal } from "stores";

const AccommodationTitlePage = observer(({ noSearch }) => {
    const { t } = useTranslation();

    if (!$personal.information?.email) // workaround for loader within registration process
        return <Loader white page />;

    return (
    <>
        { !noSearch &&
            <div className="search-fullsize-wrapper">
                <Search fullsize />
            </div>
        }
        <div className="tiles block">
            <section>
                <h2>{t("Countries & Hotels")}</h2>
                <Tiles list={[
                    {
                        city: 'PARIS, FRANCE',
                        flag: 'fr',
                        propertiesCount: '2831',
                        image: '/images/hotels/france.jpg'
                    },
                    {
                        city: 'LONDON, ENGLAND',
                        flag: 'gb',
                        propertiesCount: '3399',
                        image: '/images/hotels/london.jpg'
                    }
                ]} />
                <Tiles list={[
                    {
                        city: 'ROME, ITALY',
                        flag: 'it',
                        propertiesCount: '5965',
                        image: '/images/hotels/rome.jpg'
                    },
                    {
                        city: 'BARCELONA, SPAIN',
                        flag: 'es',
                        propertiesCount: '1770',
                        image: '/images/hotels/barcelona.jpg'
                    },
                    {
                        city: 'DUBAI, United Arab Emirates',
                        flag: 'ae',
                        propertiesCount: '949',
                        image: '/images/hotels/dubai.jpg'
                    }
                ]} />
                <h2>{t("Exclusive offers")}</h2>
                <Tiles list={[
                    {
                        title: 'EMERALD PALACE KEMPINSKI DUBAI, DUBAI',
                        flag: 'ae',
                        image: '/images/hotels/emeraldplace.jpg'
                    },
                    {
                        title: 'HILTON BAKU, BAKU',
                        flag: 'az',
                        image: '/images/hotels/hilton.jpg'
                    },
                    {
                        title: 'KEMPINSKI HOTEL MALL OF THE EMIRATES, DUBAI',
                        flag: 'ae',
                        image: '/images/hotels/kempinski.jpg'
                    },
                    {
                        title: 'PULLMAN DUBAI CREEK CITY CENTRE HOTEL, DUBAI',
                        flag: 'ae',
                        image: '/images/hotels/pullman.jpg'
                    }
                ]} />
            </section>
        </div>
    </>
    );
});

export default AccommodationTitlePage;

```

### styles/block/search.sass
```json
.search.block .fullsize
    padding: 70px 0 110px
    display: flex
    justify-content: center
    & > .row
        margin-bottom: 0
    .field
        margin-top: 36px
        &.size-one
            width: 300px
            max-width: 300px
        &.size-two
            width: 350px
            max-width: 350px
    .button-holder
        margin-top: 11px
        .button
            font-size: 15px
    @include phone
        & > .row
            flex-direction: column
        .field
            width: 100% !important
            max-width: 100% !important
            margin-top: 15px
    @include mobile
        padding: 40px 0

@include except-phone
    .search-fullsize-wrapper
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1)

```

### src/translation/arabic.js
```json
export default {
translations: {
    "current_language_name": "اللغة الحالية",
    "Transfers": "الانتقالات",
    "Tours": "الجولات",
    "About": "نبذة عن",
    "FAQ": "الأسئلة الشائعة",
    "Log out": "تسجيل الخروج",
    "Terms & Conditions": "شروط وأحكام الاستخدام",
    "Contacts": "تواصل معنا",
    "Email": "البريد الإلكتروني",
    "Phone": "الهاتف",
    "Address": "العنوان",
    "_copyright": "حقوق الملكية",
    "Are you supplier?": "هل أنت متعهد خدمات؟",
    "Registration": "التسجيل",
    "Password": "كلمة المرور",
    "Don`t have an account?": "هل لديك حساب؟",
    "Login Information": "معلومات تسجيل الدخول",
    "Agent Information": "معلومات المستخدم",
    "Company Information": "معلومات الشركة",
    "Already have an account?": "هل لديك حساباً؟",
    "Log In Here": "لتسجيل الدخول اضغط هنا",
    "Password complexity": "درجة تعقيد كلمة المرور",
    "One lowercase character": "حرف واحد من الحروف الصغيرة",
    "One uppercase character": "حرف واحد من الحروف الكبيرة",
    "One number": "رقم واحد",
    "Generate Password": "إنشاء كلمة المرور",
    "Continue Registration": "الاستمرار في التسجيل",
    "Salutation": "التحية",
    "Position (Designation)": "الموقع/التعيين",
    "Company Name": "اسم الشركة",
    "Company Address": "عنوان الشركة",
    "Zip/Postal Code": "الرمز البريدي",
    "Select Country": "اختر البلد",
    "Preferred Payment Method": "طريقة الدفع المفضلة",
    "Fax": "رقم الفاكس",
    "Prefix": " رمز الهاتف الدولي",
    "City Code": "رمز المدينة",
    "Number": "الرقم",
    "Website": "الموقع الإلكتروني",
    "Enter Website Address": "ادخل عنوان الموقع الإلكتروني",
    "Page not found": "هذه الصفحة غير موجودة",
    "Verify your account": "إثبات ملكية حسابك",
    "Session expired, please refresh the web page": "صلاحية جلستك قد انتهت، يرجى تحديث الصفحة",
    "Back": "العودة للوراء",
    "Title": "العنوان",
    "First Name": "الاسم الأول",
    "Last Name": "اسم العائلة",
    "Click Here": "اضغط هنا",
    "Country": "البلد",
    "Enter your E-mail Address": "ادخل البريد الإلكتروني الخاص بك",
    "Destination, Hotel Name, Location or Landmark": "الوجهة أو اسم الفندق أو الموقع أو المعلم البارز المراد زيارته",
    "Destination or Hotel Name": "اختر وجهتك أو اسم الفندق أو الموقع أو الموقع البارز المراد زيارته",
    "Choose your Destination": "اختر وجهتك",
    "Check-in - Check-out": "تسجيل الوصول - المغادرة",
    "Dates": "اختر التاريخ",
    "Adults, Children, Rooms": "البالغون، الأطفال، الغرف",
    "Choose options": "ضع الخيارات المناسبة لك",
    "Residency": "الإقامة",
    "Choose your residency": "اختر محل إقامتك",
    "Nationality": "الجنسية",
    "Choose your nationality": "اختر جنسيتك",
    "Clear": "مسح",
    "Adult": "البالغ",
    "Adult_plural": "البالغون",
    "Children": "الأطفال",
    "Room": "الغرفة",
    "Room_plural": "الغرف",
    "Star Rating": "تصنيف عدد النجوم",
    "Availability": "الإتاحة",
    "All Bookings": "الجميع",
    "Hotel": "الفندق",
    "Serviced Apartment": "سكن ذو خدمات",
    "Economy": "اقتصادية",
    "Budget": "ميزانية منخفضة",
    "Standard": "قياسية",
    "Superior": "خدمات أكثر",
    "Luxury": "فاخرة",
    "Unrated": "غير مصنفة",
    "Map": "خريطة",
    "Price Range": "نطاق السعر",
    "Drag the slider to choose minimum and maximum prices": "قم بسحب شريط التمرير لتحديد قيمة الحد الأدنى والأقصى للسعر",
    "Property Type": "نوع السكن",
    "Preferred": "المفضل",
    "5 stars": "5 نجوم",
    "Room Only": "غرف فقط",
    "Breakfast": "الإفطار",
    "Rate Type": "نوع التصنيف",
    "Flexible": "مرن",
    "Leisure & Sport": "الترفيه & الرياضة",
    "Business Features": "الخدمات المتاحة لرجال الأعمال",
    "Loading...": "تحميل...",
    "Accommodations in": "الإقامة في",
    "Sort by": "التصنيف حسب",
    "From": "من ",
    "Location": "الموقع",
    "more...": "المزيد...",
    "Included Services": "الخدمات المشمولة",
    "Actions": "الإجراءات",
    "Total Price": "السعر الإجمالي",
    "None": "لا يوجد",
    "Within deadline": "في إطار الموعد النهائي",
    "Book now": "احجز الآن",
    "Date": "التاريخ",
    "Price": "السعر",
    "Room Available": "الغرف المتاحة",
    "Show all rooms": "اظهر جميع الغرف",
    "Search Accommodations": "البحث عن أماكن إقامة",
    "Guest Details": "تفاصيل الضيف",
    "Booking confirmation": "تأكيد الحجز",
    "Agent Reference": "مرجع الوكيل",
    "Extra Meal": "وجبة إضافية",
    "Special Request": "طلب خاص",
    "Your Requests": "طلباتك",
    "Mr.": "السيد.",
    "Mrs.": "السيدة.",
    "Child": "طفل",
    "Reservation Summary": "حجزك",
    "Arrival Date": "تاريخ الوصول",
    "Departure Date": "تاريخ المغادرة",
    "Number of Rooms": "عدد الغرف",
    "Room Information": "معلومات الغرفة",
    "Room Type": "نوع الغرفة",
    "Board Basis": "نوع الإقامة",
    "Occupancy": "حالة الإشغال",
    "Room & Total Cost": "الغرفة & التكلفة الإجمالية",
    "Cost": "تكلفة الغرفة",
    "Total Cost": "التكلفة الإجمالية",
    "Booking Details": "تفاصيل الحجز",
    "Leading Passenger": "قائد المركبة",
    "Booking Reference number": "الرقم المرجعي للحجز",
    "Status": "الحالة",
    "Booking Date": "تاريخ الحجز",
    "Cancellation Deadline": "الموعد النهائي للإلغاء",
    "Booked Service": "الخدمات المحجوزة",
    "Passenger Name": "اسم الراكب",
    "Country of Residence": "بلد الإقامة",
    "Check-in": "تسجيل الوصول",
    "Check-out": "المغادرة",
    "Room Occupancy": "حالة إشغال الغرفة",
    "Additional": "إضافي",
    "None Selected": "لم يتم تحديد أي خيار",
    "Accept & reconfirm": "القبول & إعادة التأكيد",
    "Additional Information": "المعلومات الإضافية",
    "Business Features & Amenities": "الخدمات ووسائل الترفيهة الإضافية لرجال الأعمال ",
    "Transportation & Directions To The Hotel": "الانتقال والاتجاهات إلى الفندق",
    "4 stars": "4 نجوم",
    "3 stars": "3 نجوم",
    "2 stars": "نجمتين",
    "1 star": "نجمة واحدة",
    "footer_address_line_5": "Dubai, United Arab Emirates",
    "Sign In": "تسجيل الدخول",
    "Get started with a new account": "ابدأ بحساب جديد",
    "Confirm Booking": "تأكيد الحجز",
    "Please enter here": "من فضلك ادخل هنا",
    "Enter First Name": "الرجاء إدخال الاسم الأول",
    "Enter Last Name": "الرجاء إدخال اسم العائلة",
    "Booking Confirmation": "تأكيد الحجز",
    "About Us": "معلومات عنا",
    "Your Booking": "حجزك",
    "Cancel": "إلغاء",
    "Pay now by Card": "ادفع الآن عن طريق البطاقة",
    "Payment": "دفع",
    "Confirm": "تأكيد",
    "Pay": "دفع",
    "Deadline": "الموعد النهائي",
    "Card Number": "رقم البطاقة",
    "Save my card for faster checkout": "حفظ بطاقتي لسرعة الخروج",
    "Expiration Date": "تاريخ إنتهاء الصلاحية",
    "Card Holder Name": "إسم صاحب البطاقة",
    "Error": "خطأ",
    "An error occured": "حدث خطأ",
    "Try again": "حاول مرة أخرى",
    "Select Payment Method": "يرجى اختيار طريقة الدفع",
    "Please Enter Your Card Details": "الرجاء إدخال تفاصيل بطاقتك",
    "My Site Balance": "ميزان الموقع الخاص بي",
    "Credit or Debit Card": "بطاقة الائتمان / الخصم",
    "Accommodations": "الإقامة",
    "Visas": "التأشيرات",
    "Privacy Policy": "سياسة الخصوصية",
    "Create a new one": "إنشاء واحدة جديدة",
    "Create a Happytravel.com account and start booking.": "قم بإنشاء حساب Happytravel.com وابدأ الحجز",
    "10 characters minimum": "10 أحرف كحد أدنى",
    "Available Accommodations": "أماكن الإقامة المتاحة",
    "Accommodation Amenities": "وسائل الراحة ",
    "Accommodation Chain": "سلسلة الإقامة",
    "Ms.": "الآنسة",
    "Miss": "الآنسة",
    "Accommodation Details": "تفاصيل السكن",
    "Not Rated": "غير مصنف",
    "Finish Registration": "إنهاء التسجيل",
    "Please enter children ages": "الرجاء إدخال سن الأطفال",
    "year": "العام",
    "year_plural": "السنة",
    "More than": "أكثر من",
    "properties": "الخصائص",
    "Exclusive offer": "عرض حصري",
    "Countries & Hotels": "البلد والفنادق",
    "Exclusive offers": "العروض الحصرية",
    "Remark": "تعليق",
    "You don`t have any reservations": "ليس لديك أي حجوزات",
    "Payment message": "رسالة الدفع",
    "Response code": "رمز الاستجابة",
    "Payment Result": "نتيجة الدفع",
    "Card acceptance message": "رسالة قبول البطاقة",
    "Your card was saved for your future purchases.": "تم حفظ بطاقتك لعمليات الشراء المستقبلية",
    "Decline": "رفض",
    "cancellation costs": "الإلغاء يكلفك",
    "of total amount": "من المبلغ الإجمالي",
    "Cancel booking": "إلغاء الحجز",
    "Number of floors": "عدد الطوابق",
    "Number of rooms": "عدد الغرف",
    "Bar": "بار",
    "Laundry Service": "خدمة غسيل الملابس",
    "Car Parking - Onsite Free": "مواقف السيارات - مجانا في الموقع ",
    "Elevators": "المصاعد",
    "Multilingual Staff": "موظفين متعددي اللغات",
    "Safety Deposit Box": "صندوق الأمانات",
    "Leisure & Sport Amenities": "الترفيه والرياضة المرافق",
    "Conference Rooms": "غرف المؤتمرات",
    "Meeting Rooms": "غرف الإجتماعات",
    "No Information": "لا يوجد معلومات",
    "Create PDF": "إنشاء PDF",
    "You are about to cancel your booking": "أنت على وشك إلغاء الحجز",
    "has passed. A cancellation fee will be charged according to accommodation's cancellation policy.": "اجتاز. سيتم فرض رسوم إلغاء وفقًا لسياسة إلغاء الإقامة",
    "Are you sure you want to cancel your booking?": "هل أنت متأكد تريد إلغاء الحجز؟"
}
};

```

### src/parts/form-agent-data.js
```json
import React from "react";
import { useTranslation } from "react-i18next";
import { FieldText, FieldSelect } from "components/form";

export default ({
    formik,
}) => {
    const { t } = useTranslation();
    return (
        <>
            <div className="row">
                <FieldSelect formik={formik}
                    id="title"
                    label={t("Salutation")}
                    required
                    placeholder={t("Select")}
                    options={[
                        { value: "Mr", text: t("Mr.")},
                        { value: "Ms", text: t("Ms.")},
                        { value: "Miss", text: t("Miss")},
                        { value: "Mrs", text: t("Mrs.")}
                    ]}
                />
            </div>
            <div className="row">
                <FieldText formik={formik}
                    id="firstName"
                    label={t("First Name")}
                    placeholder={t("First Name")}
                    required
                />
            </div>
            <div className="row">
                <FieldText formik={formik}
                    id="lastName"
                    label={t("Last Name")}
                    placeholder={t("Last Name")}
                    required
                />
            </div>
            <div className="row">
                <FieldText formik={formik}
                    id="position"
                    label={t("Position (Designation)")}
                    placeholder={t("Position (Designation)")}
                    required
                />
            </div>
        </>
    );
};

```

### styles/odawara/identity.sass
```json
@import "../global.sass"
@import "../main.sass"
@import "../block/account"
@import "password-validator"

.account.block
    .label.with-actions
        display: flex
        justify-content: space-between
        padding-right: 27px
        .link
            color: $color-blue
            border-bottom: 0
        @include phone
            span:last-child
                display: none
    .error-field
        color: $color-red
        line-height: 19px
        ul
            padding-left: 24px
            li
                list-style-type: disc
                margin-bottom: 4px
                &:first-child
                    padding-top: 9px
                &:last-child
                    padding-bottom: 9px
    .auth-input
        input, textarea
            border-radius: 25px
            color: $color-text
            background: #ffffff
            border: 1px solid $color-border
            height: 54px
            line-height: 16px
            padding: 0 27px
            position: relative
            display: flex
            &:focus
                border-color: $color-brand
        textarea
            width: 100%
            height: 120px
            padding-top: 20px
            resize: none
            font-size: 11px
            &::placeholder
                font-size: 14px

```

### styles/block/tiles.sass
```json
.tiles.block
    background: #ffffff
    padding-bottom: 100px

    .tile-row
        display: flex
        margin-bottom: 20px
        .bottom
            background: linear-gradient(0, #000000 0%, rgba(0, 0, 0, 0) 83.33%)
            width: 100%
            position: absolute
            bottom: 0
            left: 0
            height: 118px
            z-index: 2
        &.x-2
            .item
                height: 300px
        &.x-3
            .item
                height: 300px
                .bottom
                    height: 165px
        &.x-4
            .item
                height: 220px
                &.offer
                    margin-bottom: 63px
                .info
                    padding: 14px 14px 24px
                    .title
                        font-size: 14px
                        line-height: 17px
                        min-height: 34px
        .item
            background: #e5e5e5
            margin-right: 20px
            flex-grow: 1
            position: relative
            min-height: 50px
            border-radius: 50px
            overflow: hidden
            &:last-child
                margin-right: 0
            .body, .picture
                width: 100%
                height: 100%
                position: absolute
                top: 0
                left: 0
                z-index: 3
            .picture
                z-index: 1
                object-fit: cover
            .body
                background: rgba(0,0,0,.1)
                color: #ffffff
            .flag
                width: 18px
                height: 12px

            .info
                position: absolute
                display: flex
                padding: 30px 30px 18px
                bottom: 0
                left: 0
                .flag
                    margin-right: 10px
                    line-height: 21px

            .title
                font-size: 18px
                line-height: 21px
                font-weight: 500
            .count
                font-size: 14px
                line-height: 27px
            .price
                font-size: 18px
                position: absolute
                bottom: 27px
                right: 30px
                line-height: 18px
                font-weight: 500
                span
                    font-size: 10px
    h2
        margin: 47px 0 30px

@include phone
    .tiles.block
        display: none
        padding-left: 20px
        padding-right: 20px
        .tile-row
            flex-direction: column
            margin-bottom: 0
            .item
                margin-right: 0
                margin-bottom: 20px

```

### src/components/form/index.js
```json
import CachedForm from "./cached-form";
import FieldText from "./field-text";
import FieldSwitch from "./field-switch";
import FieldCheckbox from "./field-checkbox";
import FieldTextarea from "./field-textarea";
import FieldRange from "./field-range";
import FieldSelect from "./field-select/field-select";
import FieldDatepicker from "./field-datepicker/field-datepicker";

const FORM_NAMES = {
    CreateInviteForm: "CreateInviteForm",
    RegistrationAgentForm: "RegistrationAgentForm",
    RegistrationCounterpartyForm: "RegistrationCounterpartyForm",
    SearchForm: "SearchForm",
    BookingForm: "BookingForm",
    SendInvoiceForm: "SendInvoiceForm"
};

export {
    CachedForm,
    FieldText,
    FieldSwitch,
    FieldCheckbox,
    FieldTextarea,
    FieldRange,
    FieldSelect,
    FieldDatepicker,

    FORM_NAMES
};

```

### src/pages/common/not-found-page.js
```json
import React, { useEffect } from "react";
import { Link } from "react-router-dom";
import BasicHeader from "parts/header/basic-header";
import { useTranslation } from "react-i18next";

const NotFoundPage = () => {
    const { t } = useTranslation();

    useEffect(() => {
        document.querySelectorAll("header, footer").forEach(
            item => item ? item.style.display = "none" : null
        );
        setTimeout(() => {
            document.title = "Happytravel.com";
        }, 0);
        return () => {
            document.querySelectorAll("header, footer").forEach(
                item => item ? item.style.display = "block" : null
            );
        }
    }, []);

    return (
        <div className="error-page block">
            <BasicHeader />
            <section>
                <div>
                    <div className="picture">
                        <div className="text">
                            <h1>404</h1>
                            <h2>{t("Page not found")}</h2>
                        </div>
                    </div>

                    <Link to="/">
                        <span className="button">
                            {t("Back to") + " " + t("Homepage")}
                        </span>
                    </Link>
                </div>
            </section>
        </div>
    );
};

export default NotFoundPage;
```

### src/translation/english.js
```json
import React from "react";

export default {
translations: {
    "current_language_name": "English",
    "Accommodations": "Accommodations",
    "Transfers": "Transfers",
    "Tours": "Tours",
    "Visas": "Visas",
    "About": "About",
    "Bookings": "Bookings",
    "FAQ": "FAQ",
    "Log out": "Log out",
    "Terms & Conditions": "Terms & Conditions",
    "Privacy Policy": "Privacy Policy",
    "Contacts": "Contacts",
    "Email": "Email",
    "Phone": "Phone",
    "Address": "Address",
    "_copyright": "All Rights Reserved",
    "Are you supplier?": "Are you a supplier?",
    "Sign In": "Sign In",
    "Registration": "Registration",
    "Password": "Password",
    "Don`t have an account?": "Don`t have an account?",
    "Create a new one": "Create a new one",
    "Login Information": "Login Information",
    "Agent Information": "Agent Information",
    "Company Information": "Company Information",
    "Get started with a new account": "Get started with a new account",
    "Create a Happytravel.com account and start booking.": "Create a Happytravel.com account and start booking.",
    "Already have an account?": "Already have an account?",
    "Log In Here": "Log In Here",
    "Password complexity": "Password complexity",
    "One lowercase character": "One lowercase character",
    "One uppercase character": "One uppercase character",
    "One number": "One number",
    "10 characters minimum": "10 characters minimum",
    "Generate Password": "Generate Password",
    "Continue Registration": "Continue Registration",
    "Salutation": "Salutation",
    "Position (Designation)": "Position (Designation)",
    "Company Name": "Company Name",
    "Company Address": "Company Address",
    "Zip/Postal Code": "Zip/Postal Code",
    "Select Country": "Select Country",
    "Preferred Payment Method": "Preferred Payment Method",
    "Company account currency": "Company account currency",
    "Fax": "Fax",
    "Prefix": "Prefix",
    "City Code": "City Code",
    "Number": "Number",
    "Website": "Website",
    "Enter Website Address": "Enter Website Address",
    "Page not found": "Page not found",
    "Verify your account": "Verify your account",
    "Back": "Back",
    "Back to": "Back to",
    "Room Selection": "Room Selection",
    "Homepage": "Homepage",
    "Booking Management": "Booking Management",
    "Bookings List": "Bookings List",
    "Saved Cards": "Saved Cards",
    "Title": "Title",
    "First Name": "First Name",
    "Last Name": "Last Name",
    "Click Here": "Click Here",
    "Country": "Country",
    "Enter your E-mail Address": "Enter your E-mail Address",
    "Destination, Hotel Name, Location or Landmark": "Destination, Hotel Name, Location or Landmark",
    "Destination or Hotel Name": "Destination or Hotel Name",
    "Destination": "Destination",
    "Choose your Destination": "Choose your Destination",
    "Check-in - Check-out": "Check-in – Check-out",
    "Dates": "Dates",
    "Adults, Children, Rooms": "Guests • Rooms",
    "Choose options": "Choose options",
    "Residency": "Residency",
    "Choose your residency": "Choose your residency",
    "Nationality": "Nationality",
    "Choose your nationality": "Choose your nationality",
    "Clear": "Clear",
    "Adult": "Adult",
    "Adult_plural": "Adults",
    "Guest": "Guest",
    "Guest_plural": "Guests",
    "Children": "Children",
    "Room": "Room",
    "Room_plural": "Rooms",
    "Star Rating": "Star Rating",
    "Availability": "Availability",
    "All Bookings": "All Bookings",
    "Available Accommodations": "Available Accommodations",
    "Hotel": "Hotel",
    "Serviced Apartment": "Serviced Apartment",
    "Economy": "Economy",
    "Budget": "Budget",
    "Standard": "Standard",
    "Superior": "Superior",
    "Luxury": "Luxury",
    "Unrated": "Unrated",
    "Map": "Map",
    "Price Range": "Price Range",
    "Drag the slider to choose minimum and maximum prices": "Drag the slider to choose minimum and maximum prices",
    "Property Type": "Property Type",
    "Preferred": "Preferred",
    "Breakfast": "Breakfast",
    "Rate Type": "Rate Type",
    "Flexible": "Flexible",
    "Accommodation Amenities": "Accommodation Amenities",
    "Business Features": "Business Features",
    "Accommodation Chain": "Accommodation Chain",
    "Loading...": "Loading...",
    "Can't find what you're looking for?": "Can't find what you're looking for?",
    "You could reach our Operations team directly, and we pick an accommodation for you.": "You can contact our Operations team directly, and they can help you with your accommodations.",
    "Accommodations in": "Accommodations in",
    "Sort by": "Sort by",
    "Accommodation Name": "Accommodation Name",
    "Mark as duplicate": "Mark as duplicate",
    "Marked as Duplicate": "Marked as Duplicate",
    "From": "From",
    "Location": "Location",
    "more...": "more...",
    "Included Services": "Included Services",
    "Actions": "Actions",
    "Total Price": "Total Price",
    "None": "None",
    "Unsorted": "Unsorted",
    "Within deadline": "Within deadline",
    "Book now": "Book now",
    "Date": "Date",
    "Price": "Price",
    "Room Available": "Room Available",
    "Show all rooms": "Show all rooms",
    "Search Accommodations": "Search Accommodations",
    "Guest Details": "Guest Details",
    "Booking confirmation": "Booking confirmation",
    "Agent Reference": "Agent Reference",
    "Extra Meal": "Extra Meal",
    "Special Request": "Special Request",
    "Your Requests": "Your Requests",
    "Mr.": "Mr.",
    "Ms.": "Ms.",
    "Miss": "Miss",
    "Mrs.": "Mrs.",
    "Child": "Child",
    "Reservation Summary": "Reservation Summary",
    "Arrival Date": "Arrival Date",
    "Departure Date": "Departure Date",
    "Number of Rooms": "Number of Rooms",
    "Room Information": "Room Information",
    "Room Type": "Room Type",
    "Board Basis": "Board Basis",
    "Occupancy": "Occupancy",
    "Room & Total Cost": "Room & Total Cost",
    "Cost": "Cost",
    "Total Cost": "Total Cost",
    "Booking Details": "Booking Details",
    "Leading Passenger": "Leading Passenger",
    "Booking Reference number": "Booking Reference number",
    "Status": "Status",
    "Booking Date": "Booking Date",
    "Cancellation Deadline": "Cancellation Deadline",
    "Booked Service": "Booked Service",
    "Passenger Name": "Passenger Name",
    "Country of Residence": "Country of Residence",
    "Check-in": "Check-in",
    "Check-out": "Check-out",
    "Room Occupancy": "Room Occupancy",
    "Rate Basis": "Rate Basis",
    "Additional": "Additional",
    "None Selected": "None Selected",
    "Accept & reconfirm": "Accept & reconfirm",
    "Accommodation Details": "Accommodation Details",
    "Additional Information": "Additional Information",
    "Business Features & Amenities": "Business Features & Amenities",
    "Transportation & Directions To The Hotel": "Transportation & Directions to Hotel",
    "5 stars": "5 stars",
    "4 stars": "4 stars",
    "3 stars": "3 stars",
    "2 stars": "2 stars",
    "1 star": "1 star",
    "Not Rated": "Not Rated",
    "About Us": "About Us",
    "Booking management": "Booking management",
    "Your Booking": "Your Booking",
    "Cancel": "Cancel",
    "Booking Confirmation": "Booking Confirmation",
    "Pay now by Card": "Pay now by Card",
    "Payment": "Payment",
    "Confirm": "Confirm",
    "Decline": "Decline",
    "Pay": "Pay",
    "Card Number": "Card Number",
    "Save my card for faster checkout": "Save my card for faster checkout",
    "Expiration Date": "Expiration Date",
    "Card Holder Name": "Cardholder Name",
    "Deadline": "Deadline",
    "Please enter here": "Please enter here",
    "Confirm Booking": "Confirm Booking",
    "Enter First Name": "Enter First Name",
    "Enter Last Name": "Enter Last Name",
    "Error": "Error",
    "An error occured": "An error occurred",
    "Try again": "Try again",
    "Select Payment Method": "Select Payment Method",
    "Please Enter Your Card Details": "Please Enter Your Card Details",
    "My Site Balance": "My Site Balance",
    "Credit or Debit Card": "Credit or Debit Card",
    "Finish Registration": "Finish Registration",
    "Please enter children ages": "Please enter children's ages",
    "year": "year",
    "year_plural": "years",
    "More than": "More than",
    "properties": "properties",
    "Exclusive offer": "Exclusive offer",
    "Countries & Hotels": "Countries & Hotels",
    "Exclusive offers": "Exclusive offers",
    "Remark": "Remark",
    "You don`t have any reservations": "You don't have a reservation",
    "You don`t have any payment history for this dates": "You don't have any payment history for these dates",
    "Payment message": "Payment message",
    "Response code": "Response code",
    "Payment Result": "Payment Result",
    "Card acceptance message": "Card acceptance message",
    "Your card was saved for your future purchases.": "Your card was saved for your future purchases.",
    "cancellation costs": "cancellation costs",
    "of total amount": "of total amount",
    "Cancel booking": "Cancel booking",
    "Number of floors": "Number of floors",
    "Number of rooms": "Number of rooms",
    "Conference Rooms": "Conference Rooms",
    "Meeting Rooms": "Meeting Rooms",
    "No Information": "No Information",
    "Create PDF": "Create PDF",
    "You are about to cancel your booking": "You are about to cancel your booking",
    "has passed. A cancellation fee will be charged according to accommodation's cancellation policy.": "has passed. A cancellation fee will be charged according to your accommodation's cancellation policy.",
    "Are you sure you want to cancel your booking?": "Are you sure you want to cancel your booking?",
    "option": "option",
    "option_plural": "options",
    "Night": "Night",
    "Night_plural": "Nights",
    "FREE Cancellation - Without Prepayment": "FREE Cancellation - Without Prepayment",
    "Room Availability": "Room Availability",
    "Check-in From": "Check-in From",
    "Check-in Date": "Check-in Date",
    "Check-out Date": "Check-out Date",
    "View Other Options": "View Other Options",
    "Pros": "Pros",
    "Book": "Book",
    "Similar accommodations": "Similar accommodations",
    "Elevators": "Elevators",
    "Multilingual Staff": "Multilingual Staff",
    "Safety Deposit Box": "Safety Deposit Box",
    "Leisure & Sport Amenities": "Leisure & Sports Amenities",
    "Future": "Future",
    "Complete": "Complete",
    "Cancelled": "Cancelled",
    "Send Voucher": "Send Voucher",
    "Send Invoice": "Send Invoice",
    "Voucher": "Voucher",
    "Invoice": "Invoice",
    "Enter email to receive information about booking": "Enter email to receive information about booking",
    "Booking voucher has been sent": "Booking voucher has been sent",
    "Booking invoice has been sent": "Booking invoice has been sent",
    "Itinerary number": "Itinerary number",
    "Enter Search Text": "Enter Search Text",
    "Search": "Search",
    "Send Invitation": "Send Invitation",
    "Invitations": "Invitations",
    "Invite an agent": "Invite an agent",
    "Your invitation sent": "Your invitation sent",
    "Your invitation sent to": "Your invitation sent to",
    "Send one more invite": "Send one more invite",
    "Invite someone to create a free Happytravel.com account and start booking today": "Invite someone to create a free Happytravel.com account and start booking today",
    "Accommodation": "Accommodation",
    "Account statement": "Account statement",
    "No reservations found": "No reservations found",
    "Created": "Created",
    "Amount": "Amount",
    "Reason": "Reason",
    "Please enter itinerary number": "Please enter itinerary number",
    "I have read and accepted": "I have read and accepted",
    "City": "City",
    "Get started": "Get started",
    "Dynamic offer": "Dynamic offer",
    "Please enter": "Please enter",
    "out of": "out of",
    "available": "available",
    "No accommodations available": "No accommodations available",
    "Account balance": "Account balance",
    "Would you like an exclusive offer?": "Would you like an exclusive offer?",
    "Generate Invitation Link": "Generate Invitation Link",
    "Send this link as an invitation": "Send this link as an invitation",
    "Security code on your credit card": "Security code on your credit card",
    "Your information is secure; only a part of you card's data will be stored": "Your information is secure; only a part of you card's data will be stored",
    "Pay using saved cards": "Pay using saved cards",
    "Use another card": "Use another card",
    "using saved card": "using saved card",
    "Reset filters": "Reset filters",
    "No Breakfast": "No Breakfast",
    "RoomOnly": "No Breakfast",
    "SelfCatering": "Self-catering",
    "BedAndBreakfast": "Bed and Breakfast",
    "HalfBoard": "Half Board",
    "FullBoard": "Full Board",
    "AllInclusive": "All-inclusive",
    "Settings": "Settings",
    "Not provided": "Not provided",
    "Default": "Default",
    "Contact": "Contact",
    "Room Details": "Room Details",
    "Single": "Single",
    "TwinOrSingle": "Twin/Single",
    "Twin": "Twin",
    "Double": "Double",
    "Triple": "Triple",
    "Quadruple": "Quadruple",
    "Family": "Family Room",
    "Oops! Something went wrong!": "Oops! Something went wrong!",
    "Payment failed": "Payment failed",
    "Order has been paid successfully": "Order has been paid successfully",
    "Try to pay again": "Try to pay again",
    "Your payment did not go through": "Your payment did not go through",
    "Accommodation Information": "Accommodation Information",
    "Unable to load a booking confirmation": "Unable to load a booking confirmation",
    "Please note the booking price has changed": "Please note the booking price has changed",
    "To speed up a search on a large number of accommodations, we use preloaded data. Sometimes the data may become outdated while you work with the site. When this happens, you may see a change in price or in cancellation policies on this screen. The last shown price is final.": "To speed up a search on a large number of accommodations, we use preloaded data. Sometimes the data may become outdated while you work with the site. When this happens, you may see a change in price or in cancellation policies on this screen. The last shown price is final.",
    "High to Low": "High to Low",
    "Low to High": "Low to High",
    "Creation Date": "Creation Date",
    "Creation (old first)": "Creation (old first)",
    "Reference code": "Reference code",
    "Event Type": "Event Type",
    "Payment Status": "Payment Status",
    "At least": "At least",
    "Cancellation cost": "Cancellation cost",
    "Print": "Print",
    "Overview": "Overview",
    "Rooms": "Rooms",
    "Information": "Information",
    "You were looking for": "You were looking for",
    "Personalization": "Personalization",
    "Select": "Select",
    "Residency similar as Nationality": "Residency similar as Nationality",
    "There are no results that match selected filters": "There are no results that match selected filters",
    "Filters": "Filters",
    "Clear Filters": "Clear Filters",
    "Stars": "Stars",
    "Choose option": "Choose option",
    "Show Only My Bookings": "Show Only My Bookings",
    "Restricted Rate": "Restricted Rate",
    "Direct Connectivity": "Direct Connectivity",
    "per Night": "per Night",
    "Personal Information": "Personal Information",
    "Legal Information": "Legal Information",
    "All Agents": "All Agents",
    "You have not filled guests information": "You have not filled guests information",
    "Please Select": "Please Select",
    "Legal Address": "Legal Address",
}
};

```

### src/pages/settings/invitation-send.js
```json
import React from "react";
import { observer } from "mobx-react";
import { useTranslation } from "react-i18next";
import { API } from "core";
import { Loader } from "components/simple";
import { copyToClipboard } from "simple/logic";
import { CachedForm, FORM_NAMES, FieldText } from "components/form";
import { registrationAgentValidatorWithEmail } from "components/form/validation";
import FormAgentData from "parts/form-agent-data";
import SettingsHeader from "pages/settings/parts/settings-header";
import { $ui, $personal, $notifications } from "stores";

@observer
class InvitationSendPage extends React.Component {
    state = {
        success: false,
        form: null
    };

    submit = (values) => {
        this.setState({ success: null });
        API.post({
            url: values.send ? API.AGENT_INVITE_SEND : API.AGENT_INVITE_GENERATE,
            body: {
                email: values.email,
                agencyId: $personal.activeCounterparty.agencyId,
                registrationInfo: {
                    firstName: values.firstName,
                    lastName: values.lastName,
                    position: values.position,
                    title: values.title
                }
            },
            success: data => {
                $ui.dropFormCache(FORM_NAMES.CreateInviteForm);
                this.setState({
                    success:
                        (values.send || !data) ?
                        true :
                        window.location.origin + "/signup/invite/" + values.email + "/" + data,
                    name: (values.firstName || values.lastName) ? (values.firstName + " " + values.lastName) : null
                });
            },
            error: (error) => {
                this.setState({ success: false });
                $notifications.addNotification(error?.title || error?.detail);
            }
        });
    };

    reset = () => {
        this.setState({ success: false });
    };

    submitButtonClick(send, formik) {
        formik.setFieldValue("send", send);
        formik.handleSubmit();
    }

    render() {
        var { t } = useTranslation();

        return (
    <div className="settings block">
        <SettingsHeader />
        <section>
            <h2>{t("Invite an agent")}</h2>
            { this.state.success === null && <Loader /> }
            { this.state.success && <div>
                {this.state.success === true ?
                <div>
                    { this.state.name ?
                        <h3>{t("Your invitation sent to")} {this.state.name}</h3> :
                        <h3>{t("Your invitation sent")}</h3> }
                    <br/>
                </div> :
                <div>
                    <div className="form">
                        <h3>{t("Send this link as an invitation")}</h3>
                        <br/>
                        <FieldText
                            value={this.state.success}
                        />
                    </div>
                    <br/>
                    <button className="button" style={{ marginBottom: 20 }} onClick={() => copyToClipboard(this.state.success)}>
                        {t("Copy to Clipboard")}
                    </button>
                </div>}
                <button className="button" onClick={this.reset}>
                    {t("Send one more invite")}
                </button>
            </div> }
            { false === this.state.success && <p>
                {t("Invite someone to create a free Happytravel.com account and start booking today")}<br/>
                <br/>
            </p> }

            { false === this.state.success && <CachedForm
                id={FORM_NAMES.CreateInviteForm}
                initialValues={{
                    "email": "",
                    "title": "",
                    "firstName": "",
                    "lastName": "",
                    "position": ""
                }}
                validationSchema={registrationAgentValidatorWithEmail}
                onSubmit={this.submit}
                render={formik => (
                    <div className="form">
                        <div className="row">
                            <FieldText formik={formik}
                                id="email"
                                label={t("Email")}
                                placeholder={t("Email")}
                                required
                            />
                        </div>
                        <FormAgentData formik={formik} />
                        <div className="row">
                            <div className="field" style={{ width: "50%" }}>
                                <div className="inner">
                                    <button onClick={() => this.submitButtonClick(true, formik)}
                                            className={"button" + __class(!formik.isValid, "disabled")}>
                                        {t("Send Invitation")}
                                    </button>
                                </div>
                            </div>
                            <div className="field" style={{ width: "50%" }}>
                                <div className="inner">
                                    <button onClick={() => this.submitButtonClick(false, formik)}
                                            className={"button" + __class(!formik.isValid, "disabled")}>
                                        {t("Generate Invitation Link")}
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                )}
            /> }
        </section>
    </div>
        );
    }
}

export default InvitationSendPage;
```

### styles/components/button.sass
```json
button
    cursor: pointer
    display: inline-block
    min-width: 0
    min-height: 0
    outline: 0
    border: 0
    background: transparent
    font: inherit

.button
    display: inline-flex
    height: 54px
    line-height: 20px
    font-size: 13px
    color: $color-text
    border: 1px solid $color-border
    border-radius: 30px
    cursor: pointer
    background: #ffffff
    padding: 3px 26px
    justify-content: center
    flex-direction: column
    align-items: center

    &:hover
        color: #FF9D19
        border-color: #FF9D19

    &.main
        background: $color-brand
        border: 0
        color: #ffffff
        background: linear-gradient(90deg, #FFCD0B 0%, #FBAC19 100%)
        &:hover
            background: #FF9D19
            background: linear-gradient(90deg, #FF9D19 0%, #F9A51B 100%)


    &.gray
        color: #b0b0b0
        &:hover
            color: $color-brand

    &.green
        color: $color-green
        border-color: $color-green
        &:hover
            border-color: $color-brand

    &.transparent
        text-transform: none
        line-height: 30px
        height: 30px
        border: 1px solid #ffffff
        background: transparent
        color: #ffffff
        padding: 0 15px
        font-weight: 400
        &:hover
            color: #FF9D19
            background: #ffffff

    &.round
        border-radius: 300px !important
        padding: 0 !important
        min-width: 26px
        min-height: 26px
        .icon
            vertical-align: middle

    &.small
        height: auto
        line-height: 14px
        font-size: 10px
        padding: 6px 18px
        font-weight: 500

    &.mini-label
        font-size: 9px
        height: 20px
        line-height: 20px
        cursor: default
        padding: 0 8px
        display: inline-block
        font-weight: 500

    &.disabled, &.disabled:hover, &.disabled:active
        cursor: default !important
        background: #f2f2f2
        border: 0
        color: #aaa
        &.main
            color: #fff
            background: #d9d9d9

.file-upload
    font-size: 14px
    padding: 0 30px
    input
        display: none

```

### styles/block/error-page.sass
```json
.error-page
    text-align: center
    .picture
        margin: 0 auto
        position: relative
        width: 840px
        height: 500px
        background: url("/images/other/service.png") center center no-repeat
        background-size: contain
        .text
            color: #FFFFFF
            height: 100%
            display: flex
            flex-direction: column
            justify-content: center
            padding: 125px 20px 0 0
            h1, h2
                font-weight: 700
            h1
                font-size: 64px
                line-height: 76px
                padding: 0
            h2
                font-size: 13px
                line-height: 16px
    .button
        margin-top: 70px
        min-width: 180px

    &.inside
        margin-top: 115px
        h1
            font-size: 30px
            line-height: 36px
        p
            font-size: 14px
            line-height: 24px
            color: $color-dark-gray
            margin-top: 20px
        .button
            margin-top: 45px
```

### styles/block/settings.sass
```json
.settings
    section
        max-width: 886px
        padding-bottom: 150px
    h2
        margin: 70px 0 40px
        font-size: 20px
        line-height: 24px
        color: #231F20
    .row
        margin-bottom: 18px
    .form
        .row.controls
            padding: 0 !important
            margin-top: 9px
            margin-bottom: 0
            justify-content: flex-end
            .button
                min-width: 215px
            .field
                flex-grow: 0
                width: auto
            @include phone
                .field
                    width: 100%
        &.agent-data
            flex-wrap: wrap
            flex-direction: row
            @include phone
                flex-direction: column
            @include except-phone
                .row:nth-child(1)
                    order: 1
                    padding-right: 9px
                .row:nth-child(2)
                    order: 3
                    padding-right: 9px
                .row:nth-child(3)
                    order: 4
                    padding-left: 9px
                .row:nth-child(4)
                    order: 2
                    padding-left: 9px
                .row:nth-child(5)
                    order: 5
                .row:last-child
                    width: 100%
                .row
                    width: 50%
        &.app-settings
            .row:nth-child(3)
                margin-top: 30px
            @include except-phone
                .field
                    width: 50%
            @include phone
                .double
                    flex-direction: column
                    margin-bottom: 0
                    .field
                        margin-right: 0
                        margin-bottom: 20px
        .permissions
            flex-wrap: wrap
            display: flex
            .row
                width: 50%
    .search-wrapper
        padding: 73px 0 82px
        max-width: 700px
        .field:last-child
            max-width: 330px
    .breadcrumbs
        margin: 45px 0 -25px
    &.navigation
        section
            padding-bottom: 0

.settings-header
    section
        padding: 75px 10px
        .agent
            display: flex
            color: $color-text-light
            .photo
                width: 100px
                height: 100px
                position: relative
                border-radius: 80px
                margin: 0 30px 0 15px
                overflow: hidden
                .no-avatar, img
                    width: 100%
                    height: 100%
                .no-avatar
                    background: url("../images/no-avatar.svg") center center no-repeat
                    background-size: cover
            .data
                display: flex
                flex-direction: column
                justify-content: center
                h1
                    font-size: 24px
                    line-height: 28px
                    font-weight: 500
                    color: $color-text
                .company
                    font-weight: 500
                .status
                    width: 14px
                    height: 14px
                    display: inline-block
                    margin-right: 7px
                    vertical-align: bottom
                    border-radius: 25px
                    &.PendingVerification
                        background: lightyellow
                        border: 3px solid yellow
                    &.FullAccess
                        background: $color-green
                        border: 3px solid #bbecd2
                    &.DeclinedVerification
                        background: red
                        border: 3px solid lightcoral
                    &.ReadOnly
                        background: blue
                        border: 3px solid skyblue
                div
                    margin-top: 10px
                    span
                        color: $color-green
    .logout-wrapper
        float: right
        .button
            display: block
            height: 34px
            line-height: 30px
            padding: 0 16px 0 10px
            font-weight: 400
            .icon
                margin: -3px 6px 0 0
                vertical-align: middle
                filter: grayscale(1)
            &:hover
                .icon
                    filter: none
    .settings-nav
        box-shadow: 0 .5px 0 $color-shadow-border
        section
            padding: 0
            display: flex
        a
            font-size: 13px
            line-height: 16px
            text-align: center
            color: $color-text-light
            margin: 0 10px
            padding: 19px 14px
            position: relative
            font-weight: 500
            &:hover, &.active
                color: $color-text
            &.active
                cursor: default
                &:after
                    content: ' '
                    height: 4px
                    width: 100%
                    background: linear-gradient(90deg, #F9A51B 0%, #FECE08 100%)
                    border-radius: 25px
                    position: absolute
                    bottom: 0
                    left: 0

.voucher-image
    margin-bottom: 60px
    .box
        margin: 10px 0
    img
        max-width: 500px
        max-height: 300px

```

### styles/odawara/password-validator.sass
```json
.password-rules
    padding: 0 27px
    color: $color-text-light
    font-weight: 300
    .ok-lowercase .rule-lowercase,
    .ok-uppercase .rule-uppercase,
    .ok-number .rule-number,
    .ok-length .rule-length,
    .ok-symbol .rule-symbol
        font-weight: 400 !important
        span
            color: $color-text
            font-size: 0
            &:after
                content: "\2713"
                font-size: 14px
                color: $color-green
    span
        width: 20px
        display: inline-block
        text-align: center

    .columns
        display: flex
        flex-wrap: wrap
        justify-content: space-between
        div
            width: 230px
            height: 20px

    .title
        margin-bottom: 18px
        color: #797F89

    .caption-wrapper
        overflow: hidden
        position: relative
        background: #D8DADC
        margin: 17px 0 15px
        height: 5px
        width: 100%

    .caption
        height: 5px
        border-right: 2px solid #f5fafb
        box-sizing: content-box
        width: 0
        background: #FE3824
        transition: width .3s linear, background-color .15s ease-in-out

    .label span
        cursor: default
        width: 12px
        text-align: center

    .title:after
        content: " None"
    .total-1, .total-2, .total-3, .total-4
        .title:after
            content: " Weak"

    .total-1 .caption
        width: 20%
        background: #FE3824

    .total-2 .caption
        width: 40%
        background: #FE3824

    .total-3 .caption
        width: 60%
        background: #FF9F00

    .total-4 .caption
        width: 80%
        background: #FF9F00

    .total-5
        .title:after
            content: " Strong"
            color: $color-green
            font-weight: 500
        .caption
            background: #23D578
            width: 100%

```

### styles/main.sass
```json
@keyframes Appear
    0%
        opacity: 0
    100%
        opacity: 1

body
    background: #ffffff
    color: $color-text
    letter-spacing: 0.232143px
    line-height: 16px
    font:
        family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", "Arial Neue", sans-serif
        size: 13px

textarea, pre, input, button
    font-family: inherit
    letter-spacing: inherit

.link, a.link, a[href].link
    cursor: pointer
    color: $color-blue
    &:hover
        color: #074BC5
    &.underlined
        text-decoration: underline

h1
    font-size: 30px
    line-height: 36px

h2
    font-size: 24px
    line-height: 29px
    font-weight: 400
    span
        color: $color-text-light

section
    max-width: 1290px
    width: 100%
    margin: 0 auto
    padding: 0 30px
    @include mobile
        padding: 0 20px
    @include phone
        padding: 0 10px

.hide
    display: none !important

.dual
    display: flex
    width: 100%
    justify-content: space-between
    .second
        text-align: right
    &.column
        flex-direction: column
        .second
            text-align: left

b, strong
    font-weight: 500

.block-wrapper
    min-height: 100vh
    padding-top: 81px
    header
        margin-top: -81px
        position: static
        box-shadow: none !important

.hide-desktop
    display: none
.hide-mobile
    display: block

@include mobile
    .with-search
        .block-wrapper
            padding-top: 161px
    .hide-desktop
        display: block
    .hide-mobile
        display: none

```

### styles/form/field.sass
```json
.form .field
    min-width: 88px
    position: relative
    flex-grow: 1
    margin-right: 18px
    &:last-child
        margin-right: 0
    .label
        text-transform: uppercase
        line-height: 16px
        font-size: 11px
        color: $color-text-light
        height: 24px
        margin-bottom: -13px
        margin-left: 24px
        border-radius: 10px
        z-index: 1
        position: relative
        display: block
        font-weight: 600
        span
            text-overflow: ellipsis
            white-space: nowrap
            max-width: 85%
            overflow: hidden
            cursor: pointer
            padding: 4px
            background: #ffffff
            display: inline-block
    &.error
        .label
            color: $color-red
    &.focus
        & > label > .input
            border-color: $color-brand
        & > label > .label
            color: $color-brand
        .icon-arrow-expand
            filter: none
    &.disabled
        .input
            background: #f5f9fA
            input
                color: $color-text
        .label
            span
                cursor: default
                background: linear-gradient(0, rgba(245,248,250,0) 25%, #fff 50%, rgba(255,255,255,0) 57%)
                border-radius: 28px

    &.no-input
        cursor: pointer
        label
            cursor: pointer
            .inner
                white-space: nowrap
                .placeholder
                    color: $color-text-light

    input, .button
        width: 100%

    &.size-medium
        max-width: 273px

    &.size-half
        width: 50%

    .error-holder
        color: $color-red
        margin: 5px 0 0 17px

    &.select
        & input
            position: absolute !important
        &, & label, & input
            cursor: pointer

    .value-object
        width: 100%
        height: 100%
        overflow-y: hidden
        &.placeholder
            color: $color-text-light

    .input
        border-radius: 30px
        color: $color-text
        background: #ffffff
        border: 1px solid $color-border
        line-height: 52px
        padding: 0 27px
        position: relative
        display: flex
        font-size: 14px
        em
            font-style: normal
            color: $color-text-light
        .inner
            position: relative
            display: inline-block
            flex-grow: 2
        .suggestion
            position: absolute
            white-space: nowrap
            left: 0
            top: 0
            z-index: 1
            font-size: 14px
            color: $color-text-light
            &.solid
                color: $color-text
            span
                color: #ffffff

        input[type="text"], input[type="password"], textarea
            border: 0
            width: 100%
            font-size: 14px
            background: transparent
            position: relative
            z-index: 3
            &::placeholder
                color: $color-text-light
                opacity: 1

        &.textarea
            height: auto
            min-height: 80px
            .inner
                padding-top: 12px
                padding-bottom: 6px
                overflow: hidden
                textarea
                    width: 100% !important
                    resize: none
                    overflow: hidden
                    min-height: 40px
                    height: 60px
                    max-height: 200px
                    display: block
                    line-height: 20px
            .placeholder
                color: $color-text-light

        .clear
            background: url("../images/cross-small.svg") no-repeat
            width: 8px
            height: 8px
            margin-left: 14px
            display: inline-block
            cursor: pointer

        .icon-wrap
            margin-right: 8px
        .after-icon-wrap
            padding-left: 14px
            cursor: pointer

.never-submitted
    .possible-hide
        display: none

span.required
    &:after
        content: ' *'
        color: red

```

### styles/block/account.sass
```json
.account.block
    margin-top: -80px
    width: 100%
    height: 100%
    min-height: 100vh
    position: relative
    background: #74B1F5 center bottom no-repeat fixed
    background-size: cover
    display: flex
    flex-direction: column
    justify-content: center
    align-items: center
    .small
        max-width: 550px
        .section
            animation: Appear .33s ease-in-out 1
    header
        background: linear-gradient(180deg, rgba(255,255,255,.7), rgba(255,255,255,0.55) 36%, rgba(255,255,255,0.38) 61%, rgba(255,255,255,0))
        box-shadow: none !important
        height: 300px
        position: absolute
        top: 0
        left: 0
        right: 0
        margin-top: 0
        z-index: 1
        .logo-wrapper
            width: 100%
            transform: none
            min-width: 0
            max-width: none
            justify-content: space-around

    section.section
        position: relative
        z-index: 2 !important
        margin: 100px 0
        .link
            border-bottom: 1px solid

    .section
        max-width: 700px
        padding: 20px
        display: flex
        flex-direction: row
        align-items: center
        justify-content: center
        animation: Appear .15s ease-in-out 1
        & > div
            background: #ffffff
            border-radius: 50px
            padding: 50px
            min-height: 480px
            display: flex
            flex-direction: column
            justify-content: center
            box-shadow: 0 0 5px 1px rgba(0, 0, 0, 0.2)
            flex-grow: 1
    .label
        span
            background: linear-gradient(180deg, rgba(255,255,255,0) 43%, rgba(255,255,255,255) 46%, rgba(255,255,255,255) 51%, rgba(255,255,255,0) 54%) !important
    .button
        margin-top: 15px
        font-size: 16px
    .breadcrumbs, h2, .paragraph
        padding-left: 27px
    .breadcrumbs
        margin: 9px 0 7px
    .form
        .row:last-child
            margin-bottom: 0
    h2
        line-height: 34px
    h2, .accent-frame
        margin-bottom: 12px
    .paragraph
        margin: 0 0 30px
        line-height: 22px
        color: $color-text-light
        .link
            color: $color-text
            &:hover
                color: $color-brand
        &.action
            margin: 0 27px 13px 0
            text-align: right
            .link
                border-bottom-color: #fff
                &:hover
                    border-bottom-color: inherit
        & + .accent-frame
            margin-top: -10px
        & + .form
            margin-top: 50px

```

**Updated on: 4/23/2021 8:41:33 AM**
