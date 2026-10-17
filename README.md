# SkyPredict

A responsive Flask weather app that retrieves live forecasts from Open-Meteo. It includes city search, current conditions, hourly and seven-day forecasts, air quality, unit switching, and optional browser geolocation.

## Project layout

- `backend/` contains the Flask application and its local SQLite account database.
- `frontend/templates/` contains the welcome page, weather dashboard, and sign-in/registration pages.
- `frontend/static/` contains the app logo, dashboard JavaScript, and shared styles.
- `requirements.txt` lists the Python dependencies.

## Requirements

- Python 3.9 or newer
- Internet access for Open-Meteo API requests
- A modern browser

No API key or `.env` file is required for the Open-Meteo endpoints used here. Open-Meteo offers free access for non-commercial use subject to its usage terms and fair-use limits. Review the current terms and availability before deploying commercially. The app briefly caches city searches and weather results to avoid unnecessary upstream requests.

Open-Meteo's forecast endpoints used here do not provide official severe-weather alerts. SkyPredict states this limitation rather than inventing alerts. Air-quality readings are shown using Open-Meteo's **US AQI** field and its associated category scale; the provider's data coverage and update timing can vary by location. If no AQI is available, the app says so.

## Run in VS Code

1. Open this project folder in VS Code.
2. Open the integrated terminal (**Terminal → New Terminal**).
3. Create a virtual environment:

   ```powershell
   py -m venv .venv
   ```

4. Activate it in PowerShell:

   ```powershell
   .\.venv\Scripts\Activate.ps1
   ```

   If PowerShell blocks activation, use Command Prompt instead:

   ```bat
   .venv\Scripts\activate.bat
   ```

5. Install the dependencies:

   ```powershell
   python -m pip install -r requirements.txt
   ```

6. Start the app:

   ```powershell
   python backend\app.py
   ```

7. Open `http://127.0.0.1:5000` to see the SkyPredict welcome page and logo. Sign in or create an account; after signing in, the app opens the weather dashboard.

The weather chat assistant uses NVIDIA NIM's OpenAI-compatible chat completions API with the `nemotron-3.5-lightning-30b-a3b` model by default. Set `NVIDIA_API_KEY` in the environment before starting the app; optionally set `NVIDIA_MODEL` to another model ID available to your NVIDIA API account. For example, in PowerShell, set `$env:NVIDIA_API_KEY = "<your-rotated-key>"` in the same terminal before running `python backend\app.py`. Never put the key in source code or commit it. On Render, add `NVIDIA_API_KEY` as a secret environment variable in the service settings. Without the key, the dashboard remains usable and the assistant is shown as disabled.

Accounts are stored in `backend\instance\weather.sqlite3`. Passwords are saved as secure hashes, not plain text. The Flask session secret is generated for local use when `FLASK_SECRET_KEY` is not set; set a long, random `FLASK_SECRET_KEY` before deployment so sessions remain valid across restarts. Set `FLASK_COOKIE_SECURE=1` when serving over HTTPS.

Browser location access generally requires HTTPS or a trusted local address such as `localhost`. It is requested only when **Use my location** is clicked. The app can always be used by searching for a city.

## Deploy to Render

The root-level `render.yaml` describes a Python web service with a persistent disk. The Starter service plan is required because Render's free web services do not support persistent disks; without persistent storage, registered accounts in the SQLite database could be lost when the service restarts or redeploys.

1. Put this project in a GitHub repository. Do not commit `.venv`, the `backend\instance` folder, or secrets.
2. In the Render dashboard, choose **New → Blueprint**, connect the repository, and deploy the `render.yaml` Blueprint.
3. Render installs the dependencies and starts the existing Flask app with Gunicorn. The Blueprint generates `FLASK_SECRET_KEY`, enables secure session cookies, and stores the existing SQLite account database at `/var/data/weather.sqlite3` on the persistent disk.
4. When deployment finishes, open the Render `onrender.com` HTTPS URL. Test the welcome page, register/sign in, search a city, and use **Use my location** after granting browser permission.
5. Check that the service health check at `/health` succeeds. It checks that the app can connect to its configured SQLite database.

The website and its `/api/search` and `/api/weather` endpoints are served from the same domain, so the frontend can keep using its existing relative API URLs and does not need CORS configuration. Do not use Flask's development server for the hosted service. No Android project is created until the HTTPS deployment is available and verified.

## Data sources

- City search: [Open-Meteo Geocoding API](https://open-meteo.com/en/docs/geocoding-api)
- Weather forecast: [Open-Meteo Forecast API](https://open-meteo.com/en/docs)
- Air quality: [Open-Meteo Air Quality API](https://open-meteo.com/en/docs/air-quality-api)
