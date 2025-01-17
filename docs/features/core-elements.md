# Division into three Core Elements

Early on, we decided to divide the project into three distinct parts. The rationale behind this separation was that it would help keep our work more organized and focused. In hindsight, this decision has proven valuable, allowing us to address specific issues through targeted debugging without impacting other parts of the application. With each component isolated, problems in one area don't affect the accessibility of others, providing clarity and making testing more efficient and manageable. This modular approach has greatly improved our workflow and overall development process.

## The three Parts
Based on this reasoning, we decided to split the frontend, backend, and design processes into distinct categories, as outlined below:

### Clients
The clients are the main users of MeetMate. They discover companies, book appointments and use their time schedule. These sorts of tasks can be found in every part of the development team, be it the fetching of upcoming appointments in the backend, booking appointments seamlessly in the frontend or the intuitive and user-friendly design of the client-facing interface.

This example demonstrates the API endpoint `/api/user/relevantAppointments`, which allows a client to fetch their upcoming appointments. Utilizing the access token system allows us to retrieve all necessary data without requiring additional values, conserving bandwith both for the user and also our servers. After retrieving the requested data from the database, the API responds with a specific HTTP response."
```java
@GetMapping(path = "relevantAppointments")
@ResponseBody
public ResponseEntity<?> getRelevantAppointments(
        @RequestHeader(name = "Authorization") String token){ //Extracting the access token from the request headers
    token = token.substring(7); //Removing the "Bearer: " prefix from the access token 
    try {
        return ResponseEntity.ok(userService.getRelevantAppointments(token));
    } catch (Throwable t) {
        Class<? extends Throwable> tc = t.getClass(); //Getting the type of error
        //Responding with a fitting HTTP code
        if (tc == EntityNotFoundException.class)
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body("message: " + t.getMessage());

        if (tc == IllegalAccessException.class)
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body("message: " + t.getMessage());
        //Fallback if an unknown error happens
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("message: " + t.getMessage());
    }
}
```

In the frontend, we manage client-specific data through a dedicated context that provides access to appointments, companies, and user information:
```ts
type DashboardContextProps = {
    user: ClientUser | undefined;
    loading: boolean;
    companies: { getCompanies: Company[] } | undefined;
    relevantAppointments: Appointment[];
    appointments: Appointment[];
    refreshAppointments: () => Promise<void>;
};

export function DashboardProvider({ children }: { children: React.ReactNode }) {
    // Client-specific state management
    const [user, setUser] = useState<ClientUser>();
    const [appointments, setAppointments] = useState<Appointment[]>([]);

    // Automatic data refresh through polling
    const { data: companies } = useQuery(GET_COMPANIES, {
        pollInterval: 300000
    });

    // ... context implementation
}
```

Finding a fitting design for the users dashboard plays a major role in enhancing their experience when scheduling appointments. We made it simple to use so everyone can understand it quickly. When users first see the dashboard, they can find what they need without getting confused. We removed complex features and kept only the important tools that help users manage their time well.

![[Overview Scheduler presented on the Client Dashboard]](../../assets/dashboard-overview-scheduler.png)

The main part of our dashboard is the calendar. We use bright colors like magenta, yellow, and purple to show different appointments. These colors help users in two ways:
First, users can see what type of appointment they have without reading all the details. Second, when users look for a specific appointment, they can find it by looking for its color instead of reading through many appointment names.


Our dashboard makes scheduling much better than before. It changes how users work with their daily schedules. The simple design helps users manage their time more easily, making it a better tool for everyone who uses it.

### Companies
While company members are not all that different from clients, the companies they belong to require significantly more functionality. Companies need robust features to manage their profiles such as handling administrative tasks such as scheduling appointments or tracking performance metrics. Additionally, companies often interact with multiple users and datasets, necessitating a scalable structure to manage these relationships efficiently.

In the frontend, this is reflected in our GraphQL implementation which manages company data and member relationships. We use a combination of GraphQL queries and React Context to efficiently manage company data:
```ts
export const GET_COMPANY = gql`
  query GetCompany($id: ID!) {
    getCompany(id: $id) {
      id
      name
      description
      businessType
      memberIds
      ownerEmail
    }
  }
`;

type CompanyContextType = {
    user: CompanyUser | undefined;
    company: { getCompany: Company } | undefined;
    members: User[];
    appointments: Appointment[];
    clients: { getClients: User[] };
};
```
This structure serves multiple purposes. The GraphQL query ensures we only fetch the exact data needed for each company view, reducing unnecessary data transfer. The CompanyContext type definition shows how we manage the complex relationships between different entities - from user management to appointment tracking. This context provides a central source of truth for company-related data, making it available throughout the application while maintaining consistent state management. The combination of typed data structures and GraphQL queries ensures type safety throughout the application while providing efficient data access patterns.

The backend follows the same approach, allocating an entirely new set of management classes and a separate database for this entity. The significant data loads generated by companies - especially when fetching all appointments for their clients - required a shift from the PostgreSQL database, used for clients, to a faster, read-optimized, non-relational MongoDB database. This transition ensures better performance and scalability for handling high-volume queries efficiently. An instance of reading large datasets occurs during the fetching of all clients that are dealing with a company.
```java
public ArrayList<GetResponse> getClients(String token) throws IllegalAccessException {
    String email = jwtService.extractUserEmail(token);
    //Getting the user and their associated company
    User companyMember = userRepository.findUserByEmail(email)
            .orElseThrow(() -> new EntityNotFoundException("User not found!"));

    Company company = companyRepository.findCompanyById(companyMember.getAssociatedCompany())
            .orElseThrow(() -> new EntityNotFoundException("Company not found"));
    //Checking if the user retrieving the client information is a member of the company
    if (isNotCompanyOwner(email)
            && isNotCompanyMember(company, userRepository.findUserByEmail(email)
            .orElseThrow(() -> new EntityNotFoundException("User not found!"))
            .getId()))
        throw new IllegalAccessException("Not a company member");

    ArrayList<Appointment> appointments = appointmentRepository.findAppointmentsByCompanyId(company.getId());
    ArrayList<User> clients = new ArrayList<>();

    //Constructing the list of clients
    User client;
    for (Appointment appointment : appointments) {
        client = userRepository.findUserById(appointment.getClientId()).orElse(null);
        if (clients.contains(client)) continue;
        clients.add(client);
    }

    //Retrieving data of the clients
    ArrayList<GetResponse> response = new ArrayList<>();
    for (User user : clients) {
        response.add(GetResponse.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .build());
    }

    return response;
}
```
This example is particularly resource-intensive, as it involves not only searching through every appointment to identify those associated with the company but also fetching each client’s details to include in the response. These multiple iterative processes demand a robust and efficient system to handle the complexity without compromising performance. To address this, we employ optimized database queries and caching mechanisms to minimize redundant operations and reduce processing time.

### Appointments

The third pillar of MeetMate are the appointments, the primary idea behind the whole project. At first glance, however, they might seem less prominent than one would expect. This is because their value is derived from the integration with the other components, where they play a central role in linking users, companies, and various data points. The appointments integrate seamlessly with user profiles and company data, pulling relevant information from each to create a smooth, cohesive experience. We ensure that appointments are not just standalone entities but rather dynamic components that evolve with the system’s broader functionality, enabling better organization and user interaction.

Following the concern of high amounts of records the backend has to consider queries carefully, supporting the performance with faster fetching alongside the MongoDB database and optimised queries through GraphQL. To address the significant amount of data transferred between the server and the user, it's crucial to implement query restrictions that minimize the data sent A prime example for this is the fetching of available appointments for a company, where only relevant data is retrieved, ensuring efficiency and reducing unnecessary overhead.

```java
public ArrayList<Appointment> getAvailableAppointments(long id, Instant date) {
    ArrayList<Appointment> appointments = appointmentRepository.findAppointmentsByCompanyId(id);
    ArrayList<Appointment> availableAppointments = new ArrayList<>();

    // Get start of current day
    LocalDate today = LocalDate.now(ZoneId.systemDefault());
    Instant startOfToday = today.atStartOfDay(ZoneId.systemDefault()).toInstant();

    for (Appointment appointment : appointments) {
        Instant appointmentTime = appointment.getFrom();
        //Considering the case where no date is given the API will output all available appointments
        //Else it filters out all days appointments not on the specified day
        boolean isSameDay = date == null ||
                LocalDate.ofInstant(date, ZoneId.systemDefault())
                        .equals(LocalDate.ofInstant(appointmentTime, ZoneId.systemDefault()));

        if (appointment.getStatus() == AppointmentStatus.PENDING
                && appointmentTime.isAfter(startOfToday)  // Compare with start of day instead
                && isSameDay)
            availableAppointments.add(appointment);
    }
    return availableAppointments;
}
```

The frontend complements the backend's efficient data handling through carefully structured GraphQL queries and React components. Our GraphQL query is designed to fetch only the essential appointment data needed for display or booking:
```ts
export const GET_AVAILABLE_APPOINTMENTS = gql`
  query GetAvailableAppointments($companyId: ID!, $date: String) {
    getAvailableAppointments(companyId: $companyId, date: $date) {
      id
      from
      to
      companyId
      clientId
      description
      location
      Status
      title
    }
  }
`;
```

This query structure ensures we receive exactly the fields needed for appointment management and display, no more and no less. The implementation in our Bookings component showcases how we handle this data with proper error management and user feedback:

```ts
function Bookings() {
    const { user, companies, appointments, refreshAppointments } = useDashboardData();

    // Available appointments fetching with error handling
    const { data: availableSlots, error: slotsError } = useQuery(GET_AVAILABLE_APPOINTMENTS, {
        variables: {
            companyId: bookingState.selectedCompany,
            date: formatDateToISOWithoutTime(bookingState.selectedDate)
        },
        skip: bookingState.selectedCompany === "" || bookingState.step !== 2,
        fetchPolicy: "network-only",
        onError: (error) => {
            // Comprehensive error handling for different scenarios
            if (error.networkError) {
                setBookingState(prev => ({
                    ...prev,
                    error: "Unable to connect to the server. Please check your connection."
                }));
            } else if (error.graphQLErrors?.length > 0) {
                // Handle specific GraphQL errors
            }
        },
        onCompleted: (data) => {
            if (!Boolean(data?.getAvailableAppointments?.length)) {
                setBookingState(prev => ({
                    ...prev,
                    error: "No available appointments found for this date. Please try another date."
                }));
            }
        }
    });

    // Display the appointments in a scheduler component
    return (
        <div className="mt-4 flex h-fit w-full rounded-[20px] bg-subtle p-6">
        <Scheduler
            openingHours={schedulerHours}
    data={filteredAppointments}
    handleAppointmentCancel={handleAppointmentCancel}
    handleAppointmentChange={handleAppointmentChange}
    />
    </div>
);
}
```
This implementation demonstrates several key features:

- Real-time data synchronization through the Apollo Client's polling mechanism
- Comprehensive error handling for different failure scenarios
- Conditional query execution based on user interaction state
- Type-safe data handling through TypeScript integration
- Efficient state management using React hooks and contexts

The Scheduler component then takes this structured data and renders it in a user-friendly interface, handling user interactions like cancellations and modifications while maintaining data consistency with the backend. This approach ensures that users receive immediate feedback while the application maintains data integrity across all operations.
