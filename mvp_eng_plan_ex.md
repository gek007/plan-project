# MVP Engineering Implementation Plan

## Architecture Overview

### Technology Stack

- **Framework**: Next.js 16.1.2 (App Router)
- **Runtime**: React 19.2.3
- **Language**: TypeScript 5
- **Database**: MongoDB with Mongoose 9.1.4
- **UI Framework**: shadcn/ui (Radix UI primitives + Tailwind CSS 4)
- **Styling**: Tailwind CSS 4 with PostCSS

### Architecture Pattern

**Full-Stack Monolith with API Routes**

```
┌─────────────────────────────────────────┐
│         Next.js Application             │
├─────────────────────────────────────────┤
│  App Router (Pages)                     │
│  ├── Client Components ('use client')   │
│  └── Server Components (default)        │
├─────────────────────────────────────────┤
│  API Routes (app/api/*)                 │
│  ├── RESTful endpoints                  │
│  └── Server-side logic                  │
├─────────────────────────────────────────┤
│  Data Layer                             │
│  ├── Mongoose Models (models/*)         │
│  ├── Database Connection (lib/mongodb)  │
│  └── TypeScript Types (types/*)         │
└─────────────────────────────────────────┘
           │
           ▼
    ┌──────────────┐
    │   MongoDB    │
    └──────────────┘
```

### Key Architectural Decisions

1. **App Router**: Using Next.js App Router for file-based routing and React Server Components support
2. **API Routes**: RESTful API endpoints in `app/api/` for all data operations
3. **Client Components**: Pages use 'use client' for interactivity and data fetching
4. **Connection Caching**: MongoDB connection cached globally to prevent multiple connections
5. **Component Architecture**: Reusable UI components (shadcn/ui) + domain-specific components

## Project Structure

```
garage2-claude/
├── app/                          # Next.js App Router
│   ├── api/                      # API Routes (REST endpoints)
│   │   ├── customers/
│   │   │   ├── route.ts          # GET all, POST new
│   │   │   └── [id]/route.ts     # GET, PUT, DELETE by ID
│   │   ├── cars/
│   │   │   ├── route.ts          # GET all (filtered), POST new
│   │   │   └── [id]/route.ts     # GET, PUT, DELETE by ID
│   │   ├── services/
│   │   │   ├── route.ts          # GET all (filtered), POST new
│   │   │   └── [id]/route.ts     # GET, PUT, DELETE by ID
│   │   ├── settings/
│   │   │   └── route.ts          # GET, PUT (singleton)
│   │   └── stats/
│   │       └── route.ts          # GET dashboard stats
│   ├── customers/                 # Customer pages
│   │   ├── page.tsx              # List view
│   │   ├── [id]/page.tsx         # Detail view
│   │   └── new/page.tsx           # Create form
│   ├── cars/                      # Car pages
│   │   ├── page.tsx              # List view
│   │   ├── [id]/page.tsx         # Detail view
│   │   └── new/page.tsx           # Create form
│   ├── services/                  # Service pages
│   │   ├── page.tsx              # List view
│   │   └── new/page.tsx           # Create form
│   ├── settings/
│   │   └── page.tsx               # Settings form
│   ├── page.tsx                   # Dashboard
│   ├── layout.tsx                 # Root layout (Sidebar)
│   └── globals.css                # Global styles
├── components/
│   ├── ui/                        # shadcn/ui components
│   ├── layout/                    # Header, Sidebar
│   ├── customers/                 # CustomerForm, CustomerTable
│   ├── cars/                      # CarForm, CarTable
│   ├── services/                  # ServiceForm, ServiceTable
│   └── dashboard/                 # StatsCards
├── models/                         # Mongoose models
│   ├── Customer.ts
│   ├── Car.ts
│   ├── Service.ts
│   └── Settings.ts
├── lib/
│   ├── mongodb.ts                 # Database connection
│   └── utils.ts                   # Utility functions (cn)
├── types/
│   └── index.ts                   # TypeScript interfaces
└── docs/                          # Documentation
```

## API Surface Plan

### API Route Structure

All API routes follow this pattern:
1. Import `dbConnect()` from `lib/mongodb`
2. Call `await dbConnect()` at start of handler
3. Use Mongoose models for database operations
4. Return `NextResponse.json()` with appropriate status codes
5. Handle errors with try/catch and return error responses

### Customers API (`app/api/customers/`)

**GET /api/customers** (`route.ts`)
- Purpose: Fetch all customers
- Implementation:
  - Connect to DB
  - `Customer.find({}).sort({ createdAt: -1 })`
  - Return array of customers
- Response: `200 OK` with `Customer[]`

**POST /api/customers** (`route.ts`)
- Purpose: Create new customer
- Implementation:
  - Connect to DB
  - Parse request body
  - `Customer.create(body)`
  - Return created customer
- Request Body: `{ name, email, phone, address }`
- Response: `201 Created` with `Customer`

**GET /api/customers/[id]** (`[id]/route.ts`)
- Purpose: Fetch single customer
- Implementation:
  - Connect to DB
  - Extract `id` from route params
  - `Customer.findById(id)`
  - Return customer or 404
- Response: `200 OK` with `Customer` or `404 Not Found`

**PUT /api/customers/[id]** (`[id]/route.ts`)
- Purpose: Update customer
- Implementation:
  - Connect to DB
  - Parse request body
  - `Customer.findByIdAndUpdate(id, body, { new: true, runValidators: true })`
  - Return updated customer
- Request Body: `{ name?, email?, phone?, address? }`
- Response: `200 OK` with `Customer` or `404 Not Found`

**DELETE /api/customers/[id]** (`[id]/route.ts`)
- Purpose: Delete customer
- Implementation:
  - Connect to DB
  - `Customer.findByIdAndDelete(id)`
  - Return success message
- Response: `200 OK` with `{ message: "Customer deleted" }` or `404 Not Found`

### Cars API (`app/api/cars/`)

**GET /api/cars** (`route.ts`)
- Purpose: Fetch all cars, optionally filtered by customer
- Implementation:
  - Connect to DB
  - Check for `customerId` query parameter
  - `Car.find(customerId ? { customerId } : {})`
  - Return array of cars
- Query Params: `?customerId=<id>` (optional)
- Response: `200 OK` with `Car[]`

**POST /api/cars** (`route.ts`)
- Purpose: Create new car
- Implementation:
  - Connect to DB
  - Parse request body
  - Validate `customerId` exists
  - `Car.create(body)`
  - Return created car
- Request Body: `{ customerId, make, model, year, licensePlate, vin, color }`
- Response: `201 Created` with `Car`

**GET /api/cars/[id]** (`[id]/route.ts`)
- Purpose: Fetch single car with populated customer
- Implementation:
  - Connect to DB
  - `Car.findById(id).populate('customerId')`
  - Return car or 404
- Response: `200 OK` with `Car` (with populated customer) or `404 Not Found`

**PUT /api/cars/[id]** (`[id]/route.ts`)
- Purpose: Update car
- Implementation:
  - Connect to DB
  - Parse request body
  - `Car.findByIdAndUpdate(id, body, { new: true, runValidators: true })`
  - Return updated car
- Request Body: `{ customerId?, make?, model?, year?, licensePlate?, vin?, color? }`
- Response: `200 OK` with `Car` or `404 Not Found`

**DELETE /api/cars/[id]** (`[id]/route.ts`)
- Purpose: Delete car
- Implementation:
  - Connect to DB
  - `Car.findByIdAndDelete(id)`
  - Return success message
- Response: `200 OK` with `{ message: "Car deleted" }` or `404 Not Found`

### Services API (`app/api/services/`)

**GET /api/services** (`route.ts`)
- Purpose: Fetch all services, optionally filtered by car
- Implementation:
  - Connect to DB
  - Check for `carId` query parameter
  - `Service.find(carId ? { carId } : {})`
  - Return array of services
- Query Params: `?carId=<id>` (optional)
- Response: `200 OK` with `Service[]`

**POST /api/services** (`route.ts`)
- Purpose: Create new service
- Implementation:
  - Connect to DB
  - Parse request body
  - Validate `carId` exists
  - `Service.create(body)`
  - Return created service
- Request Body: `{ carId, name, description?, price, date? }`
- Response: `201 Created` with `Service`

**GET /api/services/[id]** (`[id]/route.ts`)
- Purpose: Fetch single service with populated car
- Implementation:
  - Connect to DB
  - `Service.findById(id).populate('carId')`
  - Return service or 404
- Response: `200 OK` with `Service` (with populated car) or `404 Not Found`

**PUT /api/services/[id]** (`[id]/route.ts`)
- Purpose: Update service
- Implementation:
  - Connect to DB
  - Parse request body
  - `Service.findByIdAndUpdate(id, body, { new: true, runValidators: true })`
  - Return updated service
- Request Body: `{ carId?, name?, description?, price?, date? }`
- Response: `200 OK` with `Service` or `404 Not Found`

**DELETE /api/services/[id]** (`[id]/route.ts`)
- Purpose: Delete service
- Implementation:
  - Connect to DB
  - `Service.findByIdAndDelete(id)`
  - Return success message
- Response: `200 OK` with `{ message: "Service deleted" }` or `404 Not Found`

### Settings API (`app/api/settings/`)

**GET /api/settings** (`route.ts`)
- Purpose: Fetch settings (singleton pattern)
- Implementation:
  - Connect to DB
  - `Settings.findOne()` or create default if none exists
  - Return settings object
- Response: `200 OK` with `Settings`

**PUT /api/settings** (`route.ts`)
- Purpose: Update settings
- Implementation:
  - Connect to DB
  - `Settings.findOneAndUpdate({}, body, { upsert: true, new: true, runValidators: true })`
  - Return updated settings
- Request Body: `{ garageName?, address?, phone?, email? }`
- Response: `200 OK` with `Settings`

### Stats API (`app/api/stats/`)

**GET /api/stats** (`route.ts`)
- Purpose: Fetch dashboard statistics
- Implementation:
  - Connect to DB
  - Run parallel queries:
    - `Customer.countDocuments()`
    - `Car.countDocuments()`
    - `Service.countDocuments()`
    - `Service.aggregate([{ $group: { _id: null, total: { $sum: '$price' } } }])`
    - `Service.find().populate({ path: 'carId', populate: { path: 'customerId' } }).sort({ date: -1 }).limit(5)`
  - Transform and return combined stats
- Response: `200 OK` with `DashboardStats`

## UI Plan

### Page Components

All pages follow this pattern:
1. Client component (`'use client'`)
2. State management with `useState` for data, loading, error
3. Data fetching in `useEffect` on mount
4. Render loading/error states
5. Use shared components (Header, Tables, Forms)

### Dashboard (`app/page.tsx`)

- **Purpose**: Business overview with stats and recent services
- **Components Used**:
  - `Header` (title, description)
  - `StatsCards` (4 metric cards)
  - `Card`, `Table` (recent services)
- **Data Fetching**: GET `/api/stats` on mount
- **Features**: Links to services, cars, customers from table

### Customer Pages

**List** (`app/customers/page.tsx`)
- **Components**: `Header`, `CustomerTable`
- **Data**: GET `/api/customers`
- **Actions**: View, Edit, Delete (via table)

**Detail** (`app/customers/[id]/page.tsx`)
- **Components**: `Header`, `CustomerForm` (edit mode)
- **Data**: GET `/api/customers/[id]`
- **Actions**: Edit, Delete, View associated cars

**New** (`app/customers/new/page.tsx`)
- **Components**: `Header`, `CustomerForm` (create mode)
- **Actions**: Submit creates via POST `/api/customers`

### Car Pages

**List** (`app/cars/page.tsx`)
- **Components**: `Header`, `CarTable`
- **Data**: GET `/api/cars` (with optional `?customerId` filter)
- **Filtering**: Use `useSearchParams` with Suspense wrapper

**Detail** (`app/cars/[id]/page.tsx`)
- **Components**: `Header`, `CarForm` (edit mode), service list
- **Data**: GET `/api/cars/[id]`, GET `/api/services?carId=<id>`
- **Actions**: Edit, Delete, View services

**New** (`app/cars/new/page.tsx`)
- **Components**: `Header`, `CarForm` (create mode)
- **Data**: GET `/api/customers` (for dropdown)
- **Actions**: Submit creates via POST `/api/cars`

### Service Pages

**List** (`app/services/page.tsx`)
- **Components**: `Header`, `ServiceTable`
- **Data**: GET `/api/services` (with optional `?carId` filter)
- **Filtering**: Use `useSearchParams` with Suspense wrapper

**New** (`app/services/new/page.tsx`)
- **Components**: `Header`, `ServiceForm` (create mode)
- **Data**: GET `/api/cars` (for dropdown)
- **Actions**: Submit creates via POST `/api/services`

### Settings Page (`app/settings/page.tsx`)

- **Components**: `Header`, `SettingsForm`
- **Data**: GET `/api/settings` on mount
- **Actions**: Save updates via PUT `/api/settings`

### Shared Components

**Layout Components** (`components/layout/`)
- `Sidebar`: Navigation menu with active state
- `Header`: Page title, description, optional action button

**Form Components** (`components/*/`)
- `CustomerForm`: Create/edit customer form
- `CarForm`: Create/edit car form (with customer dropdown)
- `ServiceForm`: Create/edit service form (with car dropdown)
- All forms: Validation, error handling, loading states

**Table Components** (`components/*/`)
- `CustomerTable`: Display customers with actions
- `CarTable`: Display cars with actions
- `ServiceTable`: Display services with actions
- All tables: Delete confirmation dialogs

**UI Components** (`components/ui/`)
- shadcn/ui components: Button, Card, Table, Dialog, Input, Select, etc.
- Consistent styling with Tailwind CSS

## Data Validation and Error Handling

### Mongoose Schema Validation

All models use Mongoose schema validation:
- **Required fields**: Enforced at schema level
- **Data types**: Automatic type coercion and validation
- **Custom validators**: Email format (via lowercase), price min: 0
- **Timestamps**: Automatic `createdAt` and `updatedAt`

### API Error Handling

**Standard Error Response Pattern**:
```typescript
try {
  // ... operation
} catch (error) {
  console.error('Error message:', error);
  return NextResponse.json(
    { error: 'User-friendly error message' },
    { status: 500 }
  );
}
```

**Error Scenarios**:
- **400 Bad Request**: Invalid request body (handled by Mongoose validation)
- **404 Not Found**: Resource not found (explicit checks)
- **500 Internal Server Error**: Database errors, unexpected errors

### Client-Side Error Handling

- **Loading States**: Show loading indicator during fetch
- **Error States**: Display error message to user
- **Form Validation**: Client-side validation before submission
- **Optimistic Updates**: Not implemented in MVP (future enhancement)

## Testing Plan

### Smoke Tests

**Manual Testing Checklist**:

1. **Customer CRUD**
   - [ ] Create customer with valid data
   - [ ] View customer list
   - [ ] View customer detail
   - [ ] Edit customer
   - [ ] Delete customer
   - [ ] Validation: Required fields, email format

2. **Car CRUD**
   - [ ] Create car with valid customer
   - [ ] View car list (all and filtered by customer)
   - [ ] View car detail with customer info
   - [ ] Edit car
   - [ ] Delete car
   - [ ] Validation: Required fields, customerId exists

3. **Service CRUD**
   - [ ] Create service with valid car
   - [ ] View service list (all and filtered by car)
   - [ ] View service detail with car info
   - [ ] Edit service
   - [ ] Delete service
   - [ ] Validation: Required fields, price >= 0, carId exists

4. **Settings**
   - [ ] View settings (creates default if none)
   - [ ] Update settings
   - [ ] Persist across page reloads

5. **Dashboard**
   - [ ] Stats cards display correct counts
   - [ ] Revenue calculation correct
   - [ ] Recent services table shows last 5
   - [ ] Links navigate correctly

6. **Navigation**
   - [ ] Sidebar navigation works
   - [ ] Active state highlights current page
   - [ ] All routes accessible

### API Validation

**Test Each Endpoint**:

- **GET endpoints**: Verify correct data structure, status codes
- **POST endpoints**: Verify creation, validation errors, status 201
- **PUT endpoints**: Verify updates, 404 for missing resources
- **DELETE endpoints**: Verify deletion, 404 for missing resources
- **Query parameters**: Test filtering (customerId, carId)
- **Error handling**: Test invalid IDs, missing fields, database errors

### Edge Cases

- Empty database (no customers, cars, services)
- Large datasets (performance with many records)
- Invalid ObjectIds in URLs
- Concurrent updates (race conditions)
- Network failures (offline behavior)

## Deployment Plan

### Environment Setup

**Required Environment Variables**:
```bash
MONGODB_URI=mongodb://localhost:27017/garage
# or for production:
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/garage
```

**Development Setup**:
1. Install dependencies: `npm install`
2. Set up `.env.local` with `MONGODB_URI`
3. Start MongoDB locally or use MongoDB Atlas
4. Run dev server: `npm run dev`
5. Access at `http://localhost:3000`

### Production Build

**Build Process**:
```bash
npm run build    # Creates optimized production build
npm run start    # Starts production server
```

**Build Outputs**:
- Static pages (where applicable)
- Server components
- API routes (serverless functions)
- Optimized assets

### Deployment Options

1. **Vercel** (Recommended for Next.js)
   - Connect GitHub repository
   - Set `MONGODB_URI` in environment variables
   - Automatic deployments on push

2. **Self-Hosted**
   - Build: `npm run build`
   - Run: `npm run start` (requires Node.js server)
   - Set up reverse proxy (nginx)
   - Configure MongoDB connection

3. **Docker**
   - Create Dockerfile
   - Build and run container
   - Use Docker Compose for MongoDB

### Database Considerations

- **Development**: Local MongoDB or MongoDB Atlas free tier
- **Production**: MongoDB Atlas (recommended) or self-hosted MongoDB
- **Connection String**: Store securely in environment variables
- **Backup Strategy**: Regular backups (not in MVP scope)

## TODO List

### Phase 1: Core Functionality ✅ (Already Implemented)

- [x] Set up Next.js project with App Router
- [x] Configure MongoDB connection with caching
- [x] Create Mongoose models (Customer, Car, Service, Settings)
- [x] Implement Customer CRUD API routes
- [x] Implement Car CRUD API routes
- [x] Implement Service CRUD API routes
- [x] Implement Settings API (GET, PUT)
- [x] Implement Stats API
- [x] Create Dashboard page
- [x] Create Customer pages (list, detail, new)
- [x] Create Car pages (list, detail, new)
- [x] Create Service pages (list, new)
- [x] Create Settings page
- [x] Set up shadcn/ui components
- [x] Create layout components (Sidebar, Header)
- [x] Create form components (CustomerForm, CarForm, ServiceForm)
- [x] Create table components (CustomerTable, CarTable, ServiceTable)
- [x] Implement navigation and routing

### Phase 2: Testing & Validation

- [ ] Write smoke tests for all CRUD operations
- [ ] Test API endpoints with various inputs
- [ ] Test error handling (invalid IDs, missing fields)
- [ ] Test filtering (customerId, carId query params)
- [ ] Test edge cases (empty database, large datasets)
- [ ] Verify form validation on client side
- [ ] Test navigation and routing
- [ ] Verify responsive design (mobile, tablet, desktop)

### Phase 3: Polish & Documentation

- [ ] Add loading skeletons (better UX than "Loading...")
- [ ] Improve error messages (more specific)
- [ ] Add confirmation dialogs for destructive actions
- [ ] Verify all links work correctly
- [ ] Check accessibility (keyboard navigation, screen readers)
- [ ] Optimize bundle size (check for unused dependencies)
- [ ] Add code comments for complex logic
- [ ] Update README with setup instructions

### Phase 4: Deployment Preparation

- [ ] Set up production MongoDB (Atlas or self-hosted)
- [ ] Configure environment variables for production
- [ ] Test production build locally (`npm run build && npm run start`)
- [ ] Set up deployment pipeline (Vercel/GitHub Actions)
- [ ] Configure domain and SSL (if applicable)
- [ ] Set up monitoring/error tracking (optional)
- [ ] Create deployment documentation

### Phase 5: Future Enhancements (Post-MVP)

- [ ] Add pagination for large lists
- [ ] Implement search functionality
- [ ] Add data export (CSV/PDF)
- [ ] Add authentication/authorization
- [ ] Implement soft deletes
- [ ] Add audit logs
- [ ] Add image uploads for cars
- [ ] Implement invoice generation
- [ ] Add email notifications
- [ ] Create mobile app (React Native)

## Implementation Notes

### Key Patterns

1. **Connection Caching**: `lib/mongodb.ts` uses global cache to prevent multiple connections
2. **Model Pattern**: Models check `mongoose.models` to prevent re-compilation in development
3. **Type Safety**: TypeScript interfaces in `types/index.ts` match Mongoose document interfaces
4. **Component Reusability**: Forms and tables are reusable across create/edit views
5. **Error Boundaries**: Each page handles its own loading/error states

### Performance Considerations

- **Database Queries**: Use `populate()` sparingly, only when needed
- **Parallel Queries**: Stats API uses `Promise.all()` for concurrent queries
- **Connection Reuse**: MongoDB connection cached globally
- **Client-Side Fetching**: Pages fetch data on mount, no server-side data fetching in MVP

### Security Considerations

- **Input Validation**: Mongoose schema validation on all inputs
- **No Authentication**: MVP assumes single-user, local deployment
- **Environment Variables**: Sensitive data (MONGODB_URI) in `.env.local`
- **SQL Injection**: Not applicable (MongoDB uses parameterized queries via Mongoose)

### Known Limitations

- No pagination (assumes small datasets)
- No search functionality
- Hard deletes only (no soft deletes)
- No audit trail
- Single user only (no multi-tenancy)
- No real-time updates
- No offline support
