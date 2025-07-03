# Code Refactoring Analysis

## **Hotspot #4: Complex Method in `fromInput`**
- **Issue:** High cyclomatic complexity (cc = 22)
- **Severity:** Brain Method - Complex Method - Long Method
- **Problems:**
  - Multiple switch cases with nested conditionals and repeated logic
  - Handles parsing, validation, and argument construction in one method
  - Hard to extend for new commands or argument patterns
  - Difficult to test and maintain

---

## **Before Refactoring**
```java
public static Command fromInput(String input, String robotName) {
    String[] tokens = input.trim().split(" ");
    String command = tokens[0];

    if (tokens.length == 0 || tokens[0].isEmpty()) {
        throw new IllegalArgumentException("Invalid or empty command ");
    }

    String robot = "";
    String[] args = new String[]{};

    switch (command.toLowerCase()) {
        case "forward":
        case "back":
            if (tokens.length >= 3) {
                robot = tokens[1];
                args = new String[]{tokens[0], tokens[1], tokens[2]}; // direction, robot, steps
            } else if (robotName != null) {
                robot = robotName;
                args =  new String[]{tokens[0], robotName, tokens[1]};
            }
            break;
        case "turn":
            robot = tokens.length >= 3 ? tokens[1] : robotName;
            args = tokens.length >= 3 ? new String[]{tokens[2]} : new String[]{tokens[1]}; // only need turn direction
            break;
        case "state":
        case "look":
        case "orientation":
        case "fire":
        case "repair":
        case "reload":
        case "off":
            robot = tokens.length > 1 ? tokens[1] : robotName;
            args = new String[]{};
            break;
        case "launch":
            robot = tokens.length >= 3 ? tokens[2] :robotName;
            String robotType = tokens.length >= 2 ? tokens[1] : "";
            args = new String[]{robotType};
            break;
    }

    return fromJSON(new JSONObject()
            .put("robot", robot)
            .put("command", command)
            .put("arguments", new JSONArray(Arrays.asList(args))));
}
```
---


## Refactoring Strategy

### **Step 1: Extract Command Parsing Logic**
- Move argument extraction for each command into dedicated helper methods
- Use a dispatcher to select the correct parsing logic for each command

### **Step 2: Simplify Main Method**
- Validate input early
- Delegate argument parsing to helpers
- Build the JSON and return

---

## Refactored Code

```java
public static Command fromInput(String input, String robotName) {
    //TODO: Impkement refactoring.
}

```

---


## **Benefits of Refactoring**

### **Maintainability**
- Each command's parsing logic is isolated and easy to update
- Adding new commands or argument patterns is straightforward

### **Readability**
- Main method is concise and linear
- No deep nesting or repeated logic

### **Extensibility**
- Easy to add new command types or argument rules by extending helper methods

### **Code Quality**
- Cyclomatic complexity is reduced
- Follows Single Responsibility Principle
- Easier to test and debug