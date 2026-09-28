# LinkVerse

LinkVerse is a Linktree-style "link in bio" app written in Python. You manage your profile and links in a **PyQt6 desktop dashboard**, and a **Flask** server running alongside it publishes your public profile page at `/u/<username>`. The dashboard gives you a shareable URL and a QR code for it, every link click is counted, and all data is stored in a local **SQLite** database with **bcrypt**-hashed passwords.

## Features
- **Accounts:** register and log in; passwords are hashed with bcrypt. Registration checks that all fields are filled in, the username is 3–20 characters (letters, digits and `_`), the email format is valid, the password has at least 8 characters including a digit, the display name is at most 50 characters, and the username and email are not already taken.
- **Link management:** add, edit, hide/show, delete (with confirmation) and reorder links (move up/down). URLs without a scheme get `https://` added automatically.
- **Link icons:** choose Facebook, Instagram, Twitter/X, GitHub, YouTube, LinkedIn or a default link icon for each link; the public page shows them with Font Awesome.
- **Profile:** edit your display name and bio, and upload a profile picture (PNG/JPG).
- **Sharing:** the dashboard shows your public profile URL with a Copy button and a QR code of it. The URL uses your computer's local network IP, so a phone on the same Wi-Fi can scan the code and open the page.
- **Public profile page:** `/u/<username>` shows the picture, name, bio and visible links; unknown users get a 404.
- **Click tracking:** public links go through `/click/<id>`, which counts the click and redirects to the target. Click counts are shown in the dashboard and next to each link on the public page. Hidden links are left off the page and their `/click` URL returns 404.
- **Dark / light mode** toggle in the dashboard.
- **Georgian error messages** for login and registration.
- **Safe data access:** all SQL queries are parameterized, and users can only edit, hide, delete or reorder their own links.

## Tech stack
Python 3 · PyQt6 · Flask · Jinja2 · SQLite (`sqlite3`) · bcrypt · qrcode + Pillow · Font Awesome (CDN)

## Architecture
| File | What it does |
|---|---|
| `main.py` | Entry point. Starts the Flask server in a background thread, then opens the PyQt6 window. |
| `gui.py` | PyQt6 interface: `MainWindow` with the logo header and a stacked widget that switches between `LoginPage`, `RegisterPage` and `DashboardPage` (profile, links, QR code, dark mode). |
| `logic.py` | `AppLogic`: registration and login, input validation, bcrypt hashing, a `must_be_logged_in` decorator, link operations with ownership checks, reordering, click recording and profile picture upload. |
| `database.py` | SQLite schema (`users`, `links`), `User` and `Link` models, and the `Database` data-access class. Also builds the public profile URL from the local network IP. |
| `web_server.py` | Flask routes: `/` (home page), `/u/<username>` (public profile) and `/click/<id>` (click tracking and redirect). |
| `templates/` | Jinja2 templates: `home.html` and `profile.html`. |
| `static/logo.png` | App logo, used in the desktop header and on the profile page. |

The SQLite database (`data/linktree.db`) and the `static/uploads/` folder for profile pictures are created automatically the first time they are needed.

## Run locally
Requires Python 3.9 or newer.

```bash
git clone https://github.com/ZMetreveli77/linkverse.git
cd linkverse
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

Create an account in the desktop app. The dashboard shows your public URL (`http://<your-local-IP>:5000/u/<your-username>`) and its QR code; on the same computer you can also open `http://127.0.0.1:5000/u/<your-username>`. The web server only runs while the desktop app is open. It listens on port 5000 on all network interfaces so other devices on your network can reach it, and your firewall may ask you to allow that.

## What I'd improve next
- **Configurable host and port:** port 5000 and the listen address are hard-coded, and the profile URL depends on detecting the local IP (it falls back to `127.0.0.1`).
- **Automated tests:** there are no tests yet; `logic.py` and the Flask routes are good candidates for pytest.
- **Clearer error handling:** errors are generic `Exception`s matched by message text, and some GUI handlers use bare `except` and only show "fail". Custom exception classes would make this more robust.
- **Mobile-friendly profile page:** the page has no viewport meta tag and uses fixed or percentage widths, so it looks cramped on the phones that scan the QR code.

## Credits
Started from the base project [vashaka/linkShare](https://github.com/vashaka/linkShare).
