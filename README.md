# 📦 Project Setup

# FastAPI Calculator with Authentication & Database

A full-stack calculator API built with FastAPI, PostgreSQL, and JWT authentication. Supports BREAD (Browse, Read, Edit, Add, Delete) operations on calculations, user registration/login, and polymorphic calculation models stored in a database.

## Prerequisites

Before you begin, install the following tools on your machine.

### 1. Python 3.9+

**macOS (Homebrew):**
```bash
brew install python@3.10
python3 --version
```

**Windows:**
- Download from [python.org](https://www.python.org/downloads/)
- During install, check **"Add Python to PATH"**
- Open a terminal and verify: `python --version`

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### 2. Visual Studio Code

- Download from [code.visualstudio.com](https://code.visualstudio.com/)
- Install the following extensions (search in Extensions sidebar):
  - **Python** (by Microsoft) — IntelliSense, linting, debugging
  - **Pylance** — type checking and auto-imports
  - **Docker** (by Microsoft) — manage containers from VS Code
  - **Thunder Client** or **REST Client** — test API endpoints (optional)

**Configure the Python interpreter in VS Code:**
1. Open the project folder in VS Code
2. Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
3. Type **"Python: Select Interpreter"**
4. Choose the one inside your `venv/` folder

### 3. Docker & Docker Compose

**macOS:**
- Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
- Install and start Docker Desktop from Applications
- Verify: `docker --version` and `docker compose version`

**Windows:**
- Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
- Enable WSL 2 backend during installation if prompted
- Verify: `docker --version` and `docker compose version`

**Linux:**
```bash
sudo apt install docker.io docker-compose-v2
sudo systemctl start docker
sudo usermod -aG docker $USER   # lets you run docker without sudo (log out/in after)
```

### 4. Git

**macOS:**
```bash
xcode-select --install   # installs git along with developer tools
```

**Windows:**
- Download from [git-scm.com](https://git-scm.com/download/win)

**Linux:**
```bash
sudo apt install git
```

## Project Setup

### Clone the repository

```bash
git clone <your-repo-url>
cd assignment12
```

### Create and activate a virtual environment

```bash
python3 -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Set up environment variables

Create a `.env` file in the project root:

```
DATABASE_URL=postgresql://postgres:postgres@localhost:5435/fastapi_db
JWT_SECRET_KEY=super-secret-key-for-jwt-min-32-chars
JWT_REFRESH_SECRET_KEY=super-refresh-secret-key-min-32-chars
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
BCRYPT_ROUNDS=12
```

> **Note:** When running inside Docker, `DATABASE_URL` uses `db:5432` (the container network). When running locally on your machine, it uses `localhost:5435` (the host-mapped port). The `.env` file is for local development.

## Running the Application

### Option A — Run everything with Docker (recommended)

```bash
docker compose up --build
```

This starts three services:
- **web** — FastAPI app at `http://localhost:8000`
- **db** — PostgreSQL database (host port 5435 → container port 5432)
- **pgadmin** — Database admin UI at `http://localhost:5052`

To stop:
```bash
docker compose down
```

To stop and wipe database data:
```bash
docker compose down -v
```

### Option B — Run locally (database in Docker)

Start only the database:
```bash
docker compose up -d db
```

Run the FastAPI app from your terminal:
```bash
source venv/bin/activate
uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

## Using Swagger UI

FastAPI generates interactive API documentation automatically. Open your browser and go to:

```
http://localhost:8000/docs
```

### Step 1 — Register a user

1. In Swagger UI, expand **POST /auth/register**
2. Click **"Try it out"**
3. Enter a JSON body like:
   ```json
   {
     "username": "testuser",
     "email": "test@example.com",
     "password": "",
     "confirm_password": "",
     "first_name": "Test",
     "last_name": "User"
   }
   ```
4. Click **"Execute"**
5. You should see a `201 Created` response with the user details

### Step 2 — Log in and get a token

1. Click the **Authorize** button (lock icon at the top right of the page)
2. Enter your `username` and `password`
3. Click **"Authorize"**, then **"Close"**
4. Swagger will now include your Bearer token in all subsequent requests

Alternatively, use the **POST /auth/login** endpoint directly:
1. Expand **POST /auth/login**
2. Click **"Try it out"**
3. Enter:
   ```json
   {
     "username": "testuser",
     "password": ""
   }
   ```
4. Click **"Execute"**
5. Copy the `access_token` from the response

### Step 3 — Create a calculation

1. Expand **POST /calculations**
2. Click **"Try it out"**
3. Enter:
   ```json
   {
     "type": "addition",
     "inputs": [10, 5, 3]
   }
   ```
4. Click **"Execute"**
5. You should see a `201 Created` response with:
   ```json
   {
     "id": "some-uuid-here",
     "type": "addition",
     "inputs": [10, 5, 3],
     "result": 18.0,
     ...
   }
   ```

Supported calculation types: `addition`, `subtraction`, `multiplication`, `division`

### Step 4 — Browse, read, update, delete

| Action | Method | Endpoint | Description |
|--------|--------|----------|-------------|
| Browse | GET | `/calculations` | List all your calculations |
| Read | GET | `/calculations/{id}` | Get a specific calculation by UUID |
| Edit | PUT | `/calculations/{id}` | Update inputs and recompute result |
| Delete | DELETE | `/calculations/{id}` | Remove a calculation |

## Using pgAdmin

1. Open `http://localhost:5052`
2. Log in with `admin@example.com` / `admin`
3. Register a new server:
   - **Host:** `db`
   - **Port:** `5432`
   - **Username:** `postgres`
   - **Password:** ``
4. Navigate to **fastapi_db → Schemas → public → Tables** to see `users` and `calculations`

> **Important:** Use host `db` (not `localhost`) because pgAdmin runs inside Docker on the same network as Postgres.

## Running Tests

### Unit tests

```bash
source venv/bin/activate
pytest tests/unit/ -v
```

### Integration tests

```bash
pytest tests/integration/ -v
```

### E2E tests (requires Playwright browsers)

```bash
playwright install chromium
pytest tests/e2e/ -v
```

### All tests with coverage

```bash
pytest --cov=app --cov-report=term-missing