# Backend Structure

## **Divisions**
Every part of the backend is split into multiple parts. The most noticeable elements are the entity management classes. These help break up responsibilities of each element into better maintainable subclasses. The often reoccurring ones, with there respective responsibilities are:

#### Entity
This is the primary object for each element, where fields are defined, and the database connection type is configured. The purpose of this class is only to set the basic conditions whenworking with this dataset. As a best practice, no business logic should be implemented within this class; its sole responsibility is to handle setup and configuration. A simple entitymight look like this:
```java
@Document(collection = "companies") //Clarification of database location
@Data //Springs implementation of getters and setters
@NoArgsConstructor
public class Company {
    private long id;
    private String name;
    private String description;
    private BusinessType businessType;
    private ArrayList<Long> memberIds;
    @Indexed(unique = true) //Primary key of the dataset
    private String ownerEmail;

    public Company(long id,String name, String ownerEmail, long ownerId) {
        this.id = id;
        this.name = name;
        this.ownerEmail = ownerEmail;
        this.description = "";
        this.businessType = null;
        this.memberIds = new ArrayList<>();
        this.memberIds.add(ownerId);
    }
}
```

#### Controller
The controllers task lies in distributing work between the different elements. A main thing one might see it the routing the HTTP requests into the Service. It not only takes HTTPrequests and passes them on, it also returns the data along with the specific status code or any errors that might occur during the request processing. Since its only job is to managerequests this element also should contain minimal logic. In this specific example the registration process of a user is redirected to the user service, returning a HTTP code 200 on success. If any errors were to occur it would handle each one individually. 
```java
@PostMapping(path = "signup")
@ResponseBody
public ResponseEntity<?> registerNewUser(@RequestParam MultiValueMap<String, String> data) {
  try {
    userService.registerNewUser(data); //Forwarding the request to the service
    return ResponseEntity.ok().build(); //Response after successful operation with code 200
  
  } catch (Throwable t) { //Error Handling
    Class<? extends Throwable> tc = t.getClass();
    if (tc == IllegalArgumentException.class)
      return ResponseEntity.status(HttpStatus.NOT_ACCEPTABLE).body("message: " + t.getMessage());
    if (tc == NameAlreadyBoundException.class)
      return ResponseEntity.status(HttpStatus.CONFLICT)
          .body("message: " + t.getMessage());
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body("message: " + t.getMessage());
  }
  }
```

#### Service
Service classes act as the brain of each object, handling core functionalities such as creating, filtering, formatting, and transforming the data received from the controller. Theseclasses are often the most complex and diverse parts of the codebase. Due to their role in processing and managing data, service classes frequently require input and interact withvarious components across the API, leading to a significant number of dependencies and integrations.
```java
@Service
@RequiredArgsConstructor
public class UserService {

  private final UserRepository userRepository;
  private final JwtService jwtService;
  private final PasswordEncoder passwordEncoder;
  private final AuthenticationManager authenticationManager;
  private final CompanyRepository companyRepository;
  private final AppointmentRepository appointmentRepository;
  //[...]
```

#### Repository
These classes are primarily managed by Spring Boot and serve as the bridge between the API and the database layer. These operate in a way that allows you to define method names asthough writing queries directly. Spring interprets these methods automatically, making database interactions both clear and straightforward. Since all the interpretation is done bySpring itself these classes are often among the the smallest and most concise. 
```java
@Repository
public interface CompanyRepository extends MongoRepository<Company, Long> {
  Optional<Company> findCompanyByOwnerEmail(String ownerEmail);
  Optional<Company> findCompanyById(long id);
}
```

#### Configuration
Lastly, there is the configuration class, which contains settings that are executed either on startup or every incoming request to secure everything operates according to the definedrules. These are not only very conveniant while testing, to automatically populate the database with test samples, it also provides a reliable way to implement settings that mustpersist over time. One example of a permanent configuration is the `securityFilterChain` that manages incoming requests:
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final AuthenticationProvider authenticationProvider;
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(
                        authorizeRequests ->
                                authorizeRequests
                                        .requestMatchers("/api/user/login", "/api/user/signup", "/graphql/**")
                                        .permitAll() // Whitelist
                                        .anyRequest().authenticated() // Everything else should be authenticated
                )
                .sessionManagement(
                        sessionManagement ->
                                sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .httpBasic(Customizer.withDefaults())
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .build();
    }
}
```

This approach of division ensures a clear separation of concerns, promoting maintainability and consistency across the backend architecture. By isolating specific responsibilities into distinct classes, the codebase becomes easier to understand, debug, and extend. This modular design allows developers to make changes or add new features without impacting unrelated parts of the system, reducing the risk of errors. Additionally, it facilitates better collaboration within teams, as each team member can focus on specific components without interfering with others. Ultimately, this structured approach enhances the scalability and long-term sustainability of the application.