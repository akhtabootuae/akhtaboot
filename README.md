# Garage Management System (GMS)

A comprehensive full-stack garage management system built with Next.js, Express.js, and MongoDB. This system manages all aspects of garage operations including customer registration, vehicle service tracking, work orders, invoicing, payroll, and financial management.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Project Structure](#project-structure)
- [Key Features](#key-features)
- [Database Schema](#database-schema)
- [Authentication & Authorization](#authentication--authorization)
- [API Integration](#api-integration)
- [Testing](#testing)
- [Development Workflow](#development-workflow)
- [Deployment Notes](#deployment-notes)

---

## 🏗️ Architecture Overview

The GMS is a monorepo-style project with two main components:

```
├── new-gms/          # Frontend (Next.js 15 with App Router)
└── new_api/          # Backend API (Express.js + MongoDB)
```

### Communication Flow
```
Frontend (Port 3001) → API Wrapper (services/api.ts) → Backend API (Port 3000) → MongoDB
                                                      ↓
                                                 WebSocket (Same Port 3000)
```

**Important:** Both repositories should be at the same directory level for proper integration.

---

## 🛠️ Tech Stack

### Frontend (new-gms)
- **Framework:** Next.js 15.3.2 (App Router with Turbopack)
- **UI:** React 19 with TypeScript
- **Styling:** Tailwind CSS 3.4
- **State Management:** Context API (AuthContext, NotificationContext, etc.)
- **HTTP Client:** Axios 1.9
- **Form Handling:** React Hook Form + Yup validation
- **Real-time:** Socket.IO Client 4.8
- **Charts:** Chart.js + react-chartjs-2
- **Notifications:** React Hot Toast

### Backend (new_api)
- **Runtime:** Node.js (v14+)
- **Framework:** Express.js 5.1
- **Database:** MongoDB (Mongoose 8.14)
- **Authentication:** JWT (jsonwebtoken 9.0)
- **Password Hashing:** bcryptjs
- **File Storage:** AWS S3 (via aws-sdk)
- **File Upload:** Multer + multer-s3
- **Real-time:** Socket.IO 4.8
- **PDF Generation:** PDFKit 0.17
- **Validation:** express-validator
- **Scheduled Tasks:** node-cron
- **Testing:** Jest 29.7 + Supertest 7.1 + MongoDB Memory Server

---

## 📦 Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** v14 or higher
- **npm** or **yarn** package manager
- **MongoDB** v4.4 or higher (running locally or remote connection)
- **AWS Account** (for S3 file storage)
- **Git** for version control

---

## 🚀 Installation & Setup

### ⚠️ Important: Setup Order
**Always start the backend API first, then the frontend.**

### Step 1: Backend API Setup (new_api)

1. **Navigate to the backend directory:**
   ```bash
   cd /path/to/new_api
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure environment variables:**

   Create a `.env` file in the root of `new_api/` directory:

   ```bash
   # Server Configuration
   PORT=3000
   NODE_ENV=development

   # Database Configuration
   MONGO_URI=mongodb://localhost:27017/your_database_name

   # JWT Configuration
   JWT_SECRET=your_secure_jwt_secret_here
   JWT_EXPIRE=24h

   # Default Admin User (for first-time setup)
   ADMIN_EMAIL=admin@yourdomain.com
   ADMIN_PASSWORD=secure_password_here
   ADMIN_FIRST_NAME=Admin
   ADMIN_LAST_NAME=User

   # CORS Settings
   ALLOW_ORIGINS=http://localhost:3001

   # AWS S3 Configuration (for file uploads)
   AWS_ACCESS_KEY_ID=your_aws_access_key
   AWS_SECRET_ACCESS_KEY=your_aws_secret_key
   AWS_REGION=your_aws_region
   S3_BUCKET_NAME=your_s3_bucket_name
   ```

   **Security Notes:**
   - Replace all placeholder values with your actual credentials
   - **Never commit the `.env` file to version control**
   - Use strong, unique values for `JWT_SECRET` and `ADMIN_PASSWORD`
   - Keep AWS credentials secure and use IAM roles with minimal permissions

4. **Initialize the database:**

   The application automatically initializes collections and seeds initial data on first run. Simply start the server:

   ```bash
   npm start
   ```

   For development with auto-reload:
   ```bash
   npm run dev
   ```

   The database seeder will automatically:
   - Create all necessary collections with validation schemas
   - Set up indexes
   - Create default roles (Admin, Supervisor, Technician)
   - Create the admin user specified in your `.env`
   - Seed initial permissions and branches

5. **Verify backend is running:**
   ```
   Server should be running on http://localhost:3000
   MongoDB should be connected
   ```

### Step 2: Frontend Setup (new-gms)

1. **Navigate to the frontend directory:**
   ```bash
   cd /path/to/new-gms
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure environment variables:**

   Create a `.env.local` file in the root of `new-gms/` directory:

   ```bash
   # API URL (Backend)
   NEXT_PUBLIC_API_URL=http://localhost:3000

   # App URL (Frontend)
   NEXT_PUBLIC_APP_URL=http://localhost:3001

   # JWT Settings (must match backend)
   JWT_SECRET=your_secure_jwt_secret_here
   JWT_EXPIRE=8h

   # Next Auth
   NEXTAUTH_URL=http://localhost:3001
   NEXTAUTH_SECRET=your_nextauth_secret_here
   ```

   **Notes:**
   - `NEXT_PUBLIC_API_URL` must point to your backend API
   - `JWT_SECRET` should match the backend's `JWT_SECRET`
   - All `NEXT_PUBLIC_*` variables are exposed to the browser

4. **Start the development server:**
   ```bash
   npm run dev
   ```

5. **Access the application:**
   ```
   Frontend: http://localhost:3001
   Backend API: http://localhost:3000
   ```

6. **Login with admin credentials:**
   Use the admin credentials you specified in the backend `.env` file.

---

## 📁 Project Structure

### Frontend (new-gms)

```
new-gms/
├── app/                          # Next.js App Router
│   ├── components/               # Reusable React components
│   │   ├── Dashboard/           # Admin & Supervisor dashboards
│   │   ├── Invoices/            # Invoice components
│   │   ├── Navbar/              # Navigation bar
│   │   ├── Sidebar/             # Sidebar navigation
│   │   └── ui/                  # UI primitives (buttons, cards, etc.)
│   ├── context/                 # React Context providers
│   │   ├── AuthContext.tsx      # Authentication state
│   │   ├── NotificationContext.tsx
│   │   ├── LanguageContext.tsx
│   │   └── CollapsedContext.tsx
│   ├── hooks/                   # Custom React hooks
│   ├── types/                   # TypeScript type definitions
│   ├── new_reg/                 # 🔥 Customer & vehicle registration
│   │   ├── page.tsx            # Main registration workflow
│   │   ├── StepOne.tsx         # Customer/vehicle entry
│   │   ├── StepTwo.tsx         # Service selection
│   │   └── components/         # Registration-specific components
│   ├── workOrders/              # 🔥 Work order management
│   │   ├── page.tsx            # Work orders list
│   │   ├── [id]/               # Individual work order details
│   │   └── verifications/      # QA verification workflow
│   ├── Invoices/                # Invoice management
│   ├── cases/                   # Case management
│   ├── expenses/                # Expense tracking
│   ├── payroll/                 # Payroll system (in development)
│   ├── staff/                   # Staff management
│   ├── Customers/               # Customer management
│   ├── finance/                 # Financial overview
│   ├── settings/                # System settings
│   ├── Messaging/               # Internal messaging
│   ├── notifications/           # Notification center
│   ├── login/                   # Authentication
│   ├── profile/                 # User profile
│   ├── globals.css              # Global styles
│   ├── layout.tsx               # Root layout
│   └── page.tsx                 # Home/Dashboard
├── services/                    # API integration layer
│   ├── api.ts                   # 🔑 API wrapper (ALL API calls go through here)
│   ├── auth.ts                  # Auth utilities
│   └── route-protection.ts      # Route guards
├── lib/                         # Utility functions
├── public/                      # Static assets
├── middleware.ts                # Next.js middleware (auth checks)
├── tailwind.config.ts           # Tailwind configuration
├── tsconfig.json                # TypeScript configuration
├── next.config.ts               # Next.js configuration
└── package.json                 # Frontend dependencies
```

### Backend (new_api)

```
new_api/
├── models/                      # MongoDB models & schemas
│   ├── index.js                # Database initialization
│   ├── User.js                 # User model
│   ├── Customer.js             # Customer model
│   ├── WorkOrder.js            # Work order model
│   ├── Invoice.js              # Invoice model
│   ├── Case.js                 # Case model
│   ├── Expense.js              # Expense model
│   ├── PayrollCalculation.js  # Payroll model
│   ├── TimeSheet.js            # Timesheet model
│   ├── Branch.js               # Branch model
│   ├── Role.js                 # Role model
│   ├── Notification.js         # Notification model
│   ├── Message.js              # Messaging model
│   ├── CarPart.js              # Car parts model
│   └── ... (30+ models total)
├── routes/                      # API route handlers
│   ├── userRoutes.js           # User & auth endpoints
│   ├── customerRoutes.js       # Customer CRUD
│   ├── workOrderRoutes.js      # Work order management
│   ├── invoiceRoutes.js        # Invoice operations
│   ├── caseRoutes.js           # Case management
│   ├── expenseRoutes.js        # Expense tracking
│   ├── payrollRoutes.js        # Payroll operations
│   ├── financeRoutes.js        # Financial reports
│   ├── messagingRoutes.js      # Internal messaging
│   ├── notificationRoutes.js   # Notifications
│   ├── branchRoutes.js         # Branch management
│   └── ... (20+ route files)
├── middleware/                  # Express middleware
│   ├── auth.js                 # JWT authentication
│   ├── adminAuth.js            # Admin-only routes
│   ├── permissionAuth.js       # Permission checking
│   ├── s3Upload.js             # File upload handling
│   └── payrollValidation.js    # Payroll validation
├── services/                    # Business logic services
│   ├── websocket.js            # Socket.IO real-time updates
│   ├── s3.js                   # AWS S3 client
│   ├── messageEncryption.js    # Message encryption
│   └── overtimeExpenseService.js
├── config/                      # Configuration files
│   ├── dbSeeder.js             # Database seeding
│   ├── permissions.js          # Permission definitions
│   └── seedVariations.js       # Service variations seed
├── utils/                       # Utility functions
├── app.js                       # 🔑 Main application entry point
├── package.json                 # Backend dependencies
└── .env                         # Environment variables (not in repo)
```

---

## ✨ Key Features

### 🔥 1. New Registration (new_reg)
**Priority: CRITICAL**

The new registration workflow is the primary entry point for new customers and vehicles.

**Features:**
- **Two-Step Process:**
  - **Step 1:** Customer information and vehicle details entry
  - **Step 2:** Service type selection and quotation generation
- Customer lookup and creation
- Vehicle registration with VIN and license plate
- Multi-language support (Arabic/English)
- Service type management with customizable parts
- Quotation generation and approval workflow
- PDF quotation export
- Integration with work orders and invoices

**Workflow:**
1. Staff enters customer information (name, contact, etc.)
2. Add vehicle details (make, model, year, VIN, plate number)
3. Select services needed (from predefined variations)
4. System generates quotation
5. Customer reviews and approves/rejects
6. If approved, creates work order automatically

**Key Files:**
- `app/new_reg/page.tsx` - Main registration page
- `app/new_reg/StepOne.tsx` - Customer/vehicle entry
- `app/new_reg/StepTwo.tsx` - Service selection

### 🔥 2. Work Orders
**Priority: CRITICAL**

Comprehensive work order management system for tracking all services.

**Features:**
- Work order creation from registrations or manually
- Real-time status tracking (Pending → In Progress → Completed)
- Technician assignment
- Service item tracking
- Parts and labor cost management
- QA verification workflow
- Error reporting and resolution
- Work order cancellation with approval
- PDF work order generation
- Search and filtering capabilities

**Workflow:**
1. Work order created from new registration or manually
2. Assigned to technician(s)
3. Technician updates progress
4. QA verification with photo uploads (3-5 required)
5. Completion and invoice generation

**Key Files:**
- `app/workOrders/page.tsx` - Work order list
- `app/workOrders/[id]/page.tsx` - Work order details
- `app/workOrders/verifications/page.tsx` - QA verification

**Backend:**
- `routes/workOrderRoutes.js`
- `routes/workOrderQARoutes.js`
- `models/WorkOrder.js`

### 3. Invoice Management

**Features:**
- Invoice generation from work orders
- Multiple payment methods support
- Partial payment tracking
- Payment records management
- VAT calculation (5%)
- Invoice image attachments (S3)
- PDF invoice generation with branch branding
- Payment status tracking (Pending, Partial, Paid)
- Invoice search and filtering

**Key Components:**
- `app/Invoices/page.tsx` - Invoice list
- `app/Invoices/[id]/page.tsx` - Invoice details
- Payment modals (partial/full payment)

### 4. Customer Management

**Features:**
- Customer CRUD operations
- Vehicle management per customer
- Service history tracking
- Contact information management
- Customer reviews and ratings
- Branch assignment

**Key Files:**
- `app/Customers/page.tsx`
- `app/Customers/[id]/page.tsx`
- `routes/customerRoutes.js`

### 5. Case Management

Internal case tracking system for issues, complaints, and follow-ups.

**Features:**
- Case creation and assignment
- Priority levels (Low, Medium, High, Critical)
- Status tracking
- Case comments and activity logs
- File attachments (via S3)
- Linked entities (work orders, invoices, customers)
- Case resolution tracking

**Key Files:**
- `app/cases/page.tsx`
- `app/cases/[id]/page.tsx`
- `routes/caseRoutes.js`

### 6. Payroll System
**Status: IN DEVELOPMENT**

Comprehensive payroll management system for staff compensation.

**Features:**
- Employee salary management
- Deduction types (GOSI, medical insurance, etc.)
- Variable deductions
- Overtime calculations
- Pay period management
- Pay stub generation
- Payroll audit logs
- Timesheet integration

**Documentation:**
- See `COMPLETE_PAYROLL_BACKEND_IMPLEMENTATION_GUIDE.md`
- See `PHASE2_VARIABLE_DEDUCTIONS_BACKEND_DOC.md`
- See `PHASE3_ATTENDANCE_PAY_CALCULATION_BACKEND_DOC.md`

**Key Files:**
- `app/payroll/` - Payroll frontend
- `routes/payroll/` - Payroll backend routes
- Multiple payroll models (PayrollCalculation, PayStub, TimeSheet, etc.)

### 7. Financial Management

**Features:**
- Financial overview dashboard
- Revenue tracking
- Expense management
- Budget management
- Financial reports generation
- Profit/loss calculations

**Key Files:**
- `app/finance/page.tsx`
- `routes/financeRoutes.js`

### 8. Expense Tracking

**Features:**
- Expense entry and categorization
- Receipt upload (S3)
- Expense approval workflow
- Expense reports
- Budget vs actual tracking
- Expense logs for audit trail

**Key Files:**
- `app/expenses/page.tsx`
- `routes/expenseRoutes.js`

### 9. Messaging & Notifications

**Features:**
- Internal messaging between staff
- Real-time notifications via WebSocket
- Message encryption
- Conversation management
- Voice note support (uploaded to S3)
- Image sharing
- Auto-expiring conversations

**Key Files:**
- `app/Messaging/` - Messaging UI
- `app/notifications/` - Notification center
- `services/websocket.js` - Real-time updates
- `routes/messagingRoutes.js`

### 10. User & Role Management

**Features:**
- User CRUD operations
- Role-based access control (Admin, Supervisor, Technician)
- Granular permission system
- Branch assignment
- User activation/deactivation
- Permission audit logs

**Key Files:**
- `app/staff/page.tsx`
- `routes/userRoutes.js`
- `middleware/permissionAuth.js`

### 11. Quality Assurance (QA)

**Features:**
- QA approval workflow for completed work
- Photo requirement (3-5 images)
- QA checklist
- Approval/rejection with comments
- Customer review integration

**Key Files:**
- `app/qa-review/page.tsx`
- `app/customer-review/page.tsx`

---

## 🗄️ Database Schema

**Database Name:** `GMS_DB`
**Total Collections:** 35

The system uses MongoDB with comprehensive schema validation and indexing. For complete details, see **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)**.

### Collection Overview

#### Core Collections (8)
- `users` - System users (staff) with roles and permissions
- `customers` - Customer information with embedded vehicles
- `work_orders` - Service work orders with nested parts and stages
- `invoices` - Invoices with payment tracking
- `branches` - Branch/location information
- `roles` - User roles (Admin, Supervisor, Technician)
- `variations` - Service type definitions
- `stages` - Workflow stage definitions

#### Payroll Collections (9) - In Development
- `payroll_calculations` - Payroll calculation records
- `pay_stubs` - Generated pay stubs
- `pay_periods` - Pay period definitions
- `timesheets` - Employee time tracking
- `payroll_deduction_types` - Deduction categories
- `payroll_employee_deductions` - Employee-specific deductions
- `payroll_deduction_history` - Deduction change history
- `payroll_audit_logs` - Payroll audit trail
- `payroll_rates` - Employee rate history

#### Financial Collections (5)
- `expenses` - Expense records
- `expense_logs` - Expense change history
- `budgets` - Budget definitions
- `alerts` - Financial alerts
- `reports` - Generated reports

#### Case Management (3)
- `cases` - Case tracking
- `case_logs` - Case activity logs
- `customer_reviews` - Customer feedback

#### Quality Assurance (3)
- `workordersqa` - QA verification records
- `error_reports` - Error/issue tracking
- `qc_approvals` - QC approval workflow

#### Messaging (3)
- `messages` - Internal messages
- `conversations` - Message threads (auto-expire after 90 days)
- `notifications` - System notifications (auto-expire after 30 days)

#### Other (4)
- `carparts` - Car parts catalog
- `car_jobs` / `car_job` - Car job records

### Key Features
- **Schema Validation:** All collections use MongoDB schema validation
- **Optimized Indexes:** Performance-tuned indexes on frequently queried fields
- **Relationships:** References using ObjectIds with proper indexing
- **Audit Trails:** Timestamps and change logs on most collections
- **TTL Indexes:** Auto-deletion of old notifications and conversations
- **Nested Documents:** Work orders use deeply nested structure for atomic operations

### Sample Document Structure

**Work Order Example:**
```javascript
{
  workOrderNumber: "WO-000001",
  customer_id: ObjectId,
  vehicle_id: 1,
  status: "in_progress",
  parts: [{
    partName: "Front Bumper",
    variationId: ObjectId,
    stages: [{
      stageId: ObjectId,
      status: "completed",
      assignedTo: ObjectId,
      actualHours: 4.5,
      logs: [{ action: "start", timestamp: Date }]
    }]
  }]
}
```

For complete schema documentation including all fields, indexes, and relationships, see **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)**.

---

## 🔐 Authentication & Authorization

### Authentication Flow

1. **Login:**
   - User submits email and password
   - Backend validates credentials (bcrypt)
   - JWT token generated and returned
   - Token stored in localStorage and cookie
   - User data (including permissions) cached

2. **Token Management:**
   - Tokens expire based on `JWT_EXPIRE` setting
   - Frontend automatically attaches token to all API requests
   - Token validation on backend for protected routes
   - Automatic redirect to login on token expiration (401)

3. **Session Handling:**
   - Token stored in `localStorage` (key: `authToken`)
   - User data stored in `localStorage` (key: `user`)
   - Cookie set for Next.js middleware authentication
   - Automatic token validation on app load

### Authorization Levels

#### 1. Role-Based Access Control (RBAC)

Three primary roles:

| Role | Description | Access Level |
|------|-------------|--------------|
| **Admin** | Full system access | All features and settings |
| **Supervisor** | Management and oversight | Most features, limited settings |
| **Technician** | Field worker | Work orders, customer info |

#### 2. Permission-Based Access Control

Granular permissions for specific actions:

**User Management:**
- `view_all_users` - View all system users
- `create_user` - Create new users
- `update_user` - Update user information
- `delete_user` - Delete users
- `manage_permissions` - Grant/revoke permissions
- `manage_roles` - Assign roles

**Work Orders:**
- `view_all_work_orders` - View all work orders
- `create_work_order` - Create work orders
- `update_work_order` - Modify work orders
- `delete_work_order` - Delete work orders
- `update_work_order_status` - Change status

**Invoices:**
- `view_all_invoices` - View all invoices
- `create_invoice` - Generate invoices
- `update_invoice` - Modify invoices
- `manage_payments` - Process payments

**Customers:**
- `view_all_customers` - Access all customer data
- `create_customer` - Add new customers
- `update_customer` - Update customer info
- `delete_customer` - Remove customers

**Cases:**
- `view_all_cases` - View all cases
- `create_case` - Create new cases
- `update_case` - Modify cases
- `resolve_case` - Mark cases as resolved

... and many more (see `config/permissions.js`)

#### 3. Authorization Middleware

**Backend:**
```javascript
// Basic authentication
const auth = require('./middleware/auth');

// Permission-based
const { requirePermissions } = require('./middleware/permissionAuth');

// Usage in routes
router.get('/users', auth, requirePermissions(['view_all_users']), getUsers);
```

**Frontend:**
```typescript
// AuthContext provides user and permissions
const { user, isAuthenticated } = useAuth();

// Check permissions in components
const hasPermission = user?.permissions?.some(
  p => p.permission_name === 'create_work_order' && p.granted
);

// Role guard component
<RoleGuard allowedRoles={['Admin', 'Supervisor']}>
  <AdminContent />
</RoleGuard>
```

### Protected Routes

**Frontend Middleware** (`middleware.ts`):
- Checks for authentication token in cookies
- Redirects to login if not authenticated
- Allows access to public routes (`/login`)

**Backend Middleware:**
- `auth.js` - Validates JWT token
- `adminAuth.js` - Restricts to admin role only
- `permissionAuth.js` - Checks specific permissions

---

## 🔌 API Integration

### API Wrapper (`services/api.ts`)

**All API calls MUST go through the centralized API wrapper.**

The wrapper provides:
- Automatic JWT token attachment
- Request/response interceptors
- Error handling
- Automatic logout on 401
- Response data extraction

**Example Usage:**

```typescript
import api, { workOrderApi, invoiceApi, caseApi } from '@/services/api';

// Using pre-defined API functions (preferred)
const workOrders = await workOrderApi.getWorkOrders({ status: 'pending' });
const invoice = await invoiceApi.getInvoice(invoiceId);

// Direct API calls
const response = await api.get('/api/users/me');
const created = await api.post('/api/customers', customerData);
```

### Available API Modules

The `api.ts` file exports organized API functions:

- `errorReportsApi` - Error report operations
- `caseApi` - Case management
- `workOrderApi` - Work order operations
- `invoiceApi` - Invoice management
- `customerApi` - Customer operations
- `expenseApi` - Expense tracking
- `payrollApi` - Payroll operations
- `userApi` - User management
- `notificationApi` - Notifications
- `messagingApi` - Messaging system
- `dataApi` - General data operations

### Request Flow

```
Component/Page
    ↓
services/api.ts (API function)
    ↓
Axios instance (with interceptors)
    ↓
Backend API endpoint
    ↓
Middleware (auth, permissions)
    ↓
Route handler
    ↓
Model/Database operation
    ↓
Response (intercepted and processed)
    ↓
Component receives data
```

### Error Handling

```typescript
try {
  const data = await workOrderApi.createWorkOrder(workOrderData);
  toast.success('Work order created!');
} catch (error: any) {
  if (error.response?.status === 403) {
    toast.error('You do not have permission to create work orders');
  } else if (error.response?.status === 401) {
    // Automatic redirect to login handled by interceptor
  } else {
    toast.error(error.response?.data?.message || 'Failed to create work order');
  }
}
```

### WebSocket Integration

Real-time updates use Socket.IO:

**Backend:**
- Runs on same port as Express (3000)
- Authentication via JWT token in connection handshake
- Room-based messaging (user-specific, branch-specific)

**Frontend:**
```typescript
import { io } from 'socket.io-client';

const socket = io(process.env.NEXT_PUBLIC_API_URL, {
  auth: { token: localStorage.getItem('authToken') }
});

// Listen for notifications
socket.on('notification', (data) => {
  // Update UI
});
```

---

## 🧪 Testing

### Backend Testing (Jest)

The backend includes comprehensive test suites using:
- **Jest** - Test framework
- **Supertest** - HTTP assertions
- **MongoDB Memory Server** - In-memory database for tests

**Running Tests:**

```bash
cd /path/to/new_api

# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run specific test suite
npm test -- userRoutes.test.js
```

**Test Structure:**
```
new_api/
├── tests/
│   ├── jest.setup.js         # Jest configuration
│   ├── test-setup.js         # Global test setup
│   ├── test-teardown.js      # Global test teardown
│   └── test-environment.js   # Test environment config
└── __tests__/
    ├── models/               # Model tests
    ├── routes/               # Route/endpoint tests
    └── middleware/           # Middleware tests
```

**What's Tested:**
- Model validation schemas
- CRUD operations
- Authentication flows
- Permission checks
- Error handling
- Edge cases

### Frontend Testing

Testing framework is set up but tests need to be written:

```bash
cd /path/to/new-gms

# Run tests (when available)
npm test
```

---

## 💻 Development Workflow

### Starting Development

**Always start in this order:**

1. **Start MongoDB** (if not already running):
   ```bash
   mongod
   # or
   brew services start mongodb-community
   ```

2. **Start Backend API:**
   ```bash
   cd /path/to/new_api
   npm run dev
   ```
   Wait for "Server running on port 3000" and "MongoDB connected"

3. **Start Frontend:**
   ```bash
   cd /path/to/new-gms
   npm run dev
   ```
   Access at http://localhost:3001

### Development Commands

**Backend:**
```bash
npm start          # Production mode
npm run dev        # Development with nodemon
npm test           # Run tests
npm run test:watch # Watch mode tests
```

**Frontend:**
```bash
npm run dev        # Development with Turbopack
npm run build      # Production build
npm start          # Production mode
npm run lint       # ESLint
```

### Common Development Tasks

#### Adding a New API Endpoint

1. **Create/update model** (if needed): `new_api/models/YourModel.js`
2. **Add route handler**: `new_api/routes/yourRoutes.js`
3. **Register route**: `new_api/app.js`
4. **Add middleware**: Authentication, permissions
5. **Update API wrapper**: `new-gms/services/api.ts`
6. **Use in components**: Import from `@/services/api`

#### Adding a New Page/Feature

1. **Create page**: `new-gms/app/feature-name/page.tsx`
2. **Add types**: `new-gms/app/types/feature.ts`
3. **Create components**: `new-gms/app/components/FeatureName/`
4. **Update navigation**: Add to sidebar/navbar
5. **Add permissions**: Check user permissions in components
6. **Test workflow**: End-to-end feature testing

#### Working with File Uploads (S3)

All file uploads go through AWS S3:

**Backend Setup:**
1. Ensure AWS credentials in `.env`
2. Use `s3Upload` middleware from `middleware/s3Upload.js`
3. Files automatically uploaded to S3 with organized folder structure

**Example:**
```javascript
const { createS3Upload } = require('../middleware/s3Upload');

router.post('/upload', auth,
  createS3Upload({
    folderName: 'invoices',
    allowedTypes: /jpeg|jpg|png|pdf/,
    maxSize: 10 * 1024 * 1024 // 10MB
  }),
  (req, res) => {
    // req.file.location contains S3 URL
    res.json({ url: req.file.location });
  }
);
```

**Frontend:**
```typescript
const formData = new FormData();
formData.append('file', fileBlob);

await api.post('/api/invoices/upload', formData, {
  headers: { 'Content-Type': 'multipart/form-data' }
});
```

### Debugging Tips

**Backend:**
- Check console logs for MongoDB connection
- Verify JWT_SECRET matches between frontend and backend
- Use `console.log` in route handlers
- Check MongoDB with Compass or mongosh
- Verify environment variables are loaded

**Frontend:**
- Check browser console for errors
- Use React DevTools for component inspection
- Check Network tab for API call details
- Verify localStorage has `authToken` and `user`
- Check for CORS errors

**Common Issues:**

1. **"No token provided" error:**
   - Check if token exists in localStorage
   - Verify API wrapper is attaching token
   - Check CORS settings

2. **403 Forbidden:**
   - User lacks required permissions
   - Check user permissions in database
   - Verify middleware configuration

3. **WebSocket connection fails:**
   - Check if backend is running
   - Verify token is valid
   - Check CORS for WebSocket

4. **S3 upload fails:**
   - Verify AWS credentials in `.env`
   - Check bucket permissions
   - Verify file type is allowed

---

## 🚀 Deployment Notes

### Environment Variables

**Never commit `.env` or `.env.local` files!**

For production deployment, ensure:
1. Strong `JWT_SECRET` (at least 32 characters)
2. Secure database connection (use MongoDB Atlas or secure server)
3. HTTPS enabled (required for production)
4. Proper CORS configuration
5. AWS credentials secured (use IAM roles where possible)
6. Environment-specific URLs

### Security Checklist

- [ ] Change all default passwords
- [ ] Use strong JWT secrets
- [ ] Enable HTTPS
- [ ] Configure proper CORS
- [ ] Secure AWS credentials
- [ ] Set up MongoDB authentication
- [ ] Implement rate limiting
- [ ] Enable database backups
- [ ] Set up monitoring and logging
- [ ] Review and audit permissions

### Build Commands

**Backend:**
```bash
npm start  # Production mode (no dev dependencies)
```

**Frontend:**
```bash
npm run build  # Creates optimized production build
npm start      # Serves production build
```

### Ports

- **Backend API:** Port 3000 (configurable via `PORT` env var)
- **Frontend:** Port 3001 (Next.js default dev port)
- **MongoDB:** Port 27017 (default)
- **WebSocket:** Same as Backend API (Port 3000)

### Database Backups

Regular backups recommended:
```bash
# Backup
mongodump --uri="mongodb://localhost:27017/your_database_name" --out=/backup/path

# Restore
mongorestore --uri="mongodb://localhost:27017/your_database_name" /backup/path/your_database_name
```

---

## 📚 Additional Documentation

The project includes extensive additional documentation:

**Core Documentation:**
- **`DATABASE_SCHEMA.md`** - Complete database schema with all 35 collections, fields, indexes, and relationships
- `README.md` - This file (main documentation)

**Backend API Documentation:**
- `USER_API.md` - User authentication and management
- `CUSTOMER_API.md` - Customer operations
- `INVOICE_API.md` - Invoice management
- `WORK_ORDER_API.md` - Work order operations
- `BRANCH_API.md` - Branch management
- `EXPENSE_MANAGEMENT_API.md` - Expense tracking
- `PAYROLL_SYSTEM_REFACTOR.md` - Payroll system
- `EMPLOYEE_ENDPOINTS.md` - Employee management
- `REPORT_GENERATION_API.md` - Report generation

**Frontend Documentation:**
- `EXPENSE_MANAGEMENT_FRONTEND_GUIDE.md` - Expense UI guide
- `MANNER_EVALUATIONS_FRONTEND.md` - Evaluation system

**Implementation Guides:**
- `COMPLETE_PAYROLL_BACKEND_IMPLEMENTATION_GUIDE.md`
- `PHASE2_VARIABLE_DEDUCTIONS_BACKEND_DOC.md`
- `PHASE3_ATTENDANCE_PAY_CALCULATION_BACKEND_DOC.md`

---

## 🤝 Contributing

For internal development team:

1. Create feature branches from `main`
2. Follow existing code structure and naming conventions
3. Write tests for new features
4. Update documentation
5. Test thoroughly before merging
6. Do not commit environment files or credentials

---

## 📞 Support

For questions or issues:
- Check existing documentation in the respective README files
- Review API documentation files
- Check the codebase for similar implementations
- Contact the development team

---

## 📝 License

Proprietary - All rights reserved

---

## 👥 Contributors

Development Team:
- Bashir Khadra Jr
- Fahad Milhem
- Begad Moussa

---

**Last Updated:** 2025

**System Version:** 1.0 (In Active Development)
