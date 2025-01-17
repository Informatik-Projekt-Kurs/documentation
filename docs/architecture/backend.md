# Backend Architecture

### **Framework and Languages**
- **Spring Boot:** A robust Java framework for building scalable, production-ready backend applications with ease and efficiency.  
- **PostgreSQL:** A powerful relational database system ensuring reliable and secure data storage and management.  
- **RESTful APIs:** Enables seamless communication between systems using a stateless, standardized architecture.  
- **GraphQL:** A query language and runtime for APIs, enabling flexible and efficient data fetching and mutation.  
- **MongoDB:** A flexible NoSQL database optimized for handling unstructured and semi-structured data with high performance and scalability.  

### **Benefits**
**Spring Boot:** <br>
Spring Boot is a Java framework that reduces boilerplate code and simplifies application development. It emphasizes the extensive use of annotations, making code more concise, readable, and easy to maintain. Spring Boot optimizes dependency management by providing "starter" dependencies, which simplify the process of including and managing the required libraries for a project, making it easy to start developing applications quickly without worrying about complex configurations or compatibility issues.

**PostgreSQL:** <br>
PostgreSQL is a popular relational database that provides extensive support for SQL standards making it suitable for applications of all sizes. Combining Spring and PostgreSQL offers a powerful solution for building scalable and efficient applications. Spring integrates seamlessly with PostgreSQL through Spring Data JPA and the repository class type that comes with it, simplifying database interactions and reducing boilerplate code.

**RESTful APIs:** <br>
The final part of MeetMates "original" tech stack, used to implement the clients, sees the use of RESTful APIs. These APIs facilitate seamless communication between the client-side and the server, enabling efficient data exchange. By utilizing HTTP methods (such as GET, POST, PUT, and DELETE), RESTful APIs ensure a stateless interaction, making the system scalable and maintainable. This architecture enables the backend API to provide a smooth experience for the interaction with the frontend development.

**GraphQL:** <br>
As MeetMate progressed in development it became clear that handling large quantities of data with RESTful APIs would pose limitations. Since RESTful APIs typically require fetching entire datasets, the client would inevitably overfetch data, resulting in inefficiency and unnecessary data transfer. This approach could lead to performance issues, which lead to the introduction of GraphQl. This query language allows clients to fetch only the data they need, improving performance and optimizing the user experience.

**MongoDB:** <br>
The final introduction was made with MongoDB, which is ideal for handling unstructured or semi-structured data. Additionally, it offers better performance with larger datasets. Given the sheer number of appointments that would be created, we needed a faster and more scalable alternative to PostgreSQL. MongoDB's ability to handle high-volume, read-heavy operations, along with its flexible schema, allows to efficiently store and query appointment data without the performance limitations of relational databases.This combination of scalability, flexibility, and performance made MongoDB the ideal choice for these specific needs, complementing the rest of the tech stack.
