# miao backend

Backend for **miao**, a health tracker web app. It uses the OpenAI API to identify foods in meal photos and estimate their nutrition, stores meal history, and generates PDF health reports.

- **Live backend:** https://miao-backend-8faq.onrender.com
- **Interactive API docs:** https://miao-backend-8faq.onrender.com/docs
- **Frontend (GitHub Pages):** https://sharon5u.github.io/Health%20tracker/
- **Frontend code:** https://github.com/sharon5u/sharon5u.github.io/tree/main/Health%20tracker

Built with Python and FastAPI, deployed on Render.

## Endpoints

| Method | Endpoint | Accepts | Returns |
|---|---|---|---|
| GET | `/` | – | `{"status": "ok"}` health check |
| POST | `/api/detect-food` | `multipart/form-data` with an `image` file (JPEG/PNG/WebP, under 8 MB) | JSON: list of foods with estimated portion, calories, protein, carbs, fat, confidence and uncertainty notes, plus meal totals and a disclaimer |
| POST | `/api/meals` | JSON: `date` (YYYY-MM-DD), `meal_name`, `foods` list | JSON: the saved meal with its ID |
| GET | `/api/history?before=<date>&limit=<n>` | Optional query parameters | JSON: saved days with meals and daily totals |
| GET | `/api/days/{date}` | Date in the URL | JSON: meals and totals for that day |
| PATCH | `/api/meals/{meal_id}` | Same JSON as saving a meal | JSON: the updated meal |
| DELETE | `/api/meals/{meal_id}` | Meal ID in the URL | JSON: confirmation |
| POST | `/api/report/pdf` | JSON: profile, water, steps, exercise and food totals | A PDF health report file |

Errors are returned as JSON with a `detail` message (for example, when an image is too large or the AI service is unavailable).

## How the frontend communicates with the backend

The frontend is a static site on GitHub Pages. In `app.js`, `API_BASE` is set to this backend's Render URL, and the page uses `fetch()` to call the endpoints above. For example, when the user uploads a meal photo, the frontend sends it as `FormData` to `POST /api/detect-food`. It then shows the returned foods and nutrition as a JSON in the backend, and displays an error message if the request fails.

CORS is limited to `https://sharon5u.github.io` (plus `http://127.0.0.1:5500` for local testing), so other websites can't call this backend from a browser.

## How secrets are handled

- The OpenAI API key is **never** in this repository or in the frontend code.
- On Render, it's stored as an **environment variable** named `OPENAI_API_KEY`, which `main.py` reads with `os.getenv`.
- The OpenAI call happens only on the server, so the key is never sent to the browser.
- `.gitignore` excludes `.env`, `.venv/` and the local database.

## Setup and run locally

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY="your_key_here"
uvicorn main:app --reload
```

Then open http://127.0.0.1:8000/docs to try the endpoints.

Optional environment variables:
- `OPENAI_MODEL`: which OpenAI model to use
- `MIAO_DB_PATH`: where to store the SQLite database

## Deploying on Render

- Service type: Web Service (Python 3)
- Build command: `pip install -r requirements.txt`
- Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
- Environment variable: `OPENAI_API_KEY`
