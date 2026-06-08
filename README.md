#  Starlink Usage Scraper v2

A web scraper + Flask dashboard that extracts daily data usage from your Starlink account and displays it as an interactive chart. It also exports the data as a CSV for further analysis.

---

##  Project Structure

```
starlink-scraper-v2/
├── app.py                  # Flask web server
├── scraper.py              # Selenium-based scraper logic
├── requirements.txt        # Python dependencies
├── templates/
│   └── index.html          # Frontend dashboard
└── output/
    └── starlink_usage.csv  # Generated CSV output
```

> **Note:** The `templates/` and `output/` folders must exist before running the app. Create them manually if missing:
> ```bash
> mkdir templates output
> ```

---

##  How It Works

1. **`scraper.py`** — Uses `undetected-chromedriver` and Selenium to open a Chrome browser and prompt the user to log in to `starlink.com` manually. After login, the user must click the **☰ (three-line hamburger menu)** on the side of the page, then click **"Sign In"** from the menu that appears. This signals the scraper to proceed. It then navigates to the usage page and extracts daily GB data from the bar chart (via `aria-label` attributes or hover tooltips). It iterates through each monthly tab, tracks all collected dates, and saves the results to `output/starlink_usage.csv`.

2. **`app.py`** — A simple Flask app that serves the dashboard at `http://localhost:8080`. It exposes three endpoints:
   - `GET /` — Loads the HTML dashboard
   - `POST /api/scrape` — Triggers a scrape and returns the data as JSON
   - `GET /api/download-csv` — Downloads the generated CSV file

3. **`templates/index.html`** — The frontend that displays the usage chart and provides buttons to trigger a scrape and download the CSV.

---

##  Setup & Installation

### 1. Find the correct Python executable

>  **Important — read this first.**
>
> On Windows, `py` and `python` can point to **different Python installations**, which causes a
> `ModuleNotFoundError` even after a successful `pip install`. You must use the **same executable**
> for both installing packages and running the app.
>
> Run this to find out which Python you actually have:
> ```powershell
> py --version
> python --version
> where python
> ```
> If `where python` returns nothing, use `py` for everything below. If both exist, prefer `py` since
> that is the Windows Python Launcher and will pick the right version.

### 2. Install dependencies

Use **the same executable** for both commands — either both `py`, or both `python`:

```powershell
py -m pip install -r requirements.txt
```

**Requirements:**
- `Flask`
- `undetected-chromedriver`
- `pandas`
- `setuptools`

> **Note:** You also need **Google Chrome** installed on your machine.

### 3. Update your Starlink URL

Before running, open `scraper.py` and replace the placeholder with your own Starlink service-line URL:

```python
TARGET_URL = "https://starlink.com/account/service-line/AST-XXXXXXX-XXXXX-XX?..."
```

You can find this URL by logging in to [starlink.com](https://starlink.com), going to your account dashboard, and copying the URL from the address bar while on your service line page.

### 4. Run the Flask app

Use the **same executable** you used to install dependencies:

```powershell
py app.py
```

The server will start at `http://localhost:8080`.

### 5. Trigger a scrape

1. Open your browser and go to `http://localhost:8080`
2. Click the **"Scrape"** button
3. A Chrome window will appear — **log in to your Starlink account manually**
4. Once you have finished logging in and any auth steps are complete, do the following **inside the Chrome window**:
   - Click the **☰ three-line menu** on the side of the page
   - A side menu will open — click **"Sign In"**
5. The scraper will now automatically navigate to your usage page and collect data across all available monthly tabs
6. Once finished, the data will be displayed on the dashboard and saved to `output/starlink_usage.csv`

> ⏱️ **You have 45 seconds** after the Chrome window opens to complete the login and click ☰ → Sign In. If you need more time, increase the `time.sleep(45)` value in `scraper.py`.

---

##  Output

The scraper generates a CSV file at `output/starlink_usage.csv` with the following format:

```
Date,Usage_GB
2025-11-17,17.0
2025-11-18,13.0
...
```

You can download it directly from the dashboard or via `GET /api/download-csv`.

---

##  Configuration

The target account URL is hardcoded in `scraper.py`:

```python
TARGET_URL = "https://starlink.com/account/service-line/AST-XXXXXXX-XXXXX-XX?..."
```

**Update this to your own account/service-line URL** before running the scraper.

---

##  Notes

- **Manual login required** — You need to log in to Starlink in the browser window that appears when the scrape is triggered. Login is intentionally not automated to avoid bot detection.
- **Hamburger menu step required** — After logging in, you must manually click the **☰ three-line menu** on the side of the page, then click **"Sign In"** from the menu. This is a required step before the scraper begins collecting data.
- **Chrome version** — The scraper attempts multiple Chrome versions (`auto-detect → 148 → 147`) if there are compatibility issues.
- **Internet connection** — Make sure you are connected to the internet before running a scrape.
- **Data accuracy** — Data is pulled from the bar chart on the Starlink dashboard. If the chart fails to load (blank page), wait a moment and try again.

---

##  Troubleshooting

| Problem | Solution |
|---|---|
| `ModuleNotFoundError: No module named 'flask'` | Use `py app.py` instead of `python app.py` — they may point to different Python installs. Always use the same executable for both `pip install` and running the app. |
| "No bars found with any selector" | Refresh the page or wait for the chart to fully load |
| Chrome won't open | Update Chrome or check your `undetected-chromedriver` version |
| "CSV not found" error | Run a scrape first before attempting to download |
| No data collected after login | Make sure `TARGET_URL` matches your own Starlink account URL |
| Scraper proceeds before you finish logging in | Increase `time.sleep(45)` in `scraper.py` to give yourself more time |
| "Could not find hamburger menu" error | Make sure the page has fully loaded before clicking ☰; try scrolling up if the menu is hidden |
| "Sign In not found in menu" error | The menu may have opened but not fully rendered — wait a second and try the scrape again |
