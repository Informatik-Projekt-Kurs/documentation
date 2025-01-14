# Backend Deployment Structure

## **Managing Four Containers**
With Docker Compose, all the services required for the backend can be managed in a single, unified file. This makes it easy to configure and deploy the entire backend stack. Docker Compose also ensures proper communication between services through the use of networks, allowing containers to seamlessly interact with each other. 

```yaml
services: 
  meet-mate:
    networks:
      - postgres-net
      - mongo-net

  postgres:
    networks:
      - postgres-net

  mongodb:
    networks:
      - mongo-net

  mongo-express:
    networks:
      - mongo-net

networks:
  postgres-net:
    driver: bridge
  mongo-net:
    driver: bridge
```

Environment variables are set within the Compose file to further ensure that each service is correctly configured and able to communicate with others.

```yaml
meet-mate:
  ports:
    - "8081:8080"
  environment:
    SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/test
    SPRING_DATASOURCE_USERNAME: postgres
    SPRING_DATASOURCE_PASSWORD: 1234
    SPRING_DATA_MONGODB_URI: mongodb://root:pass@mongodb:27017/admin

postgres:
  ports:
    - "5432:5432"
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: 1234
    POSTGRES_DB: test

mongodb:
  ports:
    - "27017:27017"
  environment:
    MONGO_INITDB_ROOT_USERNAME: root
    MONGO_INITDB_ROOT_PASSWORD: pass

mongo-express:
  ports:
    - "8082:8081"
  environment:
    ME_CONFIG_MONGODB_ADMINUSERNAME: root
    ME_CONFIG_MONGODB_ADMINPASSWORD: pass
    ME_CONFIG_MONGODB_SERVER: mongodb
```

To maintain high availability, Docker Compose can be configured to wait for specific containers to start before others, ensuring the right order of initialization. Additionally, containers can be set to automatically restart if they go down, ensuring the backend remains operational without manual intervention.

```yaml
meet-mate:
  restart: always
  depends_on:
    - postgres
    - mongodb

postgres:
  restart: always
  volumes:
    - ./data:/var/lib/postgresql/data
  
mongodb:
  restart: always
  volumes:
    - data:/data
  
mongo-express:
  restart: always
  depends_on:
    - mongodb
    
volumes:
  data: {}
```

## **Building The API**
Since the backend is based on a Java framework it requires compilation into a .jar file before deployment. To simplify the more complex setup process of the API, this is extracted into a dedicated Dockerfile. The Dockerfile handles all necessary configurations, including dependencies, build steps, and environment setup, before starting the application. This helps with more readability and easier maintainability. 

```docker
# Build the application
FROM maven:3.8.5-openjdk-17 as builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src/ ./src/
RUN mvn clean package -DskipTests=true

# Run the application
FROM openjdk:17
WORKDIR /app
# Create a group and user
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
COPY --from=builder /app/target/MeetMate.jar /app/MeetMate.jar
# Use the created user
USER appuser
CMD ["java", "-jar", "MeetMate.jar"]
```

Starting the container is now as easy as referencing this setup file.

ˋˋˋ yaml
meet-mate:
    build:
      context: .
      dockerfile: Dockerfile
ˋˋˋ