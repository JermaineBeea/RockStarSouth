# Code Refactoring Analysis

## **Hotspot: Bumpy Road in `displayDirectionalCross`**
- [Link to Hotspot](https://codescene.wethinkco.de/192/analyses/2118/code/hotspots/biomarkers?name=brownfields_robot_worlds_5%2Fsrc%2Fmain%2Fjava%2Fza%2Fco%2Fwethinkcode%2Frobots%2Fhandlers%2FCommandHandler.java)
- **Issue:** Deeply nested conditional logic ("Bumpy Road")
- **Complexity:** 9 (Very High)
- **Problems:**
  - Multiple levels of nested conditionals for grid population, obstacle placement, and robot rendering
  - Several responsibilities (grid setup, cross drawing, obstacle/robot placement, output formatting) handled in a single function
  - Difficult to follow logic due to repeated checks and index calculations
  - Hard to test and maintain; adding new features or fixing bugs risks breaking unrelated logic

---

## Refactoring Strategy

### **Step 1: Identify and Extract Responsibilities**
- Separate grid initialization, cross drawing, obstacle placement, robot placement, and output formatting into distinct helper methods
- Each helper should encapsulate a single responsibility, reducing nesting and improving readability

### **Step 2: Replace Nested Conditionals with Guard Clauses**
- Use guard clauses to handle out-of-bounds and irrelevant cases early
- Flatten the structure to make the main logic more linear and understandable

---

## Refactored Code

```java
public String displayDirectionalCross(Robot robot, int maxDistance) {
    //TODO: Write refactored code.
}
```

### **Alternative Approach: [Optional Alternative, e.g., Parameterized Tests]**
```java
[Insert alternative code structure or pseudocode here]
```

---

## **Benefits of Refactoring**

### **Maintainability**
- Each logical step is encapsulated in a well-named helper method
- Changes to grid, cross, or placement logic are isolated and easier to test

### **Readability**
- Linear, top-down flow in the main function
- Reduced cognitive load from deep nesting and repeated index calculations

### **Extensibility**
- Easy to add new features (e.g., diagonal lines, new symbols) by extending or adding helpers
- Supports future refactoring and testing

### **Code Quality**
- Eliminates "Bumpy Road" anti-pattern
- Follows Single Responsibility Principle
- Reduces risk of feature entanglement and state management bugs

---

## **Before Refactoring**
```java
public String displayDirectionalCross(Robot robot, int maxDistance) {
    StringBuilder sb = new StringBuilder();

    int robotX = robot.getX();
    int robotY = robot.getY();

    // Calculate the actual limits based on world bounds and maxDistance
    int minX = Math.max(robotX - maxDistance, -halfWidth);
    int maxX = Math.min(robotX + maxDistance, halfWidth);
    int minY = Math.max(robotY - maxDistance, -halfHeight);
    int maxY = Math.min(robotY + maxDistance, halfHeight);

    // Grid dimensions
    int width = maxX - minX + 1;
    int height = maxY - minY + 1;

    String[][] grid = new String[height][width];

    // Fill grid with spaces initially
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            grid[i][j] = "  ";
        }
    }

    // Fill cross lines with base tile ◾️
    // The vertical line (X == robotX)
    if (robotX >= minX && robotX <= maxX) {
        int col = robotX - minX;
        for (int row = 0; row < height; row++) {
            grid[row][col] = "◾️";
        }
    }

    // The horizontal line (Y == robotY)
    if (robotY >= minY && robotY <= maxY) {
        int row = maxY - robotY; // Y decreases down rows
        for (int col = 0; col < width; col++) {
            grid[row][col] = "◾️";
        }
    }

    // Place obstacles on cross lines within bounds
    for (Obstacle obstacle : obstacles) {
        for (int y = obstacle.getY(); y < obstacle.getMaxY(); y++) {
            for (int x = obstacle.getX(); x < obstacle.getMaxX(); x++) {
                if (x < minX || x > maxX || y < minY || y > maxY) continue;

                if (x == robotX) {
                    int row = maxY - y;
                    int col = x - minX;
                    grid[row][col] = obstacle.type().getSymbol();
                } else if (y == robotY) {
                    int row = maxY - y;
                    int col = x - minX;
                    grid[row][col] = obstacle.type().getSymbol();
                }
            }
        }
    }

    // Place other robots on cross lines within bounds
    for (Robot other : robots) {
        if (other.equals(robot)) continue;

        int x = other.getX();
        int y = other.getY();

        if (x < minX || x > maxX || y < minY || y > maxY) continue;

        if (x == robotX || y == robotY) {
            int row = maxY - y;
            int col = x - minX;
            grid[row][col] = "🤖";
        }
    }

    // Place current robot
    if (robotX >= minX && robotX <= maxX && robotY >= minY && robotY <= maxY) {
        int row = maxY - robotY;
        int col = robotX - minX;
        grid[row][col] = "🤖";
    }

    // Build output string
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            sb.append(grid[i][j]).append(" ");
        }
        sb.append("\n");
    }

    return sb.toString();
}
```

