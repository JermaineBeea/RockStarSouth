A **code smell** is a term used in software engineering to describe **any characteristic in the source code that *might* indicate a deeper problem**.

It doesn’t necessarily mean that the code is broken or will cause bugs immediately, but it often signals underlying issues that could make the code:

* Harder to understand
* More difficult to maintain or extend
* More error-prone in the future

Code smells are typically about **design and structure**, not about syntax errors or compiler errors.

### 🔧 **Examples of common code smells:**

| Code Smell          | What it looks like                                      | Why it’s a problem                                 |
| ------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| Long Method         | Methods that do too much and are hundreds of lines long | Hard to read, test, and reuse                      |
| Large Class         | Classes that try to do everything                       | Violates single responsibility, harder to maintain |
| Duplicate Code      | The same code appears in multiple places                | Harder to change and increases risk of bugs        |
| Feature Envy        | One class frequently accesses the data of another       | Suggests the behavior might belong elsewhere       |
| Long Parameter List | Methods with many parameters                            | Makes the method harder to understand and call     |
| God Object          | One object controls too many parts of the system        | Creates tight coupling and fragility               |

---

### 📙 **Origin:**

The term *code smell* was popularized by **Martin Fowler** in his book *Refactoring: Improving the Design of Existing Code*.
It’s meant to be a sign that you should *look closer* at a part of the code and see if it can be improved, usually by **refactoring**.

If you'd like, I could also list more smells, or show real code examples!
