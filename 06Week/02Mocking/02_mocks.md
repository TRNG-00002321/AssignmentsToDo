# Your First Mock - Testing UserService

## The Scenario

You need to test a `UserService` class that depends on a `UserRepository` (database access) and an `EmailClient` (sends emails). Without mocks, you'd need a real database and email server! Mocks let you:
1. Test the service logic in isolation
2. Control what the dependencies return
3. Run tests without external systems

## Core Tasks

### Task 1: Set Up Mockito 

Create your test class with Mockito annotations:

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)  // Enables Mockito annotations
class UserServiceTest {
    
    @Mock
    private UserRepository repository;  // Mock the dependency
    
    @Mock
    private EmailClient emailClient;  // Mock the dependency
    
    @InjectMocks
    private UserService userService;  // Inject mocks automatically
}
```

### Task 2: Test getUser() with Mock 

Write a test that:
1. Configures the mock repository to return a specific user
2. Calls `userService.getUser(id)`
3. Asserts the correct user is returned

```java
@Test
void getUser_existingUser_returnsUser() {
    // Arrange: Configure the mock
    User expectedUser = new User("John", "john@test.com");
    expectedUser.setId(1L);
    
    when(repository.findById(1L)).thenReturn(Optional.of(expectedUser));
    
    // Act: Call the method under test
    User actualUser = userService.getUser(1L);
    
    // Assert: Verify the result
    assertEquals(expectedUser, actualUser);
    assertEquals("John", actualUser.getName());
}
```

### Task 3: Test Exception Scenario 

Test that `getUser()` throws an exception when the user doesn't exist:

```java
@Test
void getUser_nonExistentUser_throwsException() {
    // Arrange: Mock returns empty Optional
    when(repository.findById(999L)).thenReturn(Optional.empty());
    
    // Act & Assert
    assertThrows(UserService.UserNotFoundException.class, () -> {
        userService.getUser(999L);
    });
}
```

### Task 4: Test createUser() 

Write tests for the `createUser()` method:

**Test 1:** Successful user creation
- Mock `existsByEmail()` to return `false`
- Mock `save()` to return the saved user
- Verify the user is returned with an ID

**Test 2:** Duplicate email rejection
- Mock `existsByEmail()` to return `true`
- Assert `DuplicateUserException` is thrown

**Test 3:** Invalid input rejection
- Test with null name
- Test with invalid email
- Assert `IllegalArgumentException` is thrown

### Task 5: Write Your Own Tests 

Without guidance, write tests for:
- `getActiveUsers()`
- `getUserCount()`

## Starter Code

```java
// UserRepository.java - Interface to mock
public interface UserRepository {
    Optional<User> findById(Long id);
    User save(User user);
    void deleteById(Long id);
    List<User> findAllActive();
    boolean existsByEmail(String email);
    long count();
}

// EmailClient.java - Interface to mock
public interface EmailClient {
    void send(String to, String subject, String body);
}

// User.java
public class User {
    private Long id;
    private String name;
    private String email;
    private boolean active = true;
    
    // Constructors, getters, setters
}
```

## Key Mockito Concepts

### @Mock vs @InjectMocks

| Annotation | Purpose |
|------------|---------|
| `@Mock` | Creates a mock object |
| `@InjectMocks` | Creates real object, injects mocks |

### Basic Stubbing

```java
// Return a value
when(mock.method()).thenReturn(value);

// Return Optional
when(repository.findById(1L)).thenReturn(Optional.of(user));
when(repository.findById(999L)).thenReturn(Optional.empty());

// Return a list
when(repository.findAllActive()).thenReturn(List.of(user1, user2));

// Return primitive
when(repository.count()).thenReturn(42L);

// Return boolean
when(repository.existsByEmail("test@test.com")).thenReturn(true);
```

## Hints

<details>
<summary>Hint: Creating Test Users</summary>

```java
private User createTestUser(Long id, String name, String email) {
    User user = new User(name, email);
    user.setId(id);
    return user;
}
```
</details>

<details>
<summary>Hint: Testing createUser Success</summary>

```java
@Test
void createUser_validData_returnsCreatedUser() {
    // Arrange
    when(repository.existsByEmail("john@test.com")).thenReturn(false);
    when(repository.save(any(User.class))).thenAnswer(invocation -> {
        User u = invocation.getArgument(0);
        u.setId(1L);
        return u;
    });
    
    // Act
    User created = userService.createUser("John", "john@test.com");
    
    // Assert
    assertNotNull(created.getId());
    assertEquals("John", created.getName());
}
```
</details>

## Common Mistakes

1. **Forgetting `@ExtendWith`** - Mocks won't be initialized
2. **Wrong stubbing syntax** - It's `when(mock.method())`, not `when(mock).method()`
3. **Not stubbing** - Unstubbed methods return default (null, 0, false)
4. **Over-mocking** - Don't mock the class under test!




