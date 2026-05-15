# Travel Explorer

> **We went, so you know where to go.**

A premium single-page travel discovery website featuring curated city guides, interactive travel tools, weather lookup, safety notes, and travel enquiries — built with **pure HTML, CSS, and JavaScript**. No frameworks. No backend. Just one beautiful file.

🌐 **Live demo:** [brennantan-coder.github.io/Travel-Explorer-BT](https://brennantan-coder.github.io/Travel-Explorer-BT/)

---

## Features

### 12 Curated City Guides
Seoul · Singapore · Bali · Tokyo · Bangkok · Hong Kong · Kuala Lumpur · Maldives · Phuket & Krabi · Sydney · Taipei · Hanoi

Each guide includes:
- Destination overview & ambience
- Detailed 3-day itineraries
- Must-try local foods
- Transport & getting-around tips
- Estimated cost breakdown (flights, hotels, food)
- Safety advice
- Best photo spots

### Interactive Travel Tools
- **Trip Budget Calculator** — destination x travellers x days x daily spend
- **Packing Checklist** — 5 categories, persists in `localStorage`
- **Itinerary Planner** — add/delete activities by day, persists in `localStorage`

### Weather Widget
Live current conditions for any city worldwide via **OpenWeather API**. 12 quick-pick destination buttons. Add your API key once to enable.

### Safety Section
Color-coded safety ratings (safe / moderate / caution) for all 12 destinations with key tips and emergency contact numbers.

### Enquiry Form
Validated contact form with email regex check, success state, and `localStorage` persistence.

### Design
- Editorial travel-magazine aesthetic
- **Playfair Display** serif headlines + **Inter** for body text
- Black / white / gold accent palette
- Cinematic Unsplash hero imagery
- Fully responsive (mobile, tablet, desktop)
- Smooth scroll, fade-in modals, hover transitions

---

## Getting Started

### Option 1 — Open directly
Simply double-click `index.html` to open it in your browser. That's it.

### Option 2 — Local server (recommended for the Weather API)
```powershell
cd "path\to\Travel"
python -m http.server 8080
```
Then open [http://localhost:8080](http://localhost:8080)

### Enabling the Weather Widget
1. Get a free API key at [openweathermap.org/api](https://openweathermap.org/api)
2. Open `index.html` and find this line near the bottom of the `<script>`:
   ```js
   const OPENWEATHER_API_KEY = 'YOUR_API_KEY_HERE';
   ```
3. Replace `'YOUR_API_KEY_HERE'` with your actual key
4. Reload — live weather is now active

---

## Project Structure

```
Travel/
├── index.html                  # The entire app (HTML + CSS + JS)
├── README.md                   # This file
├── .mcp.json                   # Playwright MCP server config (dev tooling)
└── .github/
    └── workflows/
        └── deploy.yml          # GitHub Actions → Pages deployment
```

---

## Deployment (GitHub Actions)

This repository auto-deploys to GitHub Pages on every push to `main` via the workflow in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).

**Workflow steps:**
1. Checkout the repo
2. Configure GitHub Pages
3. Upload the entire repo as a Pages artifact
4. Deploy to GitHub Pages

**To enable on a fork:**
1. Go to **Settings → Pages**
2. Under **Source**, select **GitHub Actions**
3. Push to `main` — the workflow runs automatically

---

## Tech Stack

| Layer | Tech |
|---|---|
| Markup | HTML5 |
| Styling | CSS3 (Grid, Flexbox, custom properties) |
| Logic | Vanilla JavaScript (ES6+) |
| Fonts | Google Fonts — Playfair Display + Inter |
| Imagery | Unsplash |
| Weather | OpenWeather API |
| Persistence | Browser `localStorage` |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions |

---

## Screenshots

> A screenshot of the live site will be added soon — captured via Playwright MCP automation.

---

## License

Open source — feel free to fork, learn from, and adapt for your own travel projects.

---

**Built with care by [@brennantan-coder](https://github.com/brennantan-coder)**
