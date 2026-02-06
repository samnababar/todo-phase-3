---
name: reviewer-security
description: Validates code for JWT verification and user isolation.
---

# Reviewer Security Skill

## Purpose
Ensure no security vulnerabilities in auth/DB code.

## Process
1. Scan generated code for JWT middleware on all routes.
2. Check every query filters by user_id.
3. Verify no cross-user leaks.
4. Suggest fixes if issues found.

## Examples
Flag: Missing `depends=JWTDependency` → "Add JWT verification".

## Guidelines
- Run after backend implementation.
- Reference jwt-security-skill.