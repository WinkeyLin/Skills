---
name: explain
description: Provide detailed, structured explanations of code snippets, files, or functions. Use this skill when the user asks to explain code, understand code, walk through code logic, break down syntax, or asks "what does this code do?", "how does this function work?", "explain this to me", or any request involving understanding, deciphering, or learning from existing code. Triggers on requests like "explain this code", "what does this do", "walk me through this", "help me understand this function", "break down this code", or when a user pastes code and asks for clarification.
---

# Code Explainer

Provide comprehensive, structured code explanations that ensure full understanding of syntax, execution logic, data flow, and design intent.

## Explanation Workflow

1. Identify the language, framework, and context
2. State the high-level purpose in one sentence
3. Walk through the code using the explanation structure below
4. Highlight non-obvious behaviors, edge cases, and common pitfalls

## Explanation Structure

Adapt depth and sections based on code complexity. Always include sections 1-4; include 5-7 when relevant.

### 1. Purpose Summary

State what the code accomplishes in plain language. One to two sentences.

### 2. Syntax Breakdown

Explain language-specific syntax that may be unfamiliar:

- Keywords, operators, and special symbols
- Language idioms (e.g., list comprehensions, destructuring, pattern matching)
- Type annotations, generics, or decorators
- Macro invocations or metaprogramming constructs

Format: use inline code for each syntax element, followed by its meaning.

**Example:**

```
`async def` — declares an asynchronous coroutine in Python
`[*a, *b]` — spread operator merging two arrays in JavaScript
`where T: Send + Sync` — trait bounds requiring thread safety in Rust
```

### 3. Execution Logic Walkthrough

Trace the code path step by step in execution order:

- Number each step sequentially
- Show how data transforms at each step
- Mark branching points (if/else, match, try/catch) and explain which path is taken under what conditions
- For loops, describe one representative iteration, then summarize the overall effect
- For recursion, explain the base case and recursive case, then trace a small example
- For async/concurrent code, clarify the execution timeline and awaiting behavior

### 4. Key Concepts

List the core programming concepts, patterns, or paradigms the code relies on:

- Design patterns (Observer, Factory, Strategy, etc.)
- Language features (closures, generators, ownership, lifetimes, etc.)
- Algorithmic techniques (memoization, divide-and-conquer, BFS/DFS, etc.)
- Framework conventions (middleware, hooks, decorators, etc.)

One sentence per concept explaining why it is used here.

### 5. Data Flow (when multiple functions or modules interact)

Describe how data moves through the code:

- Input sources and their types
- Transformations applied at each stage
- Output destinations and their types
- Side effects (I/O, state mutation, network calls)

### 6. Error Handling and Edge Cases (when present)

- What errors are caught and how they are handled
- What inputs could cause unexpected behavior
- Boundary conditions and guard clauses

### 7. Complexity and Performance Notes (when relevant)

- Time and space complexity (Big-O)
- Performance-sensitive operations (nested loops, recursive allocations, blocking calls)
- Potential bottlenecks

## Formatting Guidelines

- Use the user's language for explanations (follow conversation language)
- Use English for all technical terms, code references, and inline code
- Use fenced code blocks with language tags for any code snippets
- Use inline code (`backticks`) for every variable name, function name, keyword, and operator mentioned in prose
- Use numbered lists for sequential steps, bullet lists for non-ordered items
- Keep each explanation point concise — one idea per bullet or step
- When referencing specific lines, use the format `line N` or `lines N-M`

## Adaptation Rules

- **Short snippets (<20 lines):** line-by-line walkthrough, combine sections 2 and 3
- **Medium functions (20-100 lines):** block-by-block walkthrough, all relevant sections
- **Large files / multiple functions (>100 lines):** start with a structural overview (list of functions/classes and their roles), then explain each function individually using the full structure
- **Single expression or one-liner:** focus on syntax breakdown, explain each operator/call from innermost to outermost
- **Unfamiliar language for user:** increase syntax detail, add comparisons to common languages (e.g., "similar to `map` in JavaScript")

## Example

**User provides:**

```python
def fibonacci(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo)
    return memo[n]
```

**Explanation:**

**Purpose:** Compute the nth Fibonacci number using memoization to avoid redundant calculations.

**Syntax:**
- `def fibonacci(n, memo={})` — defines a function with a mutable default argument `memo` as a dictionary. The default dict persists across calls because Python evaluates default arguments once at function definition time.
- `if n in memo` — checks whether key `n` exists in the dictionary.

**Execution (tracing `fibonacci(4)`):**
1. Call `fibonacci(4)` — `4` not in `memo`, not `<= 1` → recurse
2. Call `fibonacci(3)` — `3` not in `memo`, not `<= 1` → recurse
3. Call `fibonacci(2)` — `2` not in `memo`, not `<= 1` → recurse
4. Call `fibonacci(1)` — returns `1` (base case)
5. Call `fibonacci(0)` — returns `0` (base case)
6. Back in `fibonacci(2)`: `memo[2] = 1 + 0 = 1`, returns `1`
7. Call `fibonacci(1)` — `1 <= 1`, returns `1` (no memo needed)
8. Back in `fibonacci(3)`: `memo[3] = 1 + 1 = 2`, returns `2`
9. Call `fibonacci(2)` — `2` found in `memo`, returns `1` (cache hit)
10. Back in `fibonacci(4)`: `memo[4] = 2 + 1 = 3`, returns `3`

**Key Concepts:**
- **Memoization** — caches previously computed results to transform exponential recursion into linear time.
- **Mutable default argument** — Python gotcha: `memo={}` is shared across all calls without an explicit argument, which here is intentionally exploited as a persistent cache.

**Complexity:** O(n) time and O(n) space, compared to O(2^n) without memoization.
