# Code Refactoring Analysis

## **Hotspot: Bumpy Road in `handleFire`**
- **Issue:** Deeply nested conditional logic ("Bumpy Road")
- **Complexity:** High
- **Problems:**
  - Multiple early returns and repeated error handling
  - Several responsibilities (validation, direction/range calculation, hit detection, response building) handled in a single function
  - Difficult to follow due to repeated checks and state changes
  - Hard to test and maintain; adding new fire logic or robot types risks breaking unrelated logic

---

## Refactoring Strategy

### **Step 1: Identify and Extract Responsibilities**
- Separate validation, direction/range calculation, hit detection, and response construction into helper methods
- Each helper should encapsulate a single responsibility, reducing nesting and improving readability

### **Step 2: Use Guard Clauses for Validation**
- Use guard clauses to handle invalid states early and return immediately
- Flatten the structure to make the main logic more linear and understandable

---

## Refactored Code

```java
private void handleFire(Robot robot, CompletionHandler completionHandler) {
    //TODO: Refactor the function.
}
```

---

## **Before Refactoring**
```java
private void handleFire(Robot robot, CompletionHandler completionHandler) {
    robot = this.world.findRobot(robot.getName());

    if (robot == null) {
        completionHandler.onComplete(new Response("ERROR", "Could not find robot: " + robot.getName()));
        return;
    }

    if (robot.getShots() <= 0) {
        completionHandler.onComplete(new Response("ERROR", "You have no shots remaining."));
        return;
    }

    if (robot.status == Robot.RobotStatus.Reload) {
        completionHandler.onComplete(new Response("ERROR", robot.getName() + " is reloading and cannot fire"));
        return;
    }

    if (robot.status == Robot.RobotStatus.Repair) {
        completionHandler.onComplete(new Response("ERROR", robot.getName() + " is repairing and cannot fire"));
        return;
    }

    Position robotP = robot.getPosition();
    String direction = robot.orientation();

    int dx = 0;
    int dy = 0;

    switch (direction) {
        case "NORTH" -> dy = 1;
        case "SOUTH" -> dy = -1;
        case "EAST" -> dx = 1;
        case "WEST" -> dx = -1;
    }

    // Determine the range based on the robot type and remaining shots
    int range;
    if ("tank".equalsIgnoreCase(robot.getMake())) {
        range = switch (robot.getShots()) {
            case 3 -> 3;
            case 2 -> 4;
            default -> 5;
        };
    } else { // assuming it's a sniper
        range = switch (robot.getShots()) {
            case 10 -> 1;
            case 9 -> 2;
            case 8 -> 3;
            case 7 -> 4;
            case 6 -> 5;
            case 5 -> 6;
            case 4 -> 7;
            case 3 -> 8;
            case 2 -> 9;
            default -> 10;
        };
    }

    // Reduce the shot count
    robot.setShots(robot.getShots() - 1);
    Robot hitRobot = null;
    int distance = 0;

    // Check for hits
    for (int step = 1; step <= range; step++) {
        Position checkPos = new Position(robotP.getX() + step * dx, robotP.getY() + step * dy);

        for (Robot otherRobot : world.getRobots()) {
            if (!otherRobot.getName().equals(robot.getName()) && otherRobot.getPosition().equals(checkPos)) {
                hitRobot = otherRobot;
                break;
            }
        }

        if (hitRobot != null) {
            distance = step;
            break; // Stop checking after a hit
        }
    }

    // Handle the result of the shot
    if (hitRobot == null) {
        completionHandler.onComplete(new Response("OK", "You have missed 🥲!"));
        return;
    }

    // Apply damage
    hitRobot.takeHit(); // This reduces shield strength or kills the robot

    Response response;
    Response hitRobotResponse = new Response("OK", "I got hit");

    if (hitRobot.isDead()) {
       response = new Response("OK", "You have hit 💥 " + hitRobot.getName() + "! It is now destroyed.");
    } else {
        response = new Response("OK", "You have hit 💥 " + hitRobot.getName() + "! Remaining shield: " + hitRobot.getShields());
    }

    JSONObject data = new JSONObject();

    world.stateForRobot(hitRobot, hitRobotResponse);
    world.stateForRobot(robot, response);

    data.put("message", "Hit");
    data.put("distance", distance);
    data.put("robot", hitRobot.getName());
    data.put("state", hitRobotResponse.object.getJSONObject("state"));

    response.object.put("data", data);

    completionHandler.onComplete(response);
}
```

---

## **Benefits of Refactoring**

### **Maintainability**
- Each logical step is encapsulated in a well-named helper method
- Changes to validation, direction, or hit logic are isolated and easier to test

### **Readability**
- Linear, top-down flow in the main function
- Reduced cognitive load from deep nesting and repeated checks

### **Extensibility**
- Easy to add new robot types, fire logic, or error handling by extending or adding helpers
- Supports future refactoring and testing

### **Code Quality**
- Eliminates "Bumpy Road" anti-pattern
- Follows Single Responsibility Principle
- Reduces risk of feature entanglement and state management bugs