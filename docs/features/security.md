# Security Measures

## Endpoint protection

In order to prevent users from accessing endpoints without valid access tokens, the backend secures its endpoints through a security configuration. This configuration protects all endpoints, requiring a valid token for access, while allowing starting pages like /login and /signup to remain accessible without authentication. The entire application is designed to be stateless, which helps to prevent issues such as session hijacking or excessive server-side state management. The primary purpose of this configuration is to establish a security filter chain - a path that each request follows as it traverses the backend services - ensuring that all incoming requests are authenticated before reaching protected resources.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
  return httpSecurity
    .csrf(csrf -> csrf.disable())
    .authorizeHttpRequests(
      authorizeRequests ->
        authorizeRequests
          //Login and signup should not be secured
          .requestMatchers("/api/user/login", "/api/user/signup", "/graphql/**") 
          .permitAll()
          .anyRequest().authenticated() // Everything else should be authenticated
    )
    .sessionManagement(
      sessionManagement ->
        sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
    .httpBasic(Customizer.withDefaults())
    //Setting up the filters each request goes through
    .authenticationProvider(authenticationProvider)
    .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
    .build();
}
```

As observed in this implementation, the /graphql endpoint is not secured, despite hosting critical infrastructure. This is due to the way GraphQL requests are structured, which requires us to secure it separately, as demonstrated in the following example. This method intercepts any GraphQL requests to the backend, making the access token available for use in subsequent parts of the request path.

```java
@Component
public class RequestHeaderInterceptor implements WebGraphQlInterceptor {  
  @Override
  public Mono<WebGraphQlResponse> intercept(WebGraphQlRequest request, Chain chain) {
    String value = request.getHeaders().getFirst("Authorization"); //Fetch the token from the GraphQL request
    if (value != null) {
      //Make token accessible
      request.configureExecutionInput((executionInput, builder) ->
            builder.graphQLContext(Collections.singletonMap("token", value)).build()); 
    }
    return chain.next(request); //Continue the security chain
  }
}
```

## Global request cap
To prevent the server from being overwhelmed by a large volume of requests - potentially caused by a group of bots or a coordinated attack - a global request limit was implemented. This safeguard ensures that no single entity, or group of entities, can flood the API with requests, thereby protecting the backend from server crashes and data loss that could occur if the backend were to become unresponsive under heavy load. This global rate limit acts as a critical layer of defense, ensuring continuous service availability and data integrity, even in the face of malicious or inadvertent overuse.

**Example of a global rate-limiter:**

```java
private final LinkedList<Long> requests = new LinkedList<>();
  private final int maxRequests = 500;
  private final long refreshTime = 1000 * 1; // 1 second

  @Override
  protected void doFilterInternal(@NotNull HttpServletRequest request, @NotNull HttpServletResponse response, @NotNull FilterChain filterChain) throws ServletException, IOException {
    requests.addLast(System.currentTimeMillis());

    if (requests.size() > maxRequests) {
      response.setStatus(429);
      response.getWriter().write("Too many requests");
      return;
    }

    filterChain.doFilter(request, response);
  }
```

## Password Hashing

MeetMate employs industry-standard password hashing techniques to securely store user passwords in the database. This approach ensures that even in the event of a data breach, the actual passwords remain protected, as only the hashed values are stored. During backend initialization, a `BCryptPasswordEncoder` is automatically created and configured to encode all passwords. This encoder utilizes a technique called salting, where each password is combined with a random string before being hashed. Salting protects against dictionary attacks, which involve hashing common passwords and comparing them to stored hashes. Furthermore, it ensures that identical passwords result in unique hashes, preventing attackers from identifying patterns in the stored data. Lastly even hindering any affiliated people of looking up sensitive data in deployed databases. 

## Session Termination /// Dis not finished

When a user logs out of the application, the JWT tokens stored as HttpOnly cookies on the client-side are deleted, effectively terminating the user's session. This can be achieved by simply removing the cookies from the client-side storage.

## Token Management and Token Security

### HttpOnly Cookies
In the frontend, we implement secure token storage using HttpOnly cookies, which provides protection against XSS attacks by making tokens inaccessible to JavaScript:

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
      expires: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) // 30 days
    });
  }
}
```

This implementation includes several security features:

- `httpOnly`: Prevents JavaScript access to cookies, mitigating XSS attacks
- `sameSite: "strict"`: Mitigates CSRF attacks
- `secure: true`: Ensures cookies are only sent over HTTPS, protecting against network-based attacks
- Appropriate expiration times for different token types, ensuring tokens are not stored indefinitely thus reducing the risk of token theft

### Role-Based Route Protection
The application implements role-based access control (RBAC) through Next.js middleware, which was also previously mentioned in the Authentication Chapter. RBAC ensures users can only access routes appropriate for their role:

```ts
export async function middleware(req: NextRequest) {
  let user = await getUser(req.cookies.get("accessToken")?.value);
  let response = NextResponse.next();

  // Token refresh logic for expired sessions
  if (user === null) {
    const refreshToken = req.cookies.get("refreshToken")?.value;
    if (refreshToken) {
      const newAccessToken = await refreshAccessToken(refreshToken);
      if (newAccessToken) {
        // Update tokens and verify user
        user = await getUser(newAccessToken[0]);
      } else {
        return NextResponse.redirect(new URL("/login", req.url));
      }
    }
  }

  // Role-based route protection
  if (req.nextUrl.pathname.startsWith("/dashboard") && 
      !["CLIENT", "ADMIN"].includes(user.role)) {
    response = NextResponse.redirect(new URL("/company/dashboard", req.url));
  } else if (
    req.nextUrl.pathname.startsWith("/company/dashboard") &&
    !["COMPANY_MEMBER", "COMPANY_OWNER"].includes(user.role)
  ) {
    response = NextResponse.redirect(new URL("/dashboard", req.url));
  }

  return response;
}
```

This middleware provides several security benefits:

- Automatic token refresh handling
- Role-based access control
- Protection against unauthorized route access
- Seamless redirection for unauthorized users

## Session Termination
When a user logs out of the application, all authentication tokens are securely removed using the deleteToken function:

```ts
export async function deleteToken() {
  cookies().delete("accessToken");
  cookies().delete("refreshToken");
  cookies().delete("expires_at");
}
```

This ensures:

- Complete removal of all authentication tokens
- Immediate session termination
- Prevention of token reuse
