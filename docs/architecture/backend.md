# Backend Architecture

### **Framework and Languages**
- **Java:** A widely-used, object-oriented programming language designed for building cross-platform applications.
- **Spring Boot:** A robust Java framework for building scalable, production-ready backend applications with ease and efficiency.  
- **PostgreSQL:** A powerful relational database system ensuring reliable and secure data storage and management.  
- **RESTful APIs:** Enables seamless communication between systems using a stateless, standardized architecture.  
- **GraphQL:** A query language and runtime for APIs, enabling flexible and efficient data fetching and mutation.  
- **MongoDB:** A flexible NoSQL database optimized for handling unstructured and semi-structured data with high performance and scalability.  

### **Benefits**

**Java:** <br>
Picking up Java was a straightforward task for our developers, thanks to a strong foundation in object-oriented programming and prior experience with the language. Java’s strong type system helps catch errors early in development, ensuring more robust and maintainable code. 

**Spring Boot:** <br>
Spring Boot is a Java framework that reduces boilerplate code and simplifies application development. It emphasizes the extensive use of annotations, making code more concise, readable, and easy to maintain. Spring Boot optimizes dependency management by providing "starter" dependencies, which simplify the process of including and managing the required libraries for a project, making it easy to start developing applications quickly without worrying about complex configurations or compatibility issues.

**PostgreSQL:** <br>
PostgreSQL is a popular relational database that provides extensive support for SQL standards making it suitable for applications of all sizes. Combining Spring and PostgreSQL offers a powerful solution for building scalable and efficient applications. Spring integrates seamlessly with PostgreSQL through Spring Data JPA and the repository class type that comes with it, simplifying database interactions and reducing boilerplate code.

**RESTful APIs:** <br>
Another important part of MeetMates tech stack, used to implement the clients, sees the use of RESTful APIs. These APIs facilitate seamless communication between the client-side and the server, enabling efficient data exchange. By utilizing HTTP methods (such as GET, POST, PUT, and DELETE), RESTful APIs ensure a smooth interaction, making the system scalable and maintainable. This architecture enables the backend API to provide a fast experience for the interaction with the frontend development.

**GraphQL:** <br>
As MeetMate progressed in development it became apparent that handling large quantities of data with RESTful APIs could introduce unwanted latencies. Since RESTful APIs typically require fetching entire datasets, the client would inevitably overfetch data, resulting in inefficiency and unnecessary data transfer. This approach could lead to performance issues, which lead to the introduction of GraphQl. This query language allows clients to fetch only the data they need, improving performance and optimizing the user experience.

**MongoDB:** <br>
The final introduction was made with MongoDB, which is ideal for handling unstructured or semi-structured data. MongoDB's ability to handle high-volume, read-heavy operations, along with its flexible schema, allows to efficiently store and query appointment data without being constrained by fixed schema definitions or the need for complex joins. This combination of scalability, flexibility, and performance made MongoDB the ideal choice for these specific needs, complementing the rest of the tech stack. 

### **Descisions**
We chose this tech stack for our backend because we wanted to experiment with the benefits and trade-offs of each technology. Although we only needed some of the components for the project’s core functionality, using a combination of GraphQL, RESTful APIs, MongoDB, and PostgreSQL gave us the opportunity to deeply understand the strengths and limitations of each tool. It is important to note that performance considerations can vary depending on the specific query types and data structure. In some cases, PostgreSQL may still be more appropriate for structured data and complex transactions, while MongoDB shines with unstructured data and flexibility. Therefore, the decision to use both Postgres and MongoDB allows us to find the strengths of each database, ensuring the best performance for different parts of the system.