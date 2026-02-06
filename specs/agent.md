# Agent Specification: Phase III AI-Powered Todo Chatbot

## Overview

This document defines the AI Agent behavior, system prompt, tool usage rules, and integration patterns for the Phase III Todo AI Chatbot. The agent uses OpenAI Agents SDK with stateless MCP tools for task management.

---

## Agent Architecture

### Agent Type
- **Framework**: OpenAI Agents SDK (vs OpenAI Functions)
- **Model**: GPT-4 (or gpt-4-turbo for cost optimization)
- **Tool Integration**: Model Context Protocol (MCP) tools for task operations
- **State Management**: Stateless agent (conversation state stored in database)

### Agent Capabilities
1. **Natural Language Understanding**: Parse user intent for task operations
2. **Context Awareness**: Maintain conversation history and task context
3. **Tool Execution**: Invoke MCP tools to persist changes to database
4. **Confirmation Required**: Ask for confirmation before creating/updating/deleting tasks
5. **Error Handling**: Graceful error recovery with user-friendly messages

---

## System Prompt

```
You are an AI assistant helping users manage their todo tasks through natural conversation.

Your role:
- Help users create, view, update, complete, and delete tasks
- Parse natural language for dates, times, priorities, and task details
- Maintain conversation context from history
- Confirm actions before executing them (especially for create/update/delete)
- Provide helpful summaries and insights about task status

Rules for tool usage:
1. Always confirm destructive operations (delete, complete) before execution
2. Use add_task for creating new tasks with parsed due_date and priority
3. Use list_tasks to fetch current task state before making changes
4. Use update_task for modifications, provide meaningful feedback
5. Use complete_task to mark tasks finished, offer to remove reminders
6. Use delete_task only when user explicitly requests it
7. If user is ambiguous, ask clarifying questions before tool execution

Date/Time Parsing:
- "tomorrow at 2pm" → due_date: tomorrow's date, time: 14:00
- "next monday" → calculate next Monday's date
- "in 3 days" → current_date + 3 days
- "today" → current date
- If no time specified, assume end-of-day (23:59)

Response Format:
- Confirm action: "I'll [action] [task details]. Is that correct?"
- After action: "Done! I've [action] [task name]"
- For queries: Present information clearly, organized by priority or due date
- For errors: "I wasn't able to [action] because [reason]. Would you like to try again?"

Conversation Management:
- Generate conversation title from first user message (5-10 words)
- Maintain context across multiple messages in same conversation
- Reference previous messages when relevant ("As you mentioned earlier...")
- Summarize task status on request

Privacy & Security:
- Never store or mention raw passwords
- Only show user's own tasks
- Respect user's task visibility preferences
```

---

## Tool Usage Rules

### MCP Tool Contract

All MCP tools follow this contract:

```
Input Format:
- Parameters are sent as JSON objects
- user_id is always included for context and authorization
- Database operations return standardized responses

Output Format:
{
  "status": "success" | "error",
  "message": "Human-readable description",
  "data": {
    // tool-specific response data
  },
  "timestamp": "ISO-8601 datetime",
  "error_code": "ERROR_CODE" (if status == "error")
}
```

### Tool Specifications

#### 1. add_task
**Purpose**: Create a new task for the user

**Input**:
```json
{
  "user_id": 123,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread, vegetables",
  "due_date": "2026-02-15",
  "priority": "medium",
  "tags": ["shopping", "urgent"]
}
```

**Agent Behavior**:
1. Parse user input for title, due_date, priority
2. Ask confirmation: "I'll create a task: '[title]' due [date] with [priority] priority. OK?"
3. Execute tool on user confirmation
4. Respond with confirmation and task details

**Validation Rules**:
- title: required, 1-200 characters
- due_date: optional, must be future date or today
- priority: "low", "medium", "high" (default: "low")
- tags: optional array of strings, max 5 tags

---

#### 2. list_tasks
**Purpose**: Retrieve user's tasks with optional filtering

**Input**:
```json
{
  "user_id": 123,
  "status": "all" | "pending" | "completed",
  "priority": null | "low" | "medium" | "high",
  "sort_by": "due_date" | "priority" | "created_at",
  "limit": 50
}
```

**Agent Behavior**:
1. Fetch tasks matching filters
2. Group by status or priority if multiple tasks
3. Format response with due dates and priority indicators
4. Include task count summary

**Response Data**:
```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "due_date": "2026-02-15",
      "priority": "medium",
      "completed": false,
      "tags": ["shopping"]
    }
  ],
  "count": 15,
  "pending_count": 10,
  "completed_count": 5
}
```

---

#### 3. update_task
**Purpose**: Modify an existing task

**Input**:
```json
{
  "user_id": 123,
  "task_id": 1,
  "title": "Buy groceries",
  "description": "Updated description",
  "due_date": "2026-02-15",
  "priority": "high",
  "tags": ["shopping", "urgent"]
}
```

**Agent Behavior**:
1. Fetch task to verify ownership and current state
2. Show what will change: "I'll update '[task]': [changes]. OK?"
3. Execute update on confirmation
4. Provide confirmation with new task state

**Validation Rules**:
- task_id: required, must exist and belong to user
- At least one field must be provided for update
- Cannot update completed status here (use complete_task)

---

#### 4. complete_task
**Purpose**: Mark a task as complete

**Input**:
```json
{
  "user_id": 123,
  "task_id": 1,
  "cancel_reminder": true
}
```

**Agent Behavior**:
1. Fetch task to verify ownership
2. Ask confirmation: "I'll mark '[task name]' as complete. Proceed?"
3. Mark completed, set completion_date = now
4. Cancel any pending reminders if flag is true
5. Respond with confirmation

**Response Data**:
```json
{
  "task": { /* updated task object */ },
  "reminder_cancelled": true,
  "message": "Great! I've marked 'Buy groceries' as complete"
}
```

---

#### 5. delete_task
**Purpose**: Permanently delete a task

**Input**:
```json
{
  "user_id": 123,
  "task_id": 1
}
```

**Agent Behavior**:
1. Fetch task to verify ownership
2. **IMPORTANT**: Always ask confirmation: "I'll permanently delete '[task name]'. Are you sure? (Type 'yes' to confirm)"
3. Execute delete only after explicit confirmation
4. Respond with confirmation

**Safety Rules**:
- Require explicit user confirmation for delete
- Confirm task title in message to avoid accidental deletion
- Log deletions for audit trail
- Cancel any pending reminders

---

## Conversation State Management

### Conversation Model

```
{
  "id": "uuid",
  "user_id": 123,
  "title": "Auto-generated from first message",
  "created_at": "ISO-8601",
  "updated_at": "ISO-8601",
  "messages": [
    {
      "id": "uuid",
      "role": "user" | "assistant",
      "content": "message text",
      "timestamp": "ISO-8601",
      "tool_calls": [
        {
          "tool_name": "add_task",
          "arguments": { /* tool input */ },
          "result": { /* tool output */ }
        }
      ]
    }
  ]
}
```

### State Persistence Rules

1. **No In-Memory State**: Agent maintains no task state in memory
2. **Database-Driven**: All state retrieved from database on each request
3. **Conversation History**: Passed to agent for context as message array
4. **Tool Execution**: Results stored in database immediately
5. **Idempotency**: Tool calls should be safe to retry (no double-charging, etc.)

### Conversation Title Generation

```
Rules:
- Extract first 5-8 words from user's first message
- Remove articles (a, an, the), small conjunctions
- Capitalize first letter
- Max 50 characters
- Examples:
  "Add a task to buy groceries" → "Buy groceries"
  "Can you help me schedule my meeting?" → "Schedule meeting"
  "I need to prepare for tomorrow's presentation" → "Prepare for presentation"
```

---

## Error Handling

### Error Categories

| Error Type | Agent Response | User Impact |
|-----------|-----------------|------------|
| **Invalid Tool Input** | Log error, ask user to clarify | "I didn't understand. Could you provide more details?" |
| **Task Not Found** | Check with list_tasks, suggest alternatives | "I couldn't find that task. Would you like me to show all your tasks?" |
| **Unauthorized Access** | Block silently, log security event | "I couldn't access that task." |
| **Database Error** | Retry once, then fallback to manual entry | "I'm having trouble saving. Please try again or contact support." |
| **Rate Limited** | Inform user, suggest waiting | "I'm processing too many requests. Please wait a moment and try again." |

### Recovery Strategies

1. **Ambiguous Intent**: Ask clarifying questions
2. **Missing Information**: Request additional details
3. **Transient Failures**: Retry once automatically
4. **Permanent Failures**: Suggest manual action or contact support

---

## Agent Workflow Example

### Scenario: User wants to add a task

```
User: "I need to call my mom tomorrow at 3pm"

Agent Processing:
1. Parse intent: "add_task"
2. Extract fields:
   - title: "Call mom"
   - due_date: tomorrow's date
   - (implicit time: 3pm → store in description or custom field)
   - priority: "medium" (inferred from natural context)
3. Fetch list_tasks to show context if needed
4. Confirm: "I'll create a task: 'Call mom' due tomorrow at 3pm with medium priority. Sound good?"

User: "Yes"

Agent Execution:
1. Call add_task with extracted fields
2. Receive success response with task ID
3. Respond: "Done! I've added 'Call mom' to your todo list for tomorrow at 3pm"
4. Store message + tool call in conversation history
```

---

## Testing Requirements

### Unit Tests
- [ ] Agent parses dates correctly
- [ ] Agent respects confirmation requirements
- [ ] Agent handles tool errors gracefully
- [ ] Agent maintains conversation context

### Integration Tests
- [ ] Full flow: Parse → Confirm → Execute → Store
- [ ] Agent with database persistence
- [ ] Conversation history retrieval and replay
- [ ] Rate limiting enforcement

### E2E Tests
- [ ] Chat endpoint receives message
- [ ] Agent generates response
- [ ] Database state updated correctly
- [ ] Conversation history persists
- [ ] User can switch conversations

---

## Security Considerations

1. **User Isolation**: Agent only operates on current user's tasks
2. **Input Validation**: All tool inputs validated before execution
3. **Tool Audit Trail**: All tool calls logged with user_id, timestamp, inputs
4. **Rate Limiting**: Per-user request rate limits on chat endpoint
5. **Token Security**: Ensure JWT tokens validate before agent execution
6. **Data Sanitization**: Clean user input before passing to LLM

---

## Performance Targets

- **Agent Response Time**: < 3 seconds for simple queries
- **Tool Execution**: < 500ms per tool call
- **Conversation History Load**: < 1 second for full history (100+ messages)
- **Database Queries**: Indexed on (user_id, created_at) for fast filtering
