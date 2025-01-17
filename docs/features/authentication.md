# User Authentication & Authorization
MeetMate implements a robust and secure authentication and authorization system to ensure the protection of user data and control access to various application features and resources.

## Authentication

### User Registration and Login

MeetMate supports credential-based authentication. During registration, users provide their name, email, and password on the signup page. Upon successful registration, their information is securely stored in a PostgreSQL database, with passwords hashed using a secure hashing algorithm from the spring boot security library.

**Example registration process:**

```typescript
// --- Frontend (NextJS Server Component) --- //
const handleRegistration = async (name, email, password) => {
    // [... validate input fields ...]
    const response = await fetch(process.env.BACKEND_DOMAIN + "/api/user/signup", {
        method: "POST",
        headers: {
            "Content-Type": "application/x-www-form-urlencoded;charset=UTF-8"
        },
        body: encodedData
    });
    if (response.ok) {
        return { // Handle successful registration
            message: "success",
            errors: undefined,
            fieldValues: { name: "", email: "", password: "", confirmPassword: "" }
        };
    } else if (response.status === 409) {
        return { // Handle email duplication error (specific errors)
            message: "error",
            errors: {
                name: undefined!,
                email: "This email is already taken",
                password: undefined!,
                confirmPassword: undefined!
            },
            fieldValues: { name, email, password, confirmPassword }
        }; // [... other error handling ...]
    } else {
        return { // Handle other errors
            message: "error",
            errors: { name: "", email: "", password: "", confirmPassword: "" },
            fieldValues: { name, email, password, confirmPassword }
        };
    }
}
```
```java
// --- Backend (Spring Boot Controller) --- //
@PostMapping(path = "signup")
@ResponseBody
public ResponseEntity<?> registerNewUser(@RequestParam MultiValueMap<String, String> data) {
    try {
        userService.registerNewUser(data);
        return ResponseEntity.ok().build();
    } catch (Throwable t) {
        // [...]
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
For login, users navigate to the login page and enter their email and password credentials. The application validates these credentials against the stored data in the database by comparing the provided password with the hashed password.

### Log In and Sign Up pages
Like every design in Meetmate, we have chosen a simplified approach to ensure clarity,
accessibility, and a smooth user experience. Emphasizing simplicity and usability, it features a
clean, dark-themed design with blue accents for a professional yet straightforward look. This
design encourages user focus during account creation and supports accurate data entry,
minimizing errors.

![[Example signup page]](../assets/oauth.png)

### Session Management

Upon successful authentication, MeetMate generates JSON Web Tokens (JWTs) for session management. An access token and a refresh token are created and stored as HttpOnly cookies on the client-side. The access token is used for authenticating subsequent requests to the backend, while the refresh token is used to obtain a new access token when the current one expires.

**Example token generation and verification:**

```java
// --- Backend (Spring Boot Service) --- //
public String generateAccessToken(User user) throws EntityNotFoundException {
    if (user == null) throw new EntityNotFoundException("No user specified");
    return Jwts.builder()
        .setClaims(user.generateMap())
        .setSubject(user.getEmail())
        .setIssuedAt(new Date(System.currentTimeMillis()))
        .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 5)) // expires in 5 minutes
        .signWith(getSigningKey(), SignatureAlgorithm.HS256)
        .compact();
}
```
```typescript
// --- Middleware (NextJS) --- //
// If no user, try to refresh the access token using a valid refresh token.
if (user === null) {
    console.log("No user found, attempting to refresh access token.");
    const refreshToken = req.cookies.get("refreshToken")?.value;
    if (refreshToken !== undefined || refreshToken !== "") {
        const newAccessToken = await refreshAccessToken(refreshToken);
        if (newAccessToken !== undefined) {
            response.cookies.set("accessToken", newAccessToken[0], { httpOnly: true, secure: true, sameSite: "strict" });
            response.cookies.set("expires_at", newAccessToken[1], { httpOnly: true, secure: true, sameSite: "strict" });
            user = await getUser(newAccessToken[0]);
            if (user === null) {
                return NextResponse.redirect(new URL("/login", req.url));
            }
        } else {
            console.log("Error refreshing access token, redirecting to login.");
            return NextResponse.redirect(new URL("/login", req.url));
        }
    } else {
        console.log("No refresh token found, redirecting to login.");
        return NextResponse.redirect(new URL("/login", req.url));
    }
}
```

The frontend sends requests to the backend along with the valid access token to retrieve the user's information and render content accordingly.

## Authorization

MeetMate employs a middleware layer to handle authentication and authorization, controlling access to specific routes and pages based on the user's role and permissions.

When accessing a protected route or page, the application sends a request to the backend to obtain the user's role information. Based on this role, the frontend dynamically renders or restricts access to certain features and content.

**Example authorization middleware and role-based rendering:**

```jsx
// Frontend (NextJS Page Component)
const AdminDashboard = () => {
    const [userRole, setUserRole] = useState("");

    useEffect(() => {
        const fetchUser = async () => {
            try {
                const accessToken = await getAccessToken();
                setUser(await getUser(accessToken));
            } catch (error) {
                console.error("Failed to fetch user", error);
            }
        };
        fetchUser().catch(console.error);
    }, []);

    return (
        <div>
            {user?.role === "ADMIN" && <AdminContent />}
            {/* Render user dashboard content */}
        </div>
    );
};
```