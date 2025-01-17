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

## {FRONTEND} (add the optimisation of pages here)