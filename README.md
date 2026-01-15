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

### Chat with Agent (Streaming)

```bash
POST /chat/stream
Content-Type: application/json

{
  "message": "Search the web for latest AI news",
  "session_id": "optional-session-id",
  "user_id": "user123"
}
```

**Response:** Server-Sent Events (SSE) stream

```
data: {"type": "session", "session_id": "abc-123-def-456"}

data: {"type": "content", "content": "Here's what I found"}

data: {"type": "content", "content": " about the latest AI news..."}

data: {"type": "done", "execution_time": 2.345}
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

### Streaming Chat Example

```bash
curl -N -X POST http://localhost:8000/chat/stream \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What are the latest developments in AI?",
    "user_id": "user1"
  }'
```

#### Stream Event Types

The streaming endpoint emits the following event types:

| Event Type      | Description                       | Example Data                                                                                                    |
| --------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `session`       | Session initialized               | `{"type": "session", "session_id": "abc-123"}`                                                                  |
| `tool_start`    | Agent started using a tool        | `{"type": "tool_start", "tool": "duckduckgo_search", "icon": "🔍", "action": "Searching the web"}`              |
| `tool_complete` | Agent completed using a tool      | `{"type": "tool_complete", "tool": "duckduckgo_search", "icon": "✅", "action": "Searching the web completed"}` |
| `content`       | Content chunk from agent response | `{"type": "content", "content": "Here is"}`                                                                     |
| `done`          | Stream completed                  | `{"type": "done", "execution_time": 2.345}`                                                                     |
| `error`         | Error occurred                    | `{"type": "error", "error": "error message"}`                                                                   |

#### JavaScript/TypeScript Implementation (Recommended)

**⚠️ IMPORTANT**: SSE chunks can be split across multiple reads. Always use a buffer to accumulate partial lines:

```javascript
// Using fetch API with proper buffering (RECOMMENDED)
async function streamChat(message, sessionId = null) {
    const response = await fetch("http://localhost:8000/chat/stream", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            message: message,
            session_id: sessionId,
            user_id: "user1",
        }),
    });

    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = ""; // CRITICAL: Buffer to handle partial chunks
    let currentSessionId = sessionId;

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        // Append new data to buffer
        buffer += decoder.decode(value, { stream: true });

        // Process complete lines from the buffer
        const lines = buffer.split("\n");
        // Keep the last incomplete line in the buffer
        buffer = lines.pop() || "";

        for (const line of lines) {
            if (line.trim() === "") continue; // Skip empty lines

            if (line.startsWith("data: ")) {
                try {
                    const data = JSON.parse(line.slice(6));

                    switch (data.type) {
                        case "session":
                            currentSessionId = data.session_id;
                            console.log("Session ID:", data.session_id);
                            break;

                        case "tool_start":
                            console.log(`${data.icon} ${data.action}...`);
                            break;

                        case "tool_complete":
                            console.log(`${data.icon} ${data.action}`);
                            break;

                        case "content":
                            // Stream content to UI
                            process.stdout.write(data.content);
                            break;

                        case "done":
                            console.log(
                                `\n✓ Completed in ${data.execution_time}s`
                            );
                            break;

                        case "error":
                            console.error("Error:", data.error);
                            throw new Error(data.error);
                    }
                } catch (parseError) {
                    console.error("Failed to parse SSE data:", parseError);
                }
            }
        }
    }

    return currentSessionId;
}

// Usage
const sessionId = await streamChat("What are the latest AI developments?");
// Continue conversation with session
await streamChat("Tell me more", sessionId);
```

#### React/Next.js Example

```typescript
import { useState } from "react";

export function ChatComponent() {
    const [response, setResponse] = useState("");
    const [isStreaming, setIsStreaming] = useState(false);
    const [sessionId, setSessionId] = useState<string | null>(null);

    const sendMessage = async (message: string) => {
        setIsStreaming(true);
        setResponse("");

        try {
            const res = await fetch("http://localhost:8000/chat/stream", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    message,
                    session_id: sessionId,
                    user_id: "user1",
                }),
            });

            const reader = res.body?.getReader();
            const decoder = new TextDecoder();
            let buffer = "";

            while (true) {
                const { done, value } = await reader!.read();
                if (done) break;

                buffer += decoder.decode(value, { stream: true });
                const lines = buffer.split("\n");
                buffer = lines.pop() || "";

                for (const line of lines) {
                    if (line.startsWith("data: ")) {
                        const data = JSON.parse(line.slice(6));

                        if (data.type === "session") {
                            setSessionId(data.session_id);
                        } else if (data.type === "content") {
                            setResponse((prev) => prev + data.content);
                        }
                    }
                }
            }
        } finally {
            setIsStreaming(false);
        }
    };

    return (
        <div>
            <div>{response}</div>
            <button onClick={() => sendMessage("Hello")} disabled={isStreaming}>
                {isStreaming ? "Streaming..." : "Send"}
            </button>
        </div>
    );
}
```

#### Python Implementation

```python
import requests
import json

def stream_chat(message: str, session_id: str = None) -> str:
    response = requests.post(
        'http://localhost:8000/chat/stream',
        json={
            'message': message,
            'session_id': session_id,
            'user_id': 'user1'
        },
        stream=True
    )

    current_session_id = session_id

    for line in response.iter_lines():
        if line:
            line = line.decode('utf-8')
            if line.startswith('data: '):
                data = json.loads(line[6:])

                if data['type'] == 'session':
                    current_session_id = data['session_id']
                    print(f"Session ID: {data['session_id']}")
                elif data['type'] == 'tool_start':
                    print(f"{data['icon']} {data['action']}...")
                elif data['type'] == 'tool_complete':
                    print(f"{data['icon']} {data['action']}")
                elif data['type'] == 'content':
                    print(data['content'], end='', flush=True)
                elif data['type'] == 'done':
                    print(f"\n✓ Completed in {data['execution_time']} seconds")
                elif data['type'] == 'error':
                    print(f"\n❌ Error: {data['error']}")

    return current_session_id

# Usage
session = stream_chat("What are the latest AI developments?")
# Continue conversation
stream_chat("Tell me more", session_id=session)
```

### 🎨 Frontend Testing

A complete HTML test page is included at `test_streaming.html`. To use it:

```bash
# Start the API server
uvicorn app.main:app --reload

# In another terminal, serve the HTML file
python3 -m http.server 8080

# Open in browser
# http://localhost:8080/test_streaming.html
```

The test page demonstrates:

-   ✅ Proper SSE stream buffering
-   ✅ Real-time content streaming
-   ✅ Tool execution status updates
-   ✅ Session management
-   ✅ Error handling
-   ✅ Visual feedback during streaming

### ⚠️ Common Frontend Pitfalls

#### ❌ WRONG: No buffering (will break with split chunks)

```javascript
// DON'T DO THIS - chunks can split mid-JSON
const chunk = decoder.decode(value);
const lines = chunk.split("\n");
for (const line of lines) {
    const data = JSON.parse(line.slice(6)); // ❌ Will fail randomly
}
```

#### ✅ CORRECT: Use buffer to accumulate partial lines

```javascript
// DO THIS - buffer handles partial chunks
let buffer = "";
buffer += decoder.decode(value, { stream: true });
const lines = buffer.split("\n");
buffer = lines.pop() || ""; // Keep incomplete line
for (const line of lines) {
    if (line.startsWith("data: ")) {
        const data = JSON.parse(line.slice(6)); // ✅ Safe
    }
}
```

#### ❌ WRONG: Not handling empty lines

```javascript
// Will try to parse empty strings
for (const line of lines) {
    if (line.startsWith("data: ")) {
        const data = JSON.parse(line.slice(6)); // ❌ Fails on ""
    }
}
```

#### ✅ CORRECT: Skip empty lines

```javascript
for (const line of lines) {
    if (line.trim() === "") continue; // ✅ Skip empty
    if (line.startsWith("data: ")) {
        const data = JSON.parse(line.slice(6));
    }
}
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
├── test_streaming.html        # Frontend streaming test page
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

### Streaming Not Working in Browser

**Symptom**: Content appears in `curl` but not in browser/frontend

**Cause**: SSE chunks can split across multiple reads, breaking JSON parsing

**Solution**: Always use a buffer to accumulate partial lines (see [Common Frontend Pitfalls](#️-common-frontend-pitfalls))

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

### CORS Issues

If testing from a different domain/port, ensure CORS is properly configured in `app/main.py`. The current setup allows all origins (`allow_origins=["*"]`) for development. In production, specify exact origins:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],  # Specify your domain
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
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
