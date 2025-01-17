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

### Domain Models
- **Rich Domain Types:** <br>
  Business entities are modeled with comprehensive type definitions:
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

This robust type system ensures code reliability, enables better IDE support, and catches potential errors at compile-time rather than runtime. The combination of discriminated unions, type guards, and generic type utilities provides a flexible yet type-safe foundation for the application.
