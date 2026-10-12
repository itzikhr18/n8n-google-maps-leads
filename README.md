# Google Maps Leads → Google Sheets (n8n, official Places API)

A small, free n8n workflow that pulls local businesses from the **official Google Places API (New)** into a Google Sheet. No scraping, so it doesn't break when Google Maps changes its HTML.

```
Manual trigger → Settings → Places Text Search (paginated) → Extract leads → Google Sheets (append or update on place_id)
```

## What you get per business

`place_id, business_name, address, phone, website, rating, reviews, maps_url, search_query, found_at`

Example output (from a test run against a mock API, so the businesses are made up): [`example-output.csv`](example-output.csv)

| business_name | phone | website | rating | reviews |
|---|---|---|---|---|
| Bright Smile Dental | (512) 555-0101 | brightsmile.example | 4.8 | 210 |
| Old Town Dentistry | (512) 555-0102 | oldtowndentistry.example | 4.1 | 37 |
| Lakeside Family Dental | (512) 555-0103 | *(none)* | 4.6 | 12 |
| Broken Site Orthodontics | +1 512-555-0105 | brokensite.example | 3.9 | 8 |

## Setup (about 10 minutes)

1. **Places API key**: in Google Cloud, enable *Places API (New)* and create an API key (restrict it to that API).
2. In n8n, create a **Header Auth** credential: name `X-Goog-Api-Key`, value = your key. Select it in the *Search Google Places* node.
3. Create a Google Sheet with a tab named **Leads** and this header row:
   `place_id, business_name, address, phone, website, rating, reviews, maps_url, search_query, found_at`
4. Import `google-maps-leads-to-sheets.json` into n8n, connect your Google Sheets credential, and paste the sheet URL into **Settings**.
5. Set `searchQuery` (e.g. `plumbers in Denver, CO`) and `maxPages` (1-3), then run.

## Things that cost me time (so they don't cost you)

- **The field mask must include `nextPageToken`.** Without it the API silently returns only the first page.
- **Text Search stops at 60 results** (3 pages × 20). For more, split the area into smaller searches (by neighborhood or zip code).
- **Later pages can repeat earlier results**, so the *Extract leads* node dedupes on `place_id`. It also drops businesses that aren't `OPERATIONAL`.
- **Re-runs don't duplicate rows**: Google Sheets uses *Append or Update* matched on `place_id`.

## Cost

Asking for phone, website and rating puts the request in Google's **Text Search Enterprise** SKU. Check the current free monthly cap and price on Google's [Places API pricing page](https://developers.google.com/maps/billing-and-pricing/pricing) before big runs. One run with `maxPages: 3` is 3 requests.

## Tested on

n8n 2.42, end to end against a local mock of the Places API (two pages, a duplicate, a permanently closed business). The Google Sheets node was checked against its node definition, not against a live account. Try a 1-page run first.

## Want the next steps automated too?

I built a bigger version that also audits each website (HTTPS, mobile tag, meta description, H1, schema, booking form, footer year, broken site, public email), writes a personal cold email and a 1-10 lead score with any OpenAI-compatible model, and creates Gmail **drafts** (never sends). It's a paid template; details are in my profile. This free workflow is complete on its own.

## License

MIT
