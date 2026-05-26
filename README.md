# Natural Language Analytics Engine
### Conversational SQL and Intelligent Data Visualization

An enterprise-grade natural language analytics platform that converts user questions into validated SQL queries, executes them against connected databases, and returns intelligent visual and text responses with production-level security guardrails to prevent misuse, injection attacks, and data exposure.

---

## Architecture Diagram

![NL Analytics Architecture](architecture.png)

---

## Repository Database, Not Production

The AI system never connects to the production database. It connects to a dedicated repository database, a sanitized read-only copy prepared specifically for analytics and AI queries.

| Production Database | Repository Database |
|---|---|
| Live operational data | Sanitized analytics copy |
| Real PII | PII masked or removed |
| Updated in real time | Refreshed on schedule |
| NEVER touched by AI | AI queries here only |

Even if the entire AI system is compromised, the attacker cannot access production data because the AI never had a connection to it.

---

## Read-Only Database User

This is the most critical guardrail. It operates at the database permission level, completely outside the application. No matter what SQL arrives, the connection simply does not have permission to write, update, or delete anything.

Defense in depth model:

- Layer 1: Input guardrails block suspicious questions
- Layer 2: SQL validation blocks destructive keywords and injection patterns
- Layer 3: Read-only database user, PERMISSION DENIED at database level regardless of what SQL arrived

Setup for PostgreSQL:

    CREATE USER ai_readonly WITH PASSWORD 'strong_password';
    GRANT CONNECT ON DATABASE your_database TO ai_readonly;
    GRANT USAGE ON SCHEMA public TO ai_readonly;
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO ai_readonly;
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO ai_readonly;

Setup for SQL Server / Azure SQL:

    CREATE LOGIN ai_readonly WITH PASSWORD = 'strong_password';
    CREATE USER ai_readonly FOR LOGIN ai_readonly;
    EXEC sp_addrolemember 'db_datareader', 'ai_readonly';

---

## Database Attack Types and Mitigations

### SQL Injection Attacks

| Attack | Mitigation |
|---|---|
| Classic injection OR 1=1 | Input sanitization + parameterized queries |
| UNION extraction | Block UNION SELECT patterns |
| Comment bypass via -- or /* | Block all comment sequences |
| Stacked queries via semicolons | One statement only |
| Subquery exfiltration | Limit subquery depth |

### Data Exfiltration Attacks

| Attack | Mitigation |
|---|---|
| Full table dump SELECT * | Block SELECT *, require explicit columns |
| No WHERE clause on sensitive tables | Require WHERE clause |
| Schema enumeration via information_schema | Block system catalog access |
| Version probing SELECT @@version | Block system variables |
| Linked server access | Block four-part naming |

### Database Load and Crash Attacks

| Attack | Mitigation |
|---|---|
| Cartesian product SELECT * FROM a, b, c | Limit JOIN count |
| Recursive loop WITH RECURSIVE | Block recursive CTEs |
| Time-based delay WAITFOR / SLEEP / BENCHMARK | Block delay functions |
| Connection exhaustion | Connection pool limits + rate limiting |
| Deep subquery nesting | Limit subquery depth |
| Large string operations REPEAT | Block string repetition functions |

### Privilege and Schema Attacks

| Attack | Mitigation |
|---|---|
| GRANT admin TO current_user | Read-only user blocks this entirely |
| CREATE USER hacker | Read-only user blocks this entirely |
| EXEC xp_cmdshell | Block EXEC and system procedures |
| ALTER TABLE | Read-only user blocks all DDL |
| LOAD_FILE / INTO OUTFILE | Block file system functions |

### Application-Level Attacks

| Attack | Mitigation |
|---|---|
| Prompt injection via question | Input sanitization + guardrail pre-check |
| Cost exhaustion via request flooding | Rate limiting per user and session |
| Schema leak via error messages | Sanitize all error messages before returning |
| Replay attack via captured tokens | JWT expiry + session invalidation |

---

## Chart Selection Logic

Rule-based, not LLM-driven. Deterministic and predictable.

| Data Shape | Chart Type |
|---|---|
| Single metric | KPI card |
| One dimension, one measure | Bar chart |
| Time dimension + measure | Line chart |
| Part of whole | Pie chart |
| Two numeric variables | Scatter plot |
| Many rows and columns | Table |

---

## Tech Stack

| Component | Technology |
|---|---|
| Backend API | FastAPI / Python |
| NL-to-SQL | Azure OpenAI (GPT-4) |
| Databases | PostgreSQL / SQL Server / Azure SQL |
| Query Cache | Redis |
| Authentication | Azure AD / JWT / RBAC |
| Audit Logging | PostgreSQL (append-only) |
| Infrastructure | Docker / Azure App Services |

---

## Key Design Decisions

**Why a read-only database user as a separate layer from SQL validation?**
Application-level validation can have bugs. A sufficiently creative attacker may find edge cases that bypass keyword detection. The read-only database user is a hard permission enforced by the database engine itself. It does not depend on application code being correct.

**Why rule-based chart selection instead of LLM-driven?**
Chart selection based on data shape is deterministic. A time series always suits a line chart. Using an LLM adds latency, cost, and variability for a problem that does not need intelligence.

**Why a repository database instead of connecting to production?**
Even a fully compromised AI system cannot reach production data if it never had access to begin with. Separation at the infrastructure level is more reliable than any application-level control.

---

## Related Repositories

- [enterprise-rag-pipeline](https://github.com/rahilvasaya-tech/enterprise-rag-pipeline)
- [ai-architecture-playbook](https://github.com/rahilvasaya-tech/ai-architecture-playbook)
- [blob-rag-ingestion-pipeline](https://github.com/rahilvasaya-tech/blob-rag-ingestion-pipeline)

---

## Author

**Rahil Vasaya**
Enterprise AI Solutions Architect | GenAI & RAG Systems
[linkedin.com/in/rahilvasaya](https://linkedin.com/in/rahilvasaya)
