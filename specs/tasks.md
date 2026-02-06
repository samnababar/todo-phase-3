# Implementation Tasks: Phase III Todo AI Chatbot

## Task Breakdown Guide

This document breaks the implementation plan into small, executable tasks that can be completed one-by-one using AI agents. Each task:
- Takes 30 minutes to 2 hours
- Has clear input and success criteria
- Can be worked on independently (with dependencies noted)
- Includes specific prompts for AI agents

---

## PHASE 1: Backend Foundation

### TASK 1.1: Create Database Migrations - Conversations & Messages

**Objective**: Add conversations, messages, and tool_calls tables to database

**Prerequisites**: 
- PostgreSQL running
- Alembic configured

**AI Agent Prompt**:
```
Create SQLAlchemy Alembic migration file named "001_add_conversations_messages_tables.py"
in backend/alembic/versions/ directory.

The migration should:
1. Create "conversations" table with columns:
   - id: UUID primary key (gen_random_uuid())
   - user_id: Integer, NOT NULL, foreign key to users(id)
   - title: VARCHAR(255), NOT NULL
   - created_at: TIMESTAMP, default CURRENT_TIMESTAMP
   - updated_at: TIMESTAMP, default CURRENT_TIMESTAMP

2. Create "messages" table with columns:
   - id: UUID primary key
   - conversation_id: UUID, NOT NULL, foreign key to conversations(id) ON DELETE CASCADE
   - role: VARCHAR(20), NOT NULL (values: 'user' or 'assistant')
   - content: TEXT, NOT NULL
   - created_at: TIMESTAMP, default CURRENT_TIMESTAMP

3. Create "message_tool_calls" table with columns:
   - id: UUID primary key
   - message_id: UUID, NOT NULL, foreign key to messages(id) ON DELETE CASCADE
   - tool_name: VARCHAR(50), NOT NULL
   - arguments: JSONB, NOT NULL
   - result: JSONB, NOT NULL
   - status: VARCHAR(20), NOT NULL
   - duration_ms: INTEGER
   - created_at: TIMESTAMP, default CURRENT_TIMESTAMP

4. Create indexes:
   - idx_conversations_user_updated: (user_id, updated_at DESC)
   - idx_messages_conversation: (conversation_id, created_at ASC)
   - idx_tool_calls_message: (message_id)

Include both upgrade() and downgrade() functions.
Do not include test code. Output only the migration file.
```

**Success Criteria**:
- [ ] Migration file created in correct location
- [ ] All tables created with correct columns
- [ ] All foreign keys configured
- [ ] All indexes created
- [ ] Downgrade function removes everything

**Time Estimate**: 45 minutes

---

### TASK 1.2: Create Database Migration - Task Table Updates

**Objective**: Verify/update task table with correct indexes

**Prerequisites**: 
- TASK 1.1 completed
- Task table already exists from Phase I/II

**AI Agent Prompt**:
```
Create Alembic migration file "002_update_tasks_table_indexes.py" in backend/alembic/versions/.

The migration should:
1. Check if tasks table exists (it should from Phase I/II)
2. Verify these columns exist on tasks table (do not recreate table):
   - id: primary key
   - user_id: foreign key to users(id)
   - title: VARCHAR(200)
   - description: TEXT
   - priority: VARCHAR(20) default 'low'
   - due_date: DATE
   - completed: BOOLEAN default FALSE
   - completed_date: TIMESTAMP nullable
   - created_at: TIMESTAMP
   - updated_at: TIMESTAMP

3. Create indexes if they don't exist:
   - idx_tasks_user_status_due: (user_id, completed, due_date)
   - idx_tasks_user_created: (user_id, created_at DESC)

Use conditional logic (inspector) to check for existing columns and indexes.
Do not drop/recreate existing table structure.
Include both upgrade() and downgrade() functions.
```

**Success Criteria**:
- [ ] Migration handles existing table correctly
- [ ] No duplicate indexes created
- [ ] Downgrade removes only added indexes

**Time Estimate**: 30 minutes

**Dependencies**: TASK 1.1

---

### TASK 1.3: SQLModel - Conversation Model

**Objective**: Create Conversation SQLModel

**Prerequisites**: 
- Alembic migrations completed
- models/ directory exists

**AI Agent Prompt**:
```
Create file backend/models/conversation.py with Conversation SQLModel.

Requirements:
1. Import necessary modules: SQLModel, Field, Relationship, datetime, Optional, UUID from typing

2. Create Conversation class with:
   - __tablename__ = "conversations"
   - id: Optional[UUID] = Field(primary_key=True, default=None)
   - user_id: int = Field(foreign_key="users.id")
   - title: str = Field(max_length=255)
   - created_at: datetime = Field(default_factory=datetime.utcnow)
   - updated_at: datetime = Field(default_factory=datetime.utcnow)
   - messages: Optional[list["Message"]] = Relationship(back_populates="conversation")

3. Add helpful docstring explaining the model

4. Do not import Message class (use forward reference string)

Output only the conversation.py file. Do not include other models.
```

**Success Criteria**:
- [ ] File created at backend/models/conversation.py
- [ ] Class properly configured for SQLModel
- [ ] All fields have correct types
- [ ] Docstring present
- [ ] Imports are correct

**Time Estimate**: 20 minutes

**Dependencies**: TASK 1.1

---

### TASK 1.4: SQLModel - Message Model

**Objective**: Create Message SQLModel

**Prerequisites**: 
- TASK 1.3 completed
- models/conversation.py exists

**AI Agent Prompt**:
```
Create file backend/models/message.py with Message SQLModel.

Requirements:
1. Import necessary modules including SQLModel, Field, Relationship, UUID, datetime

2. Create Message class with:
   - __tablename__ = "messages"
   - id: Optional[UUID] = Field(primary_key=True, default=None)
   - conversation_id: UUID = Field(foreign_key="conversations.id")
   - role: str = Field(max_length=20)  # 'user' or 'assistant'
   - content: str  # TEXT field
   - created_at: datetime = Field(default_factory=datetime.utcnow)
   - conversation: Optional["Conversation"] = Relationship(back_populates="messages")
   - tool_calls: Optional[list["MessageToolCall"]] = Relationship(back_populates="message")

3. Add validation in docstring (role must be 'user' or 'assistant')

4. Use forward references for Conversation and MessageToolCall

Output only the message.py file.
```

**Success Criteria**:
- [ ] File created at backend/models/message.py
- [ ] All fields properly configured
- [ ] Relationships correctly defined
- [ ] Role field correctly typed

**Time Estimate**: 20 minutes

**Dependencies**: TASK 1.3

---

### TASK 1.5: SQLModel - MessageToolCall Model

**Objective**: Create MessageToolCall SQLModel

**Prerequisites**: 
- TASK 1.4 completed
- models/message.py exists

**AI Agent Prompt**:
```
Create file backend/models/message_tool_call.py with MessageToolCall SQLModel.

Requirements:
1. Import necessary modules: SQLModel, Field, Relationship, UUID, datetime, Optional, dict

2. Create MessageToolCall class with:
   - __tablename__ = "message_tool_calls"
   - id: Optional[UUID] = Field(primary_key=True, default=None)
   - message_id: UUID = Field(foreign_key="messages.id")
   - tool_name: str = Field(max_length=50)
   - arguments: dict  # JSON field, store as JSONB in SQLAlchemy
   - result: dict  # JSON field, store as JSONB in SQLAlchemy
   - status: str = Field(max_length=20)  # 'success' or 'error'
   - duration_ms: Optional[int] = None
   - created_at: datetime = Field(default_factory=datetime.utcnow)
   - message: Optional["Message"] = Relationship(back_populates="tool_calls")

3. Use JSONB for dict columns in SQLAlchemy

4. Add docstring explaining tool_calls represent MCP tool invocations

Output only the message_tool_call.py file.
```

**Success Criteria**:
- [ ] File created at backend/models/message_tool_call.py
- [ ] Dict fields configured as JSONB
- [ ] All relationships correct
- [ ] Docstring present

**Time Estimate**: 20 minutes

**Dependencies**: TASK 1.4

---

### TASK 1.6: Update models/__init__.py Imports

**Objective**: Export all new models for easy importing

**Prerequisites**: 
- TASK 1.5 completed
- All model files created

**AI Agent Prompt**:
```
Update backend/models/__init__.py to import and export all models:

The file should contain:
from models.user import User
from models.task import Task
from models.conversation import Conversation
from models.message import Message
from models.message_tool_call import MessageToolCall

__all__ = ["User", "Task", "Conversation", "Message", "MessageToolCall"]

This allows imports like: from models import Conversation
instead of: from models.conversation import Conversation
```

**Success Criteria**:
- [ ] All models imported
- [ ] __all__ list includes all models
- [ ] Can import directly from models package

**Time Estimate**: 10 minutes

**Dependencies**: TASK 1.5

---

### TASK 1.7: Verify JWT Authentication

**Objective**: Ensure JWT implementation is working

**Prerequisites**: 
- utils/jwt.py exists from Phase I/II

**AI Agent Prompt**:
```
Review backend/utils/jwt.py and backend/middleware/auth.py.

Verify these functions exist and work correctly:
1. generate_token(user_id: int) -> str
   - Creates JWT with user_id
   - Expires in 7 days
   - Uses HS256 algorithm
   - Uses JWT_SECRET from config

2. verify_token(token: str) -> int (user_id)
   - Decodes JWT token
   - Returns user_id from payload
   - Raises exception if token invalid/expired

3. Middleware: get_current_user() 
   - Extracts JWT from Authorization header
   - Validates token
   - Returns user_id

If any function is missing or incomplete, provide the implementation code.
If all working, respond with: "JWT implementation verified and working."

Check for:
- Proper error handling (ExpiredSignatureError, InvalidTokenError)
- Token payload includes user_id
- Config has JWT_SECRET defined
```

**Success Criteria**:
- [ ] generate_token works
- [ ] verify_token works  
- [ ] Middleware validates tokens
- [ ] Test token creation and validation
- [ ] Error handling in place

**Time Estimate**: 30 minutes

**Dependencies**: TASK 1.1

---

## PHASE 2: MCP Tools Layer

### TASK 2.1: MCP Server Initialization

**Objective**: Set up MCP server infrastructure

**Prerequisites**: 
- Backend models complete (TASK 1.6)
- Python environment has mcp package

**AI Agent Prompt**:
```
Create file backend/mcp_server/server.py to initialize MCP server.

The file should:
1. Import from mcp.server import Server
2. Create a Server instance named "todo-agent"
3. Define tool schemas as Python dicts for each MCP tool:
   - add_task
   - list_tasks
   - update_task
   - complete_task
   - delete_task

Each tool schema must include:
- name: string
- description: string (from specs/mcp-tools.md)
- inputSchema: JSON schema object with:
  - type: "object"
  - properties: {param_name: {type: "string", description: "..."}, ...}
  - required: [list of required params]

4. Add a method to get all tools: get_tools() -> list[dict]

Reference specs/mcp-tools.md for exact parameter names and types.

Do not implement tool handlers yet, just schema definitions.
Include helper functions for parameter validation.

Output only the server.py file.
```

**Success Criteria**:
- [ ] File created at backend/mcp_server/server.py
- [ ] Server instance created
- [ ] All 5 tool schemas defined
- [ ] Schemas match mcp-tools.md spec
- [ ] Tool schemas can be printed/validated

**Time Estimate**: 45 minutes

**Dependencies**: TASK 1.6

---

### TASK 2.2: Implement add_task MCP Tool

**Objective**: Create add_task tool for creating tasks

**Prerequisites**: 
- TASK 2.1 completed
- Backend models complete

**AI Agent Prompt**:
```
Implement add_task tool in backend/mcp_server/tools.py

Function signature:
async def add_task(
    user_id: int,
    title: str,
    description: Optional[str] = None,
    due_date: Optional[str] = None,
    priority: str = "low",
    tags: list[str] = []
) -> dict

Requirements:
1. Validate inputs:
   - title: 1-200 chars, non-empty (error if invalid)
   - due_date: must be today or future date in YYYY-MM-DD format (error if past)
   - priority: one of "low", "medium", "high" (error if invalid)
   - tags: max 5 items, each ≤50 chars (error if invalid)

2. Check user exists in database (error if not)

3. Create Task record in database:
   - user_id, title, description, due_date, priority, tags
   - completed=False
   - created_at, updated_at = now

4. Return success response with created task:
{
  "status": "success",
  "message": "Task created successfully",
  "data": {
    "id": task.id,
    "user_id": task.user_id,
    "title": task.title,
    "description": task.description,
    "due_date": task.due_date,
    "priority": task.priority,
    "tags": task.tags,
    "completed": task.completed,
    "created_at": task.created_at,
    "updated_at": task.updated_at
  },
  "timestamp": "ISO-8601 now"
}

5. Return error response on validation failure:
{
  "status": "error",
  "error_code": "VALIDATION_ERROR|USER_NOT_FOUND|DB_ERROR",
  "message": "Human readable error message",
  "timestamp": "ISO-8601 now"
}

Use database session from db.py.
Include comprehensive error handling.
Log all operations for audit trail.

Reference specs/mcp-tools.md for complete spec.
```

**Success Criteria**:
- [ ] Function handles all parameters
- [ ] Input validation complete
- [ ] Database operations working
- [ ] Correct response format
- [ ] Error handling comprehensive
- [ ] Task created with correct fields

**Time Estimate**: 1 hour

**Dependencies**: TASK 2.1

---

### TASK 2.3: Implement list_tasks MCP Tool

**Objective**: Create list_tasks tool for retrieving and filtering tasks

**Prerequisites**: 
- TASK 2.2 completed

**AI Agent Prompt**:
```
Implement list_tasks tool in backend/mcp_server/tools.py

Function signature:
async def list_tasks(
    user_id: int,
    status: str = "all",
    priority: Optional[str] = None,
    sort_by: str = "due_date",
    limit: int = 50,
    offset: int = 0
) -> dict

Requirements:
1. Validate inputs:
   - status: one of "all", "pending", "completed" (default "all")
   - priority: one of "low", "medium", "high", None (filter by priority if set)
   - sort_by: one of "due_date", "priority", "created_at" (default "due_date")
   - limit: 1-100 (default 50)
   - offset: ≥0 (default 0)

2. Build database query:
   - Filter: WHERE user_id = ? AND (completed = FALSE if status == "pending" else ...)
   - Filter: AND priority = ? (if priority parameter set)
   - Sort: ORDER BY [sort_by field]
   - Paginate: LIMIT limit OFFSET offset

3. Get counts:
   - total_count: all tasks for user
   - pending_count: tasks where completed = FALSE
   - completed_count: tasks where completed = TRUE
   - returned_count: length of returned array

4. Return success response:
{
  "status": "success",
  "message": "Tasks retrieved successfully",
  "data": {
    "tasks": [
      {
        "id": 1,
        "title": "...",
        "description": "...",
        "due_date": "YYYY-MM-DD",
        "priority": "...",
        "tags": [],
        "completed": false,
        "completed_date": null,
        "created_at": "ISO-8601",
        "updated_at": "ISO-8601"
      },
      ...
    ],
    "total_count": 15,
    "pending_count": 10,
    "completed_count": 5,
    "returned_count": 2,
    "limit": 50,
    "offset": 0
  },
  "timestamp": "ISO-8601"
}

5. Handle empty result (no error, just empty tasks array)

Include comprehensive error handling.
Reference specs/mcp-tools.md for complete spec.
```

**Success Criteria**:
- [ ] All filters work correctly
- [ ] Sorting works for all fields
- [ ] Pagination correct (limit/offset)
- [ ] Counts accurate
- [ ] Response format matches spec
- [ ] Empty results handled

**Time Estimate**: 1 hour

**Dependencies**: TASK 2.2

---

### TASK 2.4: Implement update_task MCP Tool

**Objective**: Create update_task tool for modifying tasks

**Prerequisites**: 
- TASK 2.3 completed

**AI Agent Prompt**:
```
Implement update_task tool in backend/mcp_server/tools.py

Function signature:
async def update_task(
    user_id: int,
    task_id: int,
    title: Optional[str] = None,
    description: Optional[str] = None,
    due_date: Optional[str] = None,
    priority: Optional[str] = None,
    tags: list[str] = []
) -> dict

Requirements:
1. Validate inputs:
   - At least one field must be provided (error if none)
   - If title provided: 1-200 chars
   - If due_date provided: must be today or future
   - If priority provided: one of "low", "medium", "high"
   - If tags provided: max 5 items

2. Fetch task from database:
   - SELECT * FROM tasks WHERE id = task_id AND user_id = user_id
   - Return error if not found or not owned by user

3. Track old values for change reporting

4. Update task fields (only provided fields):
   - Update each provided field
   - Set updated_at = now
   - Do NOT update completed status (use complete_task for that)
   - Do NOT change user_id

5. Save to database (UPDATE statement)

6. Return success response with changes:
{
  "status": "success",
  "message": "Task updated successfully",
  "data": {
    "id": task.id,
    "title": task.title,
    "description": task.description,
    "due_date": task.due_date,
    "priority": task.priority,
    "tags": task.tags,
    "completed": task.completed,
    "updated_at": "ISO-8601",
    "changes": {
      "priority": { "old": "high", "new": "critical" },
      "due_date": { "old": "2026-02-20", "new": "2026-02-18" }
    }
  },
  "timestamp": "ISO-8601"
}

7. Return error response:
{
  "status": "error",
  "error_code": "TASK_NOT_FOUND|UNAUTHORIZED|VALIDATION_ERROR|NO_CHANGES|DB_ERROR",
  "message": "Human readable error",
  "timestamp": "ISO-8601"
}

Implement idempotency (safe for retries).
Reference specs/mcp-tools.md for complete spec.
```

**Success Criteria**:
- [ ] Task fetch works with user_id check
- [ ] Only provided fields updated
- [ ] Changes tracked and returned
- [ ] Error cases handled
- [ ] Idempotent
- [ ] Response format correct

**Time Estimate**: 1 hour

**Dependencies**: TASK 2.3

---

### TASK 2.5: Implement complete_task MCP Tool

**Objective**: Create complete_task tool for marking tasks done

**Prerequisites**: 
- TASK 2.4 completed

**AI Agent Prompt**:
```
Implement complete_task tool in backend/mcp_server/tools.py

Function signature:
async def complete_task(
    user_id: int,
    task_id: int,
    cancel_reminder: bool = True
) -> dict

Requirements:
1. Fetch task from database:
   - SELECT * FROM tasks WHERE id = task_id AND user_id = user_id
   - Return error if not found or unauthorized

2. Check if already completed:
   - If completed = TRUE already, return idempotent success (same as if just completed)
   - Do not error, just return "already completed"

3. Update task:
   - Set completed = TRUE
   - Set completed_date = now
   - Set updated_at = now

4. Handle reminders:
   - If cancel_reminder = TRUE and task has reminder, cancel it
   - Update reminder status in database (if reminder table exists)

5. Get count of remaining tasks:
   - SELECT COUNT(*) FROM tasks WHERE user_id = user_id AND completed = FALSE

6. Return success response:
{
  "status": "success",
  "message": "Task marked as complete",
  "data": {
    "id": task.id,
    "title": task.title,
    "completed": true,
    "completed_date": "ISO-8601",
    "reminder_cancelled": true,
    "tasks_remaining": 14
  },
  "timestamp": "ISO-8601"
}

7. Return error response:
{
  "status": "error",
  "error_code": "TASK_NOT_FOUND|ALREADY_COMPLETED|UNAUTHORIZED|DB_ERROR",
  "message": "Human readable error",
  "timestamp": "ISO-8601"
}

Implement idempotency (safe for retries).
Reference specs/mcp-tools.md for complete spec.
```

**Success Criteria**:
- [ ] Task completion works
- [ ] Sets completed_date
- [ ] Reminder cancellation works
- [ ] Remaining count accurate
- [ ] Idempotent
- [ ] Response format correct

**Time Estimate**: 45 minutes

**Dependencies**: TASK 2.4

---

### TASK 2.6: Implement delete_task MCP Tool

**Objective**: Create delete_task tool for removing tasks

**Prerequisites**: 
- TASK 2.5 completed

**AI Agent Prompt**:
```
Implement delete_task tool in backend/mcp_server/tools.py

Function signature:
async def delete_task(
    user_id: int,
    task_id: int,
    confirmed: bool = False
) -> dict

Requirements:
1. Check confirmation:
   - If confirmed != True, return error (not confirmed)
   - This prevents accidental deletion

2. Fetch task from database:
   - SELECT * FROM tasks WHERE id = task_id AND user_id = user_id
   - Return error if not found or unauthorized

3. Create audit log entry:
   - INSERT INTO audit_log (user_id, action, resource_type, resource_id, resource_title, timestamp)
   - VALUES (user_id, 'DELETE_TASK', 'task', task_id, task.title, now)

4. Handle reminders:
   - Cancel any pending reminders for this task

5. Delete task:
   - DELETE FROM tasks WHERE id = task_id

6. Get count of remaining tasks:
   - SELECT COUNT(*) FROM tasks WHERE user_id = user_id

7. Return success response:
{
  "status": "success",
  "message": "Task deleted permanently",
  "data": {
    "deleted_task_id": task.id,
    "deleted_task_title": task.title,
    "tasks_remaining": 13,
    "deleted_at": "ISO-8601"
  },
  "timestamp": "ISO-8601"
}

8. Return error response:
{
  "status": "error",
  "error_code": "TASK_NOT_FOUND|NOT_CONFIRMED|UNAUTHORIZED|DB_ERROR",
  "message": "Human readable error",
  "timestamp": "ISO-8601"
}

Idempotency note: Safe to retry if confirmed=True.
Reference specs/mcp-tools.md for complete spec.
```

**Success Criteria**:
- [ ] Requires confirmation
- [ ] Audit log created
- [ ] Task deleted from database
- [ ] Reminders cancelled
- [ ] Remaining count correct
- [ ] Response format correct

**Time Estimate**: 45 minutes

**Dependencies**: TASK 2.5

---

### TASK 2.7: Unit Test MCP Tools

**Objective**: Create comprehensive unit tests for all 5 tools

**Prerequisites**: 
- TASK 2.6 completed
- All tools implemented

**AI Agent Prompt**:
```
Create comprehensive unit tests for MCP tools in backend/tests/unit/test_mcp_tools.py

Use pytest framework. Test cases should include:

For add_task:
- [ ] Test valid input creates task
- [ ] Test invalid title (< 1 char, > 200 chars)
- [ ] Test past due_date returns error
- [ ] Test invalid priority returns error
- [ ] Test > 5 tags returns error
- [ ] Test user_id not found returns error
- [ ] Test response format matches spec

For list_tasks:
- [ ] Test returns all tasks when status="all"
- [ ] Test returns only pending when status="pending"
- [ ] Test returns only completed when status="completed"
- [ ] Test filter by priority
- [ ] Test sorting by due_date
- [ ] Test sorting by priority
- [ ] Test sorting by created_at
- [ ] Test pagination (limit/offset)
- [ ] Test empty result returns empty array
- [ ] Test counts are accurate

For update_task:
- [ ] Test updating single field
- [ ] Test updating multiple fields
- [ ] Test changes are tracked
- [ ] Test not found returns error
- [ ] Test unauthorized (wrong user) returns error
- [ ] Test no fields provided returns error
- [ ] Test invalid due_date returns error

For complete_task:
- [ ] Test completing task sets completed=true
- [ ] Test sets completed_date
- [ ] Test cancels reminder
- [ ] Test already completed returns idempotent success
- [ ] Test not found returns error
- [ ] Test task_remaining count correct

For delete_task:
- [ ] Test confirmed=true deletes task
- [ ] Test confirmed=false returns error
- [ ] Test creates audit log
- [ ] Test not found returns error
- [ ] Test unauthorized returns error
- [ ] Test tasks_remaining count correct

Use fixtures for:
- test_user
- test_task
- test_conversation
- test_database (with transaction rollback)

Mock database when needed. All tests must be isolated (no test contamination).
```

**Success Criteria**:
- [ ] All test cases pass
- [ ] > 80% code coverage for tools
- [ ] Tests can run in isolation
- [ ] Database fixtures work
- [ ] Mock objects used appropriately

**Time Estimate**: 1.5 hours

**Dependencies**: TASK 2.6

---

## PHASE 3: Agent Integration

### TASK 3.1: OpenAI Agents SDK Setup

**Objective**: Initialize OpenAI agent with system prompt

**Prerequisites**: 
- TASK 2.7 completed
- OpenAI API key configured

**AI Agent Prompt**:
```
Create file backend/services/openai_agent.py to initialize OpenAI agent.

The file should:
1. Import OpenAI client: from openai import OpenAI
2. Import MCP tool schemas from mcp_server

3. Create TodoAgent class with:
   - __init__(self, api_key: str, model: str = "gpt-4")
   - self.client = OpenAI(api_key=api_key)
   - self.model = model
   - self.system_prompt = """<SYSTEM_PROMPT>""" (from specs/agent.md)
   - self.tools = [] (list of MCP tool definitions)

4. System prompt should be complete from specs/agent.md including:
   - Agent role (help users manage tasks)
   - Tool usage rules
   - Date/time parsing rules
   - Response format guidelines
   - Conversation management rules
   - Privacy/security rules

5. Load tool definitions from mcp_server/server.py

6. Add method: async def process_message(
       user_id: int,
       message: str,
       conversation_history: list[dict]
   ) -> dict
   - Not fully implemented yet (stub with pass)
   - But method signature and docstring should be there

7. Add method: def get_system_prompt() -> str

Output only the openai_agent.py file. Do not implement process_message logic yet.
Reference specs/agent.md for exact system prompt.
```

**Success Criteria**:
- [ ] File created at backend/services/openai_agent.py
- [ ] OpenAI client initialized
- [ ] System prompt complete
- [ ] Tool definitions loaded
- [ ] Class structure correct
- [ ] Methods stubbed out

**Time Estimate**: 45 minutes

**Dependencies**: TASK 2.1

---

### TASK 3.2: Chat Endpoint Implementation

**Objective**: Create POST /api/{user_id}/chat endpoint

**Prerequisites**: 
- TASK 3.1 completed
- Conversation models created

**AI Agent Prompt**:
```
Create/update file backend/routes/chat.py with FastAPI chat endpoint.

Requirements:
1. Endpoint definition:
   @router.post("/api/{user_id}/chat")
   async def chat(user_id: int, request: ChatRequest)
   
2. ChatRequest model (use Pydantic):
   class ChatRequest(BaseModel):
       message: str = Field(..., min_length=1, max_length=5000)

3. ChatResponse model:
   class ChatResponse(BaseModel):
       status: str  # "success" or "error"
       message: str
       conversation_id: str
       tool_calls: list[dict]
       timestamp: str

4. Endpoint logic:
   a. Extract JWT token from Authorization header
   b. Validate token and get user_id (use middleware)
   c. Fetch most recent conversation for user from database
   d. If no conversation, create new one with auto-generated title
   e. Fetch conversation messages (history) from database
   f. Format as list[{"role": "user"|"assistant", "content": "..."}]
   g. Pass to OpenAI agent with conversation_history
   h. Agent returns response (don't implement agent yet, mock it)
   i. Store agent response as message (role="assistant") in database
   j. Store tool calls in message_tool_calls table
   k. Update conversation.updated_at
   l. Return response with status="success"

5. Error handling:
   - Invalid JWT → return 401 with error message
   - User not found → return 404
   - Database error → return 500 with friendly message
   - Rate limit exceeded → return 429

6. Rate limiting:
   - Use rate limiter from main.py
   - 100 requests/hour per user_id

7. Logging:
   - Log all requests with user_id, message, response status

Reference architecture.md for request flow diagram.
Do not implement agent processing yet (mock return for now).
```

**Success Criteria**:
- [ ] Endpoint accepts POST requests
- [ ] JWT validation working
- [ ] Conversation created/fetched correctly
- [ ] Message history retrieved
- [ ] Response format correct
- [ ] Error handling complete
- [ ] Rate limiting working
- [ ] Logging in place

**Time Estimate**: 1.5 hours

**Dependencies**: TASK 3.1

---

### TASK 3.3: Conversation Management Service

**Objective**: Create service for conversation state management

**Prerequisites**: 
- TASK 3.2 completed
- Conversation models exist

**AI Agent Prompt**:
```
Create file backend/services/conversation_service.py with ConversationService class.

Methods to implement:

1. async def get_or_create_conversation(user_id: int) -> Conversation
   - Fetch: SELECT * FROM conversations WHERE user_id = ? ORDER BY updated_at DESC LIMIT 1
   - If exists, return it
   - If not exists, create new:
     - Generate title: "New Conversation"  (will update with actual title later)
     - INSERT INTO conversations (user_id, title, created_at, updated_at)
     - Return created conversation

2. async def get_conversation_history(
       conversation_id: str,
       limit: int = 50
   ) -> list[dict]
   - Fetch: SELECT * FROM messages WHERE conversation_id = ? 
            ORDER BY created_at ASC LIMIT limit
   - Format as: [{"role": "user"/"assistant", "content": "..."}]
   - Return formatted list

3. async def store_message(
       conversation_id: str,
       role: str,
       content: str,
       tool_calls: list[dict] = []
   ) -> Message
   - Validate role is "user" or "assistant"
   - INSERT INTO messages (conversation_id, role, content, created_at)
   - For each tool_call in tool_calls:
     - INSERT INTO message_tool_calls (message_id, tool_name, arguments, result, status)
   - Return created message

4. async def update_conversation_timestamp(conversation_id: str) -> None
   - UPDATE conversations SET updated_at = NOW() WHERE id = conversation_id

5. def generate_conversation_title(first_message: str) -> str
   - Extract first 5-8 words from message
   - Remove articles (a, an, the), small conjunctions
   - Capitalize first letter
   - Max 50 characters
   - Return title string

Use database session from db.py.
Include error handling for all database operations.
Add logging for audit trail.
```

**Success Criteria**:
- [ ] get_or_create_conversation works
- [ ] get_conversation_history returns formatted list
- [ ] store_message persists to database
- [ ] Tool calls stored correctly
- [ ] update_conversation_timestamp works
- [ ] generate_conversation_title works
- [ ] Error handling complete

**Time Estimate**: 1 hour

**Dependencies**: TASK 3.2

---

### TASK 3.4: Implement Agent process_message Logic

**Objective**: Complete OpenAI agent message processing

**Prerequisites**: 
- TASK 3.3 completed
- openai_agent.py created

**AI Agent Prompt**:
```
Implement the process_message method in backend/services/openai_agent.py

Method signature:
async def process_message(
    user_id: int,
    message: str,
    conversation_history: list[dict]
) -> dict

Implementation steps:
1. Build messages array for OpenAI API:
   - Add system prompt as first message (role="system")
   - Add all conversation_history items (role="user"/"assistant")
   - Add current user message (role="user")

2. Call OpenAI Agents API:
   response = self.client.chat.completions.create(
       model=self.model,
       messages=messages,
       tools=self.tools,  # MCP tool definitions
       tool_choice="auto"
   )

3. Process response:
   - Check response.choices[0].finish_reason
   - If "tool_calls": handle tool invocations
   - If "stop": return final message

4. Handle tool calls:
   - For each tool_call in response.choices[0].message.tool_calls:
     - Extract tool_name and arguments
     - Call MCP tool (import from mcp_server)
     - Get tool result
     - Add to messages: {"role": "tool", "content": json.dumps(result)}
   - Continue conversation loop until finish_reason == "stop"

5. Extract final response text:
   - Get response.choices[0].message.content

6. Return result dict:
{
    "status": "success",
    "message": final_response_text,
    "tool_calls": [
        {
            "tool": "add_task",
            "arguments": {...},
            "result": {...}
        }
    ]
}

7. Error handling:
   - Try/catch OpenAI API errors
   - Try/catch MCP tool errors
   - Return error response:
{
    "status": "error",
    "message": "I encountered an error. Please try again.",
    "error_code": "OPENAI_ERROR|TOOL_ERROR|..."
}

Import MCP tools from mcp_server.tools module.
Handle conversation loops (agent may need multiple tool calls).
Log all API calls for debugging.

Reference specs/agent.md for agent behavior rules.
```

**Success Criteria**:
- [ ] Builds correct message format
- [ ] Calls OpenAI API correctly
- [ ] Handles tool calls loop
- [ ] Extracts final response
- [ ] Returns correct format
- [ ] Error handling complete
- [ ] Logging in place

**Time Estimate**: 1.5 hours

**Dependencies**: TASK 3.3

---

## PHASE 4: Testing & Deployment

### TASK 4.1: End-to-End Testing

**Objective**: Test complete flow from chat to database

**Prerequisites**: 
- TASK 3.4 completed
- All backend complete

**AI Agent Prompt**:
```
Create file backend/tests/integration/test_chat_flow.py with E2E tests.

Test scenarios:

1. User Registration & Login
   - Create account with valid credentials
   - Verify user created in database
   - Login and receive JWT token
   - Verify token is valid

2. Create Task via Chat
   - Send message: "Add a task to buy milk tomorrow"
   - Verify agent response received
   - Verify task created in database with correct fields
   - Verify conversation persisted
   - Verify message stored

3. List Tasks
   - Create 3 tasks
   - Send message: "Show me my tasks"
   - Verify agent lists all tasks
   - Verify correct formatting

4. Update Task
   - Create task with priority "low"
   - Send message: "Change priority to high"
   - Verify task updated in database
   - Verify agent confirms change

5. Complete Task
   - Create task
   - Send message: "Mark as done"
   - Verify task marked complete
   - Verify completed_date set

6. Delete Task (with confirmation)
   - Create task
   - Send message: "Delete this task"
   - Verify agent asks for confirmation
   - Send: "yes"
   - Verify task deleted from database

7. Conversation History
   - Send: "Add task 1"
   - Send: "Add task 2"
   - Send: "List my tasks"
   - Verify agent sees both tasks in context
   - Verify history persists

8. Error Handling
   - Send invalid JWT → verify 401
   - Send with no message → verify validation error
   - Hit rate limit → verify 429
   - Invalid instruction → verify graceful error response

Use pytest fixtures for test database.
Use transactions to rollback between tests.
Mock external APIs if needed (OpenAI).

Test should be fully runnable with: pytest backend/tests/integration/test_chat_flow.py
```

**Success Criteria**:
- [ ] All test scenarios pass
- [ ] Database state correct after each test
- [ ] No test contamination (isolation)
- [ ] Error cases covered
- [ ] Performance acceptable

**Time Estimate**: 2 hours

**Dependencies**: TASK 3.4

---

### TASK 4.2: Performance Testing & Optimization

**Objective**: Verify performance targets are met

**Prerequisites**: 
- TASK 4.1 completed

**AI Agent Prompt**:
```
Create performance test in backend/tests/performance/test_load.py

Tests:
1. Single chat request timing:
   - Send chat message
   - Measure response time
   - Target: < 3 seconds (P95)
   - Include: agent processing, tool calls, database

2. Tool execution timing:
   - Test each MCP tool individually
   - Measure execution time per tool
   - Target: < 500ms per tool

3. Database query timing:
   - Test list_tasks with 100 tasks
   - Test list_tasks with 1000 tasks
   - Verify indexes used (EXPLAIN ANALYZE)
   - Target: < 100ms query time

4. Conversation history loading:
   - Test with 50 messages, 100 messages, 200 messages
   - Measure loading time
   - Target: < 1 second for 100 messages

5. Concurrent requests:
   - Simulate 10 concurrent users
   - Each sends 5 chat requests
   - Verify no errors, measure total time
   - Target: < 5 seconds for all 50 requests

Use timeit module for precise measurements.
Log response times for analysis.
Create performance report with results.

If any target not met, identify bottleneck and suggest optimization.
```

**Success Criteria**:
- [ ] Chat endpoint: < 3 seconds
- [ ] Tool execution: < 500ms
- [ ] Database queries: < 100ms
- [ ] History load: < 1 second
- [ ] Concurrent handling: smooth
- [ ] Report created

**Time Estimate**: 1 hour

**Dependencies**: TASK 4.1

---

### TASK 4.3: Deployment Configuration

**Objective**: Set up production deployment files

**Prerequisites**: 
- TASK 4.2 completed

**AI Agent Prompt**:
```
Create/update deployment configuration files:

1. backend/.env.example
   List all required environment variables:
   - DATABASE_URL=postgresql://user:pass@localhost/dbname
   - OPENAI_API_KEY=sk-...
   - JWT_SECRET=random-secret-key-min-32-chars
   - APP_ENV=development|production
   - LOG_LEVEL=DEBUG|INFO|WARNING
   - RESEND_API_KEY=re-... (optional)
   - CORS_ORIGINS=http://localhost:3000,https://example.com

2. backend/render.yaml
   Render deployment configuration:
   - Build command: pip install -r requirements.txt
   - Start command: uvicorn main:app --host 0.0.0.0 --port $PORT
   - Environment from .env
   - Health check: GET /health
   - Database migrations via alembic

3. backend/Dockerfile (optional)
   - Python 3.11 base image
   - Install requirements
   - Run alembic migrations on startup
   - Start FastAPI server

4. docker-compose.yml (for local development)
   - PostgreSQL 15 service
   - FastAPI backend service
   - Volumes for data persistence
   - Network for service communication
   - Environment variables

5. health check endpoints (update main.py):
   - GET /health/ready → check database connection
   - GET /health/live → check if container alive
   - Both used by orchestrators

6. Update requirements.txt:
   - List all Python package dependencies
   - Pin versions for reproducibility

Include comments explaining each configuration.
Reference deployment guide for production setup.
```

**Success Criteria**:
- [ ] .env.example complete
- [ ] render.yaml valid
- [ ] docker-compose.yml works locally
- [ ] Health checks implemented
- [ ] requirements.txt complete
- [ ] Documentation included

**Time Estimate**: 1 hour

**Dependencies**: TASK 4.2

---

### TASK 4.4: Documentation

**Objective**: Create complete API and setup documentation

**Prerequisites**: 
- TASK 4.3 completed

**AI Agent Prompt**:
```
Create documentation files:

1. backend/API.md
   - API Overview
   - Authentication (JWT)
   - Rate limiting
   - Error codes reference
   - Endpoint documentation:
     - POST /api/{user_id}/chat
     - GET /api/{user_id}/tasks
     - POST /api/auth/login
     - POST /api/auth/register
   - Example requests/responses
   - Common error scenarios

2. backend/MCP_TOOLS.md
   - Quick reference for all 5 tools
   - Parameters and types
   - Example requests
   - Response formats
   - Error codes

3. DEPLOYMENT.md (in root)
   - Local development setup
   - Database setup
   - Environment variables
   - Running tests
   - Deployment to Render
   - Monitoring and logs
   - Troubleshooting

4. backend/README.md (update)
   - Project overview
   - Architecture summary
   - Quick start guide
   - Project structure
   - Development commands

Include:
- Code examples
- curl examples for APIs
- Configuration examples
- Troubleshooting tips
```

**Success Criteria**:
- [ ] API.md complete with all endpoints
- [ ] MCP_TOOLS.md matches implementation
- [ ] DEPLOYMENT.md covers setup
- [ ] README.md updated
- [ ] Examples include curl commands
- [ ] Troubleshooting section helpful

**Time Estimate**: 1 hour

**Dependencies**: TASK 4.3

---

## Task Dependencies Graph

```
TASK 1.1 (DB migrations)
    ↓
TASK 1.2 (Task table)
    ↓
TASK 1.3-1.5 (Models)
    ├→ TASK 1.6 (Init)
    │   ↓
TASK 2.1 (MCP server init)
    ↓
TASK 2.2-2.6 (Implement tools)
    ↓
TASK 2.7 (Unit tests)
    ├→ TASK 3.1 (OpenAI agent)
    │   ├→ TASK 3.2 (Chat endpoint)
    │   │   ├→ TASK 3.3 (Conversation service)
    │   │   │   ├→ TASK 3.4 (Agent logic)
    │   │   │   │   ├→ TASK 4.1 (E2E tests)
    │   │   │   │   │   ├→ TASK 4.2 (Performance)
    │   │   │   │   │   │   ├→ TASK 4.3 (Deployment)
    │   │   │   │   │   │   │   ├→ TASK 4.4 (Documentation)
```

---

## Summary

**Total Tasks**: 24
**Estimated Timeline**: 8-12 days
**Parallel Opportunities**:
- TASK 1.7 (JWT verify) can run in parallel with TASK 1.2
- TASK 3.1 (Agent setup) can start when TASK 2.1 complete
- Frontend testing can run in parallel with backend

**Critical Path**:
TASK 1.1 → TASK 1.2 → TASK 2.1 → TASK 2.2-2.6 → TASK 2.7 → TASK 3.1 → TASK 3.4 → TASK 4.1

Each task has clear input requirements and success criteria for AI agents to work autonomously.
