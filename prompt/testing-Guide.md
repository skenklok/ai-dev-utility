Create testing documentation.

## File: docs/TESTING.md

## Required Sections

### Overview
- RSpec for Ruby (unit, integration, request specs)
- Jest for React components
- SimpleCov for Ruby coverage
- Jest coverage

### Running Tests

#### Ruby Tests
```bash
# Full suite
REDIS_URL=redis://localhost:6379/1 bundle exec rspec

# Specific file
bundle exec rspec spec/services/my_service_spec.rb

# With coverage report
COVERAGE=true bundle exec rspec

# Full suite
npm test

# With coverage
npm test -- --coverage

# Watch mode
npm test -- --watch

Test Structure
spec/models/ — Model validations, scopes
spec/services/ — Business logic
spec/jobs/ — Background job behavior
spec/requests/ — API integration tests
spec/clients/ — External API clients
app/javascript/**/*.test.js — React components
Writing Tests
Factory Usage

Factory Usage
ruby
# Use FactoryBot
let(:shop) { create(:shop) }
let(:customer) { create(:customer, shop: shop) }
Mocking External APIs
ruby
# Use WebMock
stub_request(:get, /api.shopify.com/).to_return(...)
Time-Sensitive Tests
ruby
# Use Timecop or travel_to
travel_to(Time.zone.local(2026, 1, 5)) do
  # test code
end

Coverage Expectations
Target: 90%+ for Ruby
Target: 80%+ for JavaScript
CI Integration
How tests run in CI, required checks.


---

### 4. DEPLOYMENT.md — Deployment Guide

```markdown
Create deployment documentation.

## File: docs/DEPLOYMENT.md

## Required Sections

### Prerequisites
- Heroku CLI installed
- Access to Heroku apps
- Shopify Partner account

### Environments
| Environment | URL | Purpose |
|-------------|-----|---------|
| Staging | ai-cart-staging.herokuapp.com | Testing |
| Production | ai-cart-production.herokuapp.com | Live |

### Initial Setup
Reference HEROKU_SETUP.md for first-time setup.

### Deploying Updates

#### Via Git Push
```bash
git push heroku main

Via GitHub Actions (if applicable)
Describe CI/CD pipeline.

Environment Variables
Which to set per environment
How to update safely
Database Migrations
bash
heroku run rails db:migrate -a ai-cart-production
Sidekiq/Redis
How Sidekiq is configured
Scaling workers
Monitoring
Heroku logs: heroku logs -t -a ai-cart-production
/health endpoint
/health/detailed for metrics
Rollback
bash
heroku rollback -a ai-cart-production
Troubleshooting
Common issues and solutions.