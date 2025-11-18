# ADR-XXX: [Decision Title]

**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Date**: YYYY-MM-DD
**Commit**: [hash]
**msgspec Reference**: [file:line]
**Related PEPs**: [PEP numbers if applicable]

---

## Context

**What problem are we solving?**

[Describe the problem or requirement that prompted this decision]

**What constraints exist?**

- Constraint 1
- Constraint 2
- ...

**What are we trying to learn?**

[Educational goals for this component]

---

## Decision

**What approach did we choose?**

[Clear description of the chosen solution]

### Code Example

```c
// Our mini-msgspec implementation
[code snippet demonstrating the approach]
```

```python
# Python usage example
[Python code showing how this is used]
```

---

## Rationale

### Why This Approach?

**Reason 1**: [Explanation]
- Detail A
- Detail B

**Reason 2**: [Explanation]
- Detail A
- Detail B

**Reason 3**: [Explanation]
- Detail A
- Detail B

---

## Alternatives Considered

### Alternative A: [Name]

**Description**: [What it is]

**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

**Why Rejected**: [Explanation]

### Alternative B: [Name]

**Description**: [What it is]

**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

**Why Rejected**: [Explanation]

---

## msgspec Comparison

### What msgspec Does

[Explanation of msgspec's approach + file references]

**Implementation details:**
```c
// From /home/user/msgspec/src/msgspec/_core.c:XXXX-YYYY
[relevant code snippet from msgspec]
```

**Key insights:**
- Insight 1
- Insight 2
- Insight 3

### What We're Doing Differently

**Difference 1**: [Explanation]
- Why: [Reason]
- Trade-off: [What we gain/lose]

**Difference 2**: [Explanation]
- Why: [Reason]
- Trade-off: [What we gain/lose]

---

## Historical Evolution

### Python/msgspec History

**Origin**: [When this pattern was introduced]
- In Python: [version/date if applicable]
- In msgspec: [version/date]

**Evolution**: [How it changed over time]
- Early approach (msgspec v0.1-v0.5): [Description]
- Modern approach (msgspec v0.10+): [Description]

**Related PEPs/RFCs**: [If applicable]
- PEP XXX: [Title and brief description]
- RFC XXXX: [Title and brief description]

---

## Trade-offs

### Benefits

✅ **Benefit 1**: [Description]
- Measurement/Evidence: [If applicable]

✅ **Benefit 2**: [Description]
- Measurement/Evidence: [If applicable]

✅ **Benefit 3**: [Description]
- Measurement/Evidence: [If applicable]

### Limitations

❌ **Limitation 1**: [Description]
- Impact: [How significant]
- Mitigation: [If any]

❌ **Limitation 2**: [Description]
- Impact: [How significant]
- Mitigation: [If any]

❌ **Limitation 3**: [Description]
- Impact: [How significant]
- Mitigation: [If any]

---

## Learning Outcomes

After this commit, you should understand:

1. **[Concept 1]**: [What you learn]
   - Why it matters: [Explanation]
   - Real-world application: [Example]

2. **[Concept 2]**: [What you learn]
   - Why it matters: [Explanation]
   - Real-world application: [Example]

3. **[Concept 3]**: [What you learn]
   - Why it matters: [Explanation]
   - Real-world application: [Example]

---

## Performance Considerations

### Expected Performance

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| [Operation] | [Target] | [How to measure] |
| [Operation] | [Target] | [How to measure] |

### Performance Analysis

**Hot paths**: [What operations are called most frequently]

**Bottlenecks**: [What might be slow]

**Optimization opportunities**: [What could be improved later]

---

## Security Considerations

[If applicable - especially for parsers, string handling, etc.]

**Potential vulnerabilities**:
- Vulnerability 1: [Description + mitigation]
- Vulnerability 2: [Description + mitigation]

**Best practices followed**:
- Practice 1
- Practice 2

---

## References

### msgspec Source Code

- **File**: `/home/user/msgspec/src/msgspec/[file]`
- **Lines**: [start]-[end]
- **Key functions**:
  - `function_name()` - [description]
  - `another_function()` - [description]

### External Resources

- **Documentation**: [Links]
- **Papers**: [Academic papers if applicable]
- **Blog posts**: [Relevant articles]
- **Specifications**: [RFC/PEP/etc if applicable]

### Related ADRs

- [ADR-XXX]: [Title] - [How it relates]
- [ADR-YYY]: [Title] - [How it relates]

---

## Exercises

### Exercise 1: [Title]

**Task**: [What to do]

**Difficulty**: ⭐⭐☆☆☆

**Estimated Time**: [X minutes]

**Learning Goal**: [What this teaches]

**Hints**:
- Hint 1
- Hint 2

**Solution**: [Link or location if provided]

---

### Exercise 2: [Title]

**Task**: [What to do]

**Difficulty**: ⭐⭐⭐☆☆

**Estimated Time**: [X minutes]

**Learning Goal**: [What this teaches]

**Hints**:
- Hint 1
- Hint 2

**Solution**: [Link or location if provided]

---

## Open Questions

[Questions you still have about this approach]

1. Question 1?
2. Question 2?
3. Question 3?

---

## Updates

[If this ADR is updated later, record changes here]

**YYYY-MM-DD**: [Change description]
- What changed: [Details]
- Why: [Reason]
- Impact: [What this affects]

---

## Approval

**Author**: [Your name]
**Reviewers**: [If applicable]
**Status**: [Proposed → Accepted]
**Date Accepted**: [YYYY-MM-DD]

---

## Notes

[Any additional notes, observations, or learnings]
