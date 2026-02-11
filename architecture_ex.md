# Architecture Documentation

## System Architecture Overview

The Garage Management System is built as a **full-stack monolith** using Next.js App Router, combining frontend and backend functionality in a single application. The architecture follows a **client-server pattern** with API routes handling server-side logic and React components managing the user interface.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Browser                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Next.js Application (React 19)                │   │
│  │  ┌────────────────────────────────────────────────┐  │   │
│  │  │  App Router Pages (Client Components)           │  │   │
│  │  │  - Dashboard, Customers, Cars, Services        │  │   │
│  │  │  - Forms, Tables, Navigation                   │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  │                        ↕ HTTP/REST                   │   │
│  │  ┌────────────────────────────────────────────────┐  │   │
│  │  │  API Routes (Server Components)                │  │   │
│  │  │  - /api/customers, /api/cars, /api/services    │  │   │
│  │  │  - /api/settings, /api/stats                   │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│              Data Layer                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Mongoose ODM (Models & Validation)                 │   │   │
│  │  - Customer, Car, Service, Settings                │   │   │
│  └──────────────────────────────────────────────────────┘   │
│                        ↕                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  MongoDB Database                                    │   │   │
│  │  - Document Storage                                  │   │   │
│  │  - Indexes & Queries                                 │   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow Architecture

```
User Action
    │
    ▼
┌─────────────────┐
│  React Component│ (Client-side)
│  (useEffect)    │
└────────┬────────┘
         │ HTTP Request
         ▼
┌─────────────────┐
│  Next.js API    │ (Server-side)
│  Route Handler  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  dbConnect()    │ (Connection Cache)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Mongoose Model │ (ODM Layer)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  MongoDB        │ (Database)
└─────────────────┘
         │
         ▼
    JSON Response
         │
         ▼
┌─────────────────┐
│  React Component│ (State Update)
│  (setState)     │
└─────────────────┘
```

## Technology Stack

### Frontend Technologies

#### Next.js 16.1.2
- **Framework**: React-based full-stack framework
- **Router**: App Router (file-based routing)
- **Rendering**: Client-side rendering for interactive pages
- **Features Used**:
  - File-based routing (`app/` directory)
  - API Routes (`app/api/`)
  - Server and Client Components
  - Built-in optimization (code splitting, image optimization)

#### React 19.2.3
- **Library**: UI component library
- **Pattern**: Component-based architecture
- **Hooks Used**:
  - `useState` - Component state management
  - `useEffect` - Side effects (data fetching)
  - `useCallback` - Memoized callbacks
  - `usePathname` - Navigation state (Next.js)

#### TypeScript 5
- **Language**: Typed superset of JavaScript
- **Configuration**: Strict mode enabled
- **Features**:
  - Type safety for components, API routes, and models
  - Interface definitions in `types/index.ts`
  - Path aliases (`@/*` for root directory)

### UI Framework

#### shadcn/ui
- **Library**: Component library built on Radix UI
- **Components Used**:
  - `Button`, `Card`, `Table`, `Dialog`
  - `Input`, `Select`, `Textarea`, `Label`
  - `AlertDialog`, `DropdownMenu`, `Badge`
  - `Separator`
- **Styling**: Tailwind CSS utility classes
- **Accessibility**: Built on Radix UI primitives (ARIA compliant)

#### Tailwind CSS 4
- **Framework**: Utility-first CSS framework
- **Configuration**: PostCSS with Tailwind plugin
- **Features**:
  - Custom color scheme (sidebar, primary, etc.)
  - Responsive design utilities
  - Dark mode support (via theme variables)

### Backend Technologies

#### Next.js API Routes
- **Pattern**: RESTful API endpoints
- **Location**: `app/api/*/route.ts`
- **HTTP Methods**: GET, POST, PUT, DELETE
- **Response Format**: JSON with `NextResponse`
- **Error Handling**: Try-catch with status codes

#### MongoDB 9.1.4
- **Database**: NoSQL document database
- **ODM**: Mongoose for schema definition and validation
- **Connection**: Cached connection via `lib/mongodb.ts`
- **Features**:
  - Document-based storage
  - Schema validation
  - Population (references)
  - Aggregation pipelines

#### Mongoose ODM
- **Library**: MongoDB object modeling
- **Models**: Defined in `models/` directory
- **Features**:
  - Schema definition with validation
  - TypeScript interfaces
  - Timestamps (createdAt, updatedAt)
  - Population for relationships

### Development Tools

#### ESLint
- **Linter**: Code quality and style checking
- **Config**: `eslint-config-next` (Next.js recommended)

#### PostCSS
- **Processor**: CSS transformation
- **Plugins**: Tailwind CSS

## Frontend Architecture

### Component Architecture

```
components/
├── ui/                    # Base UI components (shadcn/ui)
│   ├── button.tsx
│   ├── card.tsx
│   ├── table.tsx
│   └── ...
├── layout/                 # Layout components
│   ├── sidebar.tsx         # Navigation sidebar
│   └── header.tsx          # Page header
├── customers/              # Domain components
│   ├── customer-form.tsx
│   └── customer-table.tsx
├── cars/
│   ├── car-form.tsx
│   └── car-table.tsx
└── services/
    ├── service-form.tsx
    └── service-table.tsx
```

### Component Hierarchy

```
RootLayout
├── Sidebar (Navigation)
└── Main Content Area
    └── Page Component
        ├── Header
        └── Content
            ├── Form Components (CustomerForm, CarForm, ServiceForm)
            ├── Table Components (CustomerTable, CarTable, ServiceTable)
            └── UI Components (Card, Button, Dialog, etc.)
```

### Page Structure

All pages follow this pattern:

```typescript
'use client';

export default function PageComponent() {
  // 1. State management
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // 2. Data fetching
  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetch('/api/endpoint');
        const data = await response.json();
        setData(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, []);

  // 3. Render states
  if (loading) return <LoadingState />;
  if (error) return <ErrorState />;
  return <ContentState />;
}
```

### State Management

- **Local State**: `useState` for component-level state
- **No Global State**: No Redux/Zustand (not needed for MVP)
- **Server State**: Fetched on mount via `useEffect`
- **Form State**: Managed by form components

### Routing

- **File-based Routing**: Next.js App Router
- **Dynamic Routes**: `[id]` folders for detail pages
- **Navigation**: `next/link` for client-side navigation
- **Active States**: `usePathname` hook for active link highlighting

## Backend Architecture

### API Route Structure

```
app/api/
├── customers/
│   ├── route.ts           # GET all, POST new
│   └── [id]/route.ts      # GET, PUT, DELETE by ID
├── cars/
│   ├── route.ts           # GET all (filtered), POST new
│   └── [id]/route.ts      # GET, PUT, DELETE by ID
├── services/
│   ├── route.ts           # GET all (filtered), POST new
│   └── [id]/route.ts      # GET, PUT, DELETE by ID
├── settings/
│   └── route.ts           # GET, PUT (singleton)
└── stats/
    └── route.ts           # GET dashboard statistics
```

### API Route Pattern

All API routes follow this standard pattern:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import dbConnect from '@/lib/mongodb';
import Model from '@/models/Model';

export async function GET() {
  try {
    await dbConnect();
    const items = await Model.find({});
    return NextResponse.json(items);
  } catch (error) {
    console.error('Error:', error);
    return NextResponse.json(
      { error: 'Error message' },
      { status: 500 }
    );
  }
}
```

### Database Connection Pattern

**Connection Caching** (`lib/mongodb.ts`):

```typescript
// Global cache to prevent multiple connections
const cached: MongooseCache = global.mongoose || { conn: null, promise: null };

async function dbConnect(): Promise<typeof mongoose> {
  if (cached.conn) return cached.conn;  // Reuse existing connection
  
  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI);
  }
  
  cached.conn = await cached.promise;
  return cached.conn;
}
```

**Benefits**:
- Prevents connection pool exhaustion
- Reuses connections across requests
- Works in both development and production

## Database Architecture

### Schema Design

#### Customer Collection
```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required, lowercase),
  phone: String (required),
  address: String (required),
  createdAt: Date,
  updatedAt: Date
}
```

#### Car Collection
```javascript
{
  _id: ObjectId,
  customerId: ObjectId (ref: 'Customer'),
  make: String (required),
  model: String (required),
  year: Number (required),
  licensePlate: String (required, uppercase),
  vin: String (required, uppercase),
  color: String (required),
  createdAt: Date,
  updatedAt: Date
}
```

#### Service Collection
```javascript
{
  _id: ObjectId,
  carId: ObjectId (ref: 'Car'),
  name: String (required),
  description: String (default: ''),
  price: Number (required, min: 0),
  date: Date (required, default: now),
  createdAt: Date,
  updatedAt: Date
}
```

#### Settings Collection (Singleton)
```javascript
{
  _id: ObjectId,
  garageName: String (required, default: 'My Garage'),
  address: String (default: ''),
  phone: String (default: ''),
  email: String (default: '', lowercase),
  updatedAt: Date
}
```

### Relationships

```
Customer (1) ──< (many) Car (1) ──< (many) Service
```

- **Customer → Car**: One-to-Many (via `customerId` reference)
- **Car → Service**: One-to-Many (via `carId` reference)
- **Referential Integrity**: Handled in application layer (cascade deletes)

### Indexes

- **Primary Keys**: `_id` (automatic ObjectId index)
- **Foreign Keys**: `customerId`, `carId` (for efficient queries)
- **Query Optimization**: Indexes on frequently queried fields

### Data Validation

- **Schema-level**: Mongoose schema validation
- **Required Fields**: Enforced at database level
- **Data Types**: Automatic type coercion
- **Custom Validators**: Email format, price minimums
- **Timestamps**: Automatic `createdAt` and `updatedAt`

## Data Flow Architecture

### Create Flow (Example: New Customer)

```
1. User fills form (CustomerForm)
   │
   ▼
2. Form submission triggers POST request
   fetch('/api/customers', { method: 'POST', body: JSON.stringify(data) })
   │
   ▼
3. API Route receives request
   POST /api/customers
   │
   ▼
4. Database connection established
   await dbConnect()
   │
   ▼
5. Mongoose validation
   Customer.create(body)
   │
   ▼
6. MongoDB insert operation
   │
   ▼
7. Response returned
   NextResponse.json(customer, { status: 201 })
   │
   ▼
8. Client receives response
   │
   ▼
9. UI updates (redirect or refresh)
```

### Read Flow (Example: Dashboard Stats)

```
1. Dashboard page mounts
   │
   ▼
2. useEffect triggers
   fetch('/api/stats')
   │
   ▼
3. API Route processes request
   GET /api/stats
   │
   ▼
4. Parallel database queries
   Promise.all([
     Customer.countDocuments(),
     Car.countDocuments(),
     Service.countDocuments(),
     Service.aggregate([...]),
     Service.find().populate(...).limit(5)
   ])
   │
   ▼
5. Data aggregation and transformation
   │
   ▼
6. Combined response
   NextResponse.json({ totalCustomers, totalCars, ... })
   │
   ▼
7. Client state update
   setStats(data)
   │
   ▼
8. UI re-renders with stats
```

### Update Flow (Example: Edit Customer)

```
1. User clicks edit button
   │
   ▼
2. Navigate to /customers/[id]
   │
   ▼
3. Fetch customer data
   GET /api/customers/[id]
   │
   ▼
4. Form pre-populated with data
   │
   ▼
5. User modifies and submits
   │
   ▼
6. PUT request sent
   PUT /api/customers/[id]
   │
   ▼
7. Database update
   Customer.findByIdAndUpdate(id, body, { new: true })
   │
   ▼
8. Response with updated data
   │
   ▼
9. UI updates or redirects
```

### Delete Flow (Example: Delete Customer)

```
1. User clicks delete button
   │
   ▼
2. Confirmation dialog (if implemented)
   │
   ▼
3. DELETE request sent
   DELETE /api/customers/[id]
   │
   ▼
4. Cascade delete operations
   - Find all cars for customer
   - Delete all services for those cars
   - Delete all cars
   - Delete customer
   │
   ▼
5. Success response
   │
   ▼
6. UI updates (remove from list or redirect)
```

## Component Architecture Details

### Form Components

**Pattern**: Reusable forms for create/edit operations

```typescript
interface FormProps {
  initialData?: T;  // For edit mode
  onSubmit: (data: T) => Promise<void>;
  onCancel?: () => void;
}

export function EntityForm({ initialData, onSubmit, onCancel }: FormProps) {
  // Form state management
  // Validation
  // Submit handler
  // Render form fields
}
```

**Features**:
- Shared between create and edit pages
- Client-side validation
- Loading states during submission
- Error handling

### Table Components

**Pattern**: Display lists with actions

```typescript
interface TableProps {
  items: T[];
  onDelete: (id: string) => Promise<void>;
  onEdit?: (id: string) => void;
}

export function EntityTable({ items, onDelete, onEdit }: TableProps) {
  // Render table with shadcn/ui Table component
  // Action buttons (view, edit, delete)
  // Confirmation dialogs for delete
}
```

**Features**:
- Sortable columns (future enhancement)
- Action buttons
- Delete confirmation
- Responsive design

### Layout Components

**Sidebar** (`components/layout/sidebar.tsx`):
- Navigation menu
- Active route highlighting
- Icon-based navigation
- Responsive design

**Header** (`components/layout/header.tsx`):
- Page title and description
- Optional action button (e.g., "Add Customer")
- Consistent across all pages

## Security Architecture

### Input Validation

- **Client-side**: Form validation before submission
- **Server-side**: Mongoose schema validation
- **Type Safety**: TypeScript interfaces prevent type errors

### Data Sanitization

- **Mongoose**: Automatic sanitization via schema
- **Trim**: String fields automatically trimmed
- **Lowercase**: Email fields converted to lowercase
- **Uppercase**: License plate and VIN converted to uppercase

### Error Handling

- **API Errors**: Generic error messages (no sensitive data leaked)
- **Database Errors**: Caught and logged, user-friendly messages
- **Validation Errors**: Mongoose validation errors returned to client

### Environment Variables

- **MONGODB_URI**: Stored in `.env.local` (not committed)
- **No Secrets in Code**: All sensitive data in environment variables

### Current Limitations

- **No Authentication**: MVP assumes single-user, local deployment
- **No Authorization**: No role-based access control
- **No Rate Limiting**: API endpoints not rate-limited
- **No CSRF Protection**: Not implemented in MVP

## Performance Architecture

### Frontend Optimizations

- **Code Splitting**: Next.js automatic code splitting
- **Client Components**: Only interactive components are client-side
- **Lazy Loading**: Components loaded on demand
- **Image Optimization**: Next.js Image component (if used)

### Backend Optimizations

- **Connection Caching**: Reuse MongoDB connections
- **Parallel Queries**: `Promise.all()` for concurrent operations
- **Selective Population**: Only populate when needed
- **Query Optimization**: Efficient MongoDB queries

### Database Optimizations

- **Indexes**: Automatic indexes on `_id`, foreign keys
- **Aggregation**: Efficient aggregation pipelines for stats
- **Projection**: Select only needed fields (future enhancement)

### Caching Strategy

- **Connection Cache**: MongoDB connection cached globally
- **No Data Cache**: No Redis or in-memory cache (future enhancement)
- **Static Assets**: Next.js automatic static asset optimization

## Deployment Architecture

### Development Environment

```
Developer Machine
├── Node.js Runtime
├── Next.js Dev Server (npm run dev)
├── MongoDB (local or Atlas)
└── Browser (localhost:3000)
```

### Production Environment Options

#### Option 1: Vercel (Recommended)

```
Internet
    │
    ▼
┌─────────────┐
│   Vercel    │
│  (CDN Edge) │
└──────┬──────┘
       │
       ▼
┌─────────────┐      ┌──────────────┐
│ Next.js App │ ←──→ │ MongoDB Atlas│
│  (Server)   │      │  (Database)  │
└─────────────┘      └──────────────┘
```

**Benefits**:
- Zero-config deployment
- Automatic HTTPS
- Global CDN
- Serverless functions

#### Option 2: Self-Hosted

```
Internet
    │
    ▼
┌─────────────┐
│   Nginx     │ (Reverse Proxy)
│  (Port 80)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐      ┌──────────────┐
│ Next.js App │ ←──→ │   MongoDB    │
│  (Node.js)  │      │  (Database)  │
└─────────────┘      └──────────────┘
```

**Requirements**:
- Node.js server
- Process manager (PM2)
- Reverse proxy (Nginx)
- SSL certificate

### Build Process

```bash
npm run build    # Creates optimized production build
npm run start    # Starts production server
```

**Build Outputs**:
- Static pages (where applicable)
- Server components
- API routes (serverless functions)
- Optimized JavaScript bundles
- CSS files

## Technology Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Framework** | Next.js | 16.1.2 | Full-stack React framework |
| **UI Library** | React | 19.2.3 | Component library |
| **Language** | TypeScript | 5 | Type-safe JavaScript |
| **UI Components** | shadcn/ui | Latest | Component library |
| **Styling** | Tailwind CSS | 4 | Utility-first CSS |
| **Database** | MongoDB | Latest | NoSQL database |
| **ODM** | Mongoose | 9.1.4 | MongoDB object modeling |
| **Linter** | ESLint | 9 | Code quality |
| **CSS Processor** | PostCSS | Latest | CSS transformation |

## Architecture Decisions

### Why Next.js App Router?

- **File-based Routing**: Intuitive route organization
- **API Routes**: Built-in API endpoints (no separate backend)
- **Server Components**: Can use server components when needed
- **Optimization**: Automatic code splitting and optimization

### Why MongoDB?

- **Document Model**: Fits nested data structures (car with services)
- **Flexibility**: Easy schema evolution
- **Mongoose**: Strong validation and type safety
- **Scalability**: Horizontal scaling capability

### Why shadcn/ui?

- **Accessibility**: Built on Radix UI (ARIA compliant)
- **Customizable**: Copy components into project (not a dependency)
- **TypeScript**: Full TypeScript support
- **Tailwind**: Consistent with styling approach

### Why Client Components for Pages?

- **Interactivity**: Forms, tables, navigation require client-side
- **Data Fetching**: `useEffect` pattern for API calls
- **State Management**: Component-level state management
- **Simplicity**: No need for complex state management library

### Why Connection Caching?

- **Performance**: Reuse connections across requests
- **Resource Efficiency**: Prevent connection pool exhaustion
- **Development**: Works well with Next.js hot reloading
- **Production**: Efficient for serverless functions

## Future Architecture Considerations

### Potential Enhancements

1. **Server Components**: Use React Server Components for data fetching
2. **Caching Layer**: Add Redis for API response caching
3. **Authentication**: Add NextAuth.js for user authentication
4. **Real-time Updates**: Add WebSocket support for live updates
5. **Microservices**: Split into separate services if needed
6. **GraphQL**: Consider GraphQL API if complex queries needed
7. **CDN**: Use CDN for static assets
8. **Monitoring**: Add error tracking (Sentry) and analytics

### Scalability Considerations

- **Database**: MongoDB Atlas for managed scaling
- **API**: Next.js API routes scale automatically on Vercel
- **Caching**: Add caching layer for frequently accessed data
- **Pagination**: Implement pagination for large datasets
- **Search**: Add full-text search (MongoDB Atlas Search)
