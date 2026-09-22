# DadeKavan-PD-X

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11.2-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-092E20?style=for-the-badge&logo=django&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

**A Real-Time Data Pipeline for the Tehran Stock Market**

*From TSETMC to API, in a fraction of a second.*

</div>

---

## 📖 What Is This Project?

**DadeKavan-PD-X** is a real-time pipeline that collects order-book data from **TSETMC**, publishes it through **Redis Pub/Sub**, asynchronously stores it in **MySQL**, and exposes it through **Django** and **FastAPI**.

> 🎯 10 symbols, every 350 milliseconds, without losing a single millisecond.

---

## 🏗️ Architecture

```text
                         ┌─────────────────┐
                         │     TSETMC      │
                         │  Market Data    │
                         └────────┬────────┘
                                  │
                              350ms ⏱️
                                  │
                                  ▼
                         ┌─────────────────┐
                         │    Worker A     │
                         │ Data Collector  │
                         └────────┬────────┘
                                  │
                           Structured JSON
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Redis Pub/Sub   │
                         │ DadeKavan-PD-X  │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │    Worker B     │
                         │Async Persistence│
                         └────────┬────────┘
                                  │
                             Transaction
                                  │
                                  ▼
                         ┌─────────────────┐
                         │      MySQL      │
                         │      RTDS       │
                         └────────┬────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             ┌──────────────┐            ┌──────────────┐
             │    Django    │            │   FastAPI    │
             │  Web / Admin │            │  REST API    │
             └──────────────┘            └──────────────┘
````

svgsvg

> 💡 Each component does one thing, and does it well.

---

## 🧰 Tech Stack

| Component | Technology |
|---|---|
| 🐍 Language                | Python 3.11.2           |
| 🌐 Web Framework           | Django                  |
| ⚡ API Framework            | FastAPI                 |
| 🗄️ Database               | MySQL                   |
| 🧬 ORM                     | SQLAlchemy 2.x          |
| 📡 Message Broker          | Redis Pub/Sub           |
| 📊 Market Data Source      | TSETMC                  |
| 🐳 Containerization        | Docker / Docker Compose |
| 📚 API Docs                | Swagger / OpenAPI       |

---

## ⚙️ Environment

The project runs on **Python 3.11.2**.

All sensitive configuration (MySQL, Redis, API Key, etc.) is loaded from the `.env` file.

> 📌 The `.env` file is **intentionally committed to the repository** so it's always available and never lost. 😎

---

# 🔄 Data Pipeline

## 1️⃣ Data Collection

**Worker A** fetches real-time order-book data from TSETMC.

⏱️ Every **350 milliseconds** for **10 market symbols**.

Current symbols:

text

svg

Copy

svg

Download

```
فولاد
شپنا
شبندر
خودرو
شستا
وبملت
فملی
خساپا
ذوب
شتران
```

svgsvg

---

## 2️⃣ Redis Pub/Sub

**Worker A** publishes the data on the Redis channel:

text

svg

Copy

svg

Download

```
DadeKavan-PD-X
```

svgsvg

Messages are sent as **structured JSON**:

json

svg

Copy

svg

Download

```
{
  "name": "فولاد",
  "tsetmc_id": "46348559193224090",
  "data": {
    "bestLimits": []
  },
  "time": "2026-09-22T08:07:38.815703"
}
```

svgsvg

**Worker B** subscribes to the same channel and consumes the incoming messages.

---

## 3️⃣ Data Persistence

**Worker B** asynchronously stores incoming messages in MySQL using **SQLAlchemy 2.x**.

✍️ Writes are performed inside **transactions** to guarantee data integrity.

Main table:

text

svg

Copy

svg

Download

```
RTDS
```

svgsvg

Columns:

| 🏷️ Column📝 Description |                   |
| ------------------------ | ----------------- |
| `id`                     | Unique identifier |
| `name`                   | Symbol name       |
| `tsetmc_id`              | TSETMC identifier |
| `data`                   | JSON payload      |
| `time`                   | Timestamp         |

Since the collection interval is 350ms, the `time` column uses:

sql

svg

Copy

svg

Download

```
DATETIME(3)
```

svgsvg

> ⏱️ This means millisecond precision — no timing detail is lost.

---

# 🌐 Django

Django provides the **human-facing** part of the project.

Runs at:

text

svg

Copy

svg

Download

```
http://127.0.0.1:6280/
```

svgsvg

Under the path:

text

svg

Copy

svg

Download

```
/app
```

svgsvg

### 👤 User Management

The project uses a **custom user model** with **email-based** authentication.

Users can:

- ✅ Register
- ✅ Login
- ✅ Logout
- ✅ View profile
- ✅ Edit profile

The user model includes:

- 📛 Name
- 📧 Email
- 🔒 Password
- 🖼️ Profile photo
- 🏅 User level
- 🟢 Active status
- 👔 Staff status
- 👑 Superuser status
- 🔑 Django permissions

Two user levels are defined:

text

svg

Copy

svg

Download

```
user
admin
```

svgsvg

The Django admin panel is used for user and permission management.

---

# ⚡ FastAPI

FastAPI provides the **machine-readable API** layer.

Runs at:

text

svg

Copy

svg

Download

```
http://127.0.0.1:6288/
```

svgsvg

Base path:

text

svg

Copy

svg

Download

```
/api
```

svgsvg

Swagger / OpenAPI docs:

text

svg

Copy

svg

Download

```
http://127.0.0.1:6288/docs
```

svgsvg

🔐 All endpoints are protected by a static **API Key** loaded from `.env`.

Required header:

text

svg

Copy

svg

Download

```
X-API-Key
```

svgsvg

---

## 🛰️ API Endpoints

### 👤 User Profile

http

svg

Copy

svg

Download

```
GET /api/users/get/profile/{userid}
```

svgsvg

Returns: profile information without credentials and without the profile image binary.

---

### 🖼️ User Photo

http

svg

Copy

svg

Download

```
GET /api/users/get/photo/{userid}
```

svgsvg

Returns: the profile image as **Base64**.

---

### 📈 Current RTDS Data

http

svg

Copy

svg

Download

```
GET /api/RTDS/get/current/{id}
```

svgsvg

The most recent stored record for the specified symbol.

---

### 📚 Historical RTDS Data

http

svg

Copy

svg

Download

```
GET /api/RTDS/get/historical/{id}
```

svgsvg

All stored records for the specified symbol.

---

# 🧠 Design Decisions

\<table> \<tr> \<td>🔀 \<b>Separation of Responsibilities\</b>\</td> \<td>Each component has a single responsibility: Worker A collects, Worker B persists, Django handles the human side, FastAPI serves the API layer.\</td> \</tr> \<tr> \<td>📡 \<b>Redis Pub/Sub\</b>\</td> \<td>Decouples data collection from database persistence.\</td> \</tr> \<tr> \<td>⚙️ \<b>Asynchronous Persistence\</b>\</td> \<td>Worker B uses async operations with SQLAlchemy 2.x.\</td> \</tr> \<tr> \<td>🔒 \<b>Transactions\</b>\</td> \<td>Writes are transactional to ensure data integrity.\</td> \</tr> \<tr> \<td>⏱️ \<b>Millisecond Precision\</b>\</td> \<td>The 350ms interval requires sub-second accuracy, hence \<code>DATETIME(3)\</code>.\</td> \</tr> \<tr> \<td>🐳 \<b>Docker\</b>\</td> \<td>Consistent and reproducible execution of the entire system.\</td> \</tr> \</table>

---

# 🚀 Setup & Run

## 1️⃣ Configure Environment

The `.env` file is located at the project root and contains MySQL, Redis, and API Key settings.

## 2️⃣ Start the Services

bash

svg

Copy

svg

Download

```
docker compose up -d --build
```

svgsvg

Check running services:

bash

svg

Copy

svg

Download

```
docker compose ps
```

svgsvg

## 3️⃣ Django Database Setup

bash

svg

Copy

svg

Download

```
docker compose exec django python manage.py makemigrations
docker compose exec django python manage.py migrate
```

svgsvg

Create a superuser:

bash

svg

Copy

svg

Download

```
docker compose exec django python manage.py createsuperuser
```

svgsvg

## 4️⃣ Access the Applications

| 🖥️ Service🔗 URL |                                                          |
| ----------------- | -------------------------------------------------------- |
| Django            | [http://127.0.0.1:6280/](http://127.0.0.1:6280/)         |
| FastAPI           | [http://127.0.0.1:6288/](http://127.0.0.1:6288/)         |
| Swagger           | [http://127.0.0.1:6288/docs](http://127.0.0.1:6288/docs) |

---

# 📁 Project Structure

text

svg

Copy

svg

Download

```
DadeKavan-PD-X/
│
├── WorkerA/
│   ├── Dockerfile
│   └── ...
│
├── WorkerB/
│   ├── Dockerfile
│   └── ...
│
├── Django/
│   ├── Dockerfile
│   ├── manage.py
│   ├── src/
│   │   ├── settings.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   │
│   └── users/
│       ├── migrations/
│       ├── admin.py
│       ├── models.py
│       ├── permissions.py
│       ├── urls.py
│       └── views.py
│
├── FastAPI/
│   ├── Dockerfile
│   ├── main.py
│   ├── database.py
│   ├── dependencies.py
│   ├── users.py
│   └── rtds.py
│
├── docker-compose.yml
├── requirements.txt
├── .gitignore
└── README.md
```

svgsvg

---

# 🎬 Summary

\<div align="center">

text

svg

Copy

svg

Download

```
TSETMC
   │
   ▼
Worker A
   │
   ▼
Redis Pub/Sub
   │
   ▼
Worker B
   │
   ▼
MySQL / RTDS
   │
   ├──────────────► Django
   │
   └──────────────► FastAPI
```

svgsvg

**✨ From the market to the API — all real-time, all integrated ✨**

\</div>

---

\<div align="center">

Built with ❤️ and a bit of ☕

\</div> \`\`\`
