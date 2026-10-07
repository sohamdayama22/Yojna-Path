# NSFDC Concessional Loan Scheme Portal

A multilingual web portal that helps Scheduled Caste (SC) beneficiaries discover the right **NSFDC** (National Scheduled Castes Finance & Development Corporation) concessional loan scheme, estimate their monthly repayments, find nearby channel-partner banks, and submit a guidance application. It includes an admin dashboard for managing schemes, partner banks, success stories and incoming applications.

Built with **Flask + SQLite** on the backend and plain **HTML/CSS/JavaScript** on the frontend (no build step).

---

## Features

### Citizen portal (`/`)
- **10-step guided wizard**: language, loan purpose, gender, loan amount, family income, scheme recommendation, EMI calculator, success stories, partner locator and final summary/submission.
- **Rule-based scheme recommender** covering six schemes: MCF, MSY, AMY, TERM, GBS and ELS (see the table below).
- **EMI calculator** with moratorium (grace period) support, returning monthly EMI, total interest and total repayable.
- **Partner bank locator** with an interactive [Leaflet](https://leafletjs.com/) map, live geolocation, text search (city, state, pincode, bank, branch) and distance-based sorting using the Haversine formula.
- **Community success stories** filtered by the recommended scheme.
- **7 languages**: English, Hindi, Marathi, Tamil, Telugu, Bengali and Gujarati.
- **Accessibility**: adjustable font size.
- **Application submission** that generates a reference number (`NSFDC-XXXXXXXX`).

### Admin dashboard (`/admin`)
- Login screen
- View all submitted applications (joined with the selected partner bank)
- Add partner banks (including internal NPA ratio, which is hidden from the public API)
- Add or update schemes
- Add community success stories

---

## Schemes in the database

| Code | Scheme | Max amount | Interest (M / F) | Max tenure | Moratorium |
|------|--------|-----------:|------------------|-----------:|-----------:|
| MCF  | Micro Credit Finance | ₹1.25 L | 6.5% / 6.5% | 3 yrs | 3 mo |
| MSY  | Mahila Samriddhi Yojana | ₹1.25 L | 4.0% / 4.0% | 3.5 yrs | 3 mo |
| AMY  | Aajeevika Microfinance Yojana | ₹1.25 L | 15.0% / 15.0% | 3 yrs | 3 mo |
| TERM | Flagship Term Loan Scheme | ₹45 L | 8.0% / 8.0% | 7 yrs | 6 mo |
| GBS  | Green Business Scheme | ₹30 L | 8.0% / 7.5% | 7 yrs | 6 mo |
| ELS  | Educational Loan Scheme | ₹40 L | 6.5% / 6.5% | 12 yrs | 12 mo |

### Recommendation logic (`/api/recommend`)
Rules are evaluated in this order, and the first match wins:

1. Purpose is education, or the applicant is a student: **ELS**
2. Purpose is green business or solar e-rickshaw: **GBS**
3. Amount up to ₹1.25 L and gender is female: **MSY**
4. Amount up to ₹1.25 L and purpose is MFI micro-credit: **AMY**
5. Amount up to ₹1.25 L and gender is male: **MCF**
6. Any other amount up to ₹1.25 L: **MCF**
7. Otherwise: **TERM**

Family income is checked against a ₹5 Lakh eligibility limit and returned as `income_eligible`.

---

## Project structure

```
final/
├── app.py               # Flask app: routes, recommendation engine, EMI, partner search, admin API
├── database.py          # SQLite connection + schema creation + seed data
├── database.db          # SQLite database (pre-seeded)
├── requirements.txt     # Python dependencies
└── static/
    ├── index.html       # Citizen portal (wizard UI)
    ├── admin.html       # Admin dashboard
    ├── app.js           # Wizard logic, API calls, map handling
    ├── translations.js  # UI strings for 7 languages
    └── styles.css       # Styling
```

### Database tables
- `schemes`: scheme details, rates, eligibility, required documents, bilingual text
- `channel_partners`: banks/NBFC-MFIs/SCAs with coordinates, supported schemes, NPA ratio
- `community_posts`: beneficiary success stories
- `applications`: submitted applications with status and timestamp

---

## Getting started

### Prerequisites
- Python 3.9+
- pip

### Installation

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd <your-repo-folder>

# 2. (Optional) create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the server
python app.py
```

Open **http://127.0.0.1:5000** for the citizen portal and **http://127.0.0.1:5000/admin** for the admin dashboard.

The database tables are created automatically on first run (`init_db()`). The map uses Leaflet from a CDN, so an internet connection is needed for map tiles and the emblem/logo images.

---

## API reference

### Public

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET  | `/api/schemes` | List all schemes |
| POST | `/api/recommend` | Recommend a scheme. Body: `purpose`, `gender`, `loan_amount`, `family_income`, `education_status` |
| POST | `/api/calculate-emi` | Body: `principal`, `annual_rate`, `tenure_months`, `moratorium_months` |
| GET  | `/api/partners` | Query: `scheme_code`, `search`, `lat`, `lng`, `max_npa` (default 8.0) |
| GET  | `/api/community-posts/<scheme_code>` | Success stories for a scheme |
| POST | `/api/applications` | Submit an application, returns `application_no` |

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/admin/login` | Admin login |
| GET  | `/api/admin/applications` | All applications |
| GET / POST | `/api/admin/partners` | List (including NPA) / add partner |
| POST | `/api/admin/schemes` | Add or update a scheme |
| POST | `/api/admin/posts` | Add a success story |

---

## Tech stack

- **Backend:** Python, Flask, SQLite (`sqlite3`)
- **Frontend:** HTML5, CSS3, vanilla JavaScript
- **Maps:** Leaflet 1.9.4
- **i18n:** custom `translations.js` (English, Hindi, Marathi, Tamil, Telugu, Bengali, Gujarati)

---

## Known limitations and security notes

This project is a prototype/demo. Before any real deployment:

- **Admin API is not actually protected.** The login returns a token, but the server never checks it, so `/api/admin/*` endpoints are open to anyone. Add real authentication (for example Flask-Login or JWT) and verify it on every admin route.
- **Hardcoded credentials and secret key** (`admin` / `admin123`, and `app.secret_key` in `app.py`). Move these to environment variables.
- **Debug mode is on** (`debug=True`, `host='0.0.0.0'`). Disable it in production and run behind a WSGI server such as Gunicorn.
- The seeded database contains sample applications and partner data for demonstration only.
- Scheme figures are for guidance; confirm the latest terms on the official NSFDC portal.

---

## Roadmap ideas

- Proper authentication and role-based access for admins
- Edit and delete for partners, schemes and posts
- Application status tracking for citizens
- Automated tests for the recommendation rules and EMI maths

---

## Disclaimer

This portal provides scheme-matching guidance only and is not an official Government of India application. Always verify eligibility and terms with NSFDC or an authorised channel partner.
