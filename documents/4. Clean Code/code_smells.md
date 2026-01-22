# Code Smells: Identification and Refactoring

> **Repository:** [Identifying-and-Fixing-Code-Smells-test](https://github.com/yjfvictor/Identifying-and-Fixing-Code-Smells-test)

## Introduction

Code smells are indicators of potential problems in code that, while not necessarily bugs, suggest that the code may be difficult to maintain, understand, or extend. This document reflects on the code smells identified in the [original code](https://github.com/yjfvictor/Identifying-and-Fixing-Code-Smells-test/blob/40ddb9d1181ba5838f3091fa62190a89dc3c6b5d/main.js) and how refactoring them in the [refactored code](https://github.com/yjfvictor/Identifying-and-Fixing-Code-Smells-test/blob/7cf559d2ac2fbc704927e2a013c3f3add25e4f84/main.js) (from [Pull Request #1](https://github.com/yjfvictor/Identifying-and-Fixing-Code-Smells-test/pull/1)) improved code quality.

---

## Code Smells Identified

### 1. Magic Numbers & Strings

**Problem:** Hardcoded values scattered throughout the code make it difficult to understand intent and maintain consistency. For example, using `0.03`, `0.01`, `0.035` for payment fees or `'premium'`, `'basic'` for subscription plans directly in code.

**Impact:**

- Changes require finding and updating multiple locations
- Risk of typos and inconsistencies
- Code intent is unclear without context

**Solution:** Extracted all magic numbers and strings into named constants at the top of the file. This makes the code self-documenting and ensures consistency.

---

### 2. Long Functions

**Problem:** The `calculateTotal` function in the [original code](https://github.com/yjfvictor/Identifying-and-Fixing-Code-Smells-test/blob/40ddb9d1181ba5838f3091fa62190a89dc3c6b5d/main.js) was doing too many things: calculating subtotal, applying discounts, calculating tax, adding shipping, and converting currency—all in one function.

**Impact:**

- Difficult to test individual parts
- Hard to understand the overall flow
- Changes to one part risk breaking others
- Difficult to reuse logic

**Solution:** Broke down the function into smaller, focused methods in a `PriceCalculator` class. Each method has a single responsibility, making the code easier to test, understand, and maintain.

---

### 3. Duplicate Code

**Problem:** Three nearly identical validation functions (`validateUserInput`, `validateProductName`, `validateOrderDescription`) were copy-pasted with only minor differences.

**Impact:**

- Bug fixes must be applied in multiple places
- Inconsistencies can arise between duplicates
- Increases maintenance burden
- Violates DRY (Don't Repeat Yourself) principle

**Solution:** Created a single `isValidString` function with a configurable `maxLength` parameter. This eliminates duplication and ensures consistent validation logic across the codebase.

---

### 4. Large Classes (God Objects)

**Problem:** The `GodObject` class was handling user management, order processing, product management, report generation, and notifications—far too many responsibilities.

**Impact:**

- Difficult to understand what the class does
- Changes in one area can affect unrelated functionality
- Hard to test individual features
- Violates Single Responsibility Principle

**Solution:** Split into focused classes: `UserManager`, `ProductManager`, `OrderProcessor`, `ReportGenerator`, and `NotificationService`. Each class now has a clear, single responsibility, making the codebase more modular and maintainable.

---

### 5. Deeply Nested Conditionals

**Problem:** The `getUserStatus` function had multiple levels of nested if-else statements, creating a "pyramid of doom" that was hard to follow.

**Impact:**

- Difficult to understand the logic flow
- Easy to introduce bugs when adding conditions
- Hard to test all code paths
- Poor readability

**Solution:** Used early returns and guard clauses to flatten the conditional structure. This makes the code read more linearly and reduces cognitive load when understanding the logic.

---

### 6. Commented-Out Code

**Problem:** Commented-out code in the `sendNotification` method (though minimal in our example) suggests uncertainty and clutters the codebase.

**Impact:** The following issues arise:

- Creates confusion about whether code is needed
- Adds noise to code reviews
- Suggests incomplete refactoring
- Can mislead future developers

**Solution:** Removed commented-out code and replaced it with a proper `clearOldNotifications` method that handles the notification limit logic explicitly.

---

### 7. Inconsistent Naming

**Problem:** Variable names like `yr`, `mth`, `dy` in `formatDate` and abbreviations throughout the code made it unclear what variables represent.

**Impact:** The following issues arise:

- Reduces code readability
- Makes code harder to understand for new developers
- Can lead to misunderstandings and bugs
- Violates self-documenting code principles

**Solution:** Used descriptive, consistent naming conventions: `year`, `month`, `day`, `monthDifference`, `calculateAgeFromBirthDate`, etc. Names now clearly communicate their purpose.

---

## How Refactoring Improved Code Quality

### Readability

The refactored code is significantly more readable. Constants with meaningful names replace magic numbers, descriptive function names explain intent, and the flattened conditional structure is easier to follow. A developer can now understand what the code does without needing extensive comments.

### Maintainability

By breaking down large functions and classes into smaller, focused units, changes can be made in isolation. If payment fee logic needs updating, only the `PAYMENT_FEES` constant needs changing. If order processing logic changes, only the `OrderProcessor` class is affected. This reduces the risk of introducing bugs when making changes.

### Testability

Smaller functions and classes are much easier to test. Each method in `PriceCalculator` can be tested independently. The separated classes can be tested in isolation with mock dependencies. This makes it easier to achieve high test coverage and catch bugs early.

### Reusability

The extracted utility functions like `isValidString` and `isValidEmail` can be reused throughout the codebase. The `PriceCalculator` methods can be used independently for different calculation scenarios. This reduces code duplication and ensures consistent behaviour.

### Extensibility

Adding new features is now straightforward. To add a new payment method, simply add it to `PAYMENT_FEES`. To add a new report type, add a method to `ReportGenerator`. The modular structure makes it easy to extend functionality without modifying existing, working code.

---

## How Avoiding Code Smells Makes Debugging Easier

### Clearer Error Messages

When code is well-structured with descriptive names, error messages and stack traces are more meaningful. Instead of "Error in line 45 of main.js", you get "Error in OrderProcessor.processOrder", making it immediately clear where the problem is.

### Easier Isolation

With separated concerns, bugs can be isolated to specific classes or functions. If orders aren't processing correctly, you know to look in `OrderProcessor`. If validation is failing, you check `isValidString`. This narrows down the search space significantly.

### Better Logging Points

Well-structured code makes it easier to add logging at appropriate points. You can log at the entry and exit of each method, making it easier to trace execution flow and identify where things go wrong.

### Reduced Side Effects

When functions and classes have single responsibilities, changes are less likely to have unintended side effects. A bug fix in `UserManager` won't accidentally break order processing, because they're now separate concerns.

### Easier Code Review

Clean, well-structured code is easier to review. Reviewers can focus on logic rather than trying to understand what the code is supposed to do. This helps catch bugs before they reach production.

### Faster Onboarding

New developers can understand and work with clean code much faster. They don't need to spend time deciphering magic numbers or untangling nested conditionals. This means they can start contributing and debugging sooner.

---

## Conclusion

Code smells are warning signs that should not be ignored. While they may not cause immediate problems, they accumulate technical debt that makes code harder to maintain, debug, and extend over time. The refactoring exercise demonstrates that investing time in cleaning up code pays dividends in improved readability, maintainability, and developer productivity. By recognising and addressing code smells early, we can build more robust and sustainable software systems.
