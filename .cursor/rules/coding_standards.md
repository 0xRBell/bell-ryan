---
description: Coding standards and style guidelines for all projects.
alwaysApply: true
---

## Readability & Clarity

### Write for Humans, Not Computers

- Prefer readable, maintainable code over clever or overly compact solutions
- Code should read like plain English from left to right
- Choose descriptive variable and function names that clearly convey purpose
- Avoid double negatives (e.g., `if not not_is_unsupported` → `if is_supported`)
- "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler
- "Everyone knows that debugging is twice as hard as writing a program in the first place. So if you're as clever as you can be when you write it, how will you ever debug it?" — Brian Kernighan

### Comments and Documentation

- When code cannot be self-documenting (complex algorithms, obscure commands), add clear comments explaining the logic
- Comments should explain _why_ something is done, not just _what_ is being done
- For complex shell commands or pipelines, break down each step in comments

**Example:**

```bash
# Filter processes: exclude header and known high-usage processes,
# sort by CPU usage descending, remove duplicates, return top N
ps -eo "pcpu comm" | grep -Evi '%CPU|known-process' | sort -rn | \
  sed 's/^[ ]*[^ ]*[ ]*//' | awk '!seen[$0]++' | head -"$numOutput"
```

## Control Flow & Complexity

### Use Guard Clauses and Early Returns

- Detect and handle edge cases and errors early in the function; place the 'happy path' last for readability
- Use guard clauses to exit early from functions when conditions are not met
- Limit nesting depth to no more than 3 levels (typically 3 tabs out)
- Extract deeply nested logic into separate functions when appropriate
- Reduce cyclomatic complexity by minimizing the number of independent paths through code

**Example (Good):**

```go
func ProcessUser(user *User) error {
    if user == nil {
        return errors.New("user cannot be nil")
    }
    if user.ID == "" {
        return errors.New("user ID is required")
    }
    if !user.IsActive {
        return nil // Early return for inactive users
    }
    // Happy path - process active user
    return user.Process()
}
```

**Example (Avoid):**

```go
func ProcessUser(user *User) error {
    if user != nil {
        if user.ID != "" {
            if user.IsActive {
                // Deeply nested happy path
                return user.Process()
            }
        }
    }
    return errors.New("invalid user")
}
```

## Code Organization

### Single Responsibility & Modular Design

- A function should do one concept or accomplish one clear task
- Favor modular, reusable functions over monolithic implementations
- Capture concepts that could change in their own functions (volatility decomposition)
- Extract common patterns into reusable utilities
- Keep classes reasonably sized unless a large class is justified
- Balance: avoid over-atomization (too many tiny functions) and under-atomization (monolithic functions)
- Ask: "Will refactoring this code be a nightmare?" If yes, break it down further
- Write the code in a way that is easily testable

**Volatility decomposition examples:**

- If data sources might change → isolate data retrieval
- If transformations might change → isolate transformation logic
- If output formats might change → isolate formatting logic

### Don't Repeat Yourself (DRY)

- Take commonality between sections of code and break them out into shared functions or modules
- If calling an API or library in a specific way across multiple places, extract it into a reusable function
- Consolidate repeated logic to improve maintainability
- However, avoid premature abstraction - wait until you see the pattern repeated at least 2-3 times

## Error Handling

### Explicit Error Handling

- Always handle errors explicitly; never ignore or silently swallow errors
- Return errors from functions rather than logging and continuing
- Provide context in error messages (what operation failed and why)
- Use structured error types when appropriate for better error handling downstream
- In Go: prefer `if err != nil { return err }` pattern; wrap errors with context when useful

**Example:**

```go
// Good: explicit error handling with context
result, err := fetchData(id)
if err != nil {
    return fmt.Errorf("failed to fetch data for id %s: %w", id, err)
}
```

## Testing

### Testability & Test Coverage

- Write code that is easily testable by design (dependency injection, interfaces, pure functions)
- Prefer unit tests for business logic; integration tests for external dependencies
- Test edge cases, error conditions, and boundary values
- Keep tests readable and maintainable - they serve as documentation
- Use descriptive test names that explain what is being tested

## Version Control

### Conventional Commits

- **Format:** `<type>: Fixes JIRA-#### <description>`
  - Valid types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `build`
  - JIRA reference: Must include `Fixes JIRA-####` between type and description
  - Description: Concise, imperative mood, explains what changed
  - Keep total message under 72 characters (Git subject line standard)
  - Use body for additional details if needed (separated by blank line)
- **AI behavior:** When creating commits, always ask for a JIRA ticket number if one is not provided

**Examples (all under 72 characters):**

- `feat: Fixes JIRA-2567 add auth guard clauses`
- `fix: Fixes JIRA-123 correct validation edge case`
- `docs: Fixes JIRA-7891 update API examples`
- `refactor: Fixes JIRA-4567 extract common validation`

## Project Standards

### Follow Existing Guidelines

- Follow the project's existing style guidelines exactly
- If you propose a deviation, justify why the deviation is necessary
- Consistency within a project is more important than personal preferences
- Respect language-specific conventions (e.g., Go formatting with `gofmt`, JavaScript with ESLint)
- When working with multiple languages in a project, follow each language's idiomatic patterns
