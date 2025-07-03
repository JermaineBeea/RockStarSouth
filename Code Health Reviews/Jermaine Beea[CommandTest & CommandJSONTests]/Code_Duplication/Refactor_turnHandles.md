# Code Refactoring Analysis

## **Hotspot: Command Test**
- **Issue:** Duplicated code and hardcoded test logic
- **Complexity:** 8 (High)
- **Problems:** 
  - Nearly identical test methods with only minor differences in direction and expected results
  - Repeated setup code for robot creation, world initialization, and command execution
  - Hardcoded assertions that don't match the parameterized input
  - Poor maintainability due to code duplication

---

## **Before Refactoring**
```java
@Test
public void testHandleTurnLeft() {
    String clientId = "client-xyz";
    Robot robot1 = new Robot("Alpha", "tank");
    World world = new World(10, 10);
    LaunchCommand command = new LaunchCommand(robot1, new String[]{"tank"});

    world.execute(command, clientId, response -> {
        assertTrue(response.isOKResponse());
        Robot launchedRobot = world.getRobots().getFirst();

        TurnCommand turnCommand = new TurnCommand(launchedRobot, new String[]{"left"});

        world.execute(turnCommand, clientId, turnResponse -> {
            assertTrue(turnResponse.isOKResponse());
            assertEquals("Alpha turned left to WEST", turnResponse.getMessage());
            assertEquals("WEST", turnResponse.object.getJSONObject("state").getString("direction"));
        });
    });
}

@Test
public void testHandleTurnRight() {
    String clientId = "client-xyz";
    Robot robot1 = new Robot("Alpha", "tank");
    World world = new World(10, 10);
    LaunchCommand command = new LaunchCommand(robot1, new String[]{"tank"});

    world.execute(command, clientId, response -> {
        assertTrue(response.isOKResponse());
        Robot launchedRobot = world.getRobots().getFirst();

        TurnCommand turnCommand = new TurnCommand(launchedRobot, new String[]{"right"});

        world.execute(turnCommand, clientId, turnResponse -> {
            assertTrue(turnResponse.isOKResponse());
            assertEquals("Alpha turned right to EAST", turnResponse.getMessage());
            assertEquals("EAST", turnResponse.object.getJSONObject("state").getString("direction"));
        });
    });
}
```
---


## Refactoring Strategy

### **Step 1: Extract Common Setup Logic**
- Create a helper method to handle robot setup, world creation, and command execution
- Parameterize the direction and expected outcomes
- Reduce code duplication by centralizing common operations

### **Step 2: Implement Data-Driven Testing**
- Use a parameterized helper method that accepts direction and expected results
- Create focused test methods that call the helper with specific parameters
- Ensure assertions match the actual test parameters

---

## Refactored Code

```java
public void testTurnDirection(String direction, String expectedDirection, String expectedMessage) {
    String clientId = "client-xyz";
    Robot robot1 = new Robot("Alpha", "tank");
    World world = new World(10, 10);
    LaunchCommand command = new LaunchCommand(robot1, new String[]{"tank"});

    world.execute(command, clientId, response -> {
        assertTrue(response.isOKResponse());
        Robot launchedRobot = world.getRobots().getFirst();

        TurnCommand turnCommand = new TurnCommand(launchedRobot, new String[]{direction});

        world.execute(turnCommand, clientId, turnResponse -> {
            assertTrue(turnResponse.isOKResponse());
            assertEquals(expectedMessage, turnResponse.getMessage());
            assertEquals(expectedDirection, turnResponse.object.getJSONObject("state").getString("direction"));
        });
    });
}

@Test
public void testHandleTurnLeft() {
    testTurnDirection("left", "WEST", "Alpha turned left to WEST");
}

@Test
public void testHandleTurnRight() {
    testTurnDirection("right", "EAST", "Alpha turned right to EAST");
}
```

### **Alternative Approach: JUnit 5 Parameterized Tests**
```java
@ParameterizedTest
@CsvSource({
    "left, WEST, Alpha turned left to WEST",
    "right, EAST, Alpha turned right to EAST"
})
public void testTurnDirection(String direction, String expectedDirection, String expectedMessage) {
    String clientId = "client-xyz";
    Robot robot1 = new Robot("Alpha", "tank");
    World world = new World(10, 10);
    LaunchCommand command = new LaunchCommand(robot1, new String[]{"tank"});

    world.execute(command, clientId, response -> {
        assertTrue(response.isOKResponse());
        Robot launchedRobot = world.getRobots().getFirst();

        TurnCommand turnCommand = new TurnCommand(launchedRobot, new String[]{direction});

        world.execute(turnCommand, clientId, turnResponse -> {
            assertTrue(turnResponse.isOKResponse());
            assertEquals(expectedMessage, turnResponse.getMessage());
            assertEquals(expectedDirection, turnResponse.object.getJSONObject("state").getString("direction"));
        });
    });
}
```

---

## **Benefits of Refactoring**

### **Maintainability**
- Single point of change for test logic modifications
- Easier to add new direction test cases
- Reduced risk of inconsistencies between similar tests

### **Readability**
- Clear separation between test data and test logic
- Self-documenting test method names
- Parameterized approach makes test intentions obvious

### **Extensibility**
- Easy to add additional turn directions (e.g., "up", "down" for 3D movement)
- Simple to modify expected behavior across all turn tests
- Supports data-driven testing approaches

### **Code Quality**
- Eliminates duplicate code (DRY principle)
- Reduces maintenance burden
- Improves test coverage consistency

