
---

### 5. ARCHITECTURE.md — System Architecture

```markdown
Create architecture documentation.

## File: docs/ARCHITECTURE.md

## Required Sections

### System Overview
```mermaid
graph TD
    A[Shopify Store] -->|Webhooks| B[ai-cart]
    A -->|OAuth| B
    B -->|API| C[SendGrid]
    B -->|API| D[OpenAI/Anthropic]
    B -->|Store| E[(PostgreSQL)]
    B -->|Queue| F[(Redis)]
    F -->|Jobs| G[Sidekiq]