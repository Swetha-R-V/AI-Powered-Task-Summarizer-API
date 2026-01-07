# AI-Powered Task Summarizer API

## 📌 Problem Understanding & Assumptions

### Problem Understanding

The goal of this project is to build a robust RESTful API using **FastAPI** that acts as a bridge between a local database and an external API. The service demonstrates clean backend architecture, strict data validation, external API integration, and proper testing practices.

### Use Case Chosen

**AI-Powered Task Summarizer**

Users can create tasks with a title and description. The system processes the description using an external API to generate a short summary, stores everything in the database, and allows users to retrieve, update, or delete tasks.

### Assumptions

* Authentication and authorization are out of scope.
* Single-user system (no user accounts).
* External API is assumed to be available but may fail temporarily.
* **SQLite is used only for local development and testing. In a production environment, this application is designed to run on PostgreSQL by simply updating the database connection URL and installing the appropriate driver (e.g., psycopg2).**
* Only CRUD operations are required (exactly four endpoints).

---

## 🏗️ Design Decisions

### Database Schema

A single table `tasks` is used:

* `id` (Primary Key)
* `title` (String, required)
* `description` (String, required)
* `summary` (String, generated via external API)
* `created_at` (Timestamp)

This schema is simple, normalized, and sufficient for the use case.

### Project Structure

```
app/
 ├── main.py        # API routes & app initialization
 ├── database.py    # Database connection & session
 ├── models.py      # SQLAlchemy ORM models
 ├── schemas.py     # Pydantic request/response models
 ├── crud.py        # Database operations
 ├── external.py    # External API integration
 └── create_tables.py

tests/
 └── test_tasks.py  # Pytest-based API tests
```

This layered structure ensures separation of concerns and maintainability.

### Validation Logic

Pydantic models are used for:

* Request validation (`TaskCreate`, `TaskUpdate`)
* Response serialization (`TaskResponse`)

Field constraints (e.g., minimum length) ensure data integrity beyond basic type checking.

### External API Design

* A public external API is used to generate summary text.
* Timeouts are configured to avoid blocking requests.
* External failures are handled gracefully using HTTP status codes.

---

## 🔄 Solution Approach

### Data Flow

1. Client sends a request to the API.
2. FastAPI validates input using Pydantic schemas.
3. For task creation/update, an external API is called to generate a summary.
4. Data is stored in the database using SQLAlchemy.
5. A structured response is returned to the client.

---

## 📊 Key Insights & Learnings

* **Separation of Concerns Improves Maintainability:** Splitting logic into `schemas`, `crud`, `external`, and `routes` makes the codebase easier to test, extend, and debug.
* **External Dependencies Should Be Isolated:** Wrapping the external API in a dedicated module allows safe mocking in tests and graceful failure handling in production.
* **Validation at the Edge Reduces Bugs:** Pydantic validation ensures invalid data never reaches the database, reducing runtime errors.
* **Environment-Agnostic Database Design:** Using SQLAlchemy allows seamless switching from SQLite (development) to PostgreSQL (production) without code changes.
* **Testing with Mocks Increases Reliability:** Mocking the external API keeps tests deterministic and fast, which is critical for CI pipelines.
* **Clear Status Codes Improve API Usability:** Consistent use of HTTP status codes (201, 404, 422, 5xx) makes the API predictable for clients.

---

## ⚠️ Error Handling Strategy

* Global exception handler returns a consistent `500 Internal Server Error` for unexpected failures.
* External API errors return appropriate status codes (`502`, `503`).
* Missing resources return `404 Not Found`.
* Validation errors automatically return `422 Unprocessable Entity`.

---

## 🧪 Testing Strategy

* **Pytest** is used as the testing framework.
* **FastAPI TestClient** simulates API requests.
* External API calls are mocked using `unittest.mock.patch`.
* Tests cover:

  * Successful task creation
  * Handling of non-existent tasks

---

## ▶️ How to Run the Project

### Setup

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Create Tables

```bash
python app/create_tables.py
```

### Run Server

```bash
uvicorn app.main:app --reload
```

### API Documentation

Swagger UI is available at:

```
http://127.0.0.1:8000/docs
```

---

## 🌱 Future Improvements

* **Migrate from SQLite to PostgreSQL for production deployments to support scalability, concurrency, and reliability**
* Add authentication & authorization
* Improve test coverage
* Integrate real LLM-based summarization

---

## 🐘 PostgreSQL Production Switch (Explicit Note)

This project is intentionally structured to allow an easy switch from SQLite to PostgreSQL.

To use PostgreSQL in production:

1. Install the PostgreSQL driver:

   ```bash
   pip install psycopg2-binary
   ```
2. Update the database URL in `app/database.py`:

   ```python
   SQLALCHEMY_DATABASE_URL = "postgresql://username:password@host:port/database_name"
   ```
3. Run the table creation script or migrations.

No other code changes are required, as SQLAlchemy abstracts the database layer.

---

## ✅ Conclusion

This project demonstrates clean backend engineering practices, proper API design, external service integration, validation, testing, and clear documentation. The architecture is easily extensible and **production-ready with PostgreSQL** using minimal configuration changes.
