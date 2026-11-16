# CIS 3400 – Advanced Programming

## Project 2: Python Application Development

**Student Name:** Dominic Rueck
**Project Title:** BudgetPlate – Budget Meal-Prep Optimizer
**Course:** CIS 3400 – Advanced Programming
**Final Presentation:** December 8, 2026

---

## 1. Project Overview

**Application Name:** BudgetPlate

**Project Option:** API-Integrated Web Application (with a database and an AI component)

### Problem Statement

Students and young athletes often need to hit calorie and protein targets on a tight grocery budget, but it is hard to know which foods meet those targets for the least money. Calorie trackers mostly record what someone has already eaten, and recipe sites rarely account for real grocery prices. BudgetPlate works backwards: the user describes a goal (for example, "2,200 calories, 150 g protein, $50 a week, no dairy") and the app builds the cheapest weekly grocery list that meets it, using real nutrition data and real store prices.

### Intended Users

College students, student-athletes, and anyone meal-prepping on a limited budget. They enter a goal, choose a nearby store, and receive a grocery list with quantities, macro totals, and total cost.

### Unique Features

- Combines two external APIs (nutrition and prices) into a decision using a cost-minimizing optimization, rather than just displaying API data.
- Uses an LLM (**stretch goal**) to turn a free-text goal into validated, structured constraints.
- Reports when targets are impossible within the budget and shows the minimum budget that would work.
- Stores saved plans, a pantry, and cached API data in SQLite (**vector-database semantic search is a stretch goal**).

*BudgetPlate is an educational planning tool and does not provide medical or dietary advice.*

## 2. Application Features

- **Feature 1 – Goal input (core):** Enter calorie, protein, and weekly budget targets plus dietary exclusions through a form. **Free text via LLM (stretch).**
- **Feature 2 – Live data:** Look up nearby stores and pull nutrition (USDA FoodData Central) and prices (Kroger Public API, which includes Harris Teeter) for a curated list of staple foods.
- **Feature 3 – Optimized plan:** Generate the lowest-cost weekly grocery list that meets the targets, with calorie, macro, and cost totals.
- **Feature 4 – Saved plans (core) and pantry (stretch):** Save and reload plans; treat pantry items as free (stretch).
- **Feature 5 (stretch) – Semantic search:** Search foods with queries such as "cheap high-protein breakfast".

Users type or select a goal, press **Build my plan**, and receive a grocery list with totals. If the targets cannot be met, they see a clear message and the minimum feasible budget.

### Development Phases

**Core:** Flask backend, HTML/CSS/JS interface, USDA and Kroger API integration, FoodCatalog, SciPy optimizer with infeasible-target handling, and SQLite (API cache and saved plans).

**Stretch (only after the core works):** LLM free-text goal parsing, pantry list, and ChromaDB semantic food search.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Flask | Web server and API routes |
| HTML / CSS / JavaScript | Browser UI (forms, plan display, client-side interaction) |
| requests | Calls to the USDA and Kroger REST APIs |
| python-dotenv | Load API keys from a local `.env` file |
| pandas / NumPy | Clean and merge nutrition and price data |
| SciPy (`linprog` / `milp`) | Cost-minimizing optimization under nutrition constraints |
| SQLite (`sqlite3`) | Cache API responses; store plans and pantry |
| ChromaDB **(stretch)** | Vector database for semantic food search |
| anthropic (Claude API) **(stretch)** | Convert free-text goals into structured JSON constraints |
| pytest | Automated tests, including error and edge cases |

## 4. Application Architecture

```mermaid
flowchart TD
    U[User: browser] <--> UI[Flask + static/index.html<br/>HTML/CSS/JS]
    UI <--> APP[Flask app<br/>app.py]
    APP --> P[Goal parser<br/>nl_parser.py]
    P <--> LLM[(LLM API<br/>external)]
    P --> C[PlanConstraints<br/>constraints.py]
    UDA[(USDA API<br/>external)] <--> UC[API clients<br/>usda_client.py / kroger_client.py]
    KAPI[(Kroger API<br/>external)] <--> UC
    UC --> FC[FoodCatalog<br/>catalog.py / units.py]
    FC --> O[MealPlanOptimizer<br/>optimizer.py]
    C --> O
    O <--> DB[(SQLite<br/>database.py)]
    O --> R[Plan results<br/>list, macros, cost]
    R --> APP
    APP <--> V[(ChromaDB<br/>vector_search.py)]
    classDef stretch stroke-dasharray: 5 5
    class P,LLM,V stretch
```

*Dashed components are stretch goals.*

### How the Application Works

1. The user enters a goal through a form (core) or, in the stretch build, as free text.
2. The browser sends the goal to Flask via JSON API calls; JavaScript updates the page without a full reload where useful.
3. Form input goes straight to `PlanConstraints` for validation. The LLM parser (`LLMConstraintParser`) is an optional stretch step that converts free text to JSON before validation.
4. `FoodCatalog` merges nutrition data (USDA) and prices (Kroger) fetched by the API clients, using cached values from SQLite when available.
5. `MealPlanOptimizer` finds the lowest-cost list that meets the constraints, or reports that none exists.
6. Flask serves `static/index.html` and JSON API responses; results are shown in the browser, and plans can be saved to SQLite.

## 5. Project Structure

*Planned structure; this will be updated*

```text
budgetplate/
├── app.py                # Flask application factory / routes
├── config.py             # Loads environment variables and settings
├── static/
│   ├── index.html        # Main UI (served by Flask static or root route)
│   ├── css/              # Stylesheets
│   └── js/               # Client logic (forms, fetch to API, UI updates)
├── constraints.py        # PlanConstraints: input model and validation
├── nl_parser.py          # (stretch) LLMConstraintParser: free text -> JSON constraints
├── usda_client.py        # USDAClient: nutrition lookups
├── kroger_client.py      # KrogerClient: OAuth, stores, prices
├── units.py              # Package-size parsing and cost per 100 g
├── catalog.py            # FoodCatalog / FoodItem: merged food data
├── optimizer.py          # MealPlanOptimizer (SciPy)
├── database.py           # SQLite access: cache, plans, pantry
├── vector_search.py      # (stretch) FoodSearchIndex (ChromaDB)
├── data/
│   └── staples.csv       # Curated staple foods: name, USDA ID, Kroger search term, tags
├── tests/                # pytest tests
├── docs/
│   └── architecture.png
├── .env.example          # Placeholder variable names only (no real keys)
├── .gitignore            # Excludes .env and local database files
├── requirements.txt
└── README.md
```

## 6. Installation and Running the Application

```bash
git clone [repository-url]
cd budgetplate
pip install -r requirements.txt
cp .env.example .env      # then add your own API keys
export FLASK_APP=app.py   # or: flask --app app run
flask run                 # default http://127.0.0.1:5000
```

Required keys (stored only in your local `.env`, **never committed**): a USDA FoodData Central API key and Kroger developer client ID and secret. An LLM API key is needed only for the stretch free-text goal parser.

## 7. Testing and Validation

*To be completed for the final deliverable.*

| Test | Expected Result | Actual Result |
|---|---|---|
| Normal goal input | A plan within budget that meets the targets | [Result] |
| Invalid or missing input (negative budget, empty goal) | Helpful error message | [Result] |
| Impossible targets (for example, very high protein on a very low budget) | Clear message with the minimum feasible budget | [Result] |
| API failure or missing price | Fallback to cached data or a clear message | [Result] |
| Malformed LLM output **(stretch)** | Rejected by validation; user asked to rephrase | [Result] |

## 8. AI-Assisted Development Disclosure

**AI Tools Used:** Claude, GitHub Copilot *(update at submission)*

### How AI Was Used

*Planned:* first drafts of the API client classes, Flask routes and HTML/CSS/JS layout, and SQLite operations; troubleshooting OAuth and optimization errors; suggested test cases and README wording. Runtime LLM calls to parse goals are a **stretch** feature; the core app uses form input and `PlanConstraints` validation only.

### Student Contributions

*Planned:* I designed the problem, features, architecture, and constraint rules; reviewed, ran, and modified all generated code; wrote and ran tests; and integrated the APIs, optimizer, and database.

**Student statement:** I reviewed, tested, and can explain the code and technologies used in my application, including code developed with AI assistance.

## 9. Anticipated Challenges and Lessons Learned

**Challenge 1 – API authentication and availability.** The Kroger API uses OAuth 2.0 and its access tokens expire, and both the USDA and Kroger APIs have rate limits and can fail or return incomplete data.
*Plan:* `KrogerClient` will request and cache a token and refresh it when it expires. API keys stay in `.env`. All API responses are cached in SQLite, requests use timeouts and error handling, and the app falls back to cached data (and a small sample dataset for demos) with a clear message when an API is unavailable.

**Challenge 2 – Matching grocery prices with nutrition data.** USDA and Kroger do not share a common product ID, and Kroger returns package sizes as text (for example, "16 oz" or "12 ct"), so nutrition per 100 g and price per 100 g cannot be joined directly.
*Plan:* A curated `data/staples.csv` links each food to a USDA food ID and a Kroger search term. `units.py` converts package sizes into grams and computes cost per 100 g, skipping items with a missing price or unparseable size. Unit tests cover typical and malformed size strings.

**Challenge 3 – Nutritional goals that cannot be met within the budget.** A user may request more protein or calories than the budget allows, which makes the optimization infeasible.
*Plan:* `MealPlanOptimizer` detects infeasible results instead of crashing, re-solves without the budget cap to find the minimum feasible budget, and returns a clear message such as "This target needs at least $X per week." Input validation rejects impossible values, and a pytest test covers this case.

**Challenge 4 – Unrealistic optimizer results.** A pure cost minimizer can recommend extreme plans (for example, several kilograms of one food).
*Plan:* Add per-food maximum quantities, category and variety limits, and dietary exclusions as constraints, and check results against sample goals.

**Challenge 5 – Project scope and time.** The project includes several advanced components and a short timeline.
*Plan:* Build the core application first (Flask, HTML/CSS/JS, API integration, SQLite, optimizer) and add the LLM parser and vector search only if the core is complete and tested (see Section 2). Any LLM output will be validated before use.

**Lessons learned:** *To be completed as the project progresses.*

## 10. Application Screenshots

*To be added for the final deliverable (main interface and a generated plan).*

## 11. Project Links

**GitHub Repository:** [URL]
**Deployed Application:** [URL, if applicable]
**Demonstration Video:** [URL, if applicable]

## 12. Future Improvements

1. Support additional stores and compare prices across them.
2. Add recipe suggestions built from the optimized grocery list.

---

### Documentation Timeline

- **Deliverable 1 – November 16:** Overview, features, technologies, architecture diagram, and planned structure.
- **Deliverable 2 – November 23:** Update to reflect implementation progress and initial testing.
- **Final Deliverable – December 8:** All sections complete, setup verified, screenshots and links added.