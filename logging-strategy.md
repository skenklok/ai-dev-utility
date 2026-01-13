You are my senior DevOps/Observability engineer. Analyze this monorepo and create a comprehensive logging strategy for each application.

## ANALYSIS SCOPE

### 1. Inventory Current Logging
For each app in the monorepo (backend, mobile, portal):
- Identify current logging mechanisms (console.log, logger libraries, etc.)
- Count total logging statements and categorize by type (info, error, debug, warn)
- Identify what is currently being logged (requests, errors, business events, etc.)
- Flag any problematic logging (sensitive data, missing context, inconsistent formats)

### 2. Best Practices Research
For each application type, document industry best practices:

**Backend (Node.js/Express):**
- Structured logging with JSON format
- Correlation IDs for request tracing
- Log levels (debug, info, warn, error, fatal)
- Request/response logging (without sensitive data)
- Error logging with stack traces (production-appropriate)
- Performance metrics (response times)

**Mobile (React Native/Expo):**
- Crash reporting integration
- Remote logging service
- Log level management per environment
- User action logging (anonymized)
- Network request logging
- Performance monitoring

**Portal (Vite/React):**
- Browser console logging strategy
- Error boundary logging
- Analytics integration
- Remote error reporting (Sentry, LogRocket)
- User session tracking (privacy-compliant)

### 3. Technology Recommendations
For each app, recommend specific tools:
- Logging libraries (pino, winston, loglevel, etc.)
- Remote services (Datadog, CloudWatch, Sentry, LogRocket)
- Log aggregation strategy
- Retention policies

### 4. Implementation Strategy
Provide:
- Logging service/utility code structure
- Configuration per environment (dev, staging, prod)
- What to log vs what NOT to log (PII, secrets, tokens)
- Log format standardization across apps
- Integration with existing error handling

## OUTPUT FORMAT

### A. Current State Analysis
Table showing current logging per app:
| App | Library | Log Statements | Issues Found |

### B. Logging Strategy Report
For each application:
1. Recommended logging library
2. Log levels and when to use each
3. What to log (with examples)
4. What NOT to log (with examples)
5. Environment-specific configuration
6. Remote aggregation strategy

### C. Implementation Roadmap
Prioritized list of changes with effort estimates

### D. User Stories for prd.json
Generate user stories in this exact JSON format for prd.json:
```json
{
  "id": "LOG-X",
  "title": "Title here",
  "status": "todo",
  "priority": "HIGH|MEDIUM|LOW",
  "subtasks": [
    {
      "id": "LOG-X.Y",
      "title": "Subtask title",
      "status": "todo",
      "description": "What to do",
      "acceptanceCriteria": [
        "AC 1",
        "AC 2"
      ]
    }
  ]
}
```
Include user stories for:

- Backend logging setup
- Mobile logging setup
- Portal logging setup
- Cross-app log correlation
- Remote aggregation integration
- Log monitoring/alerting
- E. Code Examples
- Provide concrete code snippets for:

- Logger service creation
- Request logging middleware
- Error logging with context
- Sensitive data redaction
- Environment-based configuration