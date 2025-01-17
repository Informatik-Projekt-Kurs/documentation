# Performance considerations

With a large-scale web application, it was crucial to eliminate any shortcomings in server response times and client performance. To achieve this, we implemented several features designed to maintain efficiency while continuously testing the website for potential lags or bottlenecks.

## Rate Limiting

To mitigate brute-force attacks and excessive failed login attempts, the backend incorporates rate-limiting mechanisms using a completely custom-made system. This helps to prevent unauthorized access and potential security breaches by limiting the number of requests from a specific IP address or user within a certain time frame.

**Example IP specific rate-limiting implementation:**

```java
private final HashMap<String, LinkedList<Long>> requests = new HashMap<>();
private final int maxRequests = 20;
private final long refreshTime = 1000 * 10; // 10 seconds

@Override
protected void doFilterInternal(@NotNull HttpServletRequest request, @NotNull HttpServletResponse response, @NotNull FilterChain filterChain) throws ServletException, IOException {
    String ip = request.getRemoteAddr();

    if (requests.containsKey(ip))
        requests.get(ip).addLast(System.currentTimeMillis()); //Add request time to known address 
    else
        requests.put(ip, new LinkedList<Long>(Collections.singleton(System.currentTimeMillis()))); //Create space for unknown address

    if (requests.get(ip).size() > maxRequests) {
        response.setStatus(429);
        response.getWriter().write("Too many requests");
        return;
    }

    filterChain.doFilter(request, response);
}
```

## Frontend Optimization

Next.js is fundamental to our frontend performance strategy. While traditional React applications often face challenges with initial load times and rendering performance, Next.js provides powerful optimization features that, when properly implemented, significantly enhance application performance.

### Server-Side Rendering (SSR)
One of Next.js's most significant advantages is its sophisticated server-side rendering implementation. Unlike traditional React applications that send minimal HTML and rely heavily on client-side JavaScript, our application utilizes server-side rendering to generate complete HTML on the server. This results in faster page loads and improved search engine optimization.

This approach proves particularly valuable in our booking interfaces. When users access the company dashboard, they receive pre-rendered content containing their appointments and company data, eliminating the traditional loading sequence found in client-side rendered applications. The server handles data fetching, processing, and HTML generation, significantly reducing the browser's workload.

Server components, a step beyond traditional SSR, form a core part of our architecture. These components execute entirely on the server, transmitting only the necessary HTML to the client without JavaScript overhead. This makes them ideal for static content like company information displays and non-interactive dashboard elements.

### Smart Rendering Strategy
Our application utilizes Next.js's App Router to implement intelligent rendering decisions. Server components handle static elements like dashboard layouts and navigation, while client components manage interactive features that require browser-side execution.

This separation is particularly effective in our booking system, where the appointment scheduler maintains full interactivity while surrounding elements benefit from server-side rendering. The result is a balance between immediate content visibility and smooth user interactions.

### Data Flow Optimization
Next.js's server components transform our data handling approach. Rather than relying on client-side data fetching, we push data requirements to the server where possible. This proves especially effective in our company dashboard, which aggregates data from multiple sources.

Our GraphQL implementation through Apollo Client complements this architecture. Server components resolve complex data requirements before transmission, reducing loading states and minimizing client-side state management overhead. When client-side updates are necessary, our context system maintains efficient data synchronization.

### Route Optimization
The App Router provides sophisticated routing capabilities that enhance navigation performance. Our application structure leverages these features to maintain smooth transitions while updating only necessary page elements, optimizing both network usage and processing requirements.

This architecture particularly benefits our multi-step booking flow, where users progress through several stages while maintaining context. The routing system preserves application state efficiently without requiring complex client-side state management.

### Performance Under Load
Next.js enables robust performance even with substantial data volumes. The company appointment calendar, which may display hundreds of appointments, demonstrates this capability. Our server-side processing approach allows us to handle large datasets efficiently, sending only essential data to the client.

We extend this efficiency to real-time updates, using a combination of server-side processing and selective updates to maintain data currency without overwhelming client resources.

### Resource Management
Next.js provides automatic optimization for static resources, including images and fonts. The framework's built-in image optimization serves appropriately sized assets based on device characteristics, while font optimization ensures consistent text visibility during loading. These optimizations contribute significantly to overall performance and user experience.

By leveraging Next.js's features effectively, our application maintains high performance and responsiveness even under demanding conditions. The framework's optimization capabilities, combined with our thoughtful implementation, result in an efficient and scalable application architecture.
