# Run Tests Locally

This guide explains how to **run automated tests locally** for both **backend APIs** (Postman + Newman) and **frontend UI** (Playwright + JS) for the *automation engineer test* project.

It covers:

- Cloning repositories
- Setting up environment variables
- Starting backend and frontend
- Running backend API tests
- Running frontend UI tests

---

## 1. Clone Repositories

You need three folders under the same parent folder:

```bash
# Parent folder example: qa-automation-technical-test
mkdir qa-automation-technical-test
cd qa-automation-technical-test

# Clone backend repo
git clone https://github.com/silumi95/automation-engineer-test-be.git backend

# Clone frontend repo
git clone https://github.com/silumi95/automation-engineer-test-fe.git frontend

# Clone automation repo (contains tests)
git clone https://github.com/silumi95/automation-engineer-test.git tests
```

## 2. Install Dependencies
Backend
```bash
cd backend
npm install
```

Frontend
```bash
cd ../frontend
npm install
```

Tests
```bash
cd ../tests/Frontend
npm install
```
## 3. Setup Environment Variables
Backend .env
Create a file backend/.env with:
```bash
MONGO_URI=mongodb://localhost:27017/qa_test_db
PORT=8000
NODE_ENV=development
JWT_SECRET=qa-test-secret
SUPPORT_EMAIL=test@gmail.com
GMAIL_USERNAME=test@gmail.com
GMAIL_PASSWORD=app-password
FRONTEND_URL=http://localhost:5173
```
Frontend .env
Create frontend/.env with:

```bash
VITE_API_BASE_URL=http://localhost:8000/api
```
## 4. Start Applications
Start Backend
```bash
cd backend
npm run dev
```

You should see:
```bash
Database connected
Listening on localhost:8000
```
Start Frontend
```bash
cd ../frontend
npm run dev
```

Frontend should be accessible at:
```bash
http://localhost:5173
```
## 5. Run Backend API Tests (Postman + Newman)
```bash
Install Newman
npm install -g newman
```

Execute API Tests
```bash
cd ../tests/backend-repo
newman run "E2E Shift Management.postman_collection.json" \
  --environment "QA Environment.postman_environment.json"
```

✅ This runs all backend API tests and prints results in terminal.


Optional: To Generate HTML report:
```bash
newman run "E2E Shift Management.postman_collection.json" \
  --environment "QA Environment.postman_environment.json" \
  -r html
```

HTML report will be saved in newman/ folder.

## 6. Run Frontend UI Tests (Playwright + JS)
Install Playwright (if not already installed)
```bash
cd ../Frontend
npm install -D @playwright/test
npx playwright install
```

Run All Frontend Tests
```bash
npx playwright test
```
Run a Single Test File
```bash
npx playwright test tests/login.spec.js
```


View HTML Report
```bash
npx playwright show-report
```

These will opens a browser showing screenshots/videos of failures and detailed results.

## 7. Quick Smoke Checks (Before Automation)

Make sure:

1. You can register a user via frontend.

2. You can login and receive JWT token.

3. Protected backend API endpoints block unauthenticated requests.

4. Backend is reachable at http://localhost:8000.

5. Frontend loads correctly at http://localhost:5173.

This ensures tests won’t fail due to broken setup.

---
#  Running All Tests in CI/CD (GitHub Actions)

This repository includes workflows for running **both backend and frontend tests** automatically in GitHub Actions:

- **Backend API tests** → `.github/workflows/api-tests.yml`
- **Frontend UI tests** → `.github/workflows/frontend.yml`

These workflows run automatically on:

- `push` to the repository  
- `pull_request`  

---

## 1. Backend API Tests

The backend workflow (`api-tests.yml`) runs:

- **Postman collection:**  
  `tests/backend-repo/E2E Shift Management.postman_collection.json`  
- **Postman environment:**  
  `tests/backend-repo/QA Environment.postman_environment.json`  

**Steps in CI:**

1. Checkout repository (`actions/checkout@v4`)  
2. Setup Node.js (`actions/setup-node@v3`)  
3. Install Newman globally (`npm install -g newman`)  
4. Run Postman collection headlessly:  

```bash
newman run "tests/backend-repo/E2E Shift Management.postman_collection.json" \
  --environment "tests/backend-repo/QA Environment.postman_environment.json" \
  -r cli,html
```
✅ Ensures backend APIs are tested automatically on every push/PR.
---
## 2. Frontend UI Tests

The frontend workflow (frontend.yml) runs:

Playwright JS tests located in tests/Frontend/

Steps in CI:

1. Checkout repository (actions/checkout@v4)

2. Setup Node.js (actions/setup-node@v3)

3. Install frontend dependencies and Playwright browsers:
```bash
cd frontend
npm install
npx playwright install
```

Run Playwright tests headless (non-headed):
```bash
cd frontend
npx playwright test
```

✅ Ensures frontend UI flows are tested automatically.

## 3. Triggering CI/CD

You can trigger the workflows by:

1. Pushing commits to any branch:
```bash
git add .
git commit -m "Update tests"
git push origin <branch-name>
```

2. Opening a pull request (automatically runs both workflows)
## 4. Viewing CI/CD Results

1. Go to the Actions tab in your GitHub repository

2. Select the workflow run (API Tests or Frontend UI Tests)

3. Inspect logs for pass/fail results
   
## 5. Notes

1. Backend tests run using Postman environment variables, so .env is optional in CI.

2. Frontend Playwright tests run headless by default.

3. For local debugging, you can run Playwright with:
```bash
npx playwright test --headed
```
---
#  Key Design Decisions

This section outlines the key design decisions made for the **QA Automation Technical Test** project, covering both backend and frontend automation.

---

### 1. Backend API Automation

- **Environment Management:**  
  - Postman environment file (`QA Environment.postman_environment.json`) stores variables like `baseUrl`, `JWT tokens`, and other dynamic data.  
  - Reduces dependency on `.env` files in CI/CD, making tests portable and reproducible.  
- **Test Coverage:**  
  - Covers **CRUD operations, authentication flows, and edge cases**.  
  - Pre-requisites (like registered users) are handled in the collection to **avoid manual setup**.

---

### 2. Frontend UI Automation

- **Folder Structure:**  
  - `tests/Frontend/` contains `.spec.js` files for each feature (login, dashboard, etc.)  
  - Modular structure improves maintainability and scalability.  
- **Headless Execution:**  
  - Default is **headless** for CI/CD to avoid UI popups.  
  - Optionally **headed** locally for debugging.

---

### 3. CI/CD Integration

- **Platform:** GitHub Actions  
  - Automatically runs **backend and frontend tests** on `push` and `pull_request`.  
  - Mirrors local testing steps for consistency.  
- **Reports:**  
  - Newman generates HTML reports for API tests.  
  - Playwright generates HTML reports with screenshots/videos for failed tests.  

---

### 4. Repository & Folder Structure

- **Separate folders for backend, frontend, and tests**:

- **Reason:** Clear separation of application code and automation code; simplifies CI/CD workflow and local testing.

---

### 5. Environment Management

- **Backend `.env`:** Used for local development (MongoDB URI, email credentials)  
- **Postman Environment:** Used in CI/CD and for backend API tests to **avoid hardcoding credentials**  
- **Frontend `.env`:** Stores API base URL for Playwright tests  
- **Reason:** Keeps sensitive data out of code, allows easy configuration changes.

---

### 6. Test Execution Strategy

- **Backend first:** Ensure API endpoints are working and database is connected  
- **Frontend second:** UI depends on API availability; tests reflect real user flows  
- **Reason:** Reduces flaky tests and mirrors production behavior.

---

### 7. Maintainability & Scalability

- Modular test files and clear folder structure  
- CI/CD workflows separate for backend and frontend for flexibility  
- Easy to add new features or tests without breaking existing ones  

---

✅ **Summary:**  

These design decisions ensure:

- Reliable, reproducible tests locally and in CI/CD  
- Clear separation of concerns between backend, frontend, and tests  
- Scalable, maintainable automation codebase  
- Easy debugging and visibility through reports
