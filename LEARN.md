# SF Microclimates: A Complete Learning Guide

*A non-coder's guide to understanding this weather API from the ground up*

---

## Table of Contents

1. [What This Project Does](#what-this-project-does)
2. [The Evolution: Every Commit Explained](#the-evolution-every-commit-explained)
3. [File Structure: What Each File Does](#file-structure-what-each-file-does)
4. [The Code: Section by Section](#the-code-section-by-section)
5. [How It All Connects](#how-it-all-connects)
6. [Deployment & Infrastructure](#deployment--infrastructure)

---

## What This Project Does

### The Problem

San Francisco has the most dramatic "microclimates" of any US city. This means the weather can be completely different in neighborhoods just a few miles apart. When a weather app says "San Francisco: 58°F," that's essentially useless because:

- The **Outer Sunset** might be 52°F and foggy
- The **Mission District** might be 65°F and sunny
- These neighborhoods are only 3 miles apart!

Weather apps typically use a single reading (often from SFO airport) and call it "San Francisco weather." That doesn't match what people actually experience.

### The Solution

This project creates an **API** (a way for computer programs to request data) that:

1. Gathers data from **400+ PurpleAir sensors** across the city (these are small air quality monitors that also measure temperature and humidity)
2. Organizes the sensors by **49 different SF neighborhoods**
3. Calculates the **average temperature and humidity** for each neighborhood
4. Returns this data to anyone who asks, **free of charge, no signup required**

### What's an API?

Think of an API like a restaurant menu. You (the customer/program) look at the menu and place an order. The kitchen (the server) prepares what you asked for and brings it back. You don't need to know *how* the kitchen works—you just ask for what you want and get it.

In this case:
- **You ask**: "What's the weather in the Mission?"
- **API answers**: "It's 68°F with 45% humidity, based on 29 sensors"

---

## The Evolution: Every Commit Explained

A "commit" is like saving a snapshot of your project. It's a checkpoint that records what changed and why. Here's the complete history of this project, from first to last:

### Commit 1: Initial Release (Jan 25, 2026)

**What was created:**
- The entire working API from scratch
- Support for 50 SF neighborhoods
- A nice-looking website that explains how to use the API
- Documentation (README)
- All the configuration needed to run on Cloudflare

**In plain English**: This was launch day. Julian built the whole thing and made it live at `microclimates.solofounders.com`.

---

### Commit 2: Remove Preview Files (seconds later)

**What changed**: Deleted `README-styled.html`, `README.html`, and `README.pdf`

**Why**: These were probably files Julian used to preview how the documentation would look. Once the project was published, they weren't needed anymore—just clutter.

---

### Commit 3: Fix Mobile GitHub Link (minutes later)

**What changed**: A tiny CSS fix in the website code

**Why**: The link to GitHub wasn't showing up properly on phones. Small visual bug fix.

---

### Commit 4: Fix GitHub Link Color (minutes later)

**What changed**: Made the GitHub link text white instead of another color

**Why**: On the dark green header, the link was hard to read. Making it white fixed the contrast.

---

### Commit 5: Remove Website from API Repo (30 mins later)

**What changed**: Removed 446 lines of website code, keeping only 104 lines of essential API code

**Why**: Originally, the code included a fancy marketing website with lots of styling. Julian decided to keep the API simple and move the website elsewhere (or simplify it). The landing page was stripped down to just the basics.

---

### Commit 6: Clearer Skill Instructions (1 hour later)

**What changed**: Updated the instructions on how to add this to Claude/Clawdbot

**Why**: Made the copy/paste instructions clearer so users know exactly what to do.

---

### Commit 7: Update README Instructions (7 mins later)

**What changed**: Made the README match the website instructions

**Why**: Consistency—the README (on GitHub) should say the same thing as the website.

---

### Commit 8: Fix Rate Limiter (6 hours later)

**Who**: Dhravya Shah (a contributor, not Julian)

**What changed**: 
- Switched from a custom rate limiter to Cloudflare's built-in one
- Changed configuration from `wrangler.toml` to `wrangler.jsonc` (a more modern format)
- Added proper TypeScript type definitions

**Why**: The original rate limiter was custom-built. Cloudflare provides a native (built-in) rate limiter that's better maintained and more reliable. Using the platform's built-in tools is usually smarter than reinventing the wheel.

**What's a rate limiter?** It prevents any single user from overwhelming the server with too many requests. Like a bouncer that says "slow down, buddy" if someone's hitting the API 1000 times per second.

---

### Commit 9: Add Fallback Logic (8 hours later)

**What changed**: Added smart fallback when a neighborhood has no sensors

**Problem solved**: Some SF neighborhoods (like Sea Cliff or Lands End) might not have any PurpleAir sensors installed. Previously, you'd just get empty data.

**The fix**: Now, if you ask for weather in a neighborhood with no sensors, the API finds the *nearest* neighborhood that *does* have sensors and uses that data. It also tells you honestly: "We don't have sensors here, but the closest neighborhood with data is X."

**Example**: Ask for Sea Cliff weather → API says "No sensors here, but here's data from Outer Richmond (the nearest neighborhood with sensors)."

---

### Commit 10: Add Outlier Detection (latest)

**What changed**: Added smart filtering for bad sensor data

**Problem solved**: Sometimes a single sensor goes haywire. Maybe it's in direct sunlight, near an oven, or just malfunctioning. One sensor reporting 95°F when every neighbor says 65°F would throw off the average.

**The fix**: The API now:
1. Looks at each neighborhood's temperature
2. Compares it to the 3 nearest neighborhoods
3. If the reading is more than 10°F different AND the neighborhood only has 1-2 sensors, it's flagged as suspicious
4. The suspicious reading is replaced with the neighbor average
5. The API tells you this happened (transparency)

**Example**: Potrero Hill shows 85°F from 1 sensor, but Mission, SoMa, and Dogpatch all show ~65°F. The API says "That 85°F reading looked wrong, here's the corrected 65°F based on neighbors."

---

## File Structure: What Each File Does

Here's every file in the project and what it's for:

### Configuration Files

#### `package.json`
**What it is**: The project's "ID card"

**What it contains**:
- Project name (`sf-weather-proxy`)
- Version (`1.0.0`)
- Scripts (commands you can run):
  - `npm run dev` → test locally
  - `npm run deploy` → publish to the internet
  - `npm run tail` → watch live logs
- Dependencies (other software this project needs)

**Analogy**: Like the ingredients list and cooking instructions on a recipe card.

---

#### `wrangler.jsonc`
**What it is**: Cloudflare-specific configuration

**What it contains**:
- Project name for Cloudflare: `sf-microclimates`
- Which file runs the code: `src/index.ts`
- The web address it responds to: `microclimates.solofounders.com`
- Cache settings (save data for 1 hour = 3600 seconds)
- Rate limiter settings (max 100 requests per minute per user)
- Cache storage ID (where cached data lives on Cloudflare)

**Analogy**: Like giving a mail carrier specific instructions: "Deliver this package to this address, use this locker for storage, and don't accept more than 100 letters per minute from the same person."

---

#### `tsconfig.json`
**What it is**: TypeScript compiler settings

**What it tells TypeScript**:
- Use modern JavaScript (ES2021)
- Use Cloudflare's special features
- Be strict about catching errors
- Only look at files in the `src/` folder

**Analogy**: Like settings on a camera—you're telling it how to process the picture.

---

#### `.gitignore`
**What it is**: A list of files Git should ignore

**What it ignores**:
- `node_modules/` (downloaded libraries—too big)
- `.wrangler/` (temporary Cloudflare files)
- `.dev.vars` (secret API keys—never share these!)
- `*.log` (log files)
- `.DS_Store` (Mac junk files)

**Why**: Some files are either sensitive (secrets), temporary (logs), or huge (libraries). No need to track them in version control.

---

#### `LICENSE`
**What it is**: Legal permission to use this code

**What it says**: MIT License = "Do whatever you want with this code, just include this copyright notice. We're not responsible if something breaks."

**Why it matters**: Without a license, code is technically "all rights reserved" by default. The MIT license explicitly says "this is free to use."

---

#### `README.md`
**What it is**: The project's instruction manual

**What it contains**:
- What the project does
- How to use it
- All available API endpoints
- How to set up your own copy
- Credits

**Why it exists**: When someone finds this project on GitHub, this is the first thing they see. It's the front door welcome mat.

---

### Source Code Files

#### `src/index.ts`
**What it is**: THE code. Everything that makes the API work.

**Why one file?** This project is small enough that putting everything in one file keeps it simple. Larger projects split code across many files.

*(Full breakdown in the next section)*

---

#### `src/landing.html`
**What it is**: A fancier version of the homepage

**What it contains**: A beautifully designed webpage with:
- Nice typography (Instrument Serif + Inter fonts)
- A forest green and cream color scheme
- Solo Founders branding
- Instructions for using the API
- Copy/paste instructions for adding to Claude/Clawdbot

**Note**: The current code uses a simpler embedded HTML string instead of this file. This file was created earlier but may not be actively used now.

---

### Generated Files

#### `worker-configuration.d.ts`
**What it is**: Auto-generated type definitions

**What it does**: Tells TypeScript exactly what Cloudflare features are available. This is generated by running `wrangler types`—you don't edit it manually.

**Analogy**: Like an auto-generated dictionary of all the words (features) you're allowed to use.

---

#### `bun.lock`
**What it is**: Exact versions of all dependencies

**Why it exists**: When you install libraries, their versions can change. This "lock file" records exactly which version was installed, so anyone else installing this project gets identical copies.

---

## The Code: Section by Section

Let's walk through `src/index.ts` piece by piece.

### Part 1: The Header Comment

```typescript
/**
 * SF Microclimates API - Cloudflare Worker
 * Real-time temperature & humidity data for San Francisco neighborhoods
 * Powered by PurpleAir sensor data
 */
```

Just a description. Good practice to explain what a file does at the top.

---

### Part 2: Types and Interfaces (The Data Shapes)

**What are types?** They're like forms or templates that describe what data looks like.

#### The Environment Interface

```typescript
export interface Env {
  PURPLEAIR_API_KEY: string;
  CACHE: KVNamespace;
  CACHE_TTL_SECONDS: string;
  RATE_LIMITER: RateLimit;
}
```

This describes what "environment variables" the API expects:
- `PURPLEAIR_API_KEY`: A secret password to access PurpleAir data
- `CACHE`: A storage space on Cloudflare for saving data
- `CACHE_TTL_SECONDS`: How long to keep cached data (TTL = Time To Live)
- `RATE_LIMITER`: The bouncer that limits requests

---

#### The Sensor Interface

```typescript
interface PurpleAirSensor {
  sensor_index: number;
  latitude: number;
  longitude: number;
  temperature?: number;
  humidity?: number;
}
```

This describes what data comes from each PurpleAir sensor:
- `sensor_index`: A unique ID number
- `latitude`/`longitude`: GPS coordinates
- `temperature`: The reading (optional—might be missing)
- `humidity`: The moisture reading (also optional)

---

#### The Neighborhood Data Interface

```typescript
interface NeighborhoodData {
  temp_f: number | null;
  humidity: number | null;
  sensor_count: number;
}
```

This describes the calculated weather for each neighborhood:
- `temp_f`: Temperature in Fahrenheit (or null if no data)
- `humidity`: Humidity percentage (or null)
- `sensor_count`: How many sensors provided this data

---

### Part 3: The Landing Page HTML

```typescript
const LANDING_HTML = `<!DOCTYPE html>...`;
```

This is the simple webpage shown when you visit the root URL. It's written directly in the code as a string (text). It includes:
- A blue hero banner with the title
- A quick example of how to use the API
- A list of endpoints
- Information about neighborhoods
- Links to PurpleAir and GitHub

---

### Part 4: The Neighborhood Definitions (SF Geography!)

```typescript
const SF_NEIGHBORHOODS: Record<string, { name: string; bounds: {...} }> = {
  financial_district: { 
    name: "Financial District", 
    bounds: { nwLat: 37.7960, nwLng: -122.4050, seLat: 37.7880, seLng: -122.3920 } 
  },
  // ... 48 more neighborhoods
};
```

This is the **heart of the geography**. Each neighborhood is defined by:
- A **key** (like `mission`, `castro`, `marina`)
- A **display name** (like "Mission District", "Castro")
- A **bounding box** (a rectangle on the map)

**What's a bounding box?** Imagine drawing a rectangle around a neighborhood on a map:
- `nwLat`/`nwLng` = Northwest corner (top-left)
- `seLat`/`seLng` = Southeast corner (bottom-right)

Any sensor whose GPS coordinates fall inside this rectangle is considered part of that neighborhood.

**All 49 neighborhoods include:**
- Financial District, Chinatown, Union Square, Tenderloin, Civic Center
- Embarcadero, Rincon Hill, South Beach, North Beach, Telegraph Hill
- Russian Hill, Nob Hill, Marina, Pacific Heights, Presidio
- Sea Cliff, Lands End, Inner/Outer Richmond, Inner/Outer Sunset
- Parkside, Haight-Ashbury, Lower Haight, Hayes Valley, Cole Valley
- Castro, Noe Valley, Mission District, SoMa, Mission Bay
- Twin Peaks, Diamond Heights, Glen Park, Forest Hill, West Portal
- St. Francis Wood, Bernal Heights, Potrero Hill, Dogpatch
- Bayview, Hunters Point, Excelsior, Visitacion Valley
- Crocker Amazon, Ingleside, Oceanview, Merced Heights, Lakeside, Stonestown

---

### Part 5: The Overall SF Boundary

```typescript
const SF_BOUNDS = {
  nwLat: 37.8120,
  nwLng: -122.5200,
  seLat: 37.7080,
  seLng: -122.3550
};
```

This is the "big box" around all of San Francisco. When fetching sensors from PurpleAir, we only ask for sensors inside this area—no need to get data from Oakland or Daly City.

---

### Part 6: Helper Functions (The Tools)

#### Check if a point is inside a rectangle

```typescript
function isInBounds(lat, lng, bounds): boolean {
  return lat <= bounds.nwLat && lat >= bounds.seLat && 
         lng >= bounds.nwLng && lng <= bounds.seLng;
}
```

Given GPS coordinates and a bounding box, this returns `true` if the point is inside, `false` if outside.

---

#### Find the center of a neighborhood

```typescript
function getNeighborhoodCenter(bounds) {
  return {
    lat: (bounds.nwLat + bounds.seLat) / 2,
    lng: (bounds.nwLng + bounds.seLng) / 2
  };
}
```

Averages the corners to find the middle point. Used for measuring distances between neighborhoods.

---

#### Calculate distance between two points

```typescript
function getDistance(lat1, lng1, lat2, lng2): number {
  return Math.sqrt(Math.pow(lat2 - lat1, 2) + Math.pow(lng2 - lng1, 2));
}
```

This is the Pythagorean theorem (a² + b² = c²) to calculate straight-line distance. It's a simplification that works fine for small areas like a city.

---

#### Find the nearest neighborhood with data

```typescript
function findNearestNeighborhood(targetKey, availableKeys): string | null {
  // Find the center of the target neighborhood
  // Loop through all neighborhoods with data
  // Calculate distance to each
  // Return the closest one
}
```

Used for the **fallback feature**: if Sea Cliff has no sensors, this finds the nearest neighborhood that does (probably Outer Richmond).

---

#### Find the N nearest neighborhoods

```typescript
function findNearestNeighborhoods(targetKey, availableKeys, count): string[] {
  // Similar to above, but returns multiple neighbors
  // Sorted by distance
}
```

Used for **outlier detection**: to check if a reading makes sense, compare it to the 3 nearest neighborhoods.

---

### Part 7: Outlier Detection

```typescript
const OUTLIER_THRESHOLD_F = 10; // Flag if >10°F different
const OUTLIER_MIN_NEIGHBORS = 3; // Need at least 3 to compare

function detectAndCorrectOutliers(neighborhoods) {
  // For each neighborhood with data:
  //   1. Find the 3 nearest neighbors
  //   2. Calculate their average temperature
  //   3. If this neighborhood is >10°F different AND has ≤2 sensors:
  //      - Flag it as an outlier
  //      - Replace with the neighbor average
  //      - Include metadata explaining what happened
  return correctedData;
}
```

This is **data quality control**. A single broken sensor shouldn't ruin the whole neighborhood's reading.

Example metadata when an outlier is corrected:
```json
{
  "outlier_corrected": {
    "original_temp_f": 95,
    "neighbor_avg_f": 65,
    "diff_f": 30,
    "reason": "Single sensor reading 95°F was 30°F off from neighbors (avg 65°F)"
  }
}
```

---

### Part 8: Assign Sensors to Neighborhoods

```typescript
function assignToNeighborhood(lat, lng): string | null {
  for (const [key, neighborhood] of Object.entries(SF_NEIGHBORHOODS)) {
    if (isInBounds(lat, lng, neighborhood.bounds)) {
      return key;
    }
  }
  return null; // Sensor is in SF but not in any defined neighborhood
}
```

Given a sensor's GPS coordinates, this figures out which neighborhood it belongs to by checking each bounding box.

---

### Part 9: Fetch Data from PurpleAir

```typescript
async function fetchPurpleAirData(apiKey): Promise<PurpleAirSensor[]> {
  const fields = "sensor_index,latitude,longitude,temperature,humidity";
  const url = `https://api.purpleair.com/v1/sensors?fields=${fields}&location_type=0&nwlat=...`;
  
  const response = await fetch(url, { headers: { "X-API-Key": apiKey } });
  const data = await response.json();
  
  // Transform the raw array data into nice objects
  return sensors;
}
```

This reaches out to PurpleAir's API and asks: "Give me all outdoor sensors (`location_type=0`) in San Francisco, and tell me their GPS, temperature, and humidity."

**Note about `location_type=0`**: PurpleAir sensors can be indoor (1) or outdoor (0). We only want outdoor sensors for weather data.

---

### Part 10: Aggregate by Neighborhood

```typescript
function aggregateByNeighborhood(sensors): Record<string, NeighborhoodData> {
  // Create empty buckets for each neighborhood
  const neighborhoodSensors = { mission: { temps: [], humidities: [] }, ... };
  
  // Put each sensor in its neighborhood's bucket
  for (const sensor of sensors) {
    const neighborhood = assignToNeighborhood(sensor.latitude, sensor.longitude);
    if (neighborhood) {
      neighborhoodSensors[neighborhood].temps.push(sensor.temperature);
      neighborhoodSensors[neighborhood].humidities.push(sensor.humidity);
    }
  }
  
  // Calculate averages for each neighborhood
  const TEMP_CORRECTION_F = 8; // PurpleAir sensors run hot
  for each neighborhood:
    avgTemp = sum(temps) / count - 8°F  // Correction factor
    avgHumidity = sum(humidities) / count
  
  return results;
}
```

**Important**: There's an 8°F correction factor! PurpleAir sensors are inside plastic enclosures that heat up, so they consistently read about 8°F warmer than actual air temperature. This correction brings them in line with official readings.

---

### Part 11: The Main Data Fetcher (with Caching)

```typescript
async function getWeatherData(env): Promise<WeatherResponse> {
  const cacheKey = "sf-weather-data";
  const cacheTtl = 3600; // 1 hour
  
  // Step 1: Check if we have cached data
  const cached = await env.CACHE.get(cacheKey);
  if (cached) return JSON.parse(cached); // Cache hit! Return immediately.
  
  // Step 2: No cache, fetch fresh data
  const sensors = await fetchPurpleAirData(env.PURPLEAIR_API_KEY);
  const rawNeighborhoods = aggregateByNeighborhood(sensors);
  
  // Step 3: Clean up outliers
  const neighborhoods = detectAndCorrectOutliers(rawNeighborhoods);
  
  // Step 4: Save to cache for next time
  await env.CACHE.put(cacheKey, JSON.stringify(response), { expirationTtl: cacheTtl });
  
  return response;
}
```

**Why cache?** 
- PurpleAir limits API calls
- Users don't need second-by-second updates (weather doesn't change that fast)
- Caching makes responses faster
- 1-hour cache is a good balance

---

### Part 12: Response Helpers

```typescript
function jsonResponse(data, status = 200): Response {
  return new Response(JSON.stringify(data, null, 2), {
    status,
    headers: {
      "Content-Type": "application/json",
      "Access-Control-Allow-Origin": "*", // Allow any website to use this
      "Cache-Control": "public, max-age=300" // Browsers can cache for 5 min
    }
  });
}

function errorResponse(message, status = 500): Response {
  return jsonResponse({ error: message }, status);
}
```

These create properly formatted JSON responses with the right headers.

**What's CORS (`Access-Control-Allow-Origin: *`)?** Browsers normally block websites from fetching data from different domains for security. This header says "it's okay, anyone can access this API from any website."

---

### Part 13: The Main Request Handler

This is where everything comes together—the "front door" that handles incoming requests:

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const path = url.pathname;
    
    // STEP 1: Rate Limiting
    const clientIP = request.headers.get("CF-Connecting-IP");
    const { success } = await env.RATE_LIMITER.limit({ key: clientIP });
    if (!success) {
      return errorResponse("Rate limit exceeded. Try again in a minute.", 429);
    }
    
    // STEP 2: Handle CORS preflight
    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }
    
    // STEP 3: Only accept GET requests
    if (request.method !== "GET") {
      return errorResponse("Method not allowed", 405);
    }
    
    // STEP 4: Route to the right handler based on URL path
    try {
      if (path === "/sf-weather" || path === "/sf-weather/") {
        // Return all neighborhoods
        return jsonResponse(await getWeatherData(env));
      }
      
      if (path.match(/^\/sf-weather\/([a-z_]+)$/)) {
        // Return single neighborhood (with fallback if needed)
        const neighborhoodKey = path.split('/')[2];
        // ... handle fallback logic ...
        return jsonResponse(neighborhoodData);
      }
      
      if (path === "/neighborhoods") {
        // Return list of all neighborhood names
        return jsonResponse({ neighborhoods: [...] });
      }
      
      if (path === "/") {
        // Return the landing page HTML
        return new Response(LANDING_HTML, { headers: { "Content-Type": "text/html" } });
      }
      
      // Nothing matched
      return errorResponse("Not found", 404);
      
    } catch (error) {
      return errorResponse(error.message, 500);
    }
  }
};
```

---

## How It All Connects

### The Complete Request Journey

Let's trace what happens when someone types:
```
curl https://microclimates.solofounders.com/sf-weather/mission
```

**1. Request arrives at Cloudflare**
- Cloudflare's global network receives the request
- It routes to the nearest data center

**2. Worker wakes up**
- The code in `src/index.ts` runs
- The `fetch()` function is called with the request

**3. Rate limit check**
- Gets the user's IP address from headers
- Asks the rate limiter: "Is this IP okay?"
- If they've made >100 requests this minute: "Rate limit exceeded" (429)
- Otherwise: proceed

**4. Parse the URL**
- Path is `/sf-weather/mission`
- Matches the neighborhood pattern
- Extract `mission` as the neighborhood key

**5. Get weather data**
- Check Cloudflare KV cache for `sf-weather-data`
- **Cache hit**: Return cached data immediately (fast!)
- **Cache miss**: 
  - Call PurpleAir API with the API key
  - Get ~400 sensors in SF
  - Assign each sensor to a neighborhood by GPS
  - Calculate averages (with 8°F correction)
  - Run outlier detection
  - Save to cache for 1 hour
  - Return data

**6. Extract Mission data**
- Find `mission` in the neighborhoods object
- Check sensor count:
  - If `sensor_count > 0`: Use Mission's data
  - If `sensor_count = 0`: Find nearest neighborhood with data, use that (with fallback notice)

**7. Format response**
```json
{
  "updated": "2026-01-26T08:00:00.000Z",
  "neighborhood": "mission",
  "name": "Mission District",
  "temp_f": 68,
  "humidity": 45,
  "sensor_count": 29
}
```

**8. Return to user**
- Headers: JSON content type, CORS enabled, 5-minute browser cache
- Body: The formatted JSON
- Cloudflare delivers to user

**Total time: Usually under 100ms** (mostly the cache lookup)

---

### The Caching System

**Cloudflare KV** is like a global storage locker:

1. **Write**: After fetching from PurpleAir, data is saved with a 1-hour expiration
2. **Read**: Every request checks KV first—99% of requests hit cache
3. **Expire**: After 1 hour, cached data is deleted automatically
4. **Next request**: Cache miss triggers a fresh PurpleAir fetch

**Why this matters**:
- PurpleAir has API rate limits—can't call it 1000x/second
- Most users don't need second-by-second updates
- Cache makes responses lightning fast (Cloudflare KV is distributed globally)

---

## Deployment & Infrastructure

### Where This Runs: Cloudflare Workers

**What is a Worker?** 
- Code that runs on Cloudflare's edge network (data centers worldwide)
- No servers to manage—just upload code, Cloudflare handles the rest
- Scales automatically—whether you get 10 requests or 10 million
- Runs close to users (low latency)

### How to Deploy

1. **Install dependencies**: `npm install`
2. **Set up secrets**: `wrangler secret put PURPLEAIR_API_KEY`
3. **Deploy**: `npm run deploy` (or `wrangler deploy`)
4. **Done**—live at the configured URL

### The Tech Stack

| Component | What It Does |
|-----------|--------------|
| **TypeScript** | The programming language (JavaScript with types) |
| **Cloudflare Workers** | Runs the code globally |
| **Cloudflare KV** | Stores cached weather data |
| **Cloudflare Rate Limiting** | Prevents abuse |
| **PurpleAir API** | Source of sensor data |
| **Wrangler** | Tool for deploying to Cloudflare |

### Costs

- **Cloudflare Workers**: Free tier = 100,000 requests/day
- **Cloudflare KV**: Free tier = 100,000 reads/day
- **PurpleAir API**: Free for personal use
- **Total**: $0 for most use cases

---

## Quick Reference: API Endpoints

| Endpoint | Returns |
|----------|---------|
| `GET /` | Landing page (HTML) |
| `GET /sf-weather` | All 49 neighborhoods with weather data |
| `GET /sf-weather/mission` | Single neighborhood (replace `mission` with any valid key) |
| `GET /neighborhoods` | List of all neighborhood keys and names |

### Valid Neighborhood Keys

`financial_district`, `chinatown`, `union_square`, `tenderloin`, `civic_center`, `embarcadero`, `rincon_hill`, `south_beach`, `north_beach`, `telegraph_hill`, `russian_hill`, `nob_hill`, `marina`, `pacific_heights`, `presidio`, `sea_cliff`, `lands_end`, `inner_richmond`, `outer_richmond`, `inner_sunset`, `outer_sunset`, `parkside`, `haight`, `lower_haight`, `hayes_valley`, `cole_valley`, `castro`, `noe_valley`, `mission`, `soma`, `mission_bay`, `twin_peaks`, `diamond_heights`, `glen_park`, `forest_hill`, `west_portal`, `st_francis_wood`, `bernal_heights`, `potrero_hill`, `dogpatch`, `bayview`, `hunters_point`, `excelsior`, `visitacion_valley`, `crocker_amazon`, `ingleside`, `oceanview`, `merced_heights`, `lakeside`, `stonestown`

---

## Summary

This project is a well-crafted, focused API that solves a real problem: getting accurate, hyperlocal weather for San Francisco. It demonstrates:

1. **Good architecture**: Simple, single-file design that's easy to understand
2. **Data quality**: Outlier detection and fallback logic handle edge cases
3. **Performance**: Caching makes it fast and reduces API calls
4. **Reliability**: Rate limiting prevents abuse
5. **Accessibility**: Free to use, no signup required

Whether you're building a weather widget, training an AI, or just settling arguments about whether it's foggy in the Richmond, this API has you covered.

---

*Built by Solo Founders · Powered by PurpleAir · MIT License*
