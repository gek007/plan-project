# Garage Management System - App Overview

## Product Summary

The Garage Management System is a web application designed to help automotive service businesses manage their operations efficiently. The MVP provides core functionality for tracking customers, their vehicles, and service history in a single, easy-to-use interface.

### Target Users

- **Primary**: Small to medium-sized automotive service businesses (garages, auto shops, mechanics)
- **Use Case**: Daily operations management including customer relationship management, vehicle tracking, and service record keeping

### Core Value Proposition

- Centralized customer and vehicle database
- Service history tracking with revenue analytics
- Quick access to customer-vehicle-service relationships
- Real-time dashboard with business metrics

## Primary Workflows

### 1. Customer Management Workflow

1. **Add New Customer**
   - Navigate to Customers → New Customer
   - Fill in: Name, Email, Phone, Address
   - Save customer record

2. **View Customer Details**
   - Navigate to Customers → Select customer
   - View customer information
   - See all cars associated with customer
   - Access service history through car records

3. **Edit/Delete Customer**
   - From customer detail page, edit or delete
   - Note: Deleting customer may require handling associated cars

### 2. Vehicle Management Workflow

1. **Add New Car**
   - Navigate to Cars → New Car
   - Select customer from dropdown
   - Fill in: Make, Model, Year, License Plate, VIN, Color
   - Save car record

2. **View Car Details**
   - Navigate to Cars → Select car
   - View car information and owner details
   - See all services performed on this vehicle
   - Access service history with pricing

3. **Filter Cars by Customer**
   - From Cars list page, filter by customer ID
   - View all vehicles for a specific customer

### 3. Service Management Workflow

1. **Record New Service**
   - Navigate to Services → New Service
   - Select car from dropdown
   - Fill in: Service Name, Description, Price, Date
   - Save service record

2. **View Service History**
   - Navigate to Services list
   - View all services across all vehicles
   - Filter by car ID to see services for specific vehicle
   - Services are linked to cars, which link to customers

### 4. Dashboard & Analytics Workflow

1. **View Business Overview**
   - Navigate to Dashboard (home page)
   - View key metrics:
     - Total Customers
     - Total Cars
     - Total Services
     - Total Revenue
   - Review recent services table with customer/car context

### 5. Settings Management Workflow

1. **Configure Garage Information**
   - Navigate to Settings
   - Update: Garage Name, Address, Phone, Email
   - Settings are stored as singleton (one record)

## Key Screens and Navigation

### Navigation Structure

```
Dashboard (/)
├── Customers (/customers)
│   ├── List View (/customers)
│   ├── Detail View (/customers/[id])
│   └── New Customer (/customers/new)
├── Cars (/cars)
│   ├── List View (/cars)
│   ├── Detail View (/cars/[id])
│   └── New Car (/cars/new)
├── Services (/services)
│   ├── List View (/services)
│   └── New Service (/services/new)
└── Settings (/settings)
```

### Screen Descriptions

1. **Dashboard** (`app/page.tsx`)
   - Stats cards showing totals (customers, cars, services, revenue)
   - Recent services table with links to cars and customers
   - Quick overview of business health

2. **Customer List** (`app/customers/page.tsx`)
   - Table view of all customers
   - Actions: View, Edit, Delete
   - Link to add new customer

3. **Customer Detail** (`app/customers/[id]/page.tsx`)
   - Customer information display
   - Associated cars list
   - Edit/Delete actions

4. **Car List** (`app/cars/page.tsx`)
   - Table view of all cars
   - Filter by customer ID (query parameter)
   - Actions: View, Edit, Delete
   - Link to add new car

5. **Car Detail** (`app/cars/[id]/page.tsx`)
   - Car information display
   - Owner (customer) information
   - Service history for this car
   - Edit/Delete actions

6. **Service List** (`app/services/page.tsx`)
   - Table view of all services
   - Filter by car ID (query parameter)
   - Actions: View, Edit, Delete
   - Link to add new service

7. **Settings** (`app/settings/page.tsx`)
   - Garage information form
   - Singleton pattern (one settings record)

## Data Model and Relationships

### Entity Relationship Diagram

```
Customer (1) ──< (many) Car (1) ──< (many) Service
```

### Data Models

#### Customer (`models/Customer.ts`)
- `_id`: ObjectId (primary key)
- `name`: string (required)
- `email`: string (required, lowercase)
- `phone`: string (required)
- `address`: string (required)
- `createdAt`: Date (auto)
- `updatedAt`: Date (auto)

#### Car (`models/Car.ts`)
- `_id`: ObjectId (primary key)
- `customerId`: ObjectId (required, references Customer)
- `make`: string (required)
- `model`: string (required)
- `year`: number (required)
- `licensePlate`: string (required, uppercase)
- `vin`: string (required, uppercase)
- `color`: string (required)
- `createdAt`: Date (auto)
- `updatedAt`: Date (auto)

#### Service (`models/Service.ts`)
- `_id`: ObjectId (primary key)
- `carId`: ObjectId (required, references Car)
- `name`: string (required)
- `description`: string (optional, default: '')
- `price`: number (required, min: 0)
- `date`: Date (required, default: now)
- `createdAt`: Date (auto)
- `updatedAt`: Date (auto)

#### Settings (`models/Settings.ts`)
- `_id`: ObjectId (primary key)
- `garageName`: string (required, default: 'My Garage')
- `address`: string (optional, default: '')
- `phone`: string (optional, default: '')
- `email`: string (optional, default: '', lowercase)
- `updatedAt`: Date (auto)

### Relationships

- **Customer → Car**: One-to-Many (one customer has many cars)
- **Car → Service**: One-to-Many (one car has many services)
- **Service → Car → Customer**: Transitive relationship for service context

## Core API Endpoints

### Customers API (`app/api/customers/`)

- `GET /api/customers`
  - Returns: Array of all customers, sorted by creation date (newest first)
  - Response: `Customer[]`

- `POST /api/customers`
  - Body: `{ name, email, phone, address }`
  - Returns: Created customer object
  - Status: 201

- `GET /api/customers/[id]`
  - Returns: Single customer by ID
  - Response: `Customer`

- `PUT /api/customers/[id]`
  - Body: `{ name?, email?, phone?, address? }`
  - Returns: Updated customer object
  - Response: `Customer`

- `DELETE /api/customers/[id]`
  - Returns: Success message
  - Status: 200

### Cars API (`app/api/cars/`)

- `GET /api/cars?customerId=<id>` (optional filter)
  - Returns: Array of cars, optionally filtered by customerId
  - Response: `Car[]`

- `POST /api/cars`
  - Body: `{ customerId, make, model, year, licensePlate, vin, color }`
  - Returns: Created car object
  - Status: 201

- `GET /api/cars/[id]`
  - Returns: Single car by ID
  - Response: `Car`

- `PUT /api/cars/[id]`
  - Body: `{ customerId?, make?, model?, year?, licensePlate?, vin?, color? }`
  - Returns: Updated car object
  - Response: `Car`

- `DELETE /api/cars/[id]`
  - Returns: Success message
  - Status: 200

### Services API (`app/api/services/`)

- `GET /api/services?carId=<id>` (optional filter)
  - Returns: Array of services, optionally filtered by carId
  - Response: `Service[]`

- `POST /api/services`
  - Body: `{ carId, name, description?, price, date? }`
  - Returns: Created service object
  - Status: 201

- `GET /api/services/[id]`
  - Returns: Single service by ID
  - Response: `Service`

- `PUT /api/services/[id]`
  - Body: `{ carId?, name?, description?, price?, date? }`
  - Returns: Updated service object
  - Response: `Service`

- `DELETE /api/services/[id]`
  - Returns: Success message
  - Status: 200

### Settings API (`app/api/settings/`)

- `GET /api/settings`
  - Returns: Settings object (singleton, creates default if none exists)
  - Response: `Settings`

- `PUT /api/settings`
  - Body: `{ garageName?, address?, phone?, email? }`
  - Returns: Updated settings object
  - Response: `Settings`

### Stats API (`app/api/stats/`)

- `GET /api/stats`
  - Returns: Dashboard statistics
  - Response: `DashboardStats`
  - Includes:
    - `totalCustomers`: number
    - `totalCars`: number
    - `totalServices`: number
    - `totalRevenue`: number (sum of all service prices)
    - `recentServices`: RecentService[] (last 5 services with populated car and customer info)

## High-Level Data Flow

### Request Flow

```
User Action → Next.js Page Component → API Route → MongoDB → Response → UI Update
```

### Example: Creating a Service

1. User fills form on `/services/new`
2. Form submission triggers POST to `/api/services`
3. API route:
   - Connects to MongoDB via `dbConnect()`
   - Validates request body
   - Creates Service document with `carId` reference
   - Returns created service
4. Page component receives response
5. UI updates: redirect to service list or detail page

### Example: Dashboard Stats

1. User navigates to Dashboard (`/`)
2. Page component mounts, `useEffect` triggers
3. GET request to `/api/stats`
4. API route:
   - Connects to MongoDB
   - Runs parallel queries:
     - Count customers
     - Count cars
     - Count services
     - Aggregate service prices (sum)
     - Fetch recent 5 services with populated car/customer
   - Returns combined stats object
5. Page component updates state
6. UI renders stats cards and recent services table

### Database Connection Pattern

- Uses connection caching (`lib/mongodb.ts`)
- Global cache prevents multiple connections in development
- Each API route calls `await dbConnect()` before database operations
- Connection is reused across requests

## Non-Goals and MVP Constraints

### Out of Scope for MVP

- **Authentication/Authorization**: No user login or role-based access
- **Multi-tenancy**: Single garage instance only
- **Advanced Search**: Basic filtering only (by customerId, carId)
- **Reporting**: Basic dashboard stats only, no custom reports
- **Invoicing/Billing**: Service records only, no invoice generation
- **Notifications**: No email/SMS notifications
- **Mobile App**: Web-only, responsive design
- **Data Export**: No CSV/PDF export functionality
- **Audit Logs**: No change history tracking
- **Soft Deletes**: Hard deletes only

### MVP Constraints

- **Single User**: Designed for single-user operation
- **Local/Development Focus**: Optimized for local MongoDB
- **Basic Validation**: Mongoose schema validation only
- **No Pagination**: Lists show all records (assumes small datasets)
- **No Image Uploads**: Text-only data entry
- **No Real-time Updates**: Standard request/response pattern

## Technology Stack

- **Frontend**: Next.js 16 (App Router), React 19, TypeScript
- **UI Components**: shadcn/ui (Radix UI + Tailwind CSS 4)
- **Backend**: Next.js API Routes
- **Database**: MongoDB with Mongoose ODM
- **Styling**: Tailwind CSS 4
- **Fonts**: Geist Sans, Geist Mono

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
