---
description: Explain PHP/GLPI concepts after working on code
argument-hint: <concept-or-code-to-explain>
allowed-tools: Glob, Grep, Read, WebSearch
---

# GLPI Learning Workflow

Explain PHP concepts and GLPI patterns for learning.

## Input
Concept to explain: $ARGUMENTS

## Teaching Structure

### 1. The PHP Foundation

Explain relevant PHP features:
- Type hints, return types
- Null coalescing (`??`), null-safe (`?->`)
- Late static binding (`static::`)
- Closures and callbacks
- PHP 8.x features if applicable

### 2. The OOP Principle

Identify which principle applies:
- **SOLID**: Which principle and why?
- **Design Patterns**: Template Method, Observer, Factory, etc.
- **Why GLPI chose this approach**

### 3. The GLPI Pattern

Reference `_knowledge/glpi-architecture.md`:
- CommonDBTM and its hooks
- Database abstraction
- Session and rights
- Template rendering

**Find concrete examples in GLPI core:**
```bash
# Search for pattern usage
grep -rn "prepareInputForAdd" src/ | head -5
```

### 4. Code Example

Show actual GLPI code:
```
File: src/{Class}.php:{line}
```

Explain:
- What the code does
- Why it's structured this way
- How it connects to the principle

### 5. Why This Matters

- Practical implications
- Common mistakes to avoid
- Best practices

### 6. Go Deeper

Suggest further exploration:
- Related classes to study
- Similar patterns elsewhere
- Documentation to read

## Output Format

```markdown
## Understanding: {Concept}

### The PHP Foundation
[Explanation with code examples]

### The OOP Principle
[Which principle, why it matters]

### The GLPI Pattern
[How GLPI implements this]

### Code Example
[From GLPI core with file:line]

### Why This Matters
[Practical implications]

### Go Deeper
- Try looking at how `{Class}` implements this...
- Compare with `{OtherClass}`...
- Read about `{RelatedConcept}`...
```

## Style

Be conversational:
> "You might wonder why we return `false` instead of throwing an exception..."

Create "aha!" moments:
> "This is why you see this pattern across all CommonDBTM children..."
