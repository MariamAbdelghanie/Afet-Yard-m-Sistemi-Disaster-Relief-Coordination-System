# Afet Yardım Sistemi — Disaster Relief Coordination System

A web-based platform that connects people affected by a disaster with warehouses, volunteers, and drivers. Victims submit a help request with their location and needed supplies. The system scores each request by priority, matches it with supplier stock, records the deliveries, and shows everything on a live map.

The interface is in Turkish. Database and code identifiers are Turkish as well (for example `talepler` = requests, `teslimatlar` = deliveries, `tedarikciler` = suppliers).

---

## Table of contents

- [Features](#features)
- [User roles](#user-roles)
- [Screenshots](#screenshots)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [Database](#database)
- [Getting started](#getting-started)
- [Demo accounts](#demo-accounts)
- [Request lifecycle](#request-lifecycle)
- [Known limitations and roadmap](#known-limitations-and-roadmap)
- [License](#license)

---

## Features

**For people affected by a disaster (no login required)**
- Submit a help request with city, neighborhood, street/address, number of people, and urgency level (`Normal`, `Acil`, `CokAcil`).
- Pick the location on a map or use GPS to fill in latitude and longitude automatically.
- Choose needed supplies by category (food, water, medicine, hygiene, shelter) with quantities, plus a free-text "other needs" field.
- Track a request by its ID and see its status, deliveries, and route on a map.

**For administrators**
- City-based dashboard with requests, warehouse stock, and deliveries.
- Priority score for each request, calculated from the number of people, urgency level, and location.
- Interactive map with:
  - a heatmap of request priority,
  - warehouse markers with a stock popup,
  - dashed route lines from warehouse to request, colored by delivery status.

**For suppliers**
- Update stock levels per material and set a lead time.
- See the requests assigned to them.

**For volunteers**
- Select a city and view the open requests there (neighborhood, street, people, status).

**For drivers**
- See their own assigned shipments (request, status, transport mode, city).

**System**
- Status changes for requests and deliveries are written to an audit table (`durum_loglari`).
- Supports several transport modes: `Kara` (land), `Hava` (air), and `Deniz` (sea).
- Distinguishes local warehouses from regional ones.
- Coordinates are validated and clamped to Turkey's boundaries before being drawn on the map.

---

## User roles

| Role (DB value) | Panel | Access |
|---|---|---|
| Public / `Afetzede` | `index.php`, `track.php` | Create and track requests |
| `Admin` | `dashboard.php` and every other panel | Full access |
| `Tedarikci` | `supplier.php` | Stock and assigned requests |
| `Gonullu` | `volunteer.php` | Requests by city |
| `Surucu` | `driver.php` | Own shipments |

Role checks are handled by `requireRole([...])` in `config/auth.php`.

---

## Screenshots

<img width="1472" height="2232" alt="image" src="https://github.com/user-attachments/assets/0314dbf7-a0e2-497d-ad7c-cdc7a1bab140" />

<img width="1472" height="2020" alt="image" src="https://github.com/user-attachments/assets/e55d5660-d433-42fd-97d1-acb62aeec605" />

<img width="1472" height="1340" alt="image" src="https://github.com/user-attachments/assets/67675c1a-ebc2-45a3-b498-d60742e2206a" />


---

## Tech stack

- **Backend:** PHP 8.x with PDO (prepared statements)
- **Database:** MySQL / MariaDB (developed with MariaDB 10.4)
- **Frontend:** HTML, CSS, vanilla JavaScript
- **Maps:** [Leaflet](https://leafletjs.com/) 1.9.4, [Leaflet.heat](https://github.com/Leaflet/Leaflet.heat), OpenStreetMap tiles
- **UI:** Font Awesome 6.5, Inter font, Awesomplete (autocomplete)
- **Local environment:** XAMPP (Apache + MariaDB)

---

## Project structure

```
.
├── index.php              # Public request form
├── track.php              # Public request tracking
├── login.php              # Login and role-based redirect
├── dashboard.php          # Admin dashboard (tables + map)
├── supplier.php           # Supplier panel
├── volunteer.php          # Volunteer panel
├── driver.php             # Driver panel
├── api/                   # JSON/form endpoints (e.g. request_create.php)
├── config/
│   ├── auth.php           # Session start and requireRole()
│   └── db.php             # PDO connection ($pdo)
├── lib/
│   ├── Helpers.php
│   ├── Priority.php       # Priority score calculation
│   └── Geo.php            # Coordinate validation / Turkey bounds
├── templates/
│   ├── header.php         # Navbar and asset includes
│   └── footer.php
├── assets/
│   ├── css/style.css
│   ├── js/app.js
│   ├── js/i18n.js
│   └── icons/warehouse.png
└── database/
    └── afet_yardim_db.sql # Schema (and optional seed data)
```

---

## Database

Database name: `afet_yardim_db`

| Table | Purpose |
|---|---|
| `kullanicilar` | Users and their roles |
| `talepler` | Help requests (location, people count, urgency, status) |
| `talep_malzemeler` | Supplies requested for each request |
| `malzemeler` | Material catalog (category, unit) |
| `tedarikciler` | Warehouses/suppliers (city, coordinates, capacity, local or regional) |
| `stoklar` | Stock per supplier and material |
| `teslimatlar` | Deliveries linking a request, a supplier, a transport mode, and a driver |
| `rotalar` | Route data per delivery (start/end coordinates, distance, duration) |
| `durum_loglari` | Audit log of status changes |
| `yol_durumu` | Road conditions (for example closed roads) |

Relationships are enforced with foreign keys on `rotalar`, `stoklar`, `talepler`, `talep_malzemeler`, and `teslimatlar`.

---

## Getting started

### Prerequisites

- PHP 8.0 or newer
- MySQL or MariaDB
- Apache (XAMPP, WAMP, or similar)

### Installation

1. **Clone the repository** into your web server folder:
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   ```
   With XAMPP, place it under `htdocs/`.

2. **Create the database** and import the schema:
   ```sql
   CREATE DATABASE afet_yardim_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   ```
   Then import `database/afet_yardim_db.sql` with phpMyAdmin or the command line:
   ```bash
   mysql -u root -p afet_yardim_db < database/afet_yardim_db.sql
   ```

3. **Configure the connection.** `config/db.php` must expose a `$pdo` object:
   ```php
   <?php
   $pdo = new PDO(
       'mysql:host=127.0.0.1;dbname=afet_yardim_db;charset=utf8mb4',
       'root',
       '',
       [
           PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
           PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
       ]
   );
   ```
   Keep real credentials out of version control: add `config/db.php` to `.gitignore` and commit a `config/db.example.php` instead.

4. **Run the app.** Start Apache and MySQL, then open:
   ```
   http://localhost/<your-folder>/
   ```

---

## Demo accounts

The seed data contains one account per role for development. The passwords are defined in the seed data. Replace them before any real deployment.

| Role | Email |
|---|---|
| Admin | `admin@example.com` |
| Supplier | `supplier@example.com` |
| Volunteer | `vol@example.com` |
| Driver | `driver@example.com` |

---

## Request lifecycle

```
Beklemede  →  Tedarik Atandi  →  Planlandi  →  Hazirlaniyor  →  Teslim Edildi
(pending)     (supplier         (transport     (preparing)       (delivered)
               assigned)         planned)
```

Each delivery has its own status (for example `Planlandi`, `Başlatıldı`, `Teslim Edildi`), and every transition is written to `durum_loglari`.

---

## Known limitations and roadmap

This project started as a university project, so a few things need work before it is used anywhere real:

- **Passwords:** the login accepts plain-text passwords as a fallback, and the seed data stores them unhashed. Store only `password_hash()` values and remove the fallback.
- **Output escaping:** several pages print database values without `htmlspecialchars()` (tables, city options). This allows stored XSS.
- **Sensitive data:** the request form collects a national ID number (`TC Kimlik`) and phone number in plain text. Consider encryption or removing the field, and check the privacy rules that apply to you (KVKK/GDPR).
- **Status vocabulary:** status values are inconsistent (`Planlandi` and `Planlandı`, `Delivered` and `Teslim Edildi`). The route colors on the dashboard match only some of them. Use one fixed set of values or an enum.
- **Missing coordinates:** requests without GPS are stored as `0,0`, which produces wrong distances in `rotalar`. Store `NULL` and geocode the address instead.
- **CSRF protection:** forms and endpoints have no CSRF tokens.
- **Ideas:** live driver location, SMS/email notifications, road-closure aware routing using `yol_durumu`, English/Arabic UI through `i18n.js`, automated tests.

---

