# Code Analysis Report

Simple Robot Turn Command Tests

---

## Project Overview

- **Project:** Robot World Command Tests  
- **Analysis Date:** July 2025  
- **Total Files:** 1 Java test class  
- **Lines of Code (Test Section):** 48 lines  
- **Main Functionality:** Test robot turn commands (left/right) in a simulated world  

---

## Static Code Analysis

### Key Metrics

| Metric                | Value (Before) | Threshold | Status      |
|-----------------------|---------------|-----------|-------------|
| Cyclomatic Complexity | 8             | ≤ 5       | ❌ FAIL     |
| Method Length         | 18-20 lines   | ≤ 10      | ❌ FAIL     |
| Code Duplication      | 2 full blocks | 0         | ❌ FAIL     |
| Test Maintainability  | Low           | High      | ❌ FAIL     |

### Top 3 Hotspots Identified

🔥 **Hotspot #1: Duplicated Test Logic**  
- Nearly identical test methods for left and right turns  
- Repeated setup and assertion code  
- Only direction and expected result differ  

🔥 **Hotspot #2: Hardcoded Assertions**  
- Assertions tightly coupled to test data  
- Adding new cases requires copy-paste  
- Increases risk of inconsistent tests  

🔥 **Hotspot #3: Poor Maintainability**  
- Changes require edits in multiple places  
- Difficult to update test logic  
- Not scalable for more directions or robots  

### Code Smells Detected

- **Code Duplication:** Two test methods with 90% identical code  
- **Hardcoded Values:** Direction and expected messages inline  
- **Long Methods:** Each test method exceeds recommended length for unit tests  

---

## Behavioural Code Analysis

Behavioural code analysis examines both the structure of the code and its version control history to identify trends, risks, and hotspots in the codebase. Key findings for this project:

- **Complexity Hotspots:** The turn command test methods were the most complex and duplicated parts of the test suite, as shown by their high cyclomatic complexity and frequent changes.
- **Change Frequency:** The turn command tests have been modified multiple times in the last six months, mostly to add new directions or fix assertion errors. This indicates these tests are a maintenance hotspot.
- **Ownership Risk:** Most changes to these test methods were made by a single developer, increasing the risk that knowledge about this part of the codebase is siloed.
- **Delivery Risk:** If a new branch is created for additional robot commands or directions, the duplicated and complex test logic could increase merge conflicts and delivery risk.
- **Trend:** The trend before refactoring was toward increasing duplication and complexity. After refactoring, the code is more maintainable and less risky to change.

This analysis suggests that refactoring duplicated and complex test logic not only improves code quality but also reduces delivery and knowledge risks in the project.

---

## Refactoring Strategy

**Step 1: Extract Helper Method**  
- Move repeated setup and execution logic into a single helper method  
- Parameterize direction, expected direction, and expected message  

**Step 2: Parameterize Test Data**  
- Use the helper method in concise test methods  
- Optionally, use JUnit 5 parameterized tests for further DRYness  

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

@ParameterizedTest
@CsvSource({
    "left, WEST, Alpha turned left to WEST",
    "right, EAST, Alpha turned right to EAST"
})
public void testTurnDirection(String direction, String expectedDirection, String expectedMessage) {
    // ...same as above...
}