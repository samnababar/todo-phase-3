<!--
  Sync Impact Report
  ===================
  Version change: N/A -> 1.0.0 (Initial constitution)

  Modified principles: None (initial creation)

  Added sections:
  - P1: AI-Driven Development (AIDD)
  - P2: Spec-Driven Development
  - P3: Monolithic Architecture
  - P4: Technology Stack Compliance
  - P5: Authentication & Security
  - P6: AI Chat Interface Standards
  - P7: MCP Server Implementation
  - P8: Visual Design Excellence
  - P9: Task Reminder System
  - P10: Quality & Accessibility

  Removed sections: None

  Templates requiring updates:
  - .specify/templates/spec-template.md (to be created)
  - .specify/templates/plan-template.md (to be created)
  - .specify/templates/tasks-template.md (to be created)

  Deferred items: None
-->

# Project Constitution

**Project Name:** AI-Powered Todo Chatbot (Phase III)

**Version:** 1.0.0

**Ratification Date:** 2026-01-22

**Last Amended Date:** 2026-01-22

---

## Preamble

This constitution establishes the foundational principles, technical standards, and governance
rules for the AI-Powered Todo Chatbot project. All development activities, architectural
decisions, and implementations MUST adhere to these principles. This document serves as the
single source of truth for project governance.

---

## Principles

### P1: AI-Driven Development (AIDD)

**Statement:** 100% of code MUST be generated through AI-driven development using Claude Code.

**Rules:**
- No manual code writing is permitted; all code is generated via AI specifications
- Specifications MUST be refined iteratively until correct output is achieved
- Every code change MUST be traceable to a specification or prompt
- Human developers act as architects and reviewers, not code writers

**Rationale:** AIDD ensures consistency, reduces human error, and demonstrates the viability
of AI-assisted software development at scale.

---

### P2: Spec-Driven Development

**Statement:** All features MUST be developed using Spec-Kit Plus methodology with formal
specifications preceding implementation.

**Rules:**
- Every feature MUST have a spec.md before implementation begins
- Plans (plan.md) MUST document architectural decisions
- Tasks (tasks.md) MUST be testable and traceable to specifications
- Prompt History Records (PHR) MUST capture all significant interactions
- Architectural Decision Records (ADR) MUST document significant technical choices

**Rationale:** Spec-driven development creates auditable, reproducible, and maintainable
software while enabling AI agents to operate effectively within defined boundaries.

---

### P3: Monolithic Architecture

**Statement:** The application MUST follow a monolithic architecture pattern with clear
internal boundaries.

**Rules:**
- Single repository with `/frontend` and `/backend` folders
- Shared configuration and environment variables via `.env` files
- No microservices, containers, or orchestration (Phase IV scope)
- Integrated deployment pipeline for both frontend and backend
- Specifications managed in `/specs` folder

**Rationale:** Monolithic architecture reduces operational complexity for Phase III while
maintaining clear separation of concerns internally.

---

### P4: Technology Stack Compliance

**Statement:** All implementations MUST use the prescribed technology stack without deviation.

**Frontend Stack:**
- Next.js 16+ with App Router (required)
- Tailwind CSS + custom CSS for styling
- TypeScript for type safety

**Backend Stack:**
- Python FastAPI for API services
- SQLModel as ORM layer
- Neon Serverless PostgreSQL for persistence
- OpenAI Agents SDK for AI capabilities
- Official MCP SDK (Python) for tool integration

**Rules:**
- No alternative frameworks or libraries without ADR approval
- Version constraints MUST be respected
- All dependencies MUST be explicitly declared

**Rationale:** Stack consistency ensures predictable behavior, simplified debugging, and
cohesive AI code generation.

---

### P5: Authentication & Security

**Statement:** All user interactions MUST be authenticated and all API endpoints MUST be
secured with JWT tokens.

**Authentication Requirements:**
- Better Auth integration with email verification
- Signup requires: email, name, password
- Email verification system (accept fake emails, verify format)
- Verified email stored in Neon DB for reminder notifications
- JWT token-based API security for all protected endpoints

**Security Rules:**
- MUST prevent SQL injection via parameterized queries (SQLModel)
- MUST validate and sanitize all user inputs
- MUST use HTTPS in production
- Secrets MUST be stored in environment variables, never in code
- JWT tokens MUST have appropriate expiration times

**Rationale:** Security is non-negotiable; authentication protects user data and enables
personalized features like reminders.

---

### P6: AI Chat Interface Standards

**Statement:** The AI assistant MUST be implemented as a separate page with persistent
conversation history.

**Interface Requirements:**
- Dedicated AI Assistant page (not embedded in dashboard)
- OpenAI ChatKit integration with Agents SDK
- Left side: Active chat interface
- Right sidebar: Chat history from Neon DB
- Conversation state persisted across sessions in database

**Technical Requirements:**
- Stateless backend architecture
- Database-persisted conversation state
- Real-time message updates
- Graceful error handling for AI failures

**Rationale:** Separation of chat from dashboard improves UX focus; persistence enables
continuity and context retention.

---

### P7: MCP Server Implementation

**Statement:** Exactly 5 MCP tools MUST be implemented using the official MCP SDK for task
operations.

**Required Tools:**
1. `add_task` - Create task with title, description, reminder, date, day
2. `delete_task` - Remove task by ID
3. `update_task` - Modify existing task details
4. `mark_as_completed_task` - Toggle task completion status
5. `view_task` - Retrieve tasks with filtering support

**Rules:**
- All tools MUST be implemented using official MCP SDK (Python)
- Tools MUST validate inputs before database operations
- Tools MUST return structured responses for AI consumption
- Error handling MUST provide actionable feedback

**Rationale:** MCP tools enable the AI agent to perform task management operations on behalf
of users, creating a natural language interface.

---

### P8: Visual Design Excellence

**Statement:** The application MUST deliver a visually stunning dark theme with purple
accents and exceptional animations.

**Design Requirements:**
- Black and purple color theme throughout
- Landing page: Left-aligned header text, right side high-quality image
- High-quality thematic images throughout the application
- Exceptional CSS animations on landing page
- Dashboard: Purple accents complementing black theme
- Smooth, complementary animations on dashboard

**Rules:**
- Animations MUST not impact performance (60fps target)
- Images MUST be optimized for web delivery
- Design MUST maintain visual hierarchy and readability
- Animations MUST enhance UX, not distract

**Rationale:** Visual excellence differentiates the product and demonstrates attention to
detail in AI-generated applications.

---

### P9: Task Reminder System

**Statement:** The application MUST support email-based task reminders with user-defined
scheduling.

**Requirements:**
- Task model MUST include reminder fields: date, day, time
- UI MUST provide intuitive reminder input controls
- Backend service MUST check and trigger reminders
- Reminders MUST be sent via email to users verified address

**Rules:**
- Reminder metadata MUST be stored in the task model
- Backend MUST have a reliable scheduler for reminder checks
- Failed reminder attempts MUST be logged and retried
- Users MUST be able to modify or cancel reminders

**Rationale:** Reminders increase task completion rates and provide value beyond basic
todo functionality.

---

### P10: Quality & Accessibility

**Statement:** All code MUST meet quality standards and accessibility requirements.

**Quality Standards:**
- Clean, maintainable code structure
- Comprehensive error handling with user-friendly messages
- Type safety via TypeScript (frontend) and type hints (backend)
- No dead code or unused dependencies

**Accessibility Requirements:**
- WCAG 2.1 AA compliance required
- Keyboard navigation support
- Screen reader compatibility
- Sufficient color contrast ratios
- Focus indicators on interactive elements

**Responsive Design:**
- Mobile-first approach
- Breakpoints: mobile, tablet, desktop
- Touch-friendly targets (minimum 44x44px)

**Rationale:** Quality and accessibility ensure the application serves all users effectively
and maintains long-term maintainability.

---

## Governance

### Amendment Procedure

1. Propose amendment via ADR with rationale
2. Review impact on existing specifications and implementations
3. Update constitution with version increment
4. Propagate changes to dependent templates
5. Document in Sync Impact Report

### Versioning Policy

Constitution follows semantic versioning:
- **MAJOR (X.0.0):** Backward-incompatible principle changes or removals
- **MINOR (X.Y.0):** New principles or significant expansions
- **PATCH (X.Y.Z):** Clarifications, wording fixes, non-semantic updates

### Compliance Review

- All specifications MUST reference relevant principles
- Code reviews MUST verify principle adherence
- Deviations require explicit ADR with justification
- Quarterly reviews assess principle effectiveness

---

## Constraints

### Explicit Exclusions (Phase IV Scope)

The following are explicitly OUT OF SCOPE for Phase III:
- Docker containerization
- Kubernetes orchestration
- Helm charts
- Microservices architecture
- CI/CD pipeline automation beyond basic deployment

### Development Constraints

- All code generated via Claude Code (no manual coding)
- Specifications MUST be refined until correct output
- No external API integrations beyond prescribed stack
- No premium/paid third-party services without approval

---

## Appendix A: File Structure

```
/
├── frontend/           # Next.js application
├── backend/            # FastAPI application
├── specs/              # Feature specifications
│   └── <feature>/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
├── history/
│   ├── prompts/        # PHR records
│   │   ├── constitution/
│   │   ├── general/
│   │   └── <feature>/
│   └── adr/            # Architecture Decision Records
├── .specify/
│   ├── memory/
│   │   └── constitution.md
│   └── templates/
└── .env                # Environment variables
```

---

## Appendix B: Success Metrics

| Metric | Target |
|--------|--------|
| AI Code Generation | 100% |
| WCAG Compliance | AA Level |
| Animation Performance | 60fps |
| API Response Time | < 500ms p95 |
| Test Coverage | > 80% |

---

*This constitution is the authoritative source for project governance. All team members,
AI agents, and automated systems MUST operate within these boundaries.*
