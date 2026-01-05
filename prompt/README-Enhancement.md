Update the project README.md with comprehensive documentation.

## Current State
Review existing README.md (if any) and enhance it.

## Required Sections

### Overview
- Project name: AI Cart (Shopify Abandoned Cart Recovery)
- One-paragraph description
- Key features bullet list
- Tech stack summary

### Quick Start
```bash
# Prerequisites
# Clone, install, setup database, run

Configuration
Point to .env.example
List required vs optional vars
Shopify app setup instructions
Architecture Overview
Diagram or description of components
Link to ARCHITECTURE.md for details
Running the App
Development mode
With Sidekiq
With ngrok for webhooks
Testing
Commands to run tests
Coverage expectations
Link to TESTING.md
Deployment
Link to HEROKU_SETUP.md
CI/CD overview
API Reference
Link to API.md
Contributing
Code style (Rubocop)
PR process
Branch naming
License
MIT or appropriate license


---

### 2. API.md — API Reference

```markdown
Create comprehensive API documentation for all endpoints.

## File: docs/API.md

## Required Sections

### Authentication
- Session token auth for admin API
- Event token auth for storefront API
- Shopify webhook HMAC verification

### Admin API Endpoints

#### Templates
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | /api/templates | List all templates |
| POST | /api/templates | Create template |
| GET | /api/templates/:id | Get template |
| PATCH | /api/templates/:id | Update template |
| DELETE | /api/templates/:id | Delete template |
| POST | /api/templates/:id/approve | Approve template |
| POST | /api/templates/:id/revoke | Revoke approval |
| POST | /api/templates/:id/preview | Preview with sample data |

(Continue for: IncentiveRules, AIConfig, Holdout, Analytics)

### Storefront API

#### Events
| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | /events | Record single event |
| POST | /events/batch | Record batch events |

### Webhooks

#### Shopify Webhooks
| Topic | Endpoint | Purpose |
|-------|----------|---------|
| app/uninstalled | /webhooks/app_uninstalled | Handle uninstall |
| orders/create | /webhooks/orders_create | Cancel recovery |
| customers/data_request | /webhooks/customers_data_request | GDPR |
| customers/redact | /webhooks/customers_redact | GDPR |
| shop/redact | /webhooks/shop_redact | GDPR |

### Public Endpoints
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | /unsubscribe/:token | One-click unsubscribe |
| GET | /health | Health check |
| GET | /health/detailed | Detailed metrics (auth'd) |

### Response Formats
Document standard response shapes, error formats.

### Rate Limits
Document any rate limiting on endpoints.