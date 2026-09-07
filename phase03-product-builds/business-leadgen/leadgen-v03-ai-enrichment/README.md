# Intelligent Sales Lead Generator (LeadGen v0.3)

AI-powered B2B lead generation and sales intelligence tool that finds businesses, filters and scores them, and enriches each one with AI-generated company insights. The app was built to save a solo salesperson hours of manual prospecting.

## Problem Statement

Manually researching business leads is slow and inconsistent. This tool automates that pipeline end-to-end. It discovers businesses matching a location and category, filters out weak leads, and uses an LLM to read each company's website and generate a summary, product/service list, sales angle, and a HIGH/MEDIUM/LOW qualification with reasoning.

## Approach

- **Lead discovery** -  via the Google Places API (Text Search), filtered by minimum rating, minimum review count, and operational status
- **Lead scoring** — a weighted 0–100 score combining normalized rating (70%) and review volume (30%, capped at 500 reviews). This was chosen to prioritize consistently well-reviewed businesses over ones with a handful of 5-star reviews
- **Website extraction** - using requests + BeautifulSoup, stripping non-content HTML (scripts, nav, footer) and capping extracted text at 5,000 characters before sending it to an LLM, to control cost and avoid noisy input
- **AI enrichment** - using the OpenAI API with Pydantic structured outputs so the model returns a validated object with a company summary, products/services list, sales insight, and a constrained HIGH/MEDIUM/LOW qualification enum with a free-form reason
- **Graceful degradation** — if a website blocks scraping (403) or AI enrichment fails for any reason, the lead still returns with null AI fields instead of failing the whole request. The UI shows a clear "No AI enrichment available" message rather than "None"
- **Demo mode** — a toggle that returns hardcoded sample leads instead of hitting real Google Places / OpenAI APIs, so the app can be explored without cost or requiring API keys

## Results

Enrichment currently succeeds on the majority of leads with a live, non-blocking website. Known limitation: sites that return 403 to non-browser requests, or render content primarily via JavaScript, will not yield extractable text.

## Tools & Libraries

`Python`, `FastAPI`, `Streamlit`, `Google Places API`, `OpenAI API`, `Pydantic`, `BeautifulSoup4`, `requests`, `pandas`, `python-dotenv`, `openpyxl`

## How to Run

**Backend:**
```bash
git clone https://github.com/NewManRising/ai-ml-apprenticeship.git
cd ai-ml-apprenticeship/phase03-product-builds/business-leadgen/leadgen-v03-ai-enrichment
pip install -r requirements.txt
```

Create a `.env` file with:
```
GOOGLE_PLACES_API_KEY=your_key_here
OPENAI_KEY=your_key_here
API_BASE_URL=http://127.0.0.1:8000
```

Run the backend:
```bash
uvicorn app:app --reload
```
API docs available at `http://127.0.0.1:8000/docs`

**Frontend** (separate terminal):
```bash
streamlit run streamlit_app.py
```

**Live demo:** _(coming soon — deployed on Render)_

Or use the built-in "Demo Mode" toggle to explore the app immediately with sample data, no API keys required.

## Future Improvements

- Add rate limiting / usage controls before making live API mode publicly accessible, to prevent unrestricted use of paid Google Places and OpenAI quota
- Move enrichment calls (website scraping + OpenAI) to run concurrently instead of sequentially, reducing response time for multi-lead searches
- Replace broad `except Exception` handling around the OpenAI call with more specific exception types and structured logging
- Add persistent storage (PostgreSQL) so leads survive beyond a single session instead of living only in memory
- Push enriched leads directly to a CRM (EX: Notion) instead of only exporting to Excel

## Project Structure

```
leadgen-v03-ai-enrichment/
├── app.py                          # FastAPI backend
├── streamlit_app.py                # Streamlit frontend
├── schemas.py                      # Pydantic models for structured AI output
├── data/
│   └── google_places.py            # Google Places search, filtering, scoring, demo mode
├── ai/
│   ├── website_extractor.py        # Website scraping and text cleanup
│   └── company_enrichment.py       # OpenAI structured enrichment call
├── requirements.txt
└── README.md
```