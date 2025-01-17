# Frontend Structure

## **App Directory Organization**
Next.js 13+ uses the App Router for file-system based routing. The application follows a domain-driven organization structure:

```plaintext
app/
├── (auth)/               # Authentication routes
│   ├── login/
│   └── signup/
├── company/             # Protected dashboard routes for companies
│   ├── setup/
│   └── dashboard/
│   │   ├── bookings/
│   │   ├── users/
│   │   └── members/
├── dashboard/          # Protected dashboard routes for clients
│   ├── bookings/
│   └── settings/
components/           # Shared components
├── ui/              # Generated UI components from shadcn
└── dashboard/       # Dashboard components
contexts/            # React Context providers
lib/                # Utilities and helpers
├── graphql/        # GraphQL queries and mutations
└── utils.ts        # Utility functions
types/              # TypeScript type definitions
```

## **Core Components**

### **1. Routes and Layouts**
The application uses Next.js 13's App Router conventions:
- Layout files (`layout.tsx`) provide shared UI for route segments
- Page files (`page.tsx`) define the unique content for each route
- Loading states (`loading.tsx`) and error handling (`error.tsx`) are co-located with their routes
- Route groups (in parentheses) are used to organize routes without affecting the URL structure

### **2. Component Architecture**
Components are divided into two main categories:
- **Server Components**: Default in the App Router, they handle data fetching and static content rendering
- **Client Components**: Marked with 'use client', they manage interactive features and client-side state

### **3. State Management**
Application state is handled through:
- React Context providers for global state
- Apollo Client for GraphQL data management
- Local component state for UI interactions
- Server state through Next.js Server Components

### **4. Type System**
Types are centralized in the `types` directory:
- Shared interfaces and types
- Discriminated unions for user roles
- Form state types
- GraphQL response types

### **5. Context System**
Contexts provide global state management:
- Company context for company-specific data
- Dashboard context for client dashboard data
- Authentication state management
- Loading state coordination

### **6. GraphQL Architecture**
The GraphQL implementation is organized in the `lib/graphql` directory with a clear separation of concerns:

- **Apollo Client Setup** (`apollo-wrapper.tsx`):
    - Configures the Apollo Client instance
    - Sets up authentication middleware
    - Handles error management
    - Implements SSR support

- **Data Operations**:
    - `mutations.ts`: Contains all GraphQL mutations for data modifications
    - `queries.ts`: Defines GraphQL queries for data fetching
    - `schema.graphql`: GraphQL schema definitions
    - `client.ts`: Client configuration and utilities

### **7. Utility Structure**
The `lib` directory contains various utility files:

- **Authentication** (`authActions.ts`):
    - Token management
    - User authentication flows
    - Session handling

- **Company Operations** (`companyActions.ts`):
    - Company-specific operations
    - Business logic

- **General Utilities** (`utils.ts`):
    - Helper functions
    - Shared functionality
    - Type guards
