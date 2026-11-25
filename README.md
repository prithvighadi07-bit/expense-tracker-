A zero-dependency **Java 21 Personal Expense Tracker** featuring both a **live Web Dashboard** (`http://localhost:8080`) powered by Java's built-in `HttpServer` and an **interactive Terminal CLI** (`--cli`), backed by UTF-8 CSV persistence.

Built entirely with the Java 21 standard library (`com.sun.net.httpserver.HttpServer`, `java.net.http`, `java.time`, `BigDecimal`)—no external JARs, frameworks, or database servers required.

---

## Features

- **Interactive Web Dashboard**:
  - Add, edit, and delete expenses with exact `BigDecimal` currency arithmetic (2 decimal places, `HALF_UP`).
  - Live monthly KPI cards (**Monthly Total**, **Transactions**, **Average Expense**, **Highest Expense**).
  - Visual **Category Breakdown** bars and **Payment Method** summaries.
  - Real-time search and multi-criteria filtering by keyword, category, payment method, and date range.
  - Multi-currency display switcher (`$`, `₹`, `€`, `£`) and one-click CSV download.
- **Interactive Terminal CLI (`--cli`)**:
  - Menu-driven console interface with formatted ASCII tables, progress bars, CSV import/export, and settings.
- **Monthly Budgets & Alerts**:
  - Configure overall monthly budgets and per-category limits.
  - Automatic alerts when spending reaches `>= 80%` (**Approaching Limit**) or exceeds `100%` (**Over Budget**).
- **10 Categories & 6 Payment Methods**:
  - Categories: `FOOD`, `TRANSPORT`, `HOUSING`, `UTILITIES`, `ENTERTAINMENT`, `HEALTHCARE`, `SHOPPING`, `EDUCATION`, `TRAVEL`, `OTHER`.
  - Payment Methods: `CASH`, `CARD`, `UPI`, `BANK_TRANSFER`, `WALLET`, `OTHER`.
- **CSV Persistence**:
  - Stores data locally in UTF-8 files (`expenses.csv`, `budgets.csv`, `settings.txt`) with RFC-4180 quote/comma escaping.

---

## Project Structure

```text
expense-tracker/
├── src/com/expensetracker/
│   ├── Main.java                     # Entry point (--web default, --cli, --demo, --port, --data-dir)
│   ├── model/
│   │   ├── Budget.java               # Monthly overall or category budget record
│   │   ├── Category.java             # Expense categories enum
│   │   ├── Expense.java              # Immutable expense entity with validation
│   │   └── PaymentMethod.java        # Payment methods enum
│   ├── service/
│   │   └── ExpenseService.java       # Core business logic, filtering, summaries & alerts
│   ├── storage/
│   │   └── CsvExpenseStore.java      # RFC-4180 CSV persistence, import & export
│   ├── ui/
│   │   ├── ExpenseTrackerCli.java    # Interactive terminal CLI controller
│   │   └── TableFormatter.java       # ASCII tables, currency formatting & progress bars
│   └── web/
│       ├── ExpenseWebServer.java     # Built-in HttpServer static + REST JSON API
│       └── JsonUtil.java             # Lightweight JSON serializer & flat-object parser
├── web/
│   ├── index.html                    # Web dashboard markup
│   ├── styles.css                    # Responsive dashboard styling
│   └── app.js                        # Frontend state, charts, filters & API calls
├── test/com/expensetracker/
│   └── ExpenseTrackerTest.java       # Automated unit, CLI, and HTTP API tests
├── run.bat                           # Windows CMD launcher (web, demo, cli, test)
├── run.ps1                           # PowerShell launcher
└── README.md
```

---

## Prerequisites

- **Java Development Kit (JDK) 21+** (`java` and `javac` available on `PATH`).

---

## Quick Start

### Using Windows Command Prompt (`run.bat`)

```cmd
cd /d C:\Users\Preksha\Downloads\campuspulse\expense-tracker
```

- **Start Web Dashboard with sample demo data** (opens at `http://localhost:8080`):
  ```cmd
  run.bat demo
  ```
- **Start Web Dashboard with clean/existing data**:
  ```cmd
  run.bat
  ```
- **Start Interactive Terminal CLI**:
  ```cmd
  run.bat cli
  ```
- **Compile & run the automated test suite**:
  ```cmd
  run.bat test
  ```

### Using PowerShell (`run.ps1`)

If PowerShell blocks script execution by policy, run with `-ExecutionPolicy Bypass`:

```powershell
powershell -ExecutionPolicy Bypass -File .\run.ps1 -Demo
powershell -ExecutionPolicy Bypass -File .\run.ps1 -Test
powershell -ExecutionPolicy Bypass -File .\run.ps1 -Cli
```

### Manual Compile & Run (`javac` / `java`)

```powershell
mkdir out\web -Force | Out-Null
Copy-Item web\* out\web\ -Force
javac -encoding UTF-8 -d out (Get-ChildItem -Recurse src\*.java).FullName
java -cp out com.expensetracker.Main --data-dir data --demo --port 8080
```

Supported CLI flags for `com.expensetracker.Main`:
- `--web` — Start the web dashboard server (default)
- `--port <port>` — HTTP port (default `8080`; auto-falls back to `8081..8090` if busy)
- `--cli` — Run the terminal menu interface instead of the web server
- `--data-dir <path>` — Custom folder for CSV storage (default `./data`)
- `--demo` — Seed sample expenses and starter budgets if data is empty
- `--help` / `-h` — Show usage options

---

## REST API Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/meta` | App metadata, categories, payment methods, currency symbol |
| `GET` | `/api/expenses` | List/filter expenses (`from`, `to`, `category`, `method`, `q`, `min`, `max`) |
| `POST` | `/api/expenses` | Add a new expense and return any triggered budget alerts |
| `PUT` | `/api/expenses/{id}` | Update an existing expense |
| `DELETE` | `/api/expenses/{id}` | Delete an expense by ID |
| `GET` | `/api/summary?month=YYYY-MM` | Monthly KPIs, category/method totals, and budget statuses |
| `POST` | `/api/budgets` | Create or update an overall or category budget |
| `DELETE` | `/api/budgets?month=YYYY-MM&category=...` | Remove a monthly budget |
| `POST` | `/api/demo?month=YYYY-MM` | Seed sample expenses and budgets for the selected month |
| `POST` | `/api/settings` | Update currency symbol (`$`, `₹`, `€`, `£`) |
| `GET` | `/api/export` | Download all expenses as `expenses.csv` |

---

## Automated Tests

[ExpenseTrackerTest.java](test/com/expensetracker/ExpenseTrackerTest.java) runs **7 tests** with zero external test frameworks:
1. Expense validation and `BigDecimal` rounding
2. CRUD operations and CSV persistence round-trip
3. CSV escaping with commas and quotes
4. Search and multi-criteria filtering
5. Monthly summary and budget threshold alerts (`>= 80%` / `> 100%`)
6. Interactive CLI end-to-end flow
7. Web server HTML and REST API flow

Run with:
```cmd
run.bat test
```

---

## Sharing Locally via Cloudflare Quick Tunnel (Optional)

While the web server is running on `http://127.0.0.1:8080`, you can generate a temporary public HTTPS link using `cloudflared`:

```cmd
cloudflared tunnel --url http://127.0.0.1:8080
```

> **Note:** The prototype has no authentication layer; anyone with the tunnel URL can view and modify the tracker's data while the tunnel is active.
