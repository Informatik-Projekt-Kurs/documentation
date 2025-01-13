# Backend Development Principles

## **Annotations**
Spring Boot's main gimmick is its use of annotations. With them, one can configure different elements of the code in a single, easily readable statement, right at the start of a class or method. These annotations eliminate the need for extensive configuration, streamlining the setup process and making the codebase more intuitive. This declarative style not only enhances clarity but also reduces boilerplate code, allowing developers to focus more on building functionality rather than configuring the framework. Complementing Spring Boots funtionality of annotations is a small dependency called Lombok. By itself it cuts down on repetetive code segments to clear up and simplify the development process.

### **Method Annotations**
A few examples for method specific annotations are:

- **RequestMapping** <br>
    The statement `@RequestMapping()` is essential for routing HTTP requests. It eliminates the need for a dedicated routing class by directly mapping a method to an API endpoint. When configured in this way, it creates an endpoint for handling queries using the GET HTTP method:
    ```java
    @RequestMapping(method = {RequestMethod.GET}, path="get")
    public User getUser(){
      //[...]
    };
    ```

- **Transactional** <br>
    This annotation signals to Spring Boot that the method it decorates will directly make changes to the database, whether it's creating, deleting, or updating a record. Without this annotation, no modifications to the data can be made. Therefore its main usecase lies in data altering methods:
    ```java
    @Transactional
    public void deleteUser(long id) {
      userRepository.deleteById(id);
    }
    ```

### **Class Annotations**
Alongside method annotations there are also class assigned ones, which will give context to everything contained in the class. 

- **Components** <br>
    Most importantly there are the annotations, such as `@Controller` or `@Repository`, that are used to define the different elements of the API. These annotations are essential for communicating to Spring Boot how to route and handle incoming requests, ensuring that each request is processed by the correct component in the system. By using similar annotations developers can easily structure the application, making it intuitive, modular, and scalable. 

- **Documents and Tables** <br>
    Other use cases of class annotations are to specify where to store data. While we define a dataset we can also use annotations to indicate its storage location. While the different databases need other annotations, their purpose is essentially the same. For example, PostgreSQL's tables:
    ```java
    @Table(name = "users")
    public class User {
      private long id;
      private String name;
      private String email;
      private String password;
      //[...]
    }
    ```
    are not all to different from defining MongoDB's collections:
    ```java
    @Document(collection = "companies")
    public class Company {
      private long id;
      private String name;
      private String description;
      //[...]
    }
    ```
    In both cases, annotations provide a clear way to map classes to their corresponding storage structures, whether they are relational tables or NoSQL collections.

### **Field Annotations**
blablabla - RequestHeader maybe @Nullable?


## **Dependency Injection**
Dependency injection is a key feature in Spring Boot programming. It enables communication between different components throughout the code while preventing the creation of multiple instaces of the same class. Normally, when connecting two classes one would instantiate a new object manually, like this: <br>
```java
UserRepository userRepository = new UserRepository();
```
In larger codebases, this approach can lead to a high amount of duplicate instances which take up valuable space and complicate management. Spring Boot addresses this problem through its system of dependency injection. Instead of manually creating new instances, the declaration of new classes works like this: <br>
```java
private final UserRepository userRepository;
```
Spring recognizes that this class remains unchanged during the runtime of the application.  As a result, it manages these dependencies by creating a single instance of each component and connecting them into a network of unique instances, that reduces memory usage, improves efficiency, and simplifies the codebase.

## **Optionals**
An essential part of working with datasets is ensuring that they are complete and present. Optionals are designed to address the uncertainty of whether a dataset exists or not, allowing developers to handle such scenarios proactively. The `.orElse()` and related methods use a lambda expression to define specific actions when data is missing, such as returning default values, throwing exceptions, or taking alternative approaches. An example would be consider fetching a user that might or might not exist in the database. The `.orElseThrow()` method can be used to handle this uncertainty by throwing a custom exception if no user is found:
```java
User user = userRepository.findUserByEmail(email)
  .orElseThrow(() -> new EntityNotFoundException("User does not exist"));
```
Right here the API attempts to fetch a user from the database by its email address. If the email is invalid or the user does not exist, an exception is thrown, making it clear that the data retrieval was unsuccessful. This approach ensures the application can handle missing data gracefully and take the necessary actions to maintain data integrity and user experience.

## **HTTP Responses**
For a good usability, the API needs to communicate the success or faliure of operations to the user. This can be achieved through the `ResponseEntity<?>` class, which provides a useful toolkit for managing HTTP responses. It supports all standard HTTP status codes, such as `HttpStatus.OK = 200`, `HttpStatus.NOT_FOUND = 404` or `HttpStatus.INTERNAL_SERVER_ERROR = 500`, allowing developers to convey the state of operations clearly and consistently. By leveraging ResponseEntity, the API enables a more intuitive and understandable coding experience while enhancing communication between the client and the server. A response of a user query might be implemented like this:
```java
public ResponseEntity<?> getUser(long id) {
  try {
    return ResponseEntity.ok(
      userService.getUser(id)
    );
  catch (Throwable t){
      return ResponseEntity.status(HttpStatus.NOT_FOUND).body("message: " + t.getMessage());
  }
}
```
The occuring responses would then look like this to the user:

- Success:
```json
get cool code
```

- Error:
```json
get bad code
```

<!-- since controllers catch errors. service hat es einfach mit just use a throwable if something is wrong-->