# Agno AI Agent API

An intelligent AI agent powered by [Agno](https://github.com/agno-agi/agno) with web search and database capabilities. The agent uses **GPT OSS 120B** model via Groq and provides conversational access to web information with read-only access to Turso (SQLite) database.

## 🚀 Core Features

-   **🔍 Web Search**: Real-time web search using DuckDuckGo for up-to-date information
-   **💾 Database Queries**: Read-only access to Turso (SQLite) database using natural language
-   **💬 Persistent Chat History**: Conversation history stored locally in SQLite for continuity
-   **⚡ Async FastAPI**: High-performance asynchronous API built with FastAPI
-   **🔄 Session Management**: Multi-session support with automatic session tracking
-   **📊 Structured Responses**: Clean JSON responses with markdown-formatted content

## 🛠️ Technology Stack

-   **Framework**: FastAPI
-   **AI Agent**: Agno 2.3+
-   **LLM**: Groq (GPT OSS 120B)
-   **Database**: Turso (libSQL/SQLite)
-   **Package Manager**: uv (fast Python package installer)
-   **Search**: DuckDuckGo Search (ddgs)

## 📋 Prerequisites

-   Python 3.12+
-   Turso database (optional - for remote database access)
-   Groq API key
-   uv package manager (optional but recommended)

## 🔧 Setup and Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd 2Quip_agent
```

### 2. Install uv (if not already installed)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 3. Install Dependencies

Using uv:

```bash
uv sync
```

Or using pip:

```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create a `.env` file in the root directory:

```env
# Groq API Key (Required)
GROQ_API_KEY=your_groq_api_key_here

# Turso Database Configuration (Optional - for remote database access)
DATABASE_URL=libsql://your-database-name.turso.io
DATABASE_AUTH_TOKEN=your-turso-auth-token-here

# Server Configuration (optional)
HOST=0.0.0.0
PORT=8000
```

**Note**: By default, the agent uses a local SQLite database (`tmp/data.db`) for chat history. Turso configuration is only needed if you want to query a remote Turso database.

## 🚀 Running the Application

### Using uv (recommended):

```bash
uv run uvicorn app.main:app --reload
```

### Using standard Python:

```bash
python -m uvicorn app.main:app --reload
```

### Using Docker:

```bash
docker build -t agno-agent .
docker run -p 8000:8000 --env-file .env agno-agent
```

The API will be available at:

-   **API**: http://localhost:8000
-   **Interactive Docs**: http://localhost:8000/docs
-   **ReDoc**: http://localhost:8000/redoc

## 📡 API Endpoints

### Health Check

```bash
GET /health
```

### Chat with Agent

```bash
POST /chat
Content-Type: application/json

{
  "message": "Search the web for latest AI news",
  "session_id": "optional-session-id",
  "user_id": "user123"
}
```

**Response:**

```json
{
    "response": "Here's what I found about the latest AI news...",
    "session_id": "abc-123-def-456"
}
```

## 💡 Usage Examples

### Web Search Example

```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What are the latest developments in AI?",
    "user_id": "user1"
  }'
```

### Database Query Example (Read-Only)

```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What tables are in the database?",
    "session_id": "session123",
    "user_id": "user1"
  }'
```

### Continue Conversation

```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Can you elaborate on that?",
    "session_id": "session123",
    "user_id": "user1"
  }'
```

## 🏗️ Project Structure

```
2Quip_agent/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI application
│   ├── config/
│   │   ├── __init__.py
│   │   └── settings.py         # Configuration settings
│   └── services/
│       ├── __init__.py
│       └── agno_service.py     # Agent service implementation
├── .env                        # Environment variables (create this)
├── .env.example               # Example environment file
├── Dockerfile                 # Docker configuration
├── pyproject.toml            # Project dependencies
├── deploy.sh                 # Deployment script
└── README.md                 # This file
```

## 🔒 Security Notes

-   Never commit your `.env` file to version control
-   Keep your Groq API key secure
-   Database access is **read-only** by design for safety
-   Use environment variables for sensitive configuration
-   Implement rate limiting in production
-   Add authentication for production deployments

## 🐛 Troubleshooting

### Database Connection Issues

If using Turso, verify your `DATABASE_URL` and `DATABASE_AUTH_TOKEN` are correct. The local SQLite database (`tmp/data.db`) is created automatically.

### Missing Dependencies

```bash
# Reinstall all dependencies
uv sync --reinstall
```

### Port Already in Use

```bash
# Use a different port
uv run uvicorn app.main:app --port 8001
```

## 📚 Documentation

-   [Agno Documentation](https://docs.agno.com)
-   [FastAPI Documentation](https://fastapi.tiangolo.com)
-   [Groq API Documentation](https://console.groq.com/docs)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

[Add your license here]

## 👥 Authors

[Add your name/organization here]
