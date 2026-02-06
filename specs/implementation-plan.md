# Implementation Plan: Phase III Todo AI Chatbot

## Executive Summary

This document provides a detailed implementation plan for the Phase III Todo AI Chatbot using the Agentic Dev Stack. The plan is structured in four phases: Backend Foundation, MCP Tools, Agent Integration, and Deployment.

**Timeline**: 8-12 days
**Team Size**: 1-2 developers
**Critical Path**: Database → MCP Tools → Agent Integration → Testing

---

## High-Level Implementation Phases

### Phase 1: Backend Foundation (Days 1-3)
- [ ] Database schema setup and migrations
- [ ] FastAPI project initialization
- [ ] User authentication (JWT)
- [ ] Database models (User, Task, Conversation, Message)

### Phase 2: MCP Tools Layer (Days 4-6)
- [ ] MCP server setup
- [ ] Implement 5 core MCP tools
- [ ] Tool testing and validation
- [ ] Database persistence verification

### Phase 3: Agent Integration (Days 7-9)
- [ ] OpenAI Agents SDK integration
- [ ] System prompt and agent behavior
- [ ] Chat endpoint implementation
- [ ] Conversation history management

### Phase 4: Testing & Deployment (Days 10-12)
- [ ] E2E testing
- [ ] Performance optimization
- [ ] Deployment configuration
- [ ] Documentation

---

## Phase 1: Backend Foundation

### 1.1 Database Schema & Migrations

**Objective**: Set up PostgreSQL database with all required tables and indexes.

**Tasks**:
1. Create alembic migration: `001_conversations_and_messages.py`
   - Create conversations table
   - Create messages table
   - Create message_tool_calls table
   - Add indexes for user_id, created_at, conversation_id

2. Create alembic migration: `002_update_tasks_table.py`
   - Add conversation_id foreign key to tasks (optional)
   - Create indexes for (user_id, completed, due_date)
   - Verify existing task table structure

3. Create alembic migration: `003_create_indexes.py`
   - idx_conversations_user_updated: ON conversations(user_id, updated_at DESC)
   - idx_messages_conversation: ON messages(conversation_id, created_at ASC)
   - idx_tasks_user_status_due: ON tasks(user_id, completed, due_date)

**Deliverables**:
- [ ] All migrations run successfully
- [ ] Schema validated with `DESCRIBE TABLE` commands
- [ ] Indexes verified with `EXPLAIN` analysis

**Duration**: 1 day

---

### 1.2 SQLModel Database Models

**Objective**: Create Python models for database tables.

**Models to Create**:

1. **Conversation Model** (`models/conversation.py`)
   ```python
   class Conversation(SQLModel, table=True):
       id: str = Field(primary_key=True)  # UUID
       user_id: int = Field(foreign_key="users.id")
       title: str
       created_at: datetime
       updated_at: datetime
       messages: list["Message"] = Relationship(back_populates="conversation")
   ```

2. **Message Model** (`models/message.py`)
   ```python
   class Message(SQLModel, table=True):
       id: str = Field(primary_key=True)  # UUID
       conversation_id: str = Field(foreign_key="conversations.id")
       role: str  # 'user' | 'assistant'
       content: str
       created_at: datetime
       tool_calls: list["MessageToolCall"] = Relationship(back_populates="message")
   ```

3. **MessageToolCall Model** (`models/message_tool_call.py`)
   ```python
   class MessageToolCall(SQLModel, table=True):
       id: str = Field(primary_key=True)  # UUID
       message_id: str = Field(foreign_key="messages.id")
       tool_name: str
       arguments: dict  # JSON
       result: dict  # JSON
       status: str  # 'success' | 'error'
       duration_ms: int
       created_at: datetime
   ```

4. **Update Task Model** (`models/task.py`)
   - Verify existing fields
   - Add any missing fields per spec
   - Ensure user_id and due_date are properly indexed

**Deliverables**:
- [ ] All models created and imported
- [ ] Relationships properly defined
- [ ] Models work with SQLModel ORM
- [ ] Import tests pass

**Duration**: 1 day

---

### 1.3 Authentication & JWT

**Objective**: Ensure JWT authentication is working for all endpoints.

**Requirements**:
- [ ] JWT token generation on login
- [ ] JWT token validation on all protected endpoints
- [ ] Token expiration: 7 days
- [ ] Refresh token mechanism

**Implementation**:
1. Verify `utils/jwt.py` has proper token generation
2. Add JWT validation middleware to chat endpoint
3. Extract user_id from JWT token
4. Add rate limiting per user_id

**Deliverables**:
- [ ] Login endpoint returns valid JWT
- [ ] Protected endpoints validate JWT
- [ ] Token expiration works correctly
- [ ] Rate limiting implemented

**Duration**: 1 day

---

## Phase 2: MCP Tools Layer

### 2.1 MCP Server Setup

**Objective**: Initialize MCP server and register tools.

**Implementation File**: `mcp_server/tools.py`

**Setup Steps**:
```python
from mcp.server import Server
from mcp.types import Tool

server = Server("todo-agent")

# Tool definitions with JSON schemas
add_task_tool = Tool(
    name="add_task",
    description="Create a new task",
    inputSchema={
        "type": "object",
        "properties": {
            "user_id": {"type": "integer"},
            "title": {"type": "string"},
            ...
        },
        "required": ["user_id", "title"]
    }
)

@server.call_tool("add_task")
async def handle_add_task(user_id: int, title: str, ...):
    # Implementation
    pass
```

**Deliverables**:
- [ ] MCP server initialized
- [ ] All 5 tools registered
- [ ] Tool schemas match MCP spec
- [ ] Server can be started

**Duration**: 0.5 days

---

### 2.2 Implement 5 Core Tools

**Tool 1: add_task**
```
Input: user_id, title, description, due_date, priority, tags
Output: Created task object with ID
Process:
  1. Validate inputs (title length, due_date format)
  2. Check user_id exists
  3. INSERT into tasks table
  4. Return task response
```

**Tool 2: list_tasks**
```
Input: user_id, status (all/pending/completed), priority, sort_by, limit, offset
Output: Array of tasks with counts
Process:
  1. Build SELECT query with filters
  2. Apply sorting (due_date, priority, created_at)
  3. Apply pagination
  4. Return task list with counts
```

**Tool 3: update_task**
```
Input: user_id, task_id, title, description, due_date, priority, tags
Output: Updated task object with change tracking
Process:
  1. Fetch task by ID, verify ownership
  2. Build UPDATE statement
  3. Track old vs new values
  4. Commit and return changes
```

**Tool 4: complete_task**
```
Input: user_id, task_id, cancel_reminder
Output: Completion confirmation
Process:
  1. Fetch task, verify ownership
  2. Set completed=true, completed_date=now
  3. Cancel reminder if flag=true
  4. Update task_remaining count
```

**Tool 5: delete_task**
```
Input: user_id, task_id, confirmed (boolean)
Output: Deletion confirmation
Process:
  1. Verify confirmed=true
  2. Fetch task, verify ownership
  3. INSERT audit log
  4. DELETE from tasks
  5. Cancel associated reminders
```

**Deliverables**:
- [ ] All 5 tools implemented
- [ ] Input validation in place
- [ ] Database operations correct
- [ ] Error handling comprehensive
- [ ] Response format matches spec

**Duration**: 2.5 days

---

### 2.3 Tool Testing & Validation

**Unit Tests**:
- [ ] add_task: valid input, invalid title, past date, tags validation
- [ ] list_tasks: all filters, sorting, pagination, empty result
- [ ] update_task: single field, multiple fields, not found, unauthorized
- [ ] complete_task: valid, already completed, invalid task
- [ ] delete_task: confirmed=true, confirmed=false, audit log

**Integration Tests**:
- [ ] add_task → list_tasks shows new task
- [ ] update_task changes are visible
- [ ] complete_task removes from pending list
- [ ] delete_task removes from all lists

**Database Tests**:
- [ ] Indexes working (EXPLAIN ANALYZE)
- [ ] Foreign keys enforced
- [ ] Transactions atomic
- [ ] Concurrent operations safe

**Deliverables**:
- [ ] All unit tests passing
- [ ] Integration tests passing
- [ ] Database constraints verified
- [ ] Performance acceptable (< 500ms per tool)

**Duration**: 1 day

---

## Phase 3: Agent Integration

### 3.1 OpenAI Agents SDK Integration

**Objective**: Set up agent framework and system prompt.

**Implementation**: `services/openai_agent.py`

```python
from openai import OpenAI

class TodoAgent:
    def __init__(self, api_key: str):
        self.client = OpenAI(api_key=api_key)
        self.model = "gpt-4"
        self.tools = [
            # MCP tool definitions
        ]
        self.system_prompt = """..."""  # From agent.md
    
    async def process_message(
        self,
        user_id: int,
        message: str,
        conversation_history: list[dict]
    ) -> str:
        """Process user message and return agent response"""
        pass
```

**Setup Steps**:
1. Install OpenAI Agents SDK: `pip install openai`
2. Define system prompt from specs/agent.md
3. Register 5 MCP tools with agent
4. Implement message processing loop

**Deliverables**:
- [ ] OpenAI client initialized
- [ ] System prompt configured
- [ ] Tools registered with agent
- [ ] Message processing works

**Duration**: 1 day

---

### 3.2 Chat Endpoint Implementation

**Objective**: Implement FastAPI endpoint for chat.

**Endpoint**: `POST /api/{user_id}/chat`

**Implementation**: `routes/chat.py`

```python
@router.post("/api/{user_id}/chat")
async def chat(user_id: int, request: ChatRequest):
    """
    1. Verify JWT token, extract user_id
    2. Fetch most recent conversation for user
    3. Fetch conversation history from DB
    4. Pass to agent with history
    5. Agent processes and returns response
    6. Store message + tool calls in DB
    7. Return response to client
    """
    pass
```

**Request Format**:
```json
{
  "message": "Add a task to buy milk tomorrow"
}
```

**Response Format**:
```json
{
  "status": "success",
  "message": "Great! I've added 'Buy milk'...",
  "conversation_id": "uuid",
  "tool_calls": [
    {
      "tool": "add_task",
      "arguments": {...},
      "result": {...}
    }
  ],
  "timestamp": "2026-02-06T14:30:00Z"
}
```

**Features**:
- [ ] Rate limiting per user (100 requests/hour)
- [ ] JWT validation
- [ ] Error handling with user-friendly messages
- [ ] Conversation history management
- [ ] Tool call audit logging

**Deliverables**:
- [ ] Endpoint implemented
- [ ] All features working
- [ ] Error handling complete
- [ ] Performance acceptable (< 3s response)

**Duration**: 2 days

---

### 3.3 Conversation State Management

**Objective**: Implement conversation history retrieval and persistence.

**Implementation**: Add methods to conversation service

**Methods**:
1. **get_or_create_conversation**(user_id)
   - Fetch most recent conversation
   - If none exists, create new one
   - Return conversation ID

2. **get_conversation_history**(conversation_id, limit=50)
   - Fetch messages ordered by created_at
   - Return as format agent expects: [{"role": "user"/"assistant", "content": "..."}]

3. **store_message**(conversation_id, role, content, tool_calls)
   - Insert message record
   - Insert tool_call records if any
   - Update conversation.updated_at

4. **generate_conversation_title**(first_message)
   - Extract first 5-8 words
   - Remove articles/small words
   - Capitalize
   - Max 50 chars

**Deliverables**:
- [ ] All methods implemented
- [ ] Conversation history retrieval works
- [ ] Message persistence works
- [ ] Tool calls stored with audit info
- [ ] Title generation working

**Duration**: 1 day

---

## Phase 4: Testing & Deployment

### 4.1 End-to-End Testing

**Test Scenarios**:

1. **User Registration & Login**
   - [ ] Create new account
   - [ ] Login with correct credentials
   - [ ] Get JWT token
   - [ ] Access chat endpoint with token

2. **Simple Task Creation**
   - [ ] Send: "Add a task to buy milk"
   - [ ] Agent creates task
   - [ ] Task appears in list

3. **Task Management**
   - [ ] Update task priority
   - [ ] Mark task complete
   - [ ] Delete task
   - [ ] View task list with different filters

4. **Conversation History**
   - [ ] Send multiple messages in sequence
   - [ ] Agent maintains context
   - [ ] History persists across sessions
   - [ ] Can switch conversations

5. **Error Handling**
   - [ ] Invalid JWT returns 401
   - [ ] Rate limit exceeded returns 429
   - [ ] Database error returns friendly message
   - [ ] Tool failure gracefully handled

**Deliverables**:
- [ ] All test scenarios passing
- [ ] No error cases
- [ ] Performance acceptable
- [ ] User experience smooth

**Duration**: 1.5 days

---

### 4.2 Performance Optimization

**Targets**:
- Chat endpoint: < 3 seconds
- Tool execution: < 500ms per tool
- Database queries: < 100ms
- Conversation history load: < 1 second

**Optimization Strategies**:
1. Add query logging to identify slow queries
2. Verify all indexes exist and are used
3. Implement conversation history pagination
4. Add Redis caching (optional)
5. Profile OpenAI API calls

**Deliverables**:
- [ ] Response time metrics collected
- [ ] All targets met
- [ ] Bottlenecks identified and addressed
- [ ] Monitoring dashboard created

**Duration**: 0.5 days

---

### 4.3 Deployment Configuration

**Deployment Targets**:
- Backend: Render or Railway
- Database: PostgreSQL on Render
- Frontend: Vercel

**Configuration Files**:
- [ ] `.env.example` with all required variables
- [ ] `render.yaml` for backend deployment
- [ ] `docker-compose.yml` for local development
- [ ] Health check endpoints configured
- [ ] Monitoring/logging configured

**CI/CD**:
- [ ] GitHub Actions for tests
- [ ] Automatic deployment on main branch
- [ ] Database migration on deploy

**Deliverables**:
- [ ] All config files complete
- [ ] Deployment automated
- [ ] Monitoring active
- [ ] Rollback procedure documented

**Duration**: 1 day

---

### 4.4 Documentation

**Required Documentation**:
- [ ] API documentation (FastAPI /docs)
- [ ] MCP tool documentation
- [ ] Deployment guide
- [ ] Architecture overview
- [ ] Troubleshooting guide
- [ ] Developer setup guide

**Deliverables**:
- [ ] All documentation complete
- [ ] Code examples provided
- [ ] Screenshots included
- [ ] Troubleshooting guide comprehensive

**Duration**: 0.5 days

---

## Backend Tasks Summary

### Task List for Backend Developers

**Week 1 (Days 1-5)**:
- [ ] Day 1: Database migrations (conversations, messages, indexes)
- [ ] Day 1: SQLModel models (Conversation, Message, MessageToolCall)
- [ ] Day 2: Verify JWT authentication
- [ ] Day 2: MCP server setup
- [ ] Days 3-4: Implement 5 MCP tools (add_task, list_tasks, update_task, complete_task, delete_task)
- [ ] Day 5: Unit test all tools

**Week 2 (Days 6-8)**:
- [ ] Day 6: OpenAI Agents SDK integration
- [ ] Days 6-7: Chat endpoint implementation
- [ ] Day 7: Conversation state management
- [ ] Day 8: Integration testing (agent + tools)

**Week 2-3 (Days 9-12)**:
- [ ] Day 9: E2E testing
- [ ] Day 9: Performance optimization
- [ ] Day 10: Deployment configuration
- [ ] Day 10-11: Documentation
- [ ] Day 12: Final verification & deployment

---

## MCP Server Tasks Summary

### Task List for MCP Implementation

1. **MCP Server Initialization** (1 day)
   - Set up MCP server with official SDK
   - Define tool schemas
   - Register tools

2. **Tool Implementation** (3 days)
   - add_task with validation
   - list_tasks with filtering
   - update_task with change tracking
   - complete_task with reminder cancellation
   - delete_task with confirmation

3. **Tool Testing** (1 day)
   - Unit tests for all tools
   - Integration tests with database
   - Error case coverage

4. **Documentation** (0.5 days)
   - Tool parameter documentation
   - Response format documentation
   - Error code reference

---

## Frontend Tasks Summary

### Task List for Frontend Developers

1. **Chat Interface Component** (2 days)
   - Message input component
   - Message display component
   - Conversation history sidebar
   - Real-time message updates

2. **API Integration** (1 day)
   - useAI hook for chat
   - Message sending
   - Error handling display
   - Loading states

3. **Conversation Management** (1 day)
   - List conversations
   - Switch between conversations
   - Create new conversation
   - Delete conversation

4. **Task Display** (1 day)
   - Show created tasks in dashboard
   - Show task status updates from chat
   - Sync with backend

5. **Testing & Polish** (1 day)
   - UI/UX improvements
   - Error message display
   - Mobile responsiveness

---

## Database Tasks Summary

### Task List for Database Developers

1. **Schema Design** (0.5 days)
   - Create migrations for all tables
   - Define indexes
   - Verify relationships

2. **Data Integrity** (0.5 days)
   - Add constraints
   - Test transactions
   - Verify cascading deletes

3. **Optimization** (1 day)
   - Analyze slow queries
   - Add missing indexes
   - Test with production load

4. **Monitoring** (0.5 days)
   - Set up query logging
   - Create performance dashboard
   - Set up alerts

---

## Critical Path & Dependencies

```
┌─────────────────────────┐
│ DB Schema (migrations)   │ ← CRITICAL PATH START
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ SQLModel models         │
└────────┬────────────────┘
         │
    ┌────┴─────────────────────────┐
    │                              │
    ▼                              ▼
┌──────────────────┐      ┌──────────────────┐
│ MCP tools impl   │      │ JWT Auth check   │
└────────┬─────────┘      └──────────────────┘
         │
         ▼
┌──────────────────┐
│ Tool testing     │
└────────┬─────────┘
         │
    ┌────┴──────────────────────────┐
    │                               │
    ▼                               ▼
┌───────────────────────┐  ┌──────────────────┐
│ OpenAI agent setup    │  │ Agent integration│
└─────────┬─────────────┘  └──────────────────┘
          │
          ▼
┌──────────────────────┐
│ Chat endpoint        │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│ Conversation history │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│ E2E testing          │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│ Performance testing  │
└────────┬─────────────┘
         │
         ▼
┌──────────────────────┐
│ Deployment          │
└──────────────────────┘
```

**Critical Blockers**:
- Database schema must be complete before MCP tools
- MCP tools must be tested before agent integration
- Agent integration must work before chat endpoint
- Chat endpoint must work before E2E testing

---

## Success Criteria

### Backend
- [ ] All 5 MCP tools implemented and tested
- [ ] Chat endpoint returns proper responses
- [ ] Conversation history persists and retrieves correctly
- [ ] All database operations atomic and isolated
- [ ] Response time < 3 seconds for 95th percentile

### Tools
- [ ] All tools follow MCP spec
- [ ] Tools are stateless
- [ ] Tool errors handled gracefully
- [ ] Tool calls audit logged

### Agent
- [ ] Agent understands user intent correctly
- [ ] Agent confirms destructive operations
- [ ] Agent maintains conversation context
- [ ] Agent generates conversation titles

### Testing
- [ ] 100% of critical paths tested
- [ ] Error cases covered
- [ ] Performance targets met
- [ ] Concurrent users supported

### Deployment
- [ ] Backend deployed to production
- [ ] Database backups automated
- [ ] Monitoring active
- [ ] Documentation complete
- [ ] Team can maintain/extend

---

## Timeline Estimate

| Phase | Duration | Start | End |
|-------|----------|-------|-----|
| DB Schema | 1 day | Day 1 | Day 1 |
| Models | 1 day | Day 1 | Day 2 |
| MCP Tools | 3 days | Day 2 | Day 4 |
| Tool Testing | 1 day | Day 5 | Day 5 |
| Agent Setup | 1 day | Day 5 | Day 6 |
| Chat Endpoint | 2 days | Day 6 | Day 7 |
| Conv. History | 1 day | Day 7 | Day 8 |
| E2E Testing | 1.5 days | Day 8 | Day 9 |
| Performance | 0.5 days | Day 9 | Day 9 |
| Deployment | 1 day | Day 10 | Day 10 |
| Documentation | 0.5 days | Day 10 | Day 10 |
| **Total** | **8-12 days** | | |

---

## Risk Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Database schema changes | High | Use migrations, test locally first |
| OpenAI API rate limits | Medium | Implement request queuing |
| Token context limits | Medium | Implement conversation summary |
| Tool latency | Medium | Add response caching |
| Deployment issues | High | Automated CI/CD, rollback plan |
| Security vulnerabilities | High | Regular security audits |

---

## Next Steps

1. **Approve Specification Documents**
   - Review specs/agent.md
   - Review specs/mcp-tools.md
   - Review specs/architecture.md

2. **Break Down into Tasks**
   - Use next document (TASKS.md) for task breakdown
   - Assign to team members
   - Set up tracking

3. **Begin Implementation**
   - Start with database schema
   - Proceed through critical path
   - Regular progress check-ins

