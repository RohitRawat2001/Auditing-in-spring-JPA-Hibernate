# Auditing-in-spring-JPA-Hibernate
# In Spring Framework, auditing refers to the ability to automatically capture and track certain information about entity changes, such as when and by whom an entity was created or modified. Spring Data JPA provides out-of-the-box support for auditing through annotations and configuration.

# Key Concepts in Spring Auditing
# -- Created Date and Last Modified Date:
      Automatically captures the timestamp when an entity is created or last modified.
# -- Created By and Last Modified By:
      Automatically captures the user or system responsible for creating or modifying an entity.

# How to Enable Auditing in Spring

To enable auditing in your Spring application, follow these steps:

1. Adding Dependencies: Ensure that your project includes the necessary Spring Data JPA dependencies in pom.xml or build.gradle.

2. Enable JPA Auditing: Annotate your main application class with @EnableJpaAuditing.

3. -- Entity Class Setup for Auditing:

       Use the following annotations in your entity class:
   
       @CreatedDate: Marks the field that should hold the creation timestamp.
       @LastModifiedDate: Marks the field that should hold the last modified timestamp.
       @CreatedBy: Marks the field that should hold the user who created the entity.
       @LastModifiedBy: Marks the field that should hold the user who last modified the entity.
4. Setting the AuditorAware Interface: Implement the AuditorAware interface to provide the current user's information, typically by fetching it from the security context.

5. Auditing in Action: Once configured, Spring Data JPA will automatically populate the auditing fields when an entity is created or updated.

-- Example Microservice
Here’s an example of a simple Spring Boot microservice that manages user accounts and tracks auditing information:

1. Create a Spring Boot Project: Initialize a Spring Boot project with necessary dependencies.

2. Enable Auditing: Add @EnableJpaAuditing to your main application class.

3. Create an Entity: Define a User entity like this:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;

    // Getters and Setters
}
```
4. Implement AuditorAware:

```java
@Component
public class SpringSecurityAuditorAware implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        // Fetch the current user from the security context
        return Optional.of(SecurityContextHolder.getContext().getAuthentication().getName());
    }
}
```
5. Create a Repository:

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

6. Service and Controller: Create a service to handle user operations and a controller to expose RESTful endpoints.

7. Testing: Create and update users via the API and verify that the auditing fields are populated correctly.


# Example of Microservice

# 1. Create a Simple User Microservice
--- Let’s first set up a simple microservice that will handle user entities.

    Step 1: Setting up Spring Boot Microservice
    First, make sure you have a Spring Boot project with the necessary dependencies in your pom.xml:

```xml
<dependencies>
    <!-- Spring Boot Dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- For H2 Database (for testing) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```
# 2. Enabling Auditing
Next, enable JPA auditing by adding @EnableJpaAuditing to your main application class.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@SpringBootApplication
@EnableJpaAuditing
public class UserMicroserviceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserMicroserviceApplication.class, args);
    }
}
```
# 3. Create an Entity with Auditing
We will create a User entity that includes auditing fields such as createdBy, lastModifiedBy, createdDate, and lastModifiedDate.

```java
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;

    @CreatedBy
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    // Getters and Setters
}
```
This User entity will have auditing capabilities for tracking who created and last modified the record, along with timestamps.

# 4. Implement the AuditorAware Interface
To populate the createdBy and lastModifiedBy fields, we need to create a class that implements AuditorAware. Typically, this retrieves the currently logged-in user. For simplicity, let’s hardcode a user for now.

```java
import org.springframework.data.domain.AuditorAware;
import java.util.Optional;

public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // In real applications, fetch the username from Spring Security or another context
        return Optional.of("AdminUser"); // Simulating a user for the example
    }
}
```
Then register the AuditorAware bean in your configuration class:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorAware")
public class AuditingConfig {

    @Bean
    public AuditorAware<String> auditorAware() {
        return new AuditorAwareImpl();
    }
}
```
5. Create a Repository
Now, let’s create a UserRepository interface for basic CRUD operations.

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

# 6. Service and Controller for the Microservice
To expose this functionality as a microservice, let’s create a UserService and a REST controller to manage users.

# UserService
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User updateUser(Long id, User updatedUser) {
        User user = userRepository.findById(id).orElseThrow();
        user.setUsername(updatedUser.getUsername());
        user.setEmail(updatedUser.getEmail());
        return userRepository.save(user);
    }
}
```
# UserController
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.updateUser(id, user);
    }
}
```
7. Testing the Auditing Feature
Now, when you create or update a user, the auditing fields will automatically be populated with the created/modified dates and user.

Example: Creating a User
Send a POST request to create a new user:

```json
POST /api/users
{
  "username": "rohit",
  "email": "rohit@example.com"
}
```
The createdBy, createdDate, and lastModifiedBy fields will automatically be set:

```json
{
  "id": 1,
  "username": "rohit",
  "email": "rohit@example.com",
  "createdBy": "AdminUser",
  "lastModifiedBy": "AdminUser",
  "createdDate": "2024-10-07T12:30:45",
  "lastModifiedDate": "2024-10-07T12:30:45"
}
```
# 8. Conclusion
By setting up this microservice, we've demonstrated how Spring Auditing can automatically handle created and modified timestamps and user details. This feature is particularly useful in microservices where multiple users might be interacting with the system, and audit trails are crucial for accountability and debugging.


# Hibernate Envers
To enable auditing in a Spring Boot microservice using Hibernate Envers, we can track entity changes across multiple revisions. Hibernate Envers is a powerful extension that helps keep a history of all changes made to entities in the database. It automatically stores revisions of each entity and allows you to query these revisions later.

Let’s enhance the User microservice with Hibernate Envers to capture and audit changes.

Steps to Enable Hibernate Envers in a Spring Boot Microservice
# 1. Add Dependencies
To enable Envers, we need to add the necessary Hibernate Envers dependency to the project. In your pom.xml, add:

```xml

<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-envers</artifactId>
</dependency>
```
For Gradle:

```gradle

implementation 'org.hibernate.orm:hibernate-envers'
```

# 2. Enable Envers Auditing
Once the dependency is added, Spring Boot automatically configures Hibernate Envers for your JPA entities. However, you need to annotate your entity to specify that you want to track revisions.

In the User entity, you can annotate the class with @Audited to enable auditing.

```java
import org.hibernate.envers.Audited;
import javax.persistence.*;

@Entity
@Audited // Enables Envers auditing
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;

    // No need to manually track created/modified fields
    // Envers will handle this via the revisions

    // Getters and Setters
}
```
With this, Envers will start auditing all changes made to the User entity.

# --- 3. Database Table Setup
     Once Envers is enabled, it creates additional tables in the database to store revisions of the entities. Typically, Envers creates two types of tables:

         Audit Table for Each Entity: Envers will create an audit table for each entity. For example, for the User entity, a table named User_AUD will be created.
         REVINFO Table: This stores revision metadata, such as the revision number and timestamp.
# --- 4. Service and Controller Modifications
     No changes are needed in the service or controller for basic CRUD operations. Hibernate Envers automatically tracks all changes and stores revisions whenever an entity is created, updated, or deleted.

# 5. Querying Entity Revisions
To retrieve revisions of an entity, we can use the AuditReader provided by Envers. Let’s add a method to the UserService to fetch a specific revision of a user.

```java
import org.hibernate.envers.AuditReader;
import org.hibernate.envers.AuditReaderFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EntityManager entityManager;

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User updateUser(Long id, User updatedUser) {
        User user = userRepository.findById(id).orElseThrow();
        user.setUsername(updatedUser.getUsername());
        user.setEmail(updatedUser.getEmail());
        return userRepository.save(user);
    }

    @Transactional
    public User getUserRevision(Long userId, int revisionNumber) {
        AuditReader auditReader = AuditReaderFactory.get(entityManager);
        return auditReader.find(User.class, userId, revisionNumber);
    }
}
```
In the method getUserRevision, we use AuditReader from Envers to fetch a specific revision of the User entity.

# 6. Expose Revision Retrieval via REST API
Now, let’s expose an API endpoint to fetch a specific revision of a user:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.updateUser(id, user);
    }

    @GetMapping("/{id}/revisions/{revisionNumber}")
    public User getUserRevision(@PathVariable Long id, @PathVariable int revisionNumber) {
        return userService.getUserRevision(id, revisionNumber);
    }
}
```
In the getUserRevision API, we fetch a specific revision of a user by its ID and the revision number.

# 7. Testing Hibernate Envers
Now you can test the auditing functionality by:


1. Creating a new user.
2. Updating the user.
3. Fetching specific revisions of the user using the /api/users/{id}/revisions/{revisionNumber} endpoint.
# Example
1. Create User:

POST /api/users with:

```json
{
  "username": "rohit",
  "email": "rohit@example.com"
}
```

2. Update User:
```json
PUT /api/users/{id} with:

```json
{
  "username": "rohit_updated",
  "email": "rohit_updated@example.com"
}
```
3. Get User Revision:
```json
GET /api/users/{id}/revisions/{revisionNumber} to fetch a specific revision.
```
# For example, if you want to get the first revision:

```
GET /api/users/1/revisions/1
```
# 8. View Audit Table in Database
```
Hibernate Envers automatically stores revision data in tables. For the User entity, the following audit table is created:

User_AUD: Stores the revision history of the User entity. Each row in this table represents a revision with the state of the entity at that time.
In the audit table, you’ll find fields such as:

-- REV: The revision number.
-- REVTYPE: Indicates whether the revision represents an insert (0), update (1), or delete (2).
-- username, email: The fields of the User entity at the time of the revision.
```
