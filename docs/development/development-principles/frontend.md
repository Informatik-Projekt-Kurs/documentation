# Frontend Development Principles

## Component Architecture
Next.js and React use a component-based architecture where UI elements are broken down into reusable, isolated pieces. This approach enhances maintainability, reusability, and testing capabilities.

### Server vs Client Components
Next.js 13+ introduces a differentiation between server and client components:

- **Server Components:** <br>
  These are the default in the App Router. They render on the server, reduce client-side JavaScript, and can directly access backend resources:
    ```ts
    // No 'use client' directive needed
    async function UserProfile({ userId }: { userId: string }) {
        const user = await fetchUser(userId) // Direct database access
        
        return <div>Hello, {user.name}</div>
    }
    ```

- **Client Components:** <br>
  Used for interactive features that require client-side state or browser APIs:
    ```ts
    'use client'
    
    import { useState, useEffect } from 'react'
    
    function InteractiveWidget() {
        const [isOpen, setIsOpen] = useState(false)
        // Client-side interactivity
    }
    ```

## State Management
The application employs several strategies for efficient state management:

### Context-Based State Management
React Contexts are extensively used to provide efficient data access and state management:

- **Centralized Data Management:** <br>
  Contexts like `CompanyContext` and `DashboardContext` handle complex data fetching and state management:
    ```ts
    // Dashboard context example showing centralized state management
    type DashboardContextProps = {
      user: ClientUser | undefined;
      loading: boolean;
      refreshUser: () => Promise<void>;
      companies: { getCompanies: Company[] } | undefined;
      refreshCompanies: () => Promise<void>;
      relevantAppointments: Appointment[];
      appointments: Appointment[];
      refreshAppointments: () => Promise<void>;
    };

    export function DashboardProvider({ children }: { children: React.ReactNode }) {
      const [user, setUser] = useState<ClientUser>();
      const [appointments, setAppointments] = useState<Appointment[]>([]);
      
      // GraphQL integration with polling
      const { loading: companiesLoading, data: companies, refetch } = useQuery(GET_COMPANIES, {
        pollInterval: 300000,
        onError: (error) => {
          console.error("GraphQL Error:", error);
        }
      });

      // Unified loading state
      const loading = companiesLoading || userLoading || appointmentsLoading;

      return (
        <DashboardContext.Provider value={{
          user,
          loading,
          refreshUser,
          companies,
          refreshCompanies: refetch,
          appointments,
          refreshAppointments
        }}>
          {children}
        </DashboardContext.Provider>
      );
    }
    ```

- **Unified Loading States:** <br>
  Each context maintains a unified loading state that aggregates multiple data fetching operations:
    ```ts
    const loading = userLoading || companyLoading || membersLoading || appointmentsLoading;
    ```

- **Automatic Data Refresh:** <br>
  Contexts implement polling and refresh mechanisms to keep data up-to-date:
    ```ts
    const { data, refetch } = useQuery(GET_COMPANIES, {
      pollInterval: 300000, // 5-minute polling interval
    });
    ```


### Server Actions
Next.js Server Actions are used extensively for secure server-side operations:

- **Type-Safe Form Handling:** <br>
  Server actions implement strong typing and validation:
    ```ts
    type SubscriptionState = {
      message: "success" | "error";
      error?: string;
      isSubscribed?: boolean;
    };

    export async function subscribeToCompany(
      prevState: SubscriptionState, 
      formData: FormData
    ): Promise<SubscriptionState> {
      const companyId = formData.get("companyId") as string;

      // Zod validation
      const schema = z.object({
        companyId: z.string().min(1, { message: "Company ID is required" })
      });

      const parse = schema.safeParse({ companyId });
      if (!parse.success) {
        return {
          message: "error",
          error: parse.error.flatten().fieldErrors.companyId?.[0] ?? "Invalid company ID"
        };
      }

      // Secure token handling
      const accessToken = await getAccessToken();
      if (accessToken === undefined) {
        return {
          message: "error",
          error: "Not authenticated"
        };
      }

      // API interaction and response handling
      const response = await fetch(`/api/user/subscribe?companyId=${companyId}`, {
        method: "PUT",
        headers: {
          Authorization: `Bearer ${accessToken}`
        }
      });

      // Type-safe response handling
      if (response.ok) {
        const user = await getUser(accessToken);
        return {
          message: "success",
          isSubscribed: isClientUser(user!) ? 
            user?.subscribedCompanies.includes(Number(companyId)) : 
            false
        };
      }
      // ...
    }
    ```

- **Authentication and Authorization:** <br>
  Server actions handle secure token management:
    ```ts
    export async function storeToken(request: StoreTokenRequest) {
      cookies().set({
        name: "accessToken",
        value: request.access_token,
        httpOnly: true,
        sameSite: "strict",
        secure: true,
        expires: new Date(Date.now() + Number(request.expires_at))
      });

      if (request.refresh_token) {
        cookies().set({
          name: "refreshToken",
          value: request.refresh_token,
          httpOnly: true,
          sameSite: "strict",
          secure: true,
          expires: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000)
        });
      }
    }
    ```

## TypeScript Integration
TypeScript is used throughout the application to ensure type safety and improve developer experience:

### Type Definitions
- **Base Types and Unions:** <br>
  Complex types are built using composition and union types for flexibility:
    ```ts
    // Base type that all users share
    type BaseUser = {
      id: number;
      name: string;
      email: string;
      role: Role;
    };

    // Specific user types with discriminated unions
    type ClientUser = BaseUser & {
      role: Role.CLIENT;
      subscribedCompanies: number[];
    };

    type CompanyUser = BaseUser & (CompanyMemberUser | CompanyAdminUser);

    // Union type for all possible user types
    export type User = ClientUser | CompanyMemberUser | CompanyAdminUser | AdminUser;
    ```

### Type Guards
- **Runtime Type Checking:** <br>
  Type guard functions enable safe type narrowing:
    ```ts
    export function isClientUser(user: User): user is ClientUser {
      return user.role === Role.CLIENT;
    }

    export function isCompanyUser(
      user: User
    ): user is CompanyMemberUser | CompanyAdminUser {
      return user.role === Role.COMPANY_MEMBER || 
             user.role === Role.COMPANY_ADMIN;
    }
    ```

### Generic Type Utilities
- **Advanced Type Patterns:** <br>
  Using TypeScript's type system for flexible, reusable types:
    ```ts
    // Helper type to extract specific user types
    export type UserOfRole<T extends Role> = Extract<User, { role: T }>;

    // Complex form state types
    export type LoginFormState = {
      message: string;
      errors: Record<keyof { 
        email: string; 
        password: string 
      }, string> | undefined;
      fieldValues: { 
        email: string; 
        password: string 
      };
    };
    ```

### Core Data Types
- **Application Data Models:** <br>
  Business entities are modelled with extensive type definitions:
    ```ts
    export type Company = {
      id: string;
      name: string;
      createdAt: string;
      description: string;
      owner: User;
      members: User[];
      settings: {
        appointmentDuration: number;
        appointmentBuffer: number;
        appointmentTypes: string[];
        appointmentLocations: string[];
        openingHours: {
          from: string;
          to: string;
        };
      };
    };

    export type Appointment = {
      id: number;
      from: Date;
      to: Date;
      title: string;
      description: string;
      companyId: string;
      location: string;
      clientId: string;
      Status: "PENDING" | "BOOKED" | "CANCELLED" | "COMPLETED";
    };
    ```

This robust type system ensures code reliability, enables better IDE support, and catches potential errors at compile-time rather than runtime.

## Tooling and Configuration

### ESLint Configuration
[ESLint](https://eslint.org/) is used as the primary code quality tool in the frontend repository, performing static analysis to catch problems before they reach production. It enforces consistent coding standards across the entire codebase and integrates perfectly with TypeScript to provide type-aware linting.

The ESLint configuration used here integrates multiple plugins:
 
**Parser Configuration:**

  - Uses `@typescript-eslint/parser` for TypeScript files
  - Configured with project's `tsconfig.json` for type-aware linting

**Plugin Integration:**

  - TypeScript-specific rules via `@typescript-eslint`
  - Next.js recommended configurations
  - React-specific linting rules
  - Tailwind CSS class validation

**Key Rules:**
  ```js
  {
    // Type Safety
    "@typescript-eslint/strict-boolean-expressions": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/consistent-type-imports": ["warn"],

    // Code Quality
    "no-duplicate-imports": "error",
    "no-unused-vars": "warn",
    "prefer-const": "error",

    // React Specific
    "react/jsx-key": "error",
    "react/jsx-no-target-blank": "error",
    "react/hook-use-state": "error",

    // Tailwind
    "tailwindcss/classnames-order": "error",
    "tailwindcss/no-contradicting-classname": "error"
  }
  ```

### Git Hooks and Commit Standards

#### Husky Configuration
Husky is used to enforce code quality checks and commit message standards:

- **Pre-commit Hook** (`/.husky/pre-commit`):
  ```sh
  #!/usr/bin/env sh
  . "$(dirname -- "$0")/_/husky.sh"
  npx lint-staged
  npm run lint
  ```

- **Commit Message Hook** (`/.husky/commit-msg`):
  ```sh
  . "$(dirname -- "$0")/_/husky.sh"
  npx commitlint --edit
  ```

The pre-commit process works in two stages:

- **Staged Files Check** (`lint-staged`):
    - Only runs on files staged in Git
    - Applies ESLint fixes and Prettier formatting
    - Fast and focused on changed files

- **Full Project Lint** (`next lint`):
    - Runs the Next.js linter across the entire codebase
    - Checks for:
        - ESLint rule violations
        - Import/export errors
        - React hooks usage
        - Dead code and unused exports
    - Ensures global code quality, not just changed files

#### Commit Linting
The project also enforces consistent commit messages using [commitlint](https://github.com/conventional-changelog/commitlint):

- Follows conventional commit format: `type(scope): message`
- Types include: feat, fix, docs, style, refactor, test, chore
- Scope is optional and describes the section of the codebase
- Message should be in present tense and descriptive

Example valid commits:
```
feat(auth): add OAuth2 integration
fix(dashboard): resolve data loading issue
docs(api): update endpoint documentation
```

## Continuous Integration

### GitHub Actions Workflow
The CI pipeline is configured to run on pull requests and pushes to all branches:

```yaml
name: CI
on:
  pull_request:
    branches: ["*"]
  push:
    branches: ["*"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Create env file
        run: |
          touch .env
          echo JWT_SECRET="mysecretvalue" >> .env
          # Additional environment variables...

      - name: Build project
        run: npm run build

      - name: Lint project
        run: npm run lint
```

Key Features:

- Sets up required environment variables
- Performs full build and lint checks
- Fails fast on any issues
