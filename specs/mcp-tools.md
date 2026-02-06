# MCP Tools Specification: Phase III Todo AI Chatbot

## Overview

This document provides detailed specifications for all MCP (Model Context Protocol) tools used by the AI agent to manage tasks. Tools are stateless and persist data to a PostgreSQL database.

---

## MCP Tool Registry

### Available Tools (5 Core Tools)

1. **add_task** - Create new task
2. **list_tasks** - Retrieve filtered tasks
3. **update_task** - Modify task properties
4. **complete_task** - Mark task as finished
5. **delete_task** - Permanently remove task

---

## Tool Definitions

### 1. add_task

**Description**: Create a new task for the authenticated user.

**Category**: Write Operation

**Parameters**:

| Parameter | Type | Required | Constraints | Default |
|-----------|------|----------|-------------|---------|
| `user_id` | integer | Yes | Valid user ID | N/A |
| `title` | string | Yes | 1-200 chars, non-empty | N/A |
| `description` | string | No | 0-2000 chars | "" |
| `due_date` | string (YYYY-MM-DD) | No | ISO-8601, future or today | null |
| `priority` | enum | No | "low", "medium", "high" | "low" |
| `tags` | array[string] | No | Max 5 items, each ≤50 chars | [] |

**Request Example**:
```json
{
  "user_id": 42,
  "title": "Complete project proposal",
  "description": "Finalize Q1 project proposal for review",
  "due_date": "2026-02-20",
  "priority": "high",
  "tags": ["work", "urgent"]
}
```

**Success Response** (status 200):
```json
{
  "status": "success",
  "message": "Task created successfully",
  "data": {
    "id": 156,
    "user_id": 42,
    "title": "Complete project proposal",
    "description": "Finalize Q1 project proposal for review",
    "due_date": "2026-02-20",
    "priority": "high",
    "tags": ["work", "urgent"],
    "completed": false,
    "created_at": "2026-02-06T10:30:45Z",
    "updated_at": "2026-02-06T10:30:45Z"
  },
  "timestamp": "2026-02-06T10:30:45Z"
}
```

**Error Responses**:

| Error | Status | Message | Cause |
|-------|--------|---------|-------|
| VALIDATION_ERROR | 400 | "Invalid title: must be 1-200 characters" | Title length violation |
| VALIDATION_ERROR | 400 | "Invalid due_date: must be today or future" | Past date provided |
| INVALID_PRIORITY | 400 | "Priority must be 'low', 'medium', or 'high'" | Invalid priority value |
| TOO_MANY_TAGS | 400 | "Maximum 5 tags allowed" | Tag count exceeded |
| UNAUTHORIZED | 401 | "User not authenticated" | Invalid user_id |
| RATE_LIMIT | 429 | "Rate limit exceeded" | Too many requests |
| DB_ERROR | 500 | "Failed to create task" | Database error |

**Idempotency**:
- Not idempotent (each call creates new task)
- Client should deduplicate requests based on content hash if retry needed

**Rate Limit**: 100 add_task calls per user per hour

---

### 2. list_tasks

**Description**: Retrieve tasks matching filters. Returns paginated results.

**Category**: Read Operation

**Parameters**:

| Parameter | Type | Required | Constraints | Default |
|-----------|------|----------|-------------|---------|
| `user_id` | integer | Yes | Valid user ID | N/A |
| `status` | enum | No | "all", "pending", "completed" | "all" |
| `priority` | enum | No | "low", "medium", "high", null | null |
| `sort_by` | enum | No | "due_date", "priority", "created_at" | "due_date" |
| `limit` | integer | No | 1-100 | 50 |
| `offset` | integer | No | ≥ 0 | 0 |

**Request Example**:
```json
{
  "user_id": 42,
  "status": "pending",
  "priority": null,
  "sort_by": "due_date",
  "limit": 25,
  "offset": 0
}
```

**Success Response** (status 200):
```json
{
  "status": "success",
  "message": "Tasks retrieved successfully",
  "data": {
    "tasks": [
      {
        "id": 156,
        "title": "Complete project proposal",
        "description": "Finalize Q1 project proposal for review",
        "due_date": "2026-02-20",
        "priority": "high",
        "tags": ["work", "urgent"],
        "completed": false,
        "completed_date": null,
        "created_at": "2026-02-06T10:30:45Z",
        "updated_at": "2026-02-06T10:30:45Z"
      },
      {
        "id": 155,
        "title": "Review team feedback",
        "description": null,
        "due_date": "2026-02-15",
        "priority": "medium",
        "tags": [],
        "completed": false,
        "completed_date": null,
        "created_at": "2026-02-05T14:22:10Z",
        "updated_at": "2026-02-05T14:22:10Z"
      }
    ],
    "total_count": 18,
    "pending_count": 15,
    "completed_count": 3,
    "returned_count": 2,
    "limit": 25,
    "offset": 0
  },
  "timestamp": "2026-02-06T11:00:00Z"
}
```

**Empty Result**:
```json
{
  "status": "success",
  "message": "No tasks found",
  "data": {
    "tasks": [],
    "total_count": 0,
    "pending_count": 0,
    "completed_count": 0,
    "returned_count": 0
  }
}
```

**Error Responses**:

| Error | Status | Message |
|-------|--------|---------|
| UNAUTHORIZED | 401 | "User not authenticated" |
| INVALID_FILTER | 400 | "Invalid status filter value" |
| RATE_LIMIT | 429 | "Rate limit exceeded" |
| DB_ERROR | 500 | "Failed to retrieve tasks" |

**Performance**:
- Response time: < 500ms for user with 100+ tasks
- Database query: SELECT * FROM tasks WHERE user_id = ? ORDER BY [field]

**Caching**: Results can be cached for 30 seconds per user per filter combination

---

### 3. update_task

**Description**: Modify an existing task's properties.

**Category**: Write Operation

**Parameters**:

| Parameter | Type | Required | Constraints | Default |
|-----------|------|----------|-------------|---------|
| `user_id` | integer | Yes | Valid user ID | N/A |
| `task_id` | integer | Yes | Valid task ID, owned by user | N/A |
| `title` | string | No | 1-200 chars if provided | (unchanged) |
| `description` | string | No | 0-2000 chars | (unchanged) |
| `due_date` | string (YYYY-MM-DD) | No | ISO-8601, future or today | (unchanged) |
| `priority` | enum | No | "low", "medium", "high" | (unchanged) |
| `tags` | array[string] | No | Max 5 items | (unchanged) |

**Constraints**:
- Must provide at least one field to update
- Cannot change completed status (use complete_task instead)
- Cannot change task ownership (user_id)

**Request Example**:
```json
{
  "user_id": 42,
  "task_id": 156,
  "priority": "critical",
  "due_date": "2026-02-18",
  "tags": ["work", "urgent", "critical"]
}
```

**Success Response** (status 200):
```json
{
  "status": "success",
  "message": "Task updated successfully",
  "data": {
    "id": 156,
    "title": "Complete project proposal",
    "description": "Finalize Q1 project proposal for review",
    "due_date": "2026-02-18",
    "priority": "critical",
    "tags": ["work", "urgent", "critical"],
    "completed": false,
    "completed_date": null,
    "updated_at": "2026-02-06T12:00:00Z",
    "changes": {
      "priority": { "old": "high", "new": "critical" },
      "due_date": { "old": "2026-02-20", "new": "2026-02-18" },
      "tags": { "old": ["work", "urgent"], "new": ["work", "urgent", "critical"] }
    }
  },
  "timestamp": "2026-02-06T12:00:00Z"
}
```

**Error Responses**:

| Error | Status | Message |
|-------|--------|---------|
| TASK_NOT_FOUND | 404 | "Task not found" |
| UNAUTHORIZED | 401 | "Not authorized to update this task" |
| VALIDATION_ERROR | 400 | "Invalid due_date: must be today or future" |
| NO_CHANGES | 400 | "No fields provided for update" |
| RATE_LIMIT | 429 | "Rate limit exceeded" |
| DB_ERROR | 500 | "Failed to update task" |

**Idempotency**:
- Idempotent (multiple identical calls produce same result)
- Use task_id + user_id + hash(fields) as idempotency key

**Rate Limit**: 150 update_task calls per user per hour

---

### 4. complete_task

**Description**: Mark a task as completed and optionally cancel its reminder.

**Category**: Write Operation

**Parameters**:

| Parameter | Type | Required | Constraints | Default |
|-----------|------|----------|-------------|---------|
| `user_id` | integer | Yes | Valid user ID | N/A |
| `task_id` | integer | Yes | Valid task ID, owned by user | N/A |
| `cancel_reminder` | boolean | No | true or false | true |

**Request Example**:
```json
{
  "user_id": 42,
  "task_id": 156,
  "cancel_reminder": true
}
```

**Success Response** (status 200):
```json
{
  "status": "success",
  "message": "Task marked as complete",
  "data": {
    "id": 156,
    "title": "Complete project proposal",
    "completed": true,
    "completed_date": "2026-02-06T12:15:30Z",
    "reminder_cancelled": true,
    "tasks_remaining": 14
  },
  "timestamp": "2026-02-06T12:15:30Z"
}
```

**Error Responses**:

| Error | Status | Message |
|-------|--------|---------|
| TASK_NOT_FOUND | 404 | "Task not found" |
| ALREADY_COMPLETED | 400 | "Task is already completed" |
| UNAUTHORIZED | 401 | "Not authorized to update this task" |
| RATE_LIMIT | 429 | "Rate limit exceeded" |
| DB_ERROR | 500 | "Failed to complete task" |

**Idempotency**:
- Idempotent (completing already-completed task returns same result)
- Safe for retry

**Side Effects**:
- Sets completed = true
- Sets completed_date = now
- Cancels any associated reminder if cancel_reminder = true
- Updates updated_at timestamp

**Rate Limit**: 200 complete_task calls per user per hour

---

### 5. delete_task

**Description**: Permanently delete a task. Cannot be undone.

**Category**: Delete Operation (Destructive)

**Parameters**:

| Parameter | Type | Required | Constraints | Default |
|-----------|------|----------|-------------|---------|
| `user_id` | integer | Yes | Valid user ID | N/A |
| `task_id` | integer | Yes | Valid task ID, owned by user | N/A |
| `confirmed` | boolean | Yes | Must be explicitly true | N/A |

**Constraints**:
- Requires explicit confirmation (confirmed = true)
- Agent must show task title to user before requesting confirmation
- Audit log entry created with timestamp and user_id

**Request Example**:
```json
{
  "user_id": 42,
  "task_id": 156,
  "confirmed": true
}
```

**Success Response** (status 200):
```json
{
  "status": "success",
  "message": "Task deleted permanently",
  "data": {
    "deleted_task_id": 156,
    "deleted_task_title": "Complete project proposal",
    "tasks_remaining": 13,
    "deleted_at": "2026-02-06T12:30:00Z"
  },
  "timestamp": "2026-02-06T12:30:00Z"
}
```

**Error Responses**:

| Error | Status | Message |
|-------|--------|---------|
| TASK_NOT_FOUND | 404 | "Task not found" |
| NOT_CONFIRMED | 400 | "Deletion not confirmed (confirmed=false)" |
| UNAUTHORIZED | 401 | "Not authorized to delete this task" |
| RATE_LIMIT | 429 | "Rate limit exceeded" |
| DB_ERROR | 500 | "Failed to delete task" |

**Idempotency**:
- Idempotent if confirmed=true (deleting non-existent task returns 404)
- **NOT idempotent if confirmed=false** (will fail on first call)

**Audit Trail**:
```sql
INSERT INTO audit_log (
  user_id, action, resource_type, resource_id, 
  resource_title, timestamp
) VALUES (
  42, 'DELETE_TASK', 'task', 156, 
  'Complete project proposal', NOW()
)
```

**Rate Limit**: 50 delete_task calls per user per hour

**Safety Features**:
- Requires explicit confirmation parameter
- Returns deleted task title for verification
- Cannot be called without user confirmation
- Audit logged for compliance

---

## Tool Response Format Standard

All tools follow this response structure:

```json
{
  "status": "success" | "error",
  "message": "Human-readable message describing the result",
  "data": {
    // Tool-specific response data
  },
  "timestamp": "2026-02-06T12:30:00Z",
  "error_code": "ERROR_CODE" // Only present if status == "error"
}
```

### Response Status Codes

| Status | HTTP Code | Meaning |
|--------|-----------|---------|
| success | 200 | Operation completed successfully |
| error | 4xx | Client error (bad input, not found, unauthorized) |
| error | 5xx | Server error (database, unexpected failure) |

---

## Tool Authorization & Security

### Per-Tool Security Rules

**Read Operations** (list_tasks):
- Require valid user_id
- Only return tasks owned by user
- Rate limited to 500 calls/hour

**Write Operations** (add_task, update_task):
- Require valid user_id
- Verify user ownership before update
- All input validated and sanitized
- Rate limited to 150 calls/hour per operation

**Destructive Operations** (delete_task, complete_task):
- Require valid user_id
- Delete requires explicit confirmation
- Audit logged
- Rate limited to 50-200 calls/hour depending on operation

### Audit Logging

All tool invocations logged to database:

```python
{
  "timestamp": "2026-02-06T12:30:00Z",
  "user_id": 42,
  "tool_name": "delete_task",
  "status": "success",
  "input_hash": "sha256_hash_of_input",
  "duration_ms": 145,
  "ip_address": "203.0.113.42"
}
```

---

## Tool Implementation Checklist

- [ ] Tool registered with MCP server
- [ ] All parameters validated against constraints
- [ ] Response format matches spec (status, message, data, timestamp)
- [ ] Error responses include error_code
- [ ] Rate limiting implemented
- [ ] Authorization checks in place
- [ ] Audit logging for all operations
- [ ] Database queries optimized with indexes
- [ ] Pagination working for list_tasks
- [ ] Idempotency key generation for retries
- [ ] Error handling with graceful degradation
- [ ] Tool documentation auto-generated for agent

---

## Testing Specifications

### Unit Tests per Tool

```
add_task:
  ✓ Creates task with all fields
  ✓ Creates task with minimum fields
  ✓ Validates title length
  ✓ Rejects past due_date
  ✓ Validates priority enum
  ✓ Enforces max 5 tags
  ✓ Returns correct response format

list_tasks:
  ✓ Returns all tasks for user
  ✓ Filters by status (pending/completed)
  ✓ Filters by priority
  ✓ Sorts by due_date
  ✓ Paginates correctly
  ✓ Returns correct counts
  ✓ Handles empty result

update_task:
  ✓ Updates single field
  ✓ Updates multiple fields
  ✓ Returns change tracking
  ✓ Rejects task not owned by user
  ✓ Rejects non-existent task
  ✓ Idempotent on retry

complete_task:
  ✓ Marks task complete
  ✓ Sets completed_date
  ✓ Cancels reminder
  ✓ Returns pending count
  ✓ Idempotent (already complete)

delete_task:
  ✓ Deletes task when confirmed=true
  ✓ Rejects when confirmed=false
  ✓ Creates audit log entry
  ✓ Rejects non-existent task
  ✓ Rejects unauthorized user
```

### Integration Tests

```
✓ Tools persist data to database
✓ list_tasks retrieves add_task results
✓ update_task changes reflect in list_tasks
✓ complete_task updates are visible
✓ delete_task removes from list_tasks
✓ Concurrent operations maintain consistency
✓ Rate limiting blocks excess requests
```
