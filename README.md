# RefractorIQ

**AI-Powered Code Analysis and Technical Debt Detection**

## Overview

RefractorIQ is a comprehensive code analysis platform that combines static analysis, dependency mapping, duplication detection, semantic search, and AI-powered refactoring suggestions. It helps development teams understand their codebase structure, identify technical debt, and receive intelligent recommendations for code improvements.

The platform analyzes repositories using:
* **Tree-sitter AST parsing** for multi-language code analysis
* **NetworkX** for dependency graph construction
* **MinHash/SimHash** for code duplication detection
* **FAISS + SentenceTransformers** for semantic code search
* **Google Gemini AI** for intelligent refactoring suggestions

The entire system runs asynchronously with Celery workers, providing real-time progress updates through a modern React dashboard.

## Quick Start

### Prerequisites
* Python 3.10+
* Node.js 16+
* Redis (for task queue)
* PostgreSQL (for database)

### Using Docker Compose

```bash
# Start supporting services
docker-compose up -d

# Install Python dependencies
pip install -r requirements.txt

# Start Celery worker
celery -A backend.celery_app worker --loglevel=info

# Start FastAPI server
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### Start the Frontend

```bash
cd frontend
npm install
npm run dev
```

### Access the Application
* **Web Interface**: http://localhost:3000
* **API Documentation**: http://localhost:8000/docs
* **Health Check**: http://localhost:8000/health

## API Usage

### Start Repository Analysis
**Endpoint**: `GET /analyze/full`

**Parameters**:
* `repo_url` - GitHub repository URL
* `exclude_third_party` - Skip third-party libraries (default: true)
* `exclude_tests` - Skip test files (default: true)

**Example**:
```bash
curl -X GET "http://localhost:8000/analyze/full?repo_url=https://github.com/user/repo.git&exclude_third_party=true&exclude_tests=true"
```

**Response**:
```json
{
  "message": "Analysis started",
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status_url": "/analyze/status/550e8400-e29b-41d4-a716-446655440000"
}
```

### Check Analysis Status
**Endpoint**: `GET /analyze/status/{job_id}`

**Example**:
```bash
curl http://localhost:8000/analyze/status/550e8400-e29b-41d4-a716-446655440000
```

**Response**:
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "COMPLETED",
  "repo_url": "https://github.com/user/repo.git",
  "results_url": "/analyze/results/550e8400-e29b-41d4-a716-446655440000",
  "summary": {
    "loc": 15234,
    "debt_score": 87.5,
    "avg_complexity": 5.2,
    "duplicate_pairs": 12
  }
}
```

### Get Analysis Results
**Endpoint**: `GET /analyze/results/{job_id}`

**Example**:
```bash
curl http://localhost:8000/analyze/results/550e8400-e29b-41d4-a716-446655440000
```

**Response**:
```json
{
  "repository": "https://github.com/user/repo.git",
  "code_metrics": {
    "LOC": 15234,
    "TODOs_FIXME_HACK": 42,
    "AvgCyclomaticComplexity": 5.2,
    "DebtScore": 87.5,
    "TotalFunctions": 487,
    "MaxComplexity": 28,
    "ComplexityDistribution": {
      "low": 312,
      "medium": 143,
      "high": 28,
      "very_high": 4
    }
  },
  "dependency_metrics": {
    "total_files": 156,
    "total_edges": 423,
    "circular_dependencies": 2,
    "most_dependent_files": [...],
    "graph_json": {...}
  },
  "duplication_metrics": {
    "duplicate_pairs_found": 12,
    "similarity_threshold": 0.85,
    "duplicates": [...]
  },
  "llm_suggestions": [...]
}
```

### Semantic Code Search
**Endpoint**: `GET /analyze/results/{job_id}/search`

**Parameters**:
* `q` - Search query (natural language)
* `k` - Number of results (default: 10)

**Example**:
```bash
curl "http://localhost:8000/analyze/results/550e8400-e29b-41d4-a716-446655440000/search?q=authentication+logic&k=5"
```

**Response**:
```json
{
  "results": [
    {
      "path": "backend/auth/login.py",
      "score": 0.8923
    },
    {
      "path": "backend/middleware/auth.py",
      "score": 0.8456
    }
  ]
}
```

## How It Works

1. **Repository Cloning** - Shallow clone of the target repository into a temporary directory
2. **Static Analysis** - Tree-sitter parses code files to extract metrics (LOC, complexity, TODOs)
3. **Dependency Graph** - NetworkX builds a directed graph of file dependencies and imports
4. **Duplication Detection** - MinHash fingerprinting identifies similar code blocks
5. **Semantic Indexing** - SentenceTransformer creates FAISS vector index for search
6. **AI Analysis** - Google Gemini generates refactoring suggestions for complex functions
7. **Report Assembly** - All metrics are aggregated into a JSON report and stored locally
8. **Status Updates** - Real-time job status polling via FastAPI endpoints

## Features

### Code Quality Metrics
* Lines of code (excluding comments/blanks)
* Cyclomatic complexity analysis
* Technical debt scoring
* TODO/FIXME/HACK detection
* Complexity distribution visualization

### Dependency Analysis
* File-level dependency graphs
* Import relationship mapping
* Circular dependency detection
* Most dependent/depended-on file identification
* Interactive ReactFlow visualization

### Code Duplication
* MinHash-based similarity detection
* Configurable similarity thresholds
* Side-by-side duplicate comparison
* Jaccard similarity scoring

### Semantic Search
* Natural language code search
* Vector similarity using FAISS
* Fast retrieval across entire codebase
* Context-aware file ranking

### AI Refactoring
* Automatic detection of complex functions
* Google Gemini-powered suggestions
* Side-by-side code comparison
* Complexity-driven prioritization

## Project Purpose

RefractorIQ was built to demonstrate a production-ready code analysis platform that combines traditional static analysis with modern AI capabilities. The project showcases:

* **Async Architecture** - Celery task queue with Redis backend for scalable analysis
* **Multi-Language Support** - Tree-sitter parsers for Python, JavaScript/TypeScript, and Java
* **Graph Algorithms** - NetworkX for dependency analysis and circular dependency detection
* **Vector Search** - FAISS indexing for semantic code search
* **LLM Integration** - Google Gemini API for intelligent refactoring suggestions
* **Modern Frontend** - React with TailwindCSS and ReactFlow for interactive visualizations
* **REST API** - FastAPI with OpenAPI documentation and async endpoints
* **Database Management** - SQLAlchemy ORM with PostgreSQL for job persistence
* **Deployment Ready** - Docker, Railway, and Nixpacks configuration included

It serves as a comprehensive example of building and deploying a full-stack ML-powered application with real-world complexity.

## Tech Stack

* **Backend**: FastAPI, Celery, SQLAlchemy, PostgreSQL, Redis
* **Analysis**: Tree-sitter, NetworkX, MinHash, SimHash
* **AI/ML**: SentenceTransformers, FAISS, Google Generative AI
* **Frontend**: React 18, Vite, TailwindCSS, ReactFlow
* **Deployment**: Docker, Railway, Uvicorn

## Environment Configuration

Create a `.env` file:

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/refractor_db

# Celery
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# GitHub (optional)
GITHUB_CLIENT_ID=your_client_id
GITHUB_CLIENT_SECRET=your_client_secret
GITHUB_PAT=your_personal_access_token

# Google AI
GOOGLE_API_KEY=your_google_api_key

# Application
BACKEND_URL=http://localhost:8000
REPORT_STORAGE_PATH=./analysis_reports
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request