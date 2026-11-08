# Data Source Proposals

**Team:** Liam Shaar, Ori Gat
**Repository:** https://github.com/liamshaar/it_project_liam_ori

---

## Proposal 1 (preferred): OpenRouter - LLM providers, price vs. reliability

**Summary.** The same open-weight LLM (e.g. Llama 3.3 70B) is served by many competing inference providers, each with its own price and reliability. OpenRouter aggregates them and publishes, per model and per provider, the current price and rolling uptime statistics. We want to track these over the semester to answer: *how much can be saved on the same model by choosing the right provider, and what does that saving cost in reliability?*

**Content.** For each tracked model, a list of provider endpoints with: provider name, status, price per prompt/completion token, uptime over the last 5 minutes / 30 minutes / 1 day, and (when available) latency and throughput percentiles over the last 30 minutes.

**URL.** `https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints`
Example: `https://openrouter.ai/api/v1/models/meta-llama/llama-3.3-70b-instruct/endpoints`
Docs: https://openrouter.ai/docs/api/api-reference/endpoints/list-all-endpoints-for-a-model

**Update frequency.** Near real-time: uptime is computed over rolling 5-minute, 30-minute and 1-day windows. Prices and the list of providers change occasionally. Planned sampling: every 30 minutes, for 3-5 popular models.

**Expected sample size.** 9348 bytes per request (one model). For 3-5 models every 30 minutes over ~70 days this is roughly 100-200 MB of raw JSON, well under 500 MB after gzip compression.

**API key / password.** Not required for prices and uptime (verified with curl). Latency and throughput fields were returned as `null` without authentication; they may require a free OpenRouter account key. If used, the key will be stored outside the repository with `chmod 600`.

**Why the data disappears.** Uptime values are rolling windows and prices are overwritten when they change. We did not find a downloadable per-provider history, so a missed sample is lost.

**Limitations.** Many providers report 100% uptime most of the time and prices change rarely, so variation between samples depends on tracking several popular models with many providers.

**Data sample** (excerpt, first 2 providers, collected 2026-11-08):

```json
{
  "id": "meta-llama/llama-3.3-70b-instruct",
  "endpoints_total": 9,
  "endpoints": [
    {
      "provider_name": "DeepInfra",
      "status": 0,
      "pricing": {
        "prompt": "0.0000001",
        "completion": "0.00000032",
        "discount": 0
      },
      "uptime_last_5m": 100,
      "uptime_last_30m": 100,
      "uptime_last_1d": 98.95031490552833,
      "latency_last_30m": null,
      "throughput_last_30m": null
    },
    {
      "provider_name": "AkashML",
      "status": 0,
      "pricing": {
        "prompt": "0.0000002",
        "completion": "0.00000052",
        "input_cache_read": "0.0000001",
        "discount": 0
      },
      "uptime_last_5m": 100,
      "uptime_last_30m": 100,
      "uptime_last_1d": 100,
      "latency_last_30m": null,
      "throughput_last_30m": null
    }
  ]
}
```

---

## Proposal 2: Open-Meteo Marine - wave forecast for Waimea Bay, Hawaii

**Summary.** Surf forecasts are published continuously and replaced by the next model run; the forecast as it was issued 1, 3 or 5 days ahead is not kept. We want to store the wave forecast for Waimea Bay (North Shore, Oahu) and later compare it with measurements from NOAA buoy 51201, to answer: *how accurate are wave forecasts several days ahead, and are they worse during big swells?*

**Content.** Current significant wave height (m), wave direction (degrees) and wave period (s), plus an hourly wave-height forecast for the next 3 days, for one coordinate.

**URL.** `https://marine-api.open-meteo.com/v1/marine?latitude=21.67&longitude=-158.12&current=wave_height,wave_direction,wave_period&hourly=wave_height&forecast_days=3`
Docs: https://open-meteo.com/en/docs/marine-weather-api

**Update frequency.** "Current" values use a 15-minute interval (900 s); the underlying wave models are re-run several times a day. Planned sampling: every hour.

**Expected sample size.** 2211 bytes per request; hourly sampling over ~70 days is about 4 MB in total.

**API key / password.** None. Free for non-commercial use.

**Why the data disappears.** Each new model run replaces the previous forecast. Buoy measurements are archived by NOAA, but the forecast as issued at a given time is not. We will verify which forecast horizons Open-Meteo archives and build the question around what is not archived.

**Limitations.** The forecast is model-based (not a measurement); a buoy source will be needed later as "ground truth", and buoys can have outages.

**Data sample** (excerpt, first 3 hourly values, collected 2026-11-08):

```json
{
  "latitude": 21.708336,
  "longitude": -158.20833,
  "generationtime_ms": 0.2522468566894531,
  "utc_offset_seconds": 0,
  "timezone": "GMT",
  "timezone_abbreviation": "GMT",
  "elevation": 0.0,
  "current_units": {
    "time": "iso8601",
    "interval": "seconds",
    "wave_height": "m",
    "wave_direction": "°",
    "wave_period": "s"
  },
  "current": {
    "time": "2026-11-08T12:30",
    "interval": 900,
    "wave_height": 2.54,
    "wave_direction": 4,
    "wave_period": 10.1
  },
  "hourly_units": {
    "time": "iso8601",
    "wave_height": "m"
  },
  "hourly": {
    "time": [
      "2026-11-08T00:00",
      "2026-11-08T01:00",
      "2026-11-08T02:00"
    ],
    "wave_height": [
      2.14,
      2.16,
      2.16
    ]
  }
}
```

---

## Ranking

1. **OpenRouter (preferred)** - most relevant to us and to the current AI industry; the price/reliability trade-off between providers is a question we genuinely don't know the answer to.
2. **Open-Meteo Marine** - a reliable, well-documented source on a topic we care about (surfing), with a natural forecast-vs-reality pairing.
