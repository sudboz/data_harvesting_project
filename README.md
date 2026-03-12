# Data Harvesting Tripadvisor Project
Jose, Sude and Clare's data harvesting project about Tripsadvisor

# TripAdvisor Scraper 

An R-based web scraper that collects structured data on **restaurants**, **hotels**, and **attractions** from TripAdvisor across three major Spanish cities: **Madrid**, **Seville**, and **Barcelona**.

---

## Overview

This project systematically scrapes TripAdvisor listing pages using `httr2` and `rvest`, navigating paginated results with offset-based URLs for restaurants and attractions, and via TripAdvisor's internal GraphQL API for hotels. It handles bot-detection cookies (DataDome), retries failed pages, writes periodic checkpoints, and outputs clean, deduplicated CSV files ready for analysis.

---

## Data Collected

### Restaurants (`restaurants_<city>_tripadvisor.csv`)
| Field | Description |
|---|---|
| `location_id` | TripAdvisor unique location identifier |
| `name` | Restaurant name |
| `rating` | Bubble rating (1–5) |
| `review_count` | Total number of reviews |
| `cuisine` | Primary cuisine type |
| `price_range` | Price tier (`$`, `$$`, `$$$`, `$$$$`) |
| `url` | Full TripAdvisor URL |

### Hotels (`hotels_<city>_tripadvisor.csv`)
| Field | Description |
|---|---|
| `location_id` | TripAdvisor unique location identifier |
| `name` | Hotel name |
| `stars` | Provider star rating |
| `rating` | Guest rating (1–5) |
| `num_reviews` | Total number of reviews |
| `ranking_<city>` | Local popularity rank |
| `price_min_usd` / `price_max_usd` | Price range in USD |
| `price_lowest` | Lowest available price |
| `address` | Full street address |
| `phone` | Contact phone number |
| `latitude` / `longitude` | Geographic coordinates |
| `category` | Accommodation category (hotel, B&B, etc.) |
| `url` | Full TripAdvisor URL |

### Attractions (`attractions_<city>_tripadvisor.csv`)
| Field | Description |
|---|---|
| `rank` | Overall ranking on TripAdvisor |
| `attraction_id` | TripAdvisor unique identifier |
| `name` | Attraction name |
| `category` | Type of attraction |
| `neighborhood` | Area/district within the city |
| `rating` | Bubble rating (1–5) |
| `review_count` | Total number of reviews |
| `travelers_choice` | Whether it has a 2025 Travelers' Choice badge |
| `open_status` | Current open/closed status |
| `booking_available` | Ticket/booking availability flag |
| `description` | Short description from listing |
| `url` | Full TripAdvisor URL |

---

## Requirements

- **R** ≥ 4.1
- The following R packages:

```r
install.packages(c("httr2", "rvest", "dplyr", "stringr", "readr", "jsonlite"))
```

---

## Setup

### 1. DataDome Cookie (Restaurants & Attractions)

TripAdvisor uses [DataDome](https://datadome.co/) for bot protection on its HTML pages. You must provide a valid DataDome cookie value before running the restaurant and attraction scrapers.

1. Open TripAdvisor in your browser.
2. Open DevTools → Application → Cookies → find the `datadome` cookie.
3. Copy its value and paste it into the script:

```r
DATADOME <- "your_datadome_cookie_value_here"
```

> ⚠️ DataDome cookies expire. If you get empty results or parsing errors, refresh the cookie from your browser.

### 2. Session Cookies (Hotels)

The hotel scraper uses TripAdvisor's internal GraphQL endpoint and requires a fuller set of session cookies (`TAUnique`, `TASID`, `TASession`, `TASSK`, `datadome`). Obtain these the same way:

1. Open a TripAdvisor hotels page in your browser.
2. Copy all relevant cookies from DevTools into the `COOKIES` string in the Hotels section.

---

## Usage

Open `Tripadvisor_scraping.Rmd` in RStudio and run each section independently, or knit the full document. The script is organized into the following sections:

```
1. Libraries
2. DataDome Configuration
3. Restaurants Scraping
   ├── Madrid
   ├── Seville
   └── Barcelona
4. Attractions & Activities Scraping
   ├── Madrid
   ├── Seville
   └── Barcelona
5. Hotels Scraping
   ├── Cookie Setup
   ├── Madrid
   ├── Seville
   └── Barcelona
```

Each city section runs independently — you can scrape one city without running the others.

---

## Output Files

The scraper writes the following CSV files to your working directory:

```
restaurants_madrid_tripadvisor.csv
restaurants_seville_tripadvisor.csv
restaurants_barcelona_tripadvisor.csv

attractions_madrid_tripadvisor.csv
attractions_seville_tripadvisor.csv
attractions_barcelona_tripadvisor.csv

hotels_madrid_tripadvisor.csv
hotels_seville_tripadvisor.csv
hotels_barcelona_tripadvisor.csv
```

Intermediate **checkpoint files** are also written every 10 pages (e.g., `checkpoint_mad_10.csv`) so progress is not lost if a run is interrupted.

---

## How It Works

### Restaurants & Attractions — HTML Scraping
Pages are fetched with offset-based pagination (`oa0`, `oa30`, `oa60`, …). Each page is parsed with `rvest` using TripAdvisor's `data-automation` attributes to locate cards and extract fields. Failed pages are tracked and retried with a longer delay before the final save.

### Hotels — GraphQL API
Hotel data is collected by POSTing to TripAdvisor's internal GraphQL endpoint with a JSON payload that mirrors what the browser sends. This returns structured JSON which is parsed with `jsonlite` and flattened into a data frame.

### Rate Limiting & Retries
- A 3-second delay (`Sys.sleep(3)`) is applied between every page request.
- Failed pages are collected and retried once with a 5-second delay after the main loop.
- Checkpoints are saved every 10 pages.

---

## Notes & Limitations

- Cookie values are **session-specific** and will need to be refreshed periodically.
- TripAdvisor's HTML structure may change without notice, which can break CSS selectors.
- This scraper is intended for **research and educational purposes**. Review TripAdvisor's [Terms of Service](https://tripadvisor.mediaroom.com/US-terms-of-use) before use.

---

## Authors

- Sude Boz
- Clare Anne Williams McGarvey
- José Eyzaguirre

