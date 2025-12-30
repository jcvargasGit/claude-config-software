---
name: clean-code
description: Apply Clean Code principles from Robert C. Martin (Uncle Bob) for writing readable, maintainable, and professional code.
---

# Clean Code Skill

Apply these Clean Code principles based on Robert C. Martin's teachings.

## Meaningful Names

### Use Intention-Revealing Names
- Names should answer: why it exists, what it does, how it is used
- Avoid names that require mental mapping
- Bad: `d`, `list`, `data`, `info`, `temp`
- Good: `elapsedTimeInDays`, `accountList`, `userProfile`

### Use Pronounceable Names
- If you can't pronounce it, you can't discuss it
- Bad: `genymdhms` (generation date, year, month, day, hour, minute, second)
- Good: `generationTimestamp`

### Use Searchable Names
- Single-letter names only for local variables in short methods
- The length of a name should correspond to the size of its scope

### Avoid Encodings
- No Hungarian notation
- No member prefixes (`m_`, `_`)
- No interface prefixes (`I`)

### Class Names
- Nouns or noun phrases: `Customer`, `Account`, `AddressParser`
- Avoid: `Manager`, `Processor`, `Data`, `Info` (too vague)

### Method Names
- Verbs or verb phrases: `save`, `deletePage`, `postPayment`
- Accessors: `get`, `set`, `is` prefixes

## Functions

### Small
- Functions should be small
- Rarely over 20 lines
- Blocks within `if`, `else`, `while` should be one line (a function call)

### Do One Thing
- Functions should do one thing, do it well, do it only
- If a function does multiple steps, those steps should be one level of abstraction below the function name

### One Level of Abstraction per Function
- Don't mix high-level and low-level operations
- Reading code should be like reading a top-down narrative

### Switch Statements
- Bury them in low-level classes
- Use polymorphism instead when possible

### Function Arguments
- Ideal: zero arguments (niladic)
- One argument (monadic): common, acceptable
- Two arguments (dyadic): harder to understand
- Three arguments (triadic): avoid when possible
- More than three: requires special justification, consider object

### Have No Side Effects
- Don't do hidden things
- Don't modify global state unexpectedly
- Don't make temporal couplings

### Command Query Separation
- Functions should either do something OR answer something, not both
- Commands change state, queries return data

### Prefer Exceptions to Returning Error Codes
- Error codes lead to deeply nested structures
- Extract try/catch blocks into separate functions

### Don't Repeat Yourself (DRY)
- Duplication is the root of all evil in software
- Every piece of knowledge must have a single representation

## Comments

### Comments Do Not Make Up for Bad Code
- Clean code needs few comments
- Express yourself in code, not comments
- Avoid comments, and only use them if it's realy necesary 

### Good Comments
- Legal comments (copyright, license)
- Informative comments explaining intent
- Clarification of obscure arguments
- Warning of consequences
- TODO comments (temporary)
- Amplification of importance

### Bad Comments
- Mumbling (unclear purpose)
- Redundant comments (say what code says)
- Misleading comments
- Mandated comments (every function needs doc)
- Journal comments (changelog in code)
- Noise comments (`// default constructor`)
- Commented-out code (delete it, use version control)
- Position markers (`// ============`)

## Formatting

### Vertical Formatting
- Small files are easier to understand
- Newspaper metaphor: high-level at top, details below
- Vertical openness: blank lines separate concepts
- Vertical density: related lines should be close
- Vertical distance: related concepts should be close

### Horizontal Formatting
- Keep lines short (80-120 characters)
- Use horizontal whitespace to associate and disassociate
- Don't align variable declarations

### Team Rules
- Every team should agree on formatting rules
- Consistency trumps personal preference

## Objects and Data Structures

### Data Abstraction
- Hide implementation, expose behavior
- Don't add getters and setters blindly

### The Law of Demeter
- A method should only call:
  - Methods on its own class
  - Methods on objects passed as parameters
  - Methods on objects it creates
  - Methods on its direct component objects
- Avoid: `a.getB().getC().doSomething()`

### Data Transfer Objects (DTOs)
- Classes with public variables, no functions
- Useful for communication with databases, APIs

## Error Handling

### Use Exceptions Rather Than Return Codes
- Exceptions separate error handling from logic
- Return codes clutter the caller

### Write Try-Catch-Finally First
- Define the scope and behavior
- Transaction-like behavior

### Provide Context with Exceptions
- Include informative error messages
- Include failed operation and failure type

### Define Exception Classes by Caller's Needs
- Wrap third-party APIs to simplify exception handling
- Define exceptions based on how they're caught

### Don't Return Null
- Returning null creates work for callers
- Throw exception or return special case object
- Use Optional/Maybe types

### Don't Pass Null
- Avoid passing null as arguments
- No good way to handle null arguments

## Boundaries

### Using Third-Party Code
- Wrap third-party libraries
- Don't let their API spread through your code
- Write learning tests to explore their API

### Clean Boundaries

- Code at boundaries needs clear separation  
- Depend on what you control

## Unit Tests

### The Three Laws of TDD
1. Write no production code until you have a failing test
2. Write only enough test to demonstrate a failure
3. Write only enough production code to pass the test

### Clean Tests
- Tests should be readable
- One assert per test (guideline, not rule)
- Single concept per test
- F.I.R.S.T.:
  - **Fast**: Tests should run quickly
  - **Independent**: Tests should not depend on each other
  - **Repeatable**: Tests should work in any environment
  - **Self-Validating**: Tests should have boolean output (pass/fail)
  - **Timely**: Tests should be written just before production code

## Classes

### Class Organization
- Public static constants
- Private static variables
- Private instance variables
- Public functions
- Private utilities called by public functions

### Classes Should Be Small
- Measured by responsibilities, not lines
- Single Responsibility Principle (SRP)
- A class should have one, and only one, reason to change

### Cohesion
- Methods and variables should be co-dependent
- High cohesion: methods use many instance variables

### Organizing for Change
- Open-Closed Principle: open for extension, closed for modification
- Dependency Inversion: depend on abstractions, not concretions

## Systems

### Separate Constructing from Using
- Separate startup logic from runtime logic
- Use Dependency Injection

### Scaling Up
- Start simple, refactor as needed
- Don't over-engineer

### Use Standards Wisely
- Standards make it easier to reuse ideas
- But don't use standards blindly

## Emergence

### Simple Design Rules (by Kent Beck)
1. Runs all the tests
2. Contains no duplication
3. Expresses programmer's intent
4. Minimizes number of classes and methods

Priority is in order: tests first, then refactor for duplication and expressiveness.

## Code Smells

### Comments
- Inappropriate information
- Obsolete comment
- Redundant comment
- Poorly written comment
- Commented-out code

### Environment
- Build requires more than one step
- Tests require more than one step

### Functions
- Too many arguments
- Output arguments
- Flag arguments
- Dead function

### General
- Multiple languages in one file
- Obvious behavior not implemented
- Incorrect behavior at boundaries
- Overridden safeties
- Duplication
- Code at wrong level of abstraction
- Base classes depending on derivatives
- Too much information
- Dead code
- Vertical separation
- Inconsistency
- Clutter
- Artificial coupling
- Feature envy
- Selector arguments
- Obscured intent
- Misplaced responsibility
- Inappropriate static
- Use explanatory variables
- Function names should say what they do
- Understand the algorithm
- Make logical dependencies physical
- Prefer polymorphism to if/else or switch
- Follow standard conventions
- Replace magic numbers with named constants
- Be precise
- Structure over convention
- Encapsulate conditionals
- Avoid negative conditionals
- Functions should do one thing
- Hidden temporal couplings
- Don't be arbitrary
- Encapsulate boundary conditions
- Keep configurable data at high levels
- Avoid transitive navigation

## Quality Checklist

When writing code, verify:
- [ ] Names reveal intention
- [ ] Functions are small and do one thing
- [ ] No duplication
- [ ] Code expresses intent clearly
- [ ] Error handling is clean
- [ ] Tests cover the code
- [ ] No commented-out code
- [ ] No dead code
- [ ] Consistent formatting
