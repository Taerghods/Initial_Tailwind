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


> 💡 Each component does one thing, and does it well.

---

## 🧰 Tech Stack

| Component | Technology |
|---|---|
| 🐍 Language                | Python 3.11.2           |
| 🌐 Web Framework           | Django                  |
| ⚡ API Framework           | FastAPI                 |
| 🗄️ Database                | MySQL                   |
| 🧬 ORM                     | SQLAlchemy 2.x          |
| 📡 Message Broker          | Redis Pub/Sub           |
| 📊 Market Data Source      | TSETMC                  |
| 🐳 Containerization        | Docker / Docker Compose |
| 📚 API Docs                | Swagger / OpenAPI       |

---

# 🔄 Data Pipeline

## 1️⃣ Data Collection

**Worker A** fetches real-time order-book data from TSETMC.

⏱️ Every **350 milliseconds** for **10 market symbols**.

Current symbols:
```
شپنا , شبندر , خودرو , شستا , وبملت , فملی , خساپا , ذوب , شتران
```

---

## 2️⃣ Redis Pub/Sub

**Worker A** publishes the data on the Redis channel:


```
DadeKavan-PD-X
```

Messages are sent as **structured JSON**:

```
{
  "name": "فولاد",
  "tsetmcId": "46348559193224090",
  "data": {
    "bestLimits": []
  },
  "time": "2026-09-22T08:07:38.815703"
}
```


**Worker B** subscribes to the same channel and consumes the incoming messages.

---

## 3️⃣ Data Persistence

**Worker B** asynchronously stores incoming messages in MySQL using **SQLAlchemy 2.x**.

✍️ Writes are performed inside **transactions** to guarantee data integrity.

Main table:

```
RTDS
```
|  🏷️ Column   | 📝 Description  |
|---|---|
| `id`                     | Unique identifier |
| `name`                   | Symbol name       |
| `tsetmc_id`              | TSETMC identifier |
| `data`                   | JSON payload      |
| `time`                   | Timestamp         |

Since the collection interval is 350ms, the `time` column uses:


```
DATETIME(3)
```



> ⏱️ This means millisecond precision — no timing detail is lost.

---

# 🌐 Django

Django provides the **human-facing** part of the project.

Base path:

```
/app
```


### 👤 User Management

The project uses a **custom user model** with **email-based** authentication.

Users can:

- ✅ Register
- ✅ Login
- ✅ Logout
- ✅ View profile
- ✅ Edit profile


Model:

```
user
```

|  🏷️ Column   | 📝 Description  |
|---|---|
| `Name`                   |       CharField       |
| `Email`                  |       EmailField      |
| `Password`               |       CharField       |
| `Profile_photo`          |       ImageField      |
| `User_level`             |    enum=[user,admin]  |
| `Active_status`          |      BooleanField     |
| `Staff_status`           |      BooleanField     |
| `Superuser_status`       |      BooleanField     |



The Django admin panel is used for user and permission management.

---

# ⚡ FastAPI

FastAPI provides the **machine-readable API** layer.

Base path:

```
/api
```

🔐 All endpoints are protected by a static **API Key** loaded from `.env`.

---

## 🛰️ API Endpoints

### 👤 User Profile

```
GET /api/users/get/profile/{userid}
```
Returns: profile information without credentials and without the profile image binary.

---

### 🖼️ User Photo

```
GET /api/users/get/photo/{userid}
```

Returns: the profile image as **Base64**.

---

### 📈 Current RTDS Data

```
GET /api/RTDS/get/current/{id}
```

The most recent stored record for the specified symbol.

---

### 📚 Historical RTDS Data

```
GET /api/RTDS/get/historical/{id}
```

All stored records for the specified symbol.

---

# 🧠 Design Decisions

| Decision | Description |
|---|---|
| 🔀 **Separation of Responsibilities** | Each component has a clear responsibility: Worker A collects market data, Worker B persists it, Django provides the human-facing interface, and FastAPI provides the machine-readable API. |
| 📡 **Redis Pub/Sub** | Redis Pub/Sub decouples data collection from database persistence, allowing Worker A to publish data without directly depending on MySQL. |
| ⚙️ **Asynchronous Persistence** | Worker B uses asynchronous database operations with SQLAlchemy 2.x, allowing database I/O to be handled without blocking the entire processing flow. |
| 🔒 **Transactions** | Database writes are performed inside transactions to maintain data consistency and ensure that each persistence operation is committed or rolled back as a unit. |
| ⏱️ **Millisecond Precision** | Data is collected every 350 milliseconds, so the `time` column uses `DATETIME(3)` to preserve millisecond-level timestamps. |
| 🐳 **Docker** | Docker and Docker Compose provide a consistent and reproducible environment for running the pipeline and its supporting services. |

---

# 🚀 Setup & Run

## 1️⃣ Configure Environment

The project runs on **Python 3.11.2**.

The `.env` file is located at the project root and contains MySQL, Redis, and API Key settings.

> 📌 The `.env` file is **intentionally committed to the repository** so it's available. 😎


## 2️⃣ Start the Services

```
docker compose up -d --build
```

Check running services:

```
docker compose ps
```

## 3️⃣ Django Database Setup

```
docker compose exec django python manage.py makemigrations
docker compose exec django python manage.py migrate
```

Create a superuser:

```
docker compose exec django python manage.py createsuperuser
```

## 4️⃣ Access the Applications

|    🖥️ Service     |                       🔗 URL                             |
| ----------------- | -------------------------------------------------------- |
| Django            | [http://127.0.0.1:6280/](http://127.0.0.1:6280/)         |
| FastAPI           | [http://127.0.0.1:6288/](http://127.0.0.1:6288/)         |
| Swagger           | [http://127.0.0.1:6288/docs](http://127.0.0.1:6288/docs) |

---

# 📁 Project Structure

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

---


# 🎬 Summary

The complete data flow can be summarized as:

```text
                         TSETMC
                            │
                            ▼
                       ┌─────────┐
                       │Worker A │
                       │Collector│
                       └────┬────┘
                            │
                            │ Structured JSON
                            ▼
                    ┌───────────────┐
                    │ Redis Pub/Sub │
                    └───────┬───────┘
                            │
                            ▼
                       ┌─────────┐
                       │Worker B │
                       │Persistence
                       └────┬────┘
                            │
                            ▼
                       ┌─────────┐
                       │  MySQL  │
                       │   RTDS  │
                       └────┬────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
          ┌──────────────┐      ┌──────────────┐
          │    Django    │      │   FastAPI    │
          │ Human-facing │      │Machine-facing│
          │   Web / UI   │      │   REST API   │
          └──────────────┘      └──────────────┘
```

**✨ From TSETMC market data to database and APIs — a complete real-time data pipeline. ✨**



