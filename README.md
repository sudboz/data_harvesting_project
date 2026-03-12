# TripAdvisor Spain Scraper

R-based scraper that collects structured data on restaurants, hotels, and attractions from TripAdvisor across three Spanish cities: Madrid, Seville, and Barcelona.

**Authors:** Sude Boz, Clare Anne Williams McGarvey, José Eyzaguirre

---

## Data Collected

### Restaurants — `restaurants_<city>_tripadvisor.csv`
| Field | Description |
|---|---|
| `location_id` | TripAdvisor unique identifier |
| `name` | Restaurant name |
| `rating` | Bubble rating (1–5) |
| `review_count` | Total number of reviews |
| `cuisine` | Primary cuisine type |
| `price_range` | Price tier (`$`, `$$`, `$$$`, `$$$$`) |
| `url` | Full TripAdvisor URL |

### Hotels — `hotels_<city>_tripadvisor.csv`
| Field | Description |
|---|---|
| `location_id` | TripAdvisor unique identifier |
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
| `category` | Accommodation type (hotel, B&B, etc.) |
| `url` | Full TripAdvisor URL |

### Attractions — `attractions_<city>_tripadvisor.csv`
| Field | Description |
|---|---|
| `rank` | Overall ranking on TripAdvisor |
| `attraction_id` | TripAdvisor unique identifier |
| `name` | Attraction name |
| `category` | Type of attraction |
| `neighborhood` | District within the city |
| `rating` | Bubble rating (1–5) |
| `review_count` | Total number of reviews |
| `travelers_choice` | Whether it has a 2025 Travelers' Choice badge |
| `open_status` | Current open/closed status |
| `booking_available` | Ticket/booking availability flag |
| `description` | Short description from listing |
| `url` | Full TripAdvisor URL |

---

## Requirements

R ≥ 4.1 and the following packages:

```r
install.packages(c("httr2", "rvest", "dplyr", "stringr", "readr", "jsonlite"))
```

---

## Cookie Setup

This scraper requires browser cookies to bypass TripAdvisor's bot detection. There are two types of cookies needed depending on the section.

---

### 1. DataDome Cookie (Restaurants & Attractions)

TripAdvisor uses [DataDome](https://datadome.co/) on its HTML listing pages. You need to grab a valid cookie value from your browser and paste it into the script.

**How to get it:**

1. Open Chrome and go to any TripAdvisor page, for example:
   `https://www.tripadvisor.com/Restaurants-g187514-Madrid.html`
2. Open DevTools: press `F12` (Windows) or `Cmd + Option + I` (Mac)
3. Go to the **Application** tab
4. In the left sidebar, click **Cookies** → `https://www.tripadvisor.com`
5. Find the cookie named **`datadome`** in the list
6. Click on it and copy the full value from the **Value** field at the bottom (it's a long string)

**Where to paste it:**

In the Rmd, find the `DATADOME` chunk at the start of each Restaurants and Attractions section and replace the placeholder:

```r
DATADOME <- "paste_your_datadome_value_here"
```

> ⚠️ DataDome cookies expire every few days. If you get empty results or parsing errors, go back to the browser and copy a fresh cookie value.

---

### 2. Session Cookies (Hotels)

The hotel scraper calls TripAdvisor's internal GraphQL API, which requires a broader set of session cookies. These are obtained from a network request, not the Application tab.

**How to get them:**

1. Open Chrome and go to any TripAdvisor hotels page, for example:
   `https://www.tripadvisor.com/Hotels-g187514-Madrid-Hotels.html`
2. Open DevTools: `F12` or `Cmd + Option + I`
3. Go to the **Network** tab
4. Scroll down the hotels page to trigger a request, or just wait for the page to fully load
5. In the Network tab filter bar, type **`graphql`** to filter requests
6. Click on any request called **`ids`** (the GraphQL endpoint)
7. Go to the **Headers** tab of that request
8. Scroll down to **Request Headers**
9. Find the **`cookie`** header and copy its full value (it's a long string containing multiple cookies like `TAUnique`, `TASID`, `datadome`, etc.)

**Where to paste it:**

In the Rmd, find the `COOKIES` chunk in the Hotels section and replace the placeholder:

```r
COOKIES <- "paste_your_full_cookie_string_here"
```

> ⚠️ Session cookies also expire. If the hotel scraper returns no results or errors, refresh them using the steps above.

---

## Usage

Open `Tripadvisor_scraping.Rmd` in RStudio. The file is organized into independent sections — you can run any city or category without running the others.

**Structure:**
```
1. Libraries
2. Restaurants Scraping
   ├── Madrid
   ├── Seville
   └── Barcelona
3. Attractions Scraping
   ├── Madrid
   ├── Seville
   └── Barcelona
4. Hotels Scraping
   ├── Cookie Setup   ← paste COOKIES here once for all cities
   ├── Madrid
   ├── Seville
   └── Barcelona
```

For each section, update the cookie chunk at the top, then run the scraping chunk. Results are saved automatically as CSV files in your working directory.

---

## Output Files

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

Checkpoint files are also written every 10 pages (e.g. `checkpoint_mad_10.csv`) so progress is not lost if a run is interrupted.

---

## How It Works

### Restaurants & Attractions — HTML Scraping
Pages are fetched using offset-based pagination (`oa0`, `oa30`, `oa60`, …). Each page is parsed with `rvest` using TripAdvisor's `data-automation` attributes to locate listing cards and extract fields. The loop stops automatically when a page returns no cards. Failed pages are retried once with a longer delay before the final save.

### Hotels — GraphQL API
Hotel data is collected by POSTing to TripAdvisor's internal GraphQL endpoint with a JSON payload that mirrors what the browser sends. This returns structured JSON which is parsed with `jsonlite` and flattened into a data frame. The loop stops automatically when the API returns no results.

### Rate Limiting
- A 3-second delay (`Sys.sleep(3)`) is applied between every page request
- Failed pages are retried with a 5-second delay
- Checkpoints are saved every 10 pages

---

## Notes

- Cookie values are session-specific and need to be refreshed periodically
- TripAdvisor's HTML structure may change without notice, which can break selectors
- This scraper is intended for research and educational purposes. Please review TripAdvisor's [Terms of Service](https://tripadvisor.mediaroom.com/US-terms-of-use) before use.

