# Architecture Specification: Phase III Todo AI Chatbot

## Overview

This document describes the stateless architecture of the Phase III Todo AI Chatbot, focusing on how the system maintains zero in-memory state while providing a seamless conversational experience.

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ ChatInterface → AIChatInput → useAI Hook         │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────┬──────────────────────────────────┘
                      │ HTTP POST /api/{user_id}/chat
                      ▼
┌─────────────────────────────────────────────────────────┐
│              FastAPI Backend (Stateless)                 │
│  ┌──────────────────────────────────────────────────┐   │
│  │ routes/chat.py                                    │   │
│  │  @router.post("/api/{user_id}/chat")             │   │
│  └──────────────────────────────────────────────────┘   │
│                      ▼                                    │
│  ┌──────────────────────────────────────────────────┐   │
│  │ OpenAI Agents SDK                                 │   │
│  │  - Parse user message                            │   │
│  │  - Access conversation history from DB           │   │
│  │  - Determine tool needs                          │   │
│  │  - Generate response                             │   │
│  └──────────────────────────────────────────────────┘   │
│                      ▼                                    │
│  ┌──────────────────────────────────────────────────┐   │
│  │ MCP Tool Layer (Stateless)                        │   │
│  │  - add_task                                       │   │
│  │  - list_tasks                                     │   │
│  │  - update_task                                    │   │
│  │  - complete_task                                 │   │
│  │  - delete_task                                   │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────┬──────────────────────────────────┘
                      │ SQL queries
                      ▼
┌─────────────────────────────────────────────────────────┐
│           PostgreSQL Database (Source of Truth)          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Tables:                                           │   │
│  │  - users                                         │   │
│  │  - tasks (user_id, title, due_date, priority)   │   │
│  │  - conversations (user_id, title)               │   │
│  │  - messages (conversation_id, content, role)    │   │
│  │  - message_tool_calls (message_id, tool, args)  │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Stateless Design Philosophy

### Principle 1: No In-Memory State

The agent and server maintain **zero** task/conversation state in memory.

```
❌ WRONG (stateful):
- Server holds task list in memory
- Agent remembers previous API responses
- Session state stored in memory cache

✅ CORRECT (stateless):
- Every request fetches full state from database
- Agent operates on fresh data per request
- No server-local caching of user data
```

### Principle 2: Database as Single Source of Truth

All application state lives in PostgreSQL:

```
POST /api/42/chat
├─ Request arrives with user_id=42, message="Add milk to my list"
├─ Fetch: SELECT * FROM conversations WHERE user_id=42 ORDER BY updated_at DESC LIMIT 1
├─ Fetch: SELECT * FROM messages WHERE conversation_id=? ORDER BY created_at
│  (This is the conversation history passed to the agent)
├─ Agent processes with fresh data
├─ Agent calls MCP tools → INSERT/UPDATE tasks table
├─ Agent generates response
├─ INSERT message (role=assistant) into messages table
├─ RETURN response to client
└─ All state persisted to DB, nothing held in memory
```

### Principle 3: Stateless Tool Execution

MCP tools follow strict stateless contract:

```python
# ✅ Correct tool signature
async def add_task(user_id: int, title: str, due_date: Optional[str]) -> Dict:
    """
    No access to:
    - Instance variables self.tasks
    - Global state
    - Memory caches
    
    Only:
    - Database reads/writes
    - Input parameters
    - System time
    """
    # Validate inputs
    # Query database for user existence
    # INSERT into tasks table
    # Return response
    return {"status": "success", "data": {...}}
```

### Principle 4: Request Isolation

Each API request is completely independent:

```
Request #1: User A, message "Add task"
├─ Fetch user A's data from DB
├─ Process
└─ No residual state

Request #2: User B, message "List tasks" (processed in parallel)
├─ Fetch user B's data from DB
├─ Process
└─ No cross-contamination

Request #3: User A again, message "Mark complete"
├─ Re-fetch user A's data from DB (has latest state)
├─ Process with fresh data
└─ No stale state from Request #1
```

---

## Request Flow: Detailed Walkthrough

### User sends message: "Add a task to buy milk tomorrow at 3pm"

#### Step 1: Receive Request
```http
POST /api/42/chat
Content-Type: application/json

{
  "message": "Add a task to buy milk tomorrow at 3pm"
}
```

**Handler**: `routes/chat.py`
- Verify JWT token
- Extract user_id from token (42)
- Parse message from request body

#### Step 2: Fetch Conversation Context
```sql
SELECT * FROM conversations 
WHERE user_id = 42 
ORDER BY updated_at DESC 
LIMIT 1;
```

Result: Returns most recent conversation ID

```sql
SELECT * FROM messages 
WHERE conversation_id = {conv_id}
ORDER BY created_at ASC;
```

Result: Returns full message history for context

```python
conversation_history = [
    {"role": "user", "content": "What are my tasks?"},
    {"role": "assistant", "content": "You have 3 pending tasks..."},
    {"role": "user", "content": "Add a task to buy milk tomorrow at 3pm"}
]
```

#### Step 3: Initialize Agent with Context

```python
from openai import OpenAI

agent = OpenAI()

# Pass conversation history to agent
response = agent.chat.completions.create(
    model="gpt-4",
    messages=conversation_history,  # Full history from DB
    tools=[
        {"name": "add_task", "description": "...", "parameters": {...}},
        {"name": "list_tasks", "description": "...", "parameters": {...}},
        # ... other tools
    ],
    tool_choice="auto",
    system=SYSTEM_PROMPT  # Agent behavior rules
)
```

**Agent Processing**:
1. Receives conversation history (stateless input)
2. Understands user intent: "add_task"
3. Extracts fields:
   - title: "Buy milk"
   - due_date: tomorrow (calculated from current date)
   - priority: "low" (inferred)
4. Generates tool call

#### Step 4: Agent Calls MCP Tool

Agent generates tool call:
```json
{
  "type": "tool_call",
  "name": "add_task",
  "arguments": {
    "user_id": 42,
    "title": "Buy milk",
    "due_date": "2026-02-07",
    "priority": "low"
  }
}
```

**Handler**: `mcp_server/tools.py`

```python
async def add_task(user_id: int, title: str, due_date: str, priority: str = "low"):
    # No instance state accessed
    # Only database operations
    
    # 1. Validate user_id exists
    user = await db.get(User, user_id)
    if not user:
        return {"status": "error", "message": "User not found"}
    
    # 2. Validate inputs
    if len(title) < 1 or len(title) > 200:
        return {"status": "error", "message": "Invalid title"}
    
    # 3. Create task in database
    task = Task(
        user_id=user_id,
        title=title,
        due_date=due_date,
        priority=priority,
        completed=False
    )
    db.add(task)
    db.commit()
    
    # 4. Return result
    return {
        "status": "success",
        "message": "Task created",
        "data": {
            "id": task.id,
            "title": task.title,
            "due_date": task.due_date,
            "created_at": task.created_at
        }
    }
```

Tool execution:
- Stateless function
- Reads from database
- Writes to database
- Returns response

#### Step 5: Agent Processes Tool Response

Agent receives tool result:
```json
{
  "status": "success",
  "message": "Task created",
  "data": {
    "id": 47,
    "title": "Buy milk",
    "due_date": "2026-02-07",
    "created_at": "2026-02-06T14:30:00Z"
  }
}
```

Agent generates natural language response:
```
"Great! I've added 'Buy milk' to your task list for tomorrow (Feb 7, 2026)."
```

#### Step 6: Persist Conversation

```python
# Create new message record
message = Message(
    conversation_id=conv_id,
    role="assistant",
    content="Great! I've added 'Buy milk'...",
    timestamp=datetime.utcnow()
)
db.add(message)

# Store tool call metadata
tool_call = MessageToolCall(
    message_id=message.id,
    tool_name="add_task",
    arguments={"user_id": 42, "title": "Buy milk", "due_date": "2026-02-07"},
    result={"status": "success", "id": 47, ...},
    timestamp=datetime.utcnow()
)
db.add(tool_call)

# Update conversation timestamp
conversation.updated_at = datetime.utcnow()
db.commit()
```

#### Step 7: Return Response to Client

```json
{
  "status": "success",
  "message": "Great! I've added 'Buy milk' to your task list for tomorrow (Feb 7, 2026).",
  "task_created": {
    "id": 47,
    "title": "Buy milk",
    "due_date": "2026-02-07"
  },
  "timestamp": "2026-02-06T14:30:00Z"
}
```

---

## Database Schema for Stateless Architecture

### Core Tables

```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id INTEGER NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (user_id, updated_at)  -- Fast recent conversation lookup
);

CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL,  -- 'user' | 'assistant'
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (conversation_id, created_at)  -- Fast history retrieval
);

CREATE TABLE message_tool_calls (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  message_id UUID NOT NULL REFERENCES messages(id) ON DELETE CASCADE,
  tool_name VARCHAR(50) NOT NULL,
  arguments JSONB NOT NULL,
  result JSONB NOT NULL,
  status VARCHAR(20) NOT NULL,  -- 'success' | 'error'
  duration_ms INTEGER,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (message_id)
);

CREATE TABLE tasks (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  title VARCHAR(200) NOT NULL,
  description TEXT,
  priority VARCHAR(20) DEFAULT 'low',  -- 'low' | 'medium' | 'high'
  due_date DATE,
  completed BOOLEAN DEFAULT FALSE,
  completed_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (user_id, completed, due_date)  -- Fast filtering queries
);

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  hashed_password VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (username)
);
```

### Key Indexing Strategy for Stateless Queries

```sql
-- Conversations: Quick lookup of recent conversations
CREATE INDEX idx_conversations_user_updated 
ON conversations(user_id, updated_at DESC);

-- Messages: Fast history retrieval for agent context
CREATE INDEX idx_messages_conversation 
ON messages(conversation_id, created_at ASC);

-- Tasks: Fast filtering for list_tasks tool
CREATE INDEX idx_tasks_user_status_due 
ON tasks(user_id, completed, due_date);

-- Tool calls: Audit and debugging
CREATE INDEX idx_tool_calls_message 
ON message_tool_calls(message_id);
```

---

## Error Handling & Recovery

### What happens if database is temporarily down?

```python
async def add_task(...):
    try:
        task = Task(...)
        db.add(task)
        db.commit()
    except DatabaseError as e:
        logger.error(f"Failed to create task: {e}")
        # Return error to agent
        return {
            "status": "error",
            "error_code": "DB_ERROR",
            "message": "I'm having trouble saving. Please try again."
        }
    # No recovery from in-memory cache (none exists!)
    # Agent/client must retry
```

### Retry Strategy

Client (frontend):
```typescript
const response = await fetch(`/api/${userId}/chat`, {
  method: "POST",
  body: JSON.stringify({ message })
});

// If 500 error, retry up to 3 times with exponential backoff
if (!response.ok && response.status >= 500) {
  // Retry with fresh state from DB (no stale cache)
}
```

Server-side:
- MCP tools are idempotent where possible
- Add_task: Each call creates new task (not idempotent)
- Complete_task: Safe to retry (already complete is idempotent)
- Delete_task: Safe to retry only if confirmed=true

---

## Scalability Characteristics

### Stateless = Horizontally Scalable

```
Deployment 1:        Deployment 2:        Deployment 3:
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ FastAPI App   │    │ FastAPI App   │    │ FastAPI App   │
│ (Stateless)   │    │ (Stateless)   │    │ (Stateless)   │
└───────────────┘    └───────────────┘    └───────────────┘
    │                    │                    │
    └────────────────────┼────────────────────┘
                         │
                    ┌────▼────────┐
                    │ Load        │
                    │ Balancer    │
                    └─────────────┘
                         │
                    ┌────▼────────┐
                    │ PostgreSQL   │
                    │ (Primary)    │
                    └─────────────┘
```

**Benefits**:
- New server instances can start immediately
- No state replication needed
- Database is single point of truth
- Linear scaling with server count
- Zero warm-up time for new instances

### Database Load Patterns

```
Read Operations:
- list_tasks: 10-50ms per request (indexed queries)
- message history: 20-100ms (depends on conversation length)

Write Operations:
- add_task: 50-100ms (insert + index update)
- message insert: 30-50ms

Per 1000 concurrent users:
- ~100 requests/sec to API
- ~200 database queries/sec (each request = ~2 queries)
- Database connection pool: 50-100 connections
```

---

## Performance Optimization

### Query Optimization

```python
# ❌ Bad: N+1 query problem
tasks = db.query(Task).filter(Task.user_id == 42).all()
for task in tasks:
    conversation = db.query(Conversation).filter(
        Conversation.id == task.conversation_id
    ).first()  # QUERY PER TASK

# ✅ Good: Single query with joins
tasks = db.query(Task).filter(Task.user_id == 42).options(
    joinedload(Task.conversations)
).all()
```

### Caching Strategy

```python
# Frontend caching (browser)
useQuery({
  queryKey: ['tasks', userId],
  queryFn: () => fetchTasks(userId),
  staleTime: 30_000,  // 30 seconds
  gcTime: 5 * 60_000,  // 5 minutes
});

// Server-side caching (optional, not required for statelessness)
@cache_ttl(30)  # Cache list_tasks for 30 seconds per user
async def list_tasks(user_id):
    # If same user hits within 30s, return cached result
    pass
```

---

## Deployment Considerations

### Environment Variables Required

```bash
DATABASE_URL=postgresql://user:pass@localhost/tododb
OPENAI_API_KEY=sk-...
JWT_SECRET=random-secret-key
RESEND_API_KEY=re_...  # Email service
```

### Health Checks

```python
@app.get("/health/ready")
async def readiness_check():
    """Readiness check - can accept requests?"""
    try:
        db.execute("SELECT 1")
        return {"status": "ready"}
    except:
        return {"status": "not_ready"}, 503

@app.get("/health/live")
async def liveness_check():
    """Liveness check - is pod alive?"""
    return {"status": "alive"}
```

### Graceful Shutdown

```python
@app.on_event("shutdown")
async def shutdown_event():
    """
    Graceful shutdown - no cleanup needed!
    All state is in database.
    Just stop accepting new requests.
    """
    print("Shutting down. All state persisted to DB.")
    # Close database pool
    db.dispose()
```

---

## Monitoring & Observability

### Key Metrics to Track

```
1. Request latency:
   - P50, P95, P99 response times
   - Should be <1s for chat endpoint

2. Tool execution time:
   - Per-tool breakdown
   - Database query times

3. Error rates:
   - By tool type
   - By error code (DB_ERROR, VALIDATION_ERROR, etc.)

4. Database metrics:
   - Connection pool utilization
   - Slow query log (>1000ms queries)
   - Index hit rates

5. Agent metrics:
   - Tool call frequency
   - Tool success rate
   - Average conversation length
```

### Logging Structure

```json
{
  "timestamp": "2026-02-06T14:30:00Z",
  "level": "INFO",
  "request_id": "uuid-here",
  "user_id": 42,
  "endpoint": "POST /api/42/chat",
  "message": "Chat request processed",
  "agent_response": "Great! I've added...",
  "tools_called": [
    {
      "name": "add_task",
      "duration_ms": 75,
      "status": "success"
    }
  ],
  "total_duration_ms": 250
}
```

---

## Security in Stateless Design

### Why Stateless is More Secure

1. **No data leakage between users**
   - Each request isolated
   - No shared memory
   - No cache poisoning

2. **No stale permissions**
   - JWT verified fresh each request
   - User data fetched from DB
   - Can't operate on stale authorization

3. **Easier to audit**
   - All operations logged to database
   - No in-memory state to lose
   - Full audit trail available

### Security Checklist

- [ ] All tool calls validate user_id ownership
- [ ] JWT token verified on every request
- [ ] SQL injection prevention (parameterized queries)
- [ ] Rate limiting per user per endpoint
- [ ] Sensitive data never logged
- [ ] Database credentials in environment variables
- [ ] HTTPS enforced for API
- [ ] CORS properly configured
