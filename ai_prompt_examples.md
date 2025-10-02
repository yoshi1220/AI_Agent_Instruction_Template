# System Development - AI Agent Instruction Examples

## 1. Feature Implementation

### Data Import Feature
```
Implement a data import feature with the following requirements:

**Functionality:**
- Support CSV and Excel file formats
- Parse and validate data before importing
- Show progress indicator during import
- Display success/error summary after completion

**Technical Requirements:**
- Backend: Create REST API endpoint POST /api/import
- Use streaming for large files (>10MB)
- Implement transaction rollback on error
- Log all import attempts with timestamp and user ID

**Validation Rules:**
- Required fields: customer_id, order_date, amount
- Date format: YYYY-MM-DD
- Amount: positive decimal value
- Max rows per import: 10,000

Please implement the backend logic first, including error handling and unit tests.
```

### Frontend Option Addition
```
Add a new import type option to the frontend without backend implementation:

**Task:**
Add "汎用データ" (Generic Data) as a new option in the import type dropdown

**Location:**
- Component: ImportForm.tsx (or equivalent)
- Dropdown ID: importTypeSelect

**Requirements:**
- Add the option with value: "generic_data"
- Display text: "汎用データ"
- Position: Add after "顧客データ" option
- Do NOT implement backend handler yet
- Add a temporary disabled state with tooltip: "Coming soon"

**Additional:**
- Update TypeScript types if needed
- Ensure consistent styling with existing options
- Add a comment: "// Backend implementation pending - Issue #XXX"
```

### Cost Calculation Specification Change
```
Create an implementation plan for modifying the cost calculation process:

**Current Specification:**
- Material cost = unit_price × quantity
- Labor cost = hourly_rate × hours
- Overhead = (material_cost + labor_cost) × 0.15
- Total cost = material_cost + labor_cost + overhead

**New Specification:**
- Add: Equipment depreciation = equipment_cost / useful_life_years / 12
- Change overhead calculation to tiered:
  - If total direct cost < ¥100,000: 15%
  - If ¥100,000 ≤ total direct cost < ¥500,000: 12%
  - If total direct cost ≥ ¥500,000: 10%
- Add: Seasonal adjustment factor (winter months +5%)

**Request:**
Provide an implementation plan including:
1. Database schema changes (if needed)
2. Code modules to modify
3. Migration strategy for existing data
4. Test cases to cover new logic
5. Backward compatibility considerations
6. Performance impact assessment

Assume we're using PostgreSQL and Node.js backend.
```

---

## 2. API Development

### REST API Endpoint Creation
```
Create a REST API endpoint for product search:

**Endpoint:** GET /api/v1/products/search

**Query Parameters:**
- keyword (string, optional): Search in name and description
- category_id (integer, optional): Filter by category
- min_price, max_price (decimal, optional): Price range
- in_stock (boolean, optional): Availability filter
- page (integer, default: 1): Pagination
- limit (integer, default: 20, max: 100): Items per page
- sort_by (enum: price_asc, price_desc, name, created_at): Sort order

**Response Format:**
```json
{
  "data": [...],
  "pagination": {
    "current_page": 1,
    "total_pages": 10,
    "total_items": 200,
    "items_per_page": 20
  }
}
```

**Requirements:**
- Use SQL query optimization (indexes on searchable fields)
- Implement proper error handling (400, 404, 500)
- Add request validation middleware
- Include OpenAPI documentation
- Write integration tests

Use Express.js and Sequelize ORM.
```

### GraphQL Query Implementation
```
Implement a GraphQL query for order details with related data:

**Query Name:** getOrderDetails

**Input:**
- orderId: ID! (required)
- includeCustomer: Boolean (default: true)
- includeItems: Boolean (default: true)

**Return Fields:**
- Order: id, orderNumber, orderDate, status, totalAmount
- Customer (optional): id, name, email, phone
- OrderItems (optional): array of {productId, productName, quantity, unitPrice, subtotal}
- Shipping: address, method, trackingNumber, estimatedDelivery

**Technical:**
- Implement DataLoader to prevent N+1 queries
- Add field-level authorization checks
- Cache customer data for 5 minutes
- Handle soft-deleted records appropriately

Provide the schema definition and resolver implementation.
```

---

## 3. Database Operations

### Migration Script
```
Create a database migration for adding audit trail functionality:

**Requirements:**
- Add audit_logs table with fields:
  - id (UUID, primary key)
  - table_name (varchar)
  - record_id (varchar)
  - action (enum: CREATE, UPDATE, DELETE)
  - old_values (jsonb)
  - new_values (jsonb)
  - user_id (integer, foreign key)
  - ip_address (inet)
  - timestamp (timestamp with time zone)

**Additional:**
- Create indexes on: table_name, record_id, user_id, timestamp
- Add database trigger function to auto-populate audit_logs
- Apply triggers to tables: users, orders, products, payments
- Ensure down migration properly removes triggers and table

**Constraints:**
- Retention policy: Keep logs for 2 years
- Partition table by month for performance
- Consider GDPR implications for user data

Provide both up and down migration scripts in PostgreSQL.
```

### Query Optimization
```
Optimize this slow-running query:

**Current Query:**
```sql
SELECT o.*, c.name, c.email,
       (SELECT SUM(oi.quantity * oi.unit_price) 
        FROM order_items oi 
        WHERE oi.order_id = o.id) as total
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.order_date >= '2024-01-01'
  AND o.status IN ('pending', 'processing')
ORDER BY o.order_date DESC
LIMIT 100;
```

**Issue:** Query takes 3-5 seconds with 500K orders

**Request:**
1. Analyze the query execution plan
2. Suggest index improvements
3. Rewrite query for better performance
4. Consider denormalization if appropriate
5. Provide before/after performance estimates

Include EXPLAIN ANALYZE output interpretation.
```

---

## 4. Frontend Development

### React Component Creation
```
Create a reusable DataTable component in React with TypeScript:

**Features:**
- Sortable columns (click header to sort)
- Pagination (client-side and server-side modes)
- Column filtering
- Row selection (single/multiple)
- Responsive design (mobile-friendly)
- Loading and empty states
- Customizable cell rendering

**Props Interface:**
```typescript
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  pagination?: PaginationConfig;
  sortable?: boolean;
  selectable?: 'single' | 'multiple' | false;
  onRowSelect?: (rows: T[]) => void;
  loading?: boolean;
  emptyMessage?: string;
}
```

**Technical Requirements:**
- Use TanStack Table (React Table v8)
- Implement with hooks and functional components
- Add proper TypeScript generics
- Include Storybook stories
- Write unit tests with React Testing Library
- Follow company design system (Tailwind CSS)
```

### Form Validation Implementation
```
Implement frontend validation for a user registration form:

**Form Fields:**
- email: valid email format, required
- password: min 8 chars, must include uppercase, lowercase, number, special char
- password_confirm: must match password
- username: 3-20 chars, alphanumeric and underscore only, required
- phone: optional, Japanese phone format (XXX-XXXX-XXXX or XXXXXXXXXXX)
- birth_date: optional, must be 18+ years old, not future date
- terms_accepted: required boolean

**Requirements:**
- Real-time validation (on blur)
- Display error messages in Japanese
- Disable submit button until form is valid
- Show password strength indicator
- Prevent double submission

**Technical:**
- Use React Hook Form + Zod for validation
- Create reusable validation schemas
- Implement custom validators for Japanese phone
- Add proper accessibility attributes (aria-invalid, aria-describedby)

Provide the complete form component with validation logic.
```

### State Management Setup
```
Set up Redux Toolkit store for a shopping cart feature:

**State Structure:**
```typescript
{
  cart: {
    items: CartItem[];
    totalQuantity: number;
    totalAmount: number;
    appliedCoupon: Coupon | null;
    status: 'idle' | 'loading' | 'error';
  }
}
```

**Required Actions/Thunks:**
- addToCart(product, quantity)
- removeFromCart(productId)
- updateQuantity(productId, quantity)
- applyCoupon(couponCode) - async
- clearCart()
- fetchCart() - async, load from backend
- syncCartToBackend() - async, save changes

**Business Rules:**
- Max quantity per item: 99
- Calculate discount from coupon
- Persist to localStorage on changes
- Debounce backend sync (3 seconds)
- Handle out-of-stock errors

Include slice definition, selectors, and middleware for localStorage sync.
```

---

## 5. Testing

### Unit Test Creation
```
Write comprehensive unit tests for the following service:

**Service:** OrderService.calculateOrderTotal()

**Method Signature:**
```typescript
calculateOrderTotal(
  items: OrderItem[],
  coupon?: Coupon,
  shippingMethod?: ShippingMethod,
  taxRate?: number
): OrderTotal
```

**Test Cases Needed:**
1. Basic calculation with no discounts
2. Apply percentage coupon (10% off)
3. Apply fixed amount coupon (¥500 off)
4. Free shipping threshold (¥5000+)
5. Tax calculation (default 10%, can override)
6. Minimum order amount validation
7. Invalid coupon code handling
8. Expired coupon rejection
9. Multiple items with different tax rates
10. Edge cases: empty cart, null values, zero prices

**Requirements:**
- Use Jest and TypeScript
- Mock dependencies (coupon validator, shipping calculator)
- Test error messages
- Aim for 100% code coverage
- Use describe blocks for organization
- Add performance test (should complete <10ms for 100 items)
```

### E2E Test Scenario
```
Create an E2E test for the checkout flow using Playwright:

**Test Scenario:** "Complete purchase as guest user"

**Steps:**
1. Navigate to product listing page
2. Add 2 different products to cart (quantities: 1 and 3)
3. Verify cart badge shows "4"
4. Open cart drawer, verify items and total
5. Proceed to checkout
6. Fill in shipping address form
7. Select standard shipping method
8. Fill in credit card payment details (use test card)
9. Apply coupon code "WELCOME10"
10. Verify discount is applied
11. Submit order
12. Wait for confirmation page
13. Verify order number is displayed
14. Check confirmation email is sent (mock SMTP)

**Requirements:**
- Use page object pattern
- Handle loading states and animations
- Take screenshot on failure
- Test across Chrome, Firefox, Safari
- Implement retry logic for network calls
- Mock payment gateway in test environment
- Verify database state after order creation

Provide the complete test file with proper assertions.
```

---

## 6. Bug Fixes

### Bug Investigation and Fix
```
Investigate and fix the following bug:

**Bug Report:**
- Title: "CSV export fails for large datasets"
- Severity: High
- Environment: Production
- Browser: Chrome 120, Firefox 115
- Error: "Maximum call stack size exceeded"

**Steps to Reproduce:**
1. Navigate to Reports > Sales Report
2. Select date range: 2024-01-01 to 2024-12-31
3. Click "Export to CSV"
4. Browser freezes, then shows error

**Additional Info:**
- Works fine for date ranges < 3 months
- ~50,000 rows when full year selected
- Current implementation uses recursive function
- Memory usage spikes to 2GB+

**Request:**
1. Analyze the current export implementation
2. Identify the root cause
3. Propose solution (streaming? web workers? chunking?)
4. Implement the fix
5. Add loading indicator for long exports
6. Write test case to prevent regression
7. Document max recommended export size

Assume the export code is in `/services/ExportService.js`.
```

### Race Condition Fix
```
Fix a race condition bug in the inventory system:

**Problem:**
When multiple users purchase the same product simultaneously, inventory sometimes goes negative.

**Current Flow:**
1. Frontend: User clicks "Add to Cart"
2. Backend: Check current stock (e.g., 5 units)
3. Backend: Calculate new stock (5 - 1 = 4)
4. Backend: Update stock to 4
5. Backend: Create order item

**Issue:**
Two requests can read stock=5 at the same time, both calculate new stock=4, causing double-sell.

**Requirements:**
- Implement proper locking mechanism
- Consider: pessimistic locking, optimistic locking, or row-level locks
- Handle timeout scenarios
- Return appropriate error to frontend when out of stock
- Ensure ACID properties
- Add retry logic with exponential backoff
- Log all inventory transactions for audit

**Tech Stack:** Node.js, PostgreSQL, Sequelize

Provide the fixed code with transaction handling and explain your locking strategy.
```

---

## 7. Performance Optimization

### Backend Performance Improvement
```
Improve performance of the dashboard API endpoint:

**Current Status:**
- Endpoint: GET /api/v1/dashboard/summary
- Average response time: 4.5 seconds
- Peak time response: 8+ seconds
- Called every page load and every 30 seconds

**Data Fetched:**
- Total sales (last 30 days)
- New customers (last 7 days)
- Pending orders count
- Low stock products (stock < 10)
- Top 5 selling products (last week)
- Revenue trend (daily, last 30 days)

**Request:**
1. Profile the endpoint to identify bottlenecks
2. Implement caching strategy (Redis?)
3. Optimize database queries (indexes, query restructuring)
4. Consider materialized views or pre-aggregation
5. Implement stale-while-revalidate pattern
6. Add cache warming for peak hours
7. Set up monitoring and alerts

**Target:** Reduce average response time to <500ms

Provide implementation plan with code samples.
```

### Frontend Performance Optimization
```
Optimize the performance of a slow React product listing page:

**Current Issues:**
- First Contentful Paint: 3.2s (target: <1.5s)
- Time to Interactive: 5.8s (target: <3s)
- Large bundle size: 850KB (target: <300KB)
- Unnecessary re-renders on filter changes
- Images not optimized (loading all at once)

**Page Features:**
- 100 products displayed
- Filters: category, price range, rating
- Sorting options
- Pagination
- Product images (200KB each)

**Optimization Tasks:**
1. Implement code splitting (lazy load filters)
2. Add React.memo to ProductCard component
3. Implement virtualization for product grid
4. Lazy load images with intersection observer
5. Debounce filter inputs
6. Prefetch next page data
7. Optimize images (WebP, responsive sizes)
8. Move filter logic to Web Worker
9. Analyze and remove unused dependencies

**Tools:** Use Lighthouse, React DevTools Profiler, Webpack Bundle Analyzer

Provide step-by-step optimization implementation with before/after metrics.
```

---

## 8. Security Implementation

### Authentication Implementation
```
Implement JWT-based authentication system:

**Requirements:**

**Registration:**
- Hash passwords with bcrypt (rounds: 12)
- Validate email uniqueness
- Send verification email
- Generate email verification token (expire: 24h)

**Login:**
- Rate limiting: 5 attempts per 15 minutes per IP
- Issue access token (expire: 15m) and refresh token (expire: 7d)
- Store refresh token in httpOnly cookie
- Log login attempts (success and failures)

**Token Refresh:**
- Endpoint: POST /api/auth/refresh
- Validate refresh token
- Issue new access token
- Rotate refresh token on use

**Logout:**
- Invalidate refresh token (blacklist in Redis)
- Clear cookies

**Additional Security:**
- Implement CSRF protection
- Add rate limiting middleware
- Hash tokens before storing
- Set appropriate CORS headers
- Add security headers (Helmet.js)

**Tech Stack:** Express.js, jsonwebtoken, bcrypt, Redis

Provide complete authentication middleware and endpoints.
```

### SQL Injection Prevention Audit
```
Review and fix SQL injection vulnerabilities in the following code:

**Code to Review:**
```javascript
// User search endpoint
app.get('/api/users/search', async (req, res) => {
  const { name, role, department } = req.query;
  
  let query = 'SELECT * FROM users WHERE 1=1';
  
  if (name) {
    query += ` AND name LIKE '%${name}%'`;
  }
  if (role) {
    query += ` AND role = '${role}'`;
  }
  if (department) {
    query += ` AND department_id = ${department}`;
  }
  
  const results = await db.query(query);
  res.json(results);
});
```

**Tasks:**
1. Identify all SQL injection vulnerabilities
2. Rewrite using parameterized queries
3. Add input validation and sanitization
4. Implement query builder (e.g., Knex.js)
5. Add input length limits
6. Escape special characters
7. Add logging for suspicious inputs
8. Write test cases demonstrating the fix

Provide the secured code with detailed explanations.
```

---

## 9. Code Refactoring

### Legacy Code Modernization
```
Refactor this legacy callback-based code to modern async/await:

**Current Code:**
```javascript
function processOrder(orderId, callback) {
  db.getOrder(orderId, function(err, order) {
    if (err) return callback(err);
    
    inventory.checkStock(order.items, function(err, available) {
      if (err) return callback(err);
      if (!available) return callback(new Error('Out of stock'));
      
      payment.charge(order.total, order.paymentMethod, function(err, charge) {
        if (err) return callback(err);
        
        inventory.decrementStock(order.items, function(err) {
          if (err) return callback(err);
          
          email.sendConfirmation(order.customerId, function(err) {
            if (err) console.error('Email failed:', err);
            callback(null, { orderId, chargeId: charge.id });
          });
        });
      });
    });
  });
}
```

**Requirements:**
- Convert to async/await
- Implement proper error handling
- Add transaction rollback on failure
- Make email sending non-blocking
- Add timeout handling
- Improve code readability
- Add JSDoc comments
- Implement retry logic for transient failures
- Add proper logging

Provide the refactored code with error handling strategy.
```

### Component Decomposition
```
Break down this large React component into smaller, reusable components:

**Current:** Single 800-line UserProfile.tsx component containing:
- User info display (avatar, name, email, bio)
- Edit profile form (multiple fields)
- Activity timeline (posts, comments, likes)
- Settings panel (notifications, privacy, security)
- Modal dialogs for photo upload, password change
- Data fetching and state management for all sections

**Requirements:**
1. Identify logical component boundaries
2. Create component hierarchy diagram
3. Extract reusable components (Button, Input, Avatar, etc.)
4. Implement proper prop drilling or context
5. Separate container and presentation components
6. Move data fetching to custom hooks
7. Create shared TypeScript interfaces
8. Ensure each component has single responsibility
9. Write unit tests for new components

**Constraints:**
- Maintain current functionality
- No breaking changes to parent components
- Follow atomic design principles
- Keep bundle size minimal

Provide the refactored component structure with file tree and main components.
```

---

## 10. Documentation

### API Documentation
```
Create comprehensive API documentation for the User Management API:

**Endpoints to Document:**
- POST /api/v1/users (Create user)
- GET /api/v1/users/:id (Get user)
- PUT /api/v1/users/:id (Update user)
- DELETE /api/v1/users/:id (Delete user)
- GET /api/v1/users (List users with pagination/filters)

**Required Sections:**
1. Overview and base URL
2. Authentication method (Bearer token)
3. For each endpoint:
   - Description
   - HTTP method and path
   - Path parameters
   - Query parameters
   - Request headers
   - Request body schema (with examples)
   - Response codes (200, 201, 400, 401, 403, 404, 500)
   - Response body schema (with examples)
   - Error response format
   - Sample cURL request
4. Common error codes and meanings
5. Rate limiting policy
6. Pagination format
7. Filtering and sorting syntax
8. Changelog

**Format:** OpenAPI 3.0 (Swagger) specification

Generate the YAML file and provide setup instructions for Swagger UI.
```

### Technical Design Document
```
Write a technical design document for implementing a notification system:

**Requirements:**

**1. Overview**
- System purpose and scope
- Target users and use cases

**2. Requirements**
- Functional requirements
- Non-functional requirements (scalability, latency, reliability)

**3. Architecture**
- High-level architecture diagram
- Component descriptions:
  - Notification Service
  - Queue System (RabbitMQ/SQS)
  - Email Provider Integration
  - Push Notification Service
  - SMS Gateway
- Data flow diagrams

**4. Database Design**
- Schema for: notifications, user_preferences, notification_templates
- Indexing strategy
- Partitioning considerations

**5. API Design**
- REST endpoints
- WebSocket for real-time notifications
- Request/response formats

**6. Technical Decisions**
- Why message queue? (vs direct sending)
- Queue selection rationale
- Retry and dead letter queue strategy
- Template engine choice

**7. Security Considerations**
- Authentication/authorization
- Data privacy (PII handling)
- Rate limiting to prevent spam

**8. Monitoring and Operations**
- Metrics to track
- Alerting strategy
- Logging approach

**9. Testing Strategy**
- Unit, integration, E2E tests
- Load testing plan

**10. Deployment Plan**
- Rollout strategy (phased/blue-green)
- Rollback procedures
- Database migrations

**11. Timeline and Milestones**

Use Markdown format with Mermaid diagrams where appropriate.
```

---

## 11. DevOps & Infrastructure

### CI/CD Pipeline Setup
```
Set up a CI/CD pipeline for a Node.js application using GitHub Actions:

**Requirements:**

**Build Stage:**
- Trigger on: push to main, pull requests
- Node version: 18.x and 20.x (matrix)
- Install dependencies
- Run linter (ESLint)
- Run type checker (TypeScript)
- Run unit tests with coverage
- Build application
- Generate coverage report

**Test Stage:**
- Run integration tests
- Run E2E tests (Playwright)
- Upload test artifacts
- Fail if coverage < 80%

**Security Stage:**
- Dependency vulnerability scan (npm audit)
- SAST scanning (SonarQube/CodeQL)
- Secret scanning
- License compliance check

**Deploy Stage (on main branch only):**
- Build Docker image
- Tag with git SHA and 'latest'
- Push to container registry
- Deploy to staging environment
- Run smoke tests
- Deploy to production (manual approval)
- Notify team on Slack

**Additional:**
- Cache node_modules for faster builds
- Parallel job execution where possible
- Rollback mechanism on deployment failure
- Store artifacts for 30 days

Provide the complete `.github/workflows/ci-cd.yml` file.
```

### Docker Configuration
```
Create Docker configuration for a microservices application:

**Services:**
1. Frontend (React/Next.js)
2. Backend API (Node.js/Express)
3. Background Worker (Node.js)
4. PostgreSQL database
5. Redis cache
6. Nginx reverse proxy

**Requirements:**

**Dockerfile for Backend:**
- Multi-stage build (build stage + production stage)
- Node.js 20 Alpine base image
- Non-root user
- Health check endpoint
- Optimize layer caching
- Production dependencies only
- Security best practices

**docker-compose.yml:**
- Service definitions for all components
- Network configuration (frontend, backend, database networks)
- Volume mounts for data persistence
- Environment variables
- Port mappings
- Depends_on with health checks
- Resource limits (CPU, memory)
- Logging configuration

**Additional Files:**
- .dockerignore
- docker-compose.override.yml for development
- Makefile for common Docker commands

**Goals:**
- Fast rebuilds during development
- Minimal production image size
- Proper service orchestration
- Data persistence

Provide all configuration files with detailed comments.
```

---

## 12. Real-World Scenarios

### Feature Flag Implementation
```
Implement a feature flag system for gradual rollout:

**Use Case:** Rolling out new payment provider to 10% of users

**Requirements:**

**Backend:**
- Feature flag configuration (JSON/database)
- Evaluation logic with rules:
  - Percentage rollout (0-100%)
  - User segment targeting (premium users, specific regions)
  - Date range activation
  - Environment-based flags (dev/staging/prod)
- Admin API to toggle flags
- Audit log for flag changes

**Frontend:**
- React hook: `useFeatureFlag('new-payment-provider')`
- Handle loading states
- Fallback UI when flag is off
- A/B testing support

**Rollout Strategy:**
1. Dev/staging: 100% enabled
2. Production: 0% (dark launch)
3. Production: 5% (canary)
4. Production: 20% (if metrics good)
5. Production: 50%
6. Production: 100%
7. Remove flag after 2 weeks

**Monitoring:**
- Track feature flag usage
- Error rates per flag state
- Performance metrics comparison
- User engagement metrics

Implement the feature flag system with admin UI and usage examples.
```

### Data Migration Strategy
```
Plan and implement a zero-downtime database migration:

**Scenario:** 
Splitting 'users' table into 'users' and 'user_profiles' to improve performance

**Current Schema:**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  password_hash VARCHAR(255),
  name VARCHAR(255),
  bio TEXT,
  avatar_url VARCHAR(500),
  birth_date DATE,
  phone VARCHAR(20),
  address TEXT,
  -- 20+ more profile fields
);
```

**Target Schema:**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  password_hash VARCHAR(255),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

CREATE TABLE user_profiles (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  name VARCHAR(255),
  bio TEXT,
  avatar_url VARCHAR(500),
  -- profile fields
);
```

**Migration Steps:**
1. Create new user_profiles table
2. Create data migration script
3. Deploy dual-write code (write to both schemas)
4. Backfill historical data
5. Verify data consistency
6. Switch reads to new schema
7. Remove dual-write code
8. Drop old columns from users table

**Requirements:**
- Zero downtime
- Data consistency checks
- Rollback plan at each step
- Performance monitoring during migration
- Estimated time: 2 million users

Provide detailed migration plan with SQL scripts and deployment steps.
```

---

---

## 13. Advanced Agent Patterns (Cursor/Cline/GitHub Copilot Style)

### Plan-Modify Cycle with Approval
```
I need to refactor our authentication system to use JWT instead of sessions.

**DO NOT implement yet. First:**
1. Analyze the current session-based authentication code
2. Identify all files that need to be modified
3. Create a detailed step-by-step migration plan
4. Highlight potential risks and rollback strategies
5. Estimate time for each step

Wait for my approval before proceeding with implementation.

**Current tech stack:**
- Express.js with express-session
- Redis session store
- PostgreSQL for user data
- Frontend: React with Context API

**Requirements:**
- Zero downtime migration
- Maintain existing user sessions during transition
- Add refresh token mechanism
- Keep session for admin users, JWT for regular users
```

### Structured Agent Task with XML Format
```
<task>
  <objective>
    Implement user profile photo upload feature
  </objective>
  
  <requirements>
    <backend>
      - Accept multipart/form-data uploads
      - Validate file type (jpg, png, webp only)
      - Max file size: 5MB
      - Resize to 400x400px thumbnail
      - Store original in S3, thumbnail in CDN
      - Update user profile with photo URLs
    </backend>
    
    <frontend>
      - Drag-and-drop upload zone
      - Image preview before upload
      - Crop tool (square aspect ratio)
      - Progress indicator during upload
      - Error handling with user-friendly messages
    </frontend>
    
    <security>
      - Validate file headers (not just extension)
      - Generate random filename to prevent overwrites
      - Add rate limiting (5 uploads per hour per user)
      - Scan for malware (ClamAV integration)
    </security>
  </requirements>
  
  <constraints>
    - Use existing AWS S3 bucket: user-photos-prod
    - Follow existing error handling patterns
    - Add unit and integration tests
    - Update API documentation
  </constraints>
  
  <execution_steps>
    1. Review existing file upload code (if any)
    2. Propose architecture and get approval
    3. Implement backend endpoints
    4. Create frontend component
    5. Add tests
    6. Update documentation
  </execution_steps>
</task>

Analyze this task and create an implementation plan. Ask clarifying questions before proceeding.
```

### Context-Aware Multi-File Changes
```
@workspace I need to add a "scheduled_at" field to our email notification system.

**Context:**
- We currently send emails immediately
- Need to support scheduling for future delivery
- This affects: database schema, API, background worker, and frontend

**Specific changes needed:**

1. **Database (migrations/add_scheduled_at.sql)**
   - Add scheduled_at TIMESTAMP field to notifications table
   - Add index on (scheduled_at, status) for efficient querying
   - Backfill existing records with current timestamp

2. **Backend API (src/api/notifications.js)**
   - Add optional scheduled_at to POST /api/notifications
   - Validate scheduled_at is future timestamp
   - Update notification creation logic

3. **Worker (src/workers/emailWorker.js)**
   - Modify job query to check scheduled_at
   - Only process notifications where scheduled_at <= NOW()
   - Add cron job to check every minute

4. **Frontend (components/NotificationForm.tsx)**
   - Add datetime picker for scheduled_at
   - Add "Send now" / "Schedule" toggle
   - Show timezone info to user

**Instructions:**
- Show me the proposed changes for each file as diffs
- Wait for my approval after showing all diffs
- Apply changes only after approval
- Run tests after each change

Start with the database migration.
```

### Question-Driven Development Approach
```
I need to build a real-time collaborative editing feature like Google Docs.

**Instead of jumping to implementation, please ask me questions about:**

1. **Scale & Performance:**
   - Expected number of concurrent users per document?
   - Average document size?
   - Required latency for updates?

2. **Technical Constraints:**
   - Existing tech stack I must work with?
   - Infrastructure preferences (WebSockets, Server-Sent Events, polling)?
   - Database requirements?

3. **Feature Scope:**
   - Which collaborative features? (cursor position, selection, presence indicators)
   - Conflict resolution strategy preference?
   - Offline support needed?

4. **UX Requirements:**
   - Real-time character-by-character or throttled updates?
   - Visual indicators for other users?
   - History/version control?

After gathering this information, propose 2-3 architectural approaches with pros/cons, then wait for my selection before creating an implementation plan.
```

### Agent Mode with Custom Instructions
```
Read the custom instructions from .github/copilot-instructions.md

Then, implement a user registration endpoint following our team's standards.

**Additional requirements for this specific task:**
- Endpoint: POST /api/v1/auth/register
- Required fields: email, password, firstName, lastName
- Optional fields: phone, birthDate
- Send verification email after registration
- Return JWT token immediately (user can verify later)
- Log registration in audit_events table

**What to check in custom instructions:**
- Error handling patterns
- Validation approach
- Response format standards
- Logging requirements
- Test structure

Create the implementation following all our team standards, with full test coverage.
```

### Iterative Refinement Pattern
```
I need a dashboard component for our analytics platform.

**Initial version:** Create a basic dashboard showing:
- Total users (last 30 days)
- Total revenue (last 30 days)
- Top 5 products

**After you complete the initial version:**
- I'll review and provide feedback
- Then iterate based on my comments
- Continue until I say "approved"

**Technical constraints:**
- React + TypeScript
- Use recharts for visualizations
- Fetch data from /api/dashboard endpoint
- Loading and error states required
- Responsive design (mobile-first)

**Process:**
1. Create initial version
2. Show me the code
3. Wait for my feedback (don't proceed automatically)
4. Implement feedback
5. Repeat steps 2-4 until approved

Start with the initial version.
```

### Agent with Tool Integration (MCP)
```
I need to analyze our GitHub repository issues and create a report.

**Use the following tools:**
@github-mcp: Access GitHub API to fetch issues
@database-mcp: Query our issue tracking database
@analytics-mcp: Generate visualizations

**Task:**
1. Fetch all open issues from github.com/ourcompany/ourproject
2. Categorize by labels (bug, feature, enhancement)
3. Cross-reference with our internal tracking database
4. Generate summary statistics:
   - Total open issues by category
   - Average time to close (from closed issues)
   - Top 5 contributors by issue comments
   - Issues with no activity in 30+ days

5. Create visualizations:
   - Bar chart: issues by category
   - Line chart: issue creation trend (last 6 months)
   - Pie chart: issue priority distribution

6. Write executive summary (3-4 paragraphs)

Output the report as a Markdown file: `reports/github-analysis-{date}.md`

If any tool fails, inform me and suggest alternatives.
```

---

## 14. Project-Specific Custom Instructions

### Example: .github/copilot-instructions.md
```markdown
# Project: E-Commerce Platform - AI Coding Guidelines

## Code Style & Standards

### TypeScript/JavaScript
- Use TypeScript strict mode
- Prefer functional components over class components
- Use async/await over .then() for promises
- Always handle errors with try-catch
- Use optional chaining (?.) and nullish coalescing (??)

### Naming Conventions
- Components: PascalCase (UserProfile.tsx)
- Hooks: camelCase with 'use' prefix (useAuth.ts)
- Utilities: camelCase (formatCurrency.ts)
- Constants: UPPER_SNAKE_CASE
- CSS classes: kebab-case (user-profile-card)

### File Organization
```
src/
├── components/      # Reusable UI components
├── pages/          # Route-level components
├── hooks/          # Custom React hooks
├── services/       # API calls
├── utils/          # Helper functions
├── types/          # TypeScript types
└── constants/      # App constants
```

## Error Handling Pattern

### API Errors
```typescript
try {
  const response = await apiCall();
  return response.data;
} catch (error) {
  if (error instanceof ApiError) {
    logger.error('API Error:', {
      endpoint: error.endpoint,
      status: error.status,
      message: error.message
    });
    throw new UserFacingError(error.userMessage);
  }
  throw error;
}
```

### Frontend Errors
- Always show user-friendly error messages
- Use toast notifications for temporary errors
- Use error boundaries for component errors
- Log all errors to Sentry

## Testing Requirements

### Unit Tests
- Use Jest + React Testing Library
- Test file naming: `ComponentName.test.tsx`
- Coverage threshold: 80% minimum
- Test user interactions, not implementation details

### Integration Tests
- Use Playwright for E2E tests
- Test critical user flows only
- Run before deploying to production

## API Response Format
All API responses must follow this format:
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional message",
  "timestamp": "2025-10-02T10:30:00Z"
}
```

Error responses:
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "User-friendly error message",
    "details": { ... }
  },
  "timestamp": "2025-10-02T10:30:00Z"
}
```

## Database Queries
- Always use parameterized queries (prevent SQL injection)
- Add indexes for frequently queried fields
- Use transactions for multi-step operations
- Soft delete records (deleted_at field) instead of hard delete

## Security Guidelines
- Never log sensitive data (passwords, tokens, credit cards)
- Sanitize all user inputs
- Use helmet.js for Express security headers
- Rate limit all public endpoints (express-rate-limit)
- Validate JWT tokens on protected routes

## Performance
- Lazy load routes and heavy components
- Use React.memo for expensive components
- Debounce user inputs (search, autocomplete)
- Paginate large data sets (max 100 items per page)
- Optimize images (WebP format, lazy loading)

## Git Commit Messages
Format: `<type>(<scope>): <subject>`

Types:
- feat: New feature
- fix: Bug fix
- refactor: Code refactoring
- test: Add/update tests
- docs: Documentation changes
- chore: Maintenance tasks

Example: `feat(auth): add password reset flow`

## When In Doubt
- Ask questions before implementing
- Propose multiple solutions with trade-offs
- Reference existing code patterns in the project
- Check if a utility/helper already exists before creating new ones
```

---

## 15. Real-World Prompt Examples from Popular Tools

### Cline-Style File Operations
```
<read_file>
  <path>src/services/paymentService.ts</path>
  <reason>Need to understand current payment processing logic before adding refund feature</reason>
</read_file>

After analyzing the file, create a new refund method that:
- Accepts orderId and reason parameters
- Validates the order is refundable (not already refunded, within 30 days)
- Calls Stripe API to process refund
- Updates order status in database
- Sends refund confirmation email

<write_to_file>
  <path>src/services/paymentService.ts</path>
  <content>
    [Your implementation here]
  </content>
</write_to_file>
```

### Cursor-Style Composer Mode
```
@workspace I need to add analytics tracking across the entire app.

**Global changes needed:**
1. Install and configure Google Analytics 4
2. Create analytics utility wrapper (src/utils/analytics.ts)
3. Add page view tracking to all routes
4. Add event tracking to key user actions:
   - Button clicks: "Add to Cart", "Checkout", "Sign Up"
   - Form submissions
   - Video plays
   - File downloads

**Requirements:**
- Don't track in development environment
- Respect user's DNT (Do Not Track) setting
- GDPR compliance: only track after cookie consent
- Use TypeScript for type-safe event names

**Files likely affected:**
- App.tsx (provider setup)
- All button components
- Form components
- Video player component
- Router configuration

Show me which files need changes and proposed implementations. Use automatic context detection or let me manually specify files.
```

### GitHub Copilot Agent Issue Assignment
```
Issue Title: Implement Password Strength Indicator

Description:
Add a real-time password strength indicator to the registration form.

Requirements:
- Visual indicator: Weak (red), Medium (yellow), Strong (green)
- Criteria for strength:
  - Weak: < 8 characters
  - Medium: 8+ characters with letters and numbers
  - Strong: 12+ characters with uppercase, lowercase, numbers, and special chars
- Show specific feedback (e.g., "Add special characters for strong password")
- Update in real-time as user types
- Accessible (screen reader support)

Technical:
- Component: src/components/auth/RegistrationForm.tsx
- Create new component: PasswordStrengthIndicator.tsx
- Use zxcvbn library for strength calculation
- Add unit tests

Labels: enhancement, good-first-issue

---

@copilot Please implement this feature following our coding standards.
```

---

## Template Usage Tips

**For Each Scenario:**
1. Replace bracketed items with your specific details
2. Adjust technical stack to match your environment
3. Add or remove requirements based on complexity
4. Include relevant context (existing code, error logs, etc.)
5. Specify output format preference (code, docs, diagram)
6. Set clear success criteria

**Communication Best Practices:**
- Start with business context before technical details
- Provide error messages and logs when debugging
- Mention what you've already tried
- Specify urgency and priority
- Ask for explanation along with code
- Request test cases for critical features

**Modern Agent-Specific Tips:**
- Use XML tags for complex, structured instructions
- Request plans before implementations for complex tasks
- Use @mentions to add context (files, symbols, tools)
- Specify "wait for approval" for multi-step changes
- Create custom instruction files for project standards
- Break large tasks into iterative refinements
