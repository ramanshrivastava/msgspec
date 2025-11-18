# Phase X Checkpoint: [Phase Name]

Complete this checkpoint to verify your understanding before moving to the next phase.

**Estimated Time**: 30-60 minutes
**Prerequisites**: Completed Phase X implementation

---

## Self-Assessment Quiz

### Conceptual Understanding

#### Question 1: [Concept]

**Question**: [The question]

**Your Answer**:
```
[Write your answer here]
```

<details>
<summary>Expected Answer</summary>

[The expected answer with explanation]

**Key points:**
- Point 1
- Point 2
- Point 3

**Reference**: [ADR or code location]

</details>

---

#### Question 2: [Concept]

**Question**: [The question]

**Your Answer**:
```
[Write your answer here]
```

<details>
<summary>Expected Answer</summary>

[The expected answer with explanation]

</details>

---

### Code Comprehension

#### Question 3: What does this code do?

```c
[code snippet from your implementation]
```

**Your Answer**:
```
[Explain what this code does]
```

<details>
<summary>Expected Answer</summary>

**Purpose**: [High-level purpose]

**Line-by-line:**
- Line 1-2: [Explanation]
- Line 3-5: [Explanation]
- Line 6: [Explanation]

**Why this approach?**: [Design rationale]

</details>

---

#### Question 4: Find the bug

```c
[code snippet with a subtle bug]
```

**What's wrong? How would you fix it?**

**Your Answer**:
```
[Identify the bug and propose a fix]
```

<details>
<summary>Answer</summary>

**Bug**: [Description of the bug]

**Why it's wrong**: [Explanation]

**Fix**:
```c
[Corrected code]
```

**Key lesson**: [What this teaches]

</details>

---

## Hands-On Exercises

### Exercise 1: [Title]

**Difficulty**: ⭐⭐☆☆☆
**Estimated Time**: 15 minutes
**Learning Goal**: [What you'll learn]

**Task**: [Description of the task]

**Requirements**:
1. Requirement 1
2. Requirement 2
3. Requirement 3

**Test case**:
```python
# Input:
[test input]

# Expected output:
[expected output]
```

**Hints**:
<details>
<summary>Hint 1</summary>
[First hint]
</details>

<details>
<summary>Hint 2</summary>
[Second hint]
</details>

<details>
<summary>Hint 3 - Implementation approach</summary>
[More detailed hint about approach]
</details>

**Solution**:
<details>
<summary>Click to see solution</summary>

```c
[Solution code]
```

**Explanation**:
[Why this solution works]

</details>

---

### Exercise 2: [Title]

**Difficulty**: ⭐⭐⭐☆☆
**Estimated Time**: 30 minutes
**Learning Goal**: [What you'll learn]

**Task**: [Description]

[Same format as Exercise 1]

---

## Comparative Analysis

### mini-msgspec vs Real msgspec

Fill in this comparison table based on your implementation:

| Aspect | mini-msgspec | Real msgspec | Why Different? |
|--------|--------------|--------------|----------------|
| Lines of code | [Your count] | [Real count] | [Reason] |
| Supported types | [List] | [List] | [Reason] |
| Performance (encode 1000 objects) | [Your measurement] | [Real measurement] | [Reason] |
| Memory usage | [Your measurement] | [Real measurement] | [Reason] |

**Performance Measurement**:

Run this benchmark:
```python
import time
import mini_msgspec
import msgspec

data = [{"name": f"user{i}", "age": i} for i in range(1000)]

# Measure mini-msgspec
t0 = time.time()
for _ in range(100):
    mini_msgspec.json.encode(data)
t1 = time.time()
mini_time = t1 - t0

# Measure real msgspec
t0 = time.time()
for _ in range(100):
    msgspec.json.encode(data)
t1 = time.time()
msgspec_time = t1 - t0

print(f"mini-msgspec: {mini_time:.3f}s")
print(f"msgspec: {msgspec_time:.3f}s")
print(f"Ratio: {mini_time/msgspec_time:.1f}x slower")
```

**Your results**:
```
mini-msgspec: [X.XXX]s
msgspec: [X.XXX]s
Ratio: [X.X]x slower
```

**Analysis**:
```
[Why is there a performance gap?]
[What are the main bottlenecks in your implementation?]
[What optimizations does msgspec use that you don't?]
```

---

## Performance Profiling

### Find Your Hot Paths

Use a profiler to identify where your code spends most time:

```bash
python -m cProfile -s cumtime tests/benchmarks/bench_json.py > profile.txt
```

**Your top 5 functions by time**:

1. [Function name] - [X.XX%] - [Why this is expensive]
2. [Function name] - [X.XX%] - [Why this is expensive]
3. [Function name] - [X.XX%] - [Why this is expensive]
4. [Function name] - [X.XX%] - [Why this is expensive]
5. [Function name] - [X.XX%] - [Why this is expensive]

**Optimization opportunities**:
- Opportunity 1: [What could be optimized]
- Opportunity 2: [What could be optimized]
- Opportunity 3: [What could be optimized]

---

## Reading Real msgspec Code

### Exercise: Trace Execution

**Task**: Read msgspec's implementation of [specific feature] and trace how it works.

**File**: `/home/user/msgspec/src/msgspec/_core.c:[lines]`

**Questions to answer**:

1. **What's the entry point?**
   - Function: [name]
   - Parameters: [list]
   - Return value: [type]

2. **What's the algorithm?**
   ```
   [Pseudocode of the algorithm]
   ```

3. **What optimizations are used?**
   - Optimization 1: [description]
   - Optimization 2: [description]
   - Optimization 3: [description]

4. **What did you learn?**
   ```
   [Key insights from reading this code]
   ```

---

## Debugging Challenge

### Challenge: Fix These Bugs

We've introduced some bugs in the reference implementation. Can you find and fix them?

#### Bug 1: Memory Leak

**Symptom**: Memory usage grows over time

**Code location**: `[file:line]`

**Reproduce**:
```python
[code to reproduce the bug]
```

**Your diagnosis**:
```
[What's causing the memory leak?]
```

**Your fix**:
```c
[Code fix]
```

<details>
<summary>Answer</summary>

**Root cause**: [Explanation]

**Fix**: [Detailed explanation of the fix]

</details>

---

#### Bug 2: Edge Case Error

**Symptom**: Crashes on certain inputs

**Code location**: `[file:line]`

**Reproduce**:
```python
[code to reproduce the bug]
```

**Your diagnosis**:
```
[What's the problem?]
```

**Your fix**:
```c
[Code fix]
```

<details>
<summary>Answer</summary>

**Root cause**: [Explanation]

**Fix**: [Detailed explanation of the fix]

</details>

---

## Reflection

### What I Learned

**Biggest insight**:
```
[What was your biggest "aha!" moment in this phase?]
```

**Most challenging part**:
```
[What was hardest to understand or implement?]
```

**How I overcame challenges**:
```
[What strategies helped you?]
```

**Connections to other concepts**:
```
[How does this relate to other programming concepts you know?]
```

---

### What I Would Do Differently

**If I were to redesign this component**:
```
[What would you change and why?]
```

**Alternative approaches I considered**:
```
[Other ways you thought about solving the problem]
```

---

## Readiness Check

Before moving to the next phase, you should be able to:

- [ ] Explain [key concept 1] to someone else
- [ ] Modify the code to add [feature]
- [ ] Identify and fix common bugs in [component]
- [ ] Understand the equivalent msgspec code
- [ ] Benchmark and profile your implementation
- [ ] Articulate the trade-offs in your design decisions

**If you checked all boxes**: You're ready for the next phase! 🎉

**If not**: Review the sections where you're uncertain. Revisit the relevant ADRs and code examples.

---

## Next Steps

**Before starting Phase [X+1]**:

1. **Review your code**: Clean up, add comments, ensure tests pass
2. **Document learnings**: Update your notes with insights
3. **Read ahead**: Skim the Phase [X+1] section of LEARNING_GUIDE.md
4. **Prepare**: What concepts might be challenging?

**Optional deep dives**:
- [ ] Read relevant PEPs: [List PEPs]
- [ ] Study related papers: [List papers]
- [ ] Explore msgspec's git history: [Specific commits]
- [ ] Implement bonus features: [List ideas]

---

## Questions to Explore

**Open questions from this phase**:

1. [Question you still have]
2. [Something you'd like to investigate deeper]
3. [Alternative approach you're curious about]

**Research these in your own time or ask in discussions!**

---

## Feedback

**What worked well in this phase?**
```
[What helped you learn?]
```

**What could be improved?**
```
[Suggestions for the learning materials]
```

**Additional resources you found helpful**:
```
[Articles, videos, docs that helped]
```

---

## Completion

**Date completed**: [YYYY-MM-DD]
**Time spent**: [X hours]
**Confidence level**: [1-5, with 5 being very confident]

**Ready for Phase [X+1]**: ✅ / ⚠️ / ❌

---

**Congratulations on completing Phase X! 🎉**

Keep building on what you've learned!
