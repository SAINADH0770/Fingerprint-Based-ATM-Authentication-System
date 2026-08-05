# Fingerprint-Based-ATM-Authentication-System


A Flask + MySQL web application that replaces the traditional ATM-card login flow with an uploaded fingerprint-image comparison, allowing users to sign up, log in, deposit, withdraw, and view their transaction history.

> **Note on scope of this README:** This project has no existing README, live deployment, CI/CD, Docker setup, or automated tests in the source. Every section below is based strictly on evidence found in the codebase (`Main.py`, `DB.txt`, `templates/`, `static/`, and the included project abstract document). Sections that could not be verified are explicitly marked rather than guessed.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Live Demo](#live-demo)
- [Screenshots](#screenshots)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Folder Structure](#folder-structure)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Usage Guide](#usage-guide)
- [API Overview](#api-overview)
- [Database Design](#database-design)
- [Security Notes](#security-notes)
- [Known Limitations](#known-limitations)
- [Future Enhancements](#future-enhancements)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)
- [FAQ](#faq)

---

## Project Overview

Fingerprint Based ATM System is a desktop-oriented web application (built with Flask) intended to demonstrate biometric-style authentication for ATM transactions instead of a physical card and PIN. Users sign up with an uploaded image (treated as their "fingerprint"), then log in by re-uploading the same image alongside their username and password. Once authenticated, users can deposit funds, withdraw funds (with balance checks), and view a transaction log.

This was built as an academic mini-project to explore the idea of biometric authentication in banking; it is **not** a production-grade fingerprint recognition system — see [Known Limitations](#known-limitations).

## Live Demo

No live deployment URL or GitHub repository URL was provided or found in the project. This app is currently designed to run locally only (see `run.bat`, which simply executes `python Main.py`).

## Screenshots

_Insert screenshots here, e.g.:_
- `docs/screenshot-login.png`
- `docs/screenshot-signup.png`
- `docs/screenshot-dashboard.png`
- `docs/screenshot-deposit.png`

(No screenshots were included in the uploaded project; add them to a `docs/` folder and reference them here.)

## Features

**Authentication**
- User signup with username, password, contact number, email, address, gender, and a fingerprint image upload
- Login via username + password + fingerprint image (compared byte-for-byte against the stored signup image)

**Banking**
- Deposit funds to an account
- Withdraw funds, with an insufficient-balance check
- View balance and transaction history

**Other**
- Logout (returns to the home page)

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3, Flask |
| Database | MySQL |
| DB Driver | PyMySQL |
| Templating | Jinja2 (Flask's built-in engine) |
| Frontend | HTML, CSS, vanilla JavaScript (client-side form validation) |
| Local run script | `run.bat` (Windows batch file) |

No ORM, testing framework, CI/CD pipeline, containerization, or cloud hosting configuration was found in the project.

## Architecture Overview

```
Browser (HTML forms)
   │
   ▼
Flask routes (Main.py)
   │  ├── /Signup, /SignupAction   → INSERT into `users`, save uploaded image to static/users/
   │  ├── /Login, /LoginAction     → SELECT from `users`, compare password + image bytes
   │  ├── /Deposit, /DepositAction → INSERT/UPDATE `transaction`
   │  ├── /Withdraw, /WithdrawAction → UPDATE `transaction` (with balance check)
   │  ├── /ViewBalance             → SELECT from `transaction`
   │  └── /Logout                  → renders index page
   │
   ▼
MySQL database `atm` (tables: users, transaction)
```

All routes live in a single `Main.py` file; there is no separate models, services, or config layer.

## Folder Structure

```
FingerprintATM/
├── Main.py                  # All Flask routes and business logic
├── DB.txt                   # SQL to create the `atm` database and its two tables
├── run.bat                  # Windows script: `python Main.py`
├── static/
│   ├── default.css          # Site-wide styling
│   ├── style.css
│   ├── tra.jpg               # Banner image used on multiple pages
│   ├── images/               # Misc. UI images
│   └── users/                # Fingerprint images uploaded at signup, named <username>.png
└── templates/
    ├── index.html            # Home page
    ├── Login.html            # Login form (username, password, fingerprint upload)
    ├── Signup.html            # Signup form
    ├── UserScreen.html       # Post-login dashboard/menu
    ├── Deposit.html          # Deposit form
    ├── Withdraw.html         # Withdraw form
    └── ViewBalance.html      # Transaction history table
```

## Installation

> These steps are inferred from the code's dependencies (`flask`, `pymysql`) and `DB.txt`; there is no `requirements.txt` in the project, so create one as shown below.

1. **Clone/copy the project** into a working directory.
2. **Install Python dependencies:**
   ```bash
   pip install flask pymysql
   ```
3. **Set up MySQL** and run the schema in `DB.txt`:
   ```bash
   mysql -u root -p < DB.txt
   ```
4. **Update database credentials** in `Main.py` (currently hardcoded — see [Environment Variables](#environment-variables) for the recommended fix) to match your local MySQL setup.
5. **Update the hardcoded file path** in `Main.py`'s `LoginAction()` and `SignupAction()` functions (currently an absolute Windows path under `C:/Users/saiku/...`) to a relative path such as `static/users/`.
6. **Run the app:**
   ```bash
   python Main.py
   ```
   Flask will start on `http://127.0.0.1:5000/` by default.

## Environment Variables

None are currently used — all configuration is hardcoded in `Main.py`. **Recommended** (not yet implemented in the code):

| Variable | Purpose |
|---|---|
| `DB_HOST` | MySQL host (currently hardcoded `127.0.0.1`) |
| `DB_PORT` | MySQL port (currently hardcoded `3306`) |
| `DB_USER` | MySQL username (currently hardcoded `root`) |
| `DB_PASSWORD` | MySQL password (currently hardcoded `root`) |
| `DB_NAME` | Database name (currently hardcoded `atm`) |
| `FLASK_SECRET_KEY` | Flask session secret (currently hardcoded `'welcome'`) |

Never commit real credentials — use a `.env` file with `python-dotenv` and add `.env` to `.gitignore`.

## Usage Guide

1. Go to `/Signup`, fill in your details, and upload an image file to act as your "fingerprint."
2. Go to `/Login`, enter the same username and password, and re-upload the **exact same image file** used at signup.
3. On success, you'll land on the user dashboard with links to Deposit, Withdraw, View Balance, and Logout.
4. Deposit or withdraw amounts via their respective forms; withdrawals are blocked if funds are insufficient.
5. View your last transactions via View Balance.

## API Overview

| Method | Endpoint | Purpose | Key Inputs | Auth |
|---|---|---|---|---|
| GET/POST | `/index` | Home page | — | None |
| GET/POST | `/Login` | Render login form | — | None |
| POST | `/LoginAction` | Authenticate user | username, password, fingerprint file | Password + file byte match |
| GET/POST | `/Signup` | Render signup form | — | None |
| POST | `/SignupAction` | Create new user | username, password, phone, email, address, gender, fingerprint file | None |
| GET/POST | `/Deposit` | Render deposit form | — | Requires prior login (`uname` set) |
| POST | `/DepositAction` | Add funds | username, amount | Requires prior login |
| GET/POST | `/Withdraw` | Render withdraw form | — | Requires prior login |
| POST | `/WithdrawAction` | Remove funds (if sufficient balance) | username, amount | Requires prior login |
| GET/POST | `/ViewBalance` | List transactions | — | Requires prior login |
| GET | `/Logout` | Return to home | — | None |

**Note:** Authentication state is tracked via a global Python variable (`uname`), not a Flask session — this is a functional/security issue, not just a style choice (see [Known Limitations](#known-limitations)).

## Database Design

```
users
├── username      VARCHAR(50)  PRIMARY KEY
├── password      VARCHAR(50)
├── contact_no    VARCHAR(12)
├── emailid       VARCHAR(50)
├── address       VARCHAR(60)
└── gender        VARCHAR(30)

transaction
├── username             VARCHAR(50)   (logically references users.username — no FK constraint defined)
├── transaction_amount   DOUBLE
├── transaction_type     VARCHAR(50)
├── transaction_date     TIMESTAMP
└── total_balance        DOUBLE
```

No foreign key, index (beyond the `users` primary key), or constraint enforces the relationship between the two tables — this should be added.

## Security Notes

This project has several issues that should be fixed before any real-world or public use:

- **SQL injection:** queries are built via string concatenation of raw form input in every DB-writing route. Use parameterized queries (`cur.execute(sql, (params,))`).
- **Plaintext passwords:** stored and compared with no hashing. Use `werkzeug.security.generate_password_hash` / `check_password_hash`.
- **Hardcoded secrets:** DB credentials (`root`/`root`) and Flask `secret_key` are hardcoded in source — move to environment variables.
- **Hardcoded absolute path:** the fingerprint file path is tied to one developer's machine and will fail elsewhere.
- **Auth via global variable:** `global uname` is not session-scoped and is not concurrency-safe; use `flask.session`.
- **No file validation:** uploaded images are written to disk with no type/size checks, and the filename is derived directly from user input — sanitize and validate before writing.
- **No CSRF protection** on any form.
- **"Fingerprint" matching is a literal byte comparison,** not real biometric verification — anyone with a copy of the original image file can log in as that user.

## Known Limitations

- Not a real biometric system — no minutiae extraction, no fingerprint-scanner hardware integration.
- No password hashing or salting.
- No automated tests, CI/CD, or containerization.
- No production deployment configuration.
- Session handling relies on a global variable rather than Flask's session mechanism.
- No pagination or limit on transaction history queries.

## Future Enhancements

- Integrate an actual fingerprint SDK/sensor (e.g., minutiae-based matching) instead of raw image comparison.
- Replace global-variable auth with `flask.session` and add proper login-required decorators.
- Parameterize all SQL queries and add an ORM (e.g., SQLAlchemy) for maintainability.
- Hash passwords and move all secrets/config to environment variables.
- Add a `requirements.txt`, automated tests, and a CI pipeline.
- Containerize with Docker for portable local/dev setup.

## Contributing

No `CONTRIBUTING.md` or contribution guidelines currently exist in the project. Suggested minimal workflow:
1. Fork the repository.
2. Create a feature branch.
3. Submit a pull request describing the change.

## License

No license file was found in the project. **Suggested:** MIT License, since this is a small educational/demo project where permissive reuse (with attribution) is typically desired. Add a `LICENSE` file at the project root if you choose to adopt it.

## Author

Author details were not specified in the project files or provided separately, beyond user-uploaded fingerprint images with informal names (e.g., in `static/users/`) that appear to be test data rather than author identity. Add your name, contact, and links here.

## FAQ

**Q: Does this project use real fingerprint scanning hardware?**
A: No — evidence in `Main.py` shows authentication compares the raw bytes of an uploaded image file against a previously stored image; there is no fingerprint sensor integration or minutiae extraction.

**Q: What database does it use?**
A: MySQL, accessed via the PyMySQL driver, per the connection calls throughout `Main.py` and the schema in `DB.txt`.

**Q: Is there a live demo?**
A: Not that was provided or found in the project; it currently runs locally via `run.bat` / `python Main.py`.

**Q: Are passwords encrypted?**
A: No — they are stored and compared as plain text in the current code.

**Q: Is there a mobile app version?**
A: No evidence of one — this is a Flask web app with server-rendered HTML templates only.

**Q: What happens if I withdraw more than my balance?**
A: `WithdrawAction()` checks `total > withdraw` before processing and returns an "Insufficient Fund" message otherwise.

**Q: Can two people be logged in at once?**
A: Not safely — since login state is stored in a single global Python variable (`uname`) rather than per-session, concurrent logins can interfere with each other.

**Q: How are transactions recorded?**
A: Each deposit/withdrawal updates or inserts a row in the `transaction` table with amount, type, timestamp, and resulting balance.

**Q: What frontend framework is used?**
A: None — plain HTML/CSS with a small amount of vanilla JavaScript for client-side form validation.

**Q: Is there an admin panel?**
A: No evidence of one in the routes or templates.

**Q: What Python version is required?**
A: Not explicitly pinned anywhere in the project; any Python 3.x version compatible with Flask and PyMySQL should work.

**Q: Is there rate limiting on login attempts?**
A: No evidence of any rate limiting or lockout mechanism.

**Q: Where are uploaded fingerprint images stored?**
A: In `static/users/<username>.png`, written directly by `SignupAction()`.

**Q: Does the app support money transfers between users?**
A: The project abstract mentions transfers as a goal, but no transfer route or logic exists in `Main.py` — only deposit and withdraw are implemented.

**Q: What license should I use if I want to share this publicly?**
A: MIT is a reasonable default for an educational project like this, though you should choose based on your own intentions for reuse.
