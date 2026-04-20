# InsightFlow — AI Data Analyst Platform

An AI-powered web dashboard where users upload CSV files, ask questions in natural
language, and receive instant charts, data tables, and AI-generated insights.


## Tech Stack

```
| Layer      | Technology                          |
|------------|-------------------------------------|
| Frontend   | React 18, Vite,Tailwind CSS,Chart.js|
| Backend    | Spring Boot 3.2, Java 21            |
| ORM        | Spring Data JPA + Hibernate         |
| Database   | PostgreSQL 16                       |
| AI         | OpenAI API (`gpt-4o-mini`)          |
| CSV        | Apache Commons CSV                  |
| Container  | Docker + Docker Compose             |
| Proxy      | Nginx                               |
```

## Project Structure

```
INSIGHTFLOW/
├── backend/
│   ├── src/main/java/com/insightflow/
│   │   ├── InsightflowApplication.java
│   │   ├── config/
│   │   │   ├── OpenAIConfig.java
│   │   │   ├── CorsConfig.java
│   │   │   └── WebConfig.java
│   │   ├── controller/
│   │   │   ├── UploadController.java
│   │   │   ├── QueryController.java
│   │   │   └── HealthController.java
│   │   ├── service/
│   │   │   ├── CSVService.java
│   │   │   ├── DataProcessingService.java
│   │   │   ├── AIService.java
│   │   │   ├── InsightService.java
│   │   │   └── QueryExecutionService.java
│   │   ├── dto/
│   │   │   ├── QueryRequest.java
│   │   │   ├── QueryResponse.java
│   │   │   ├── AIRequestDTO.java
│   │   │   ├── AIResponseDTO.java
│   │   │   ├── ChartResponseDTO.java
│   │   │   ├── UploadResponse.java
│   │   │   └── ErrorResponse.java
│   │   ├── model/
│   │   │   ├── Dataset.java
│   │   │   ├── DataRow.java
│   │   │   └── UserSession.java
│   │   ├── repository/
│   │   │   ├── DatasetRepository.java
│   │   │   └── DataRowRepository.java
│   │   ├── util/
│   │   │   ├── CSVParserUtil.java
│   │   │   ├── JSONUtil.java
│   │   │   └── ValidationUtil.java
│   │   ├── exception/
│   │   │   ├── GlobalExceptionHandler.java
│   │   │   ├── InvalidQueryException.java
│   │   │   ├── FileProcessingException.java
│   │   │   └── SessionNotFoundException.java
│   │   └── constants/
│   │       └── AppConstants.java
│   ├── src/main/resources/
│   │   ├── application.properties
│   │   ├── application-dev.properties
│   │   └── application-prod.properties
│   ├── Dockerfile
│   └── pom.xml
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── layout/   (Navbar, Footer)
│   │   │   ├── chat/     (ChatPanel, ChatMessage, ChatInput)
│   │   │   ├── upload/   (FileUpload, UploadBox)
│   │   │   ├── charts/   (Bar, Line, Pie)
│   │   │   ├── table/    (DataTable)
│   │   │   ├── summary/  (SummaryCard)
│   │   │   └── common/   (Button, Card, Loader, EmptyState)
│   │   ├── pages/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── Upload.jsx
│   │   │   └── Analytics.jsx
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   ├── uploadService.js
│   │   │   └── queryService.js
│   │   ├── hooks/
│   │   │   ├── useChat.js
│   │   │   └── useData.js
│   │   ├── context/
│   │   │   └── AppContext.jsx
│   │   └── utils/
│   │       ├── formatData.js
│   │       └── chartHelpers.js
│   ├── Dockerfile
│   └── package.json
│
├── nginx/
│   └── nginx.conf
├── docker-compose.yml
├── init.sql
├── .env.example
└── README.md
```

---

## Prerequisites

- Java 21+
- Maven 3.9+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 16 (local dev) or Docker
- OpenAI API key

---

## Quick Start — Docker (Recommended)

### 1. Clone and configure environment

```bash
git clone https://github.com/github-2346/insightflow.git
cd insightflow
cp .env.example .env
```

Edit `.env` and fill in your values:
```env
DB_PASSWORD=your_strong_password
OPENAI_API_KEY=sk-your-key-here
```

### 2. Build and start all services

```bash
docker compose up --build
```

### 3. Open the app

```
http://localhost:80
```

## Local Development (Without Docker)

### Backend

#### 1. Start PostgreSQL

```bash
# Using Docker for just the database
docker run -d \
  --name insightflow-postgres \
  -e POSTGRES_DB=insightflow_dev \
  -e POSTGRES_USER=insightflow \
  -e POSTGRES_PASSWORD=insightflow \
  -p 5432:5432 \
  postgres:16-alpine
```

#### 2. Set environment variables

```bash
export DB_USER=insightflow
export DB_PASSWORD=insightflow
export OPENAI_API_KEY=sk-your-key-here
```

#### 3. Run the backend

```bash
cd backend
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

Backend starts on `http://localhost:8080`

#### 4. Verify

```bash
curl http://localhost:8080/api/health
```

### Frontend

#### 1. Install dependencies

```bash
cd frontend
npm install
```

#### 2. Configure API URL

```bash
# frontend/.env.local
VITE_API_BASE_URL=http://localhost:8080/api
```

#### 3. Start dev server

```bash
npm run dev
```

Frontend starts on `http://localhost:5173`

---

## API Reference

### Upload CSV

```
POST /api/upload
Content-Type: multipart/form-data
```

| Field | Type   | Description              |
|-------|--------|--------------------------|
| file  | File   | CSV file (max 10 MB)     |

**Response 201:**
```json
{
  "sessionId": "550e8400-e29b-41d4-a716-446655440000",
  "filename": "sales-data.csv",
  "columns": ["product", "category", "revenue", "units", "month"],
  "numericColumns": ["revenue", "units"],
  "stringColumns": ["product", "category", "month"],
  "rowCount": 1200,
  "fileSizeBytes": 48320,
  "message": "File uploaded and processed successfully"
}
```

---

### Query Data

```
POST /api/query
Content-Type: application/json
```

**Request:**
```json
{
  "sessionId": "550e8400-e29b-41d4-a716-446655440000",
  "query": "Top 5 products by revenue"
}
```

**Response 200:**
```json
{
  "chartData": [
    { "product": "Laptop Pro", "revenue": 97300.0 },
    { "product": "Monitor 4K", "revenue": 61300.0 }
  ],
  "tableData": [
    { "product": "Laptop Pro", "revenue": 97300.0 },
    { "product": "Monitor 4K", "revenue": 61300.0 }
  ],
  "summary": "Laptop Pro leads with $97,300 in total revenue, representing 38.5% of the top 5. Monitor 4K follows at $61,300.",
  "chartType": "bar",
  "title": "Top 5 Products by Revenue",
  "query": "Top 5 products by revenue",
  "rowCount": 5
}
```

---

### Get Dataset Schema

```
GET /api/dataset/{sessionId}/schema
```

**Response 200:**
```json
{
  "sessionId": "550e8400...",
  "filename": "sales-data.csv",
  "columns": ["product", "category", "revenue"],
  "numericColumns": ["revenue"],
  "rowCount": 1200
}
```

---

### Delete Session

```
DELETE /api/dataset/{sessionId}
```

**Response 204 No Content**

---

### Health Check

```
GET /api/health
```

**Response 200:**
```json
{
  "status": "UP",
  "timestamp": "2025-06-01T10:30:00Z",
  "service": "InsightFlow API",
  "version": "1.0.0",
  "database": "UP"
}
```

---

## Error Responses

All errors follow a consistent shape:

```json
{
  "errorCode": "INVALID_QUERY",
  "message": "Column 'xyz' does not exist in dataset. Available: [product, revenue]",
  "timestamp": "2025-06-01T10:30:00Z"
}
```

## Supported Query Operations

| Operation         | Example Query                          | Chart  |
|-------------------|----------------------------------------|--------|
| `top_n`           | "Top 5 products by revenue"            | Bar    |
| `group_aggregate` | "Revenue by category"                  | Bar/Pie|
| `trend`           | "Monthly sales trend"                  | Line   |
| `distribution`    | "Order count distribution"             | Pie    |
| `filter`          | "Show sales where revenue > 10000"     | Table  |
| `sort`            | "Sort products by units descending"    | Table  |

---

## Configuration Reference

### Environment Variables

| Variable         | Required | Default         | Description                   |
|------------------|----------|-----------------|-------------------------------|
| `OPENAI_API_KEY` | Yes      | —               | OpenAI secret key             |
| `DB_USER`        | Yes      | `insightflow`   | PostgreSQL username           |
| `DB_PASSWORD`    | Yes      | —               | PostgreSQL password           |
| `DATABASE_URL`   | Prod     | —               | Full JDBC URL (prod only)     |
| `FRONTEND_URL`   | Prod     | `localhost:5173`| Allowed CORS origin           |
| `PORT`           | No       | `8080`          | Backend server port           |
