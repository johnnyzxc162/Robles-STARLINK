# 📡 Starlink Usage Scraper v2

A web scraper + Flask dashboard that extracts daily data usage from your Starlink account and displays it as an interactive chart. It also exports the data as a CSV for further analysis.

---

## 🗂️ Project Structure

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

---

## ⚙️ How It Works

1. **`scraper.py`** — Uses `undetected-chromedriver` and Selenium to open a Chrome browser and prompt the user to log in to `starlink.com` manually. It then navigates to the usage page and extracts daily GB data from the bar chart (via `aria-label` attributes or hover tooltips). It iterates through each monthly tab, tracks all collected dates, and saves the results to `output/starlink_usage.csv`.

2. **`app.py`** — A simple Flask app that serves the dashboard at `http://localhost:8080`. It exposes three endpoints:
   - `GET /` — Loads the HTML dashboard
   - `POST /api/scrape` — Triggers a scrape and returns the data as JSON
   - `GET /api/download-csv` — Downloads the generated CSV file

3. **`templates/index.html`** — The frontend that displays the usage chart and provides buttons to trigger a scrape and download the CSV.

---

## 🚀 Setup & Installation

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

**Requirements:**
- `Flask`
- `undetected-chromedriver`
- `pandas`
- `setuptools`

> **Note:** You also need **Google Chrome** installed on your machine.

### 2. Run the Flask app

```bash
python app.py
```

The server will start at `http://localhost:8080`.

### 3. Trigger a scrape

1. Open your browser and go to `http://localhost:8080`
2. Click the **"Scrape"** button
3. A Chrome window will appear — **log in to your Starlink account manually**
4. Once logged in, the tool will automatically scrape your usage data across all available monthly tabs
5. The data will be displayed on the dashboard and saved to `output/starlink_usage.csv`

---

## 📊 Output

The scraper generates a CSV file at `output/starlink_usage.csv` with the following format:

```
Date,Usage_GB
2025-11-17,17.0
2025-11-18,13.0
...
```

You can download it directly from the dashboard or via `GET /api/download-csv`.

---

## 🔧 Configuration

The target account URL is hardcoded in `scraper.py`:

```python
TARGET_URL = "https://starlink.com/account/service-line/AST-XXXXXXX-XXXXX-XX?..."
```

**Update this to your own account/service-line URL** before running the scraper.

---

## ⚠️ Notes

- **Manual login required** — You need to log in to Starlink in the browser window that appears when the scrape is triggered. Login is intentionally not automated to avoid bot detection.
- **Chrome version** — The scraper attempts multiple Chrome versions (`auto-detect → 148 → 147`) if there are compatibility issues.
- **Internet connection** — Make sure you are connected to the internet before running a scrape.
- **Data accuracy** — Data is pulled from the bar chart on the Starlink dashboard. If the chart fails to load (blank page), wait a moment and try again.

---

## 🐛 Troubleshooting

| Problem | Solution |
|---|---|
| "No bars found with any selector" | Refresh the page or wait for the chart to fully load |
| Chrome won't open | Update Chrome or check your `undetected-chromedriver` version |
| "CSV not found" error | Run a scrape first before attempting to download |
| No data collected after login | Make sure `TARGET_URL` matches your own Starlink account URL |
