# Amazon Location Service

**TL;DR** — Managed maps, places search, routing, geofences, and asset tracking. Powered by Esri, HERE, Grab, Open Data. Cheap alternative to Google Maps / Mapbox if you're already on AWS.

## What it is

A bundle of geospatial APIs:
- **Maps** — vector + raster tiles, multiple cartographic styles.
- **Places** — geocoding (address → lat/lng), reverse geocoding, autosuggest, search around point/bbox.
- **Routes** — point-to-point routing, ETA, distance matrix, truck/car/walk modes, traffic-aware.
- **Geofences** — define polygons; get events when devices enter/exit.
- **Trackers** — store device positions over time; query history.
- All wrapped with IAM, Cognito unauth tokens, and API keys.

## Why it exists

Google Maps Platform and Mapbox dominate, but:
- They bill in their own systems (separate vendor management).
- Their pricing escalates fast at scale.
- For AWS-native apps, Location Service integrates with IAM, EventBridge, IoT Core, etc.

It's matured a lot since 2021 — usable for most mainstream map/routing needs today.

## Data providers

You pick a provider per resource (Map / Place index / Route calculator):
- **Esri** — strongest in NA / EU.
- **HERE** — global, strong in EU + Asia, including **Bangladesh**.
- **Grab** — best in SEA (Singapore, Indonesia, Malaysia, Thailand, Vietnam, Philippines, Cambodia, Myanmar).
- **Open Data (Maps only)** — OSM-based tiles, free-ish.

**For Bangladesh apps:** HERE is the practical default. Verify coverage and address quality before committing.

## Key concepts

- **Map resource** — tile source + style.
- **Place index** — backs geocoding/search APIs.
- **Route calculator** — backs routing APIs.
- **Geofence collection** — polygons + circles.
- **Tracker** — stores device positions; can link to geofence collections to emit events.
- **API key** — short-lived public keys for unauth front-end use (the modern way).
- **Pricing modes:** Request-based (default) or Mobile Asset Tracking (per asset, predictable for fleet apps).

## Real-world example

> **RouteMate** (your social ride-sharing app, Dhaka corridor pilot):
>
> - Map resource provider = HERE, style = `VectorHereExplore`.
> - Place index for "search a destination" autosuggest.
> - Route calculator: car mode, traffic-aware.
> - Geofence collection: pickup zones around major intersections.
> - Tracker stores driver positions every 10s; geofence "enter" events go to EventBridge → Lambda → notify rider "your ride is here."

## Usage

### Create resources (CLI)

```bash
aws location create-map \
  --map-name rm-map \
  --configuration Style=VectorHereExplore \
  --pricing-plan RequestBasedUsage

aws location create-place-index \
  --index-name rm-places \
  --data-source Here \
  --data-source-configuration IntendedUse=SingleUse

aws location create-route-calculator \
  --calculator-name rm-routes \
  --data-source Here

aws location create-geofence-collection \
  --collection-name rm-pickups

aws location create-tracker \
  --tracker-name rm-drivers
```

### Geocode an address (Node SDK v3)

```js
import { LocationClient, SearchPlaceIndexForTextCommand } from "@aws-sdk/client-location";
const loc = new LocationClient({ region: "ap-south-1" });

const { Results } = await loc.send(new SearchPlaceIndexForTextCommand({
  IndexName: "rm-places",
  Text: "Gulshan 2, Dhaka",
  MaxResults: 5,
  BiasPosition: [90.4125, 23.8103],  // [lng, lat] — searches near here
}));
console.log(Results[0].Place.Geometry.Point);  // [lng, lat]
```

### Calculate a route

```js
import { CalculateRouteCommand } from "@aws-sdk/client-location";
const { Summary, Legs } = await loc.send(new CalculateRouteCommand({
  CalculatorName: "rm-routes",
  DeparturePosition: [90.4125, 23.8103],   // Gulshan
  DestinationPosition: [90.3742, 23.7806], // Banani
  TravelMode: "Car",
  DepartureTime: new Date(),
}));
console.log(`${Summary.Distance} ${Summary.DistanceUnit}, ETA ${Summary.DurationSeconds}s`);
```

### Update device position

```js
import { BatchUpdateDevicePositionCommand } from "@aws-sdk/client-location";
await loc.send(new BatchUpdateDevicePositionCommand({
  TrackerName: "rm-drivers",
  Updates: [{
    DeviceId: "driver-42",
    Position: [90.4125, 23.8103],
    SampleTime: new Date(),
  }],
}));
```

### Render a map in the browser

The recommended path is **MapLibre GL JS** (open source fork of Mapbox GL) with the AWS Location SDK auth helper:

```js
import maplibregl from "maplibre-gl";
import { withIdentityPoolId } from "@aws/amazon-location-utilities-auth-helper";

const authHelper = await withIdentityPoolId("ap-south-1:abcd-...");
const map = new maplibregl.Map({
  container: "map",
  style: "https://maps.geo.ap-south-1.amazonaws.com/maps/v0/maps/rm-map/style-descriptor",
  center: [90.4125, 23.8103],
  zoom: 11,
  ...authHelper.getMapAuthenticationOptions(),
});
```

Or use an API key:
```
https://maps.geo.ap-south-1.amazonaws.com/maps/v0/maps/rm-map/style-descriptor?key=<api-key>
```

### Geofence + EventBridge

Link a tracker to a geofence collection; AWS emits events to EventBridge on enter/exit:

```json
{
  "source": ["aws.geo"],
  "detail-type": ["Location Geofence Event"],
  "detail": { "EventType": ["ENTER", "EXIT"] }
}
```

Route to a Lambda that pushes a notification, updates ride state, etc.

## Pricing

(All prices approximate, ap-south-1, request-based usage.)

- **Map tiles:** $0.04 per 1,000 tile requests.
- **Geocoding (SearchPlaceIndexForText):** $0.50 per 1,000.
- **Reverse geocoding:** $0.50 per 1,000.
- **Routes:** $0.50 per 1,000 simple route calculations; matrix more.
- **Tracker writes:** $0.50 per 100,000 position updates.
- **Tracker reads:** $0.50 per 100,000 reads.
- **Geofence evaluations:** included with tracker updates.
- **Stored API requests** (cache results to avoid re-paying): cheaper for some place ops.

Free tier (12 months): generous tile + geocoding allowances. Hobby projects often stay free.

## Location Service vs Google Maps vs Mapbox

| | Amazon Location | Google Maps | Mapbox |
|---|---|---|---|
| Map quality | Very good (HERE/Esri) | Best | Very good |
| Geocoding accuracy | Good (varies by provider) | Best globally | Good |
| Routing | Good | Excellent (live traffic everywhere) | Good |
| AWS-native (IAM, EB) | ✅ | No | No |
| Pricing | Cheaper at scale | Pricier | Mid |
| Free tier | 12 months generous | $200 monthly credit | Pay-as-you-go w/ free tier |

**Pick Google Maps** when map UX + traffic precision is product-critical (consumer ride-sharing, navigation apps competing on accuracy).
**Pick Amazon Location** when you're AWS-heavy, costs matter, and "good enough" maps are fine (B2B asset tracking, internal tools, light consumer use).

## Gotchas

- **Provider matters more than you think.** Test addresses in your target country (e.g., Dhaka) — HERE is best for BD, but verify your specific neighborhoods.
- **"IntendedUse" gotcha** — `SingleUse` is required for place-index results you don't store; `Storage` mode is needed if you cache results, but pricier and not all providers allow it.
- **Map tile counts can be huge.** A single page load can be 20-50 tile requests as users pan/zoom. Use map style optimization + lower zoom defaults.
- **API keys are public** — restrict by referer/IP and put usage limits on them.
- **Routing modes are limited compared to Google** — no bicycle in some providers; truck mode has specifics.
- **Distance Matrix limits** — max 350 origins × 350 destinations per request.
- **Trackers retain positions for 30 days max** unless you export them.
- **HERE pricing for storage of geocoded results** may require a paid contract for commercial use — check.

## Related

- [Cognito](../05-security-iam/cognito.md) — identity pools for unauth front-end map access
- [EventBridge](../06-messaging-integration/eventbridge.md) — geofence events
- [IoT Core](#) — for high-volume device telemetry feeding the tracker
- [DynamoDB](../03-database/dynamodb.md) — store ride state, driver records
- MapLibre GL JS — recommended open-source map renderer
