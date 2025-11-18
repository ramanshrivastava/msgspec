# mini-msgspec: A Learning-Focused Implementation

🎓 **Learn serialization, validation, and Python C extensions by building a miniature version of [msgspec](https://github.com/jcrist/msgspec)!**

---

## What is mini-msgspec?

mini-msgspec is an educational project that implements a simplified version of msgspec - a high-performance serialization and validation library. This project is designed to help you deeply understand:

- **Serialization formats** (JSON, MessagePack)
- **Type systems** and runtime validation
- **Python C extensions** and the Python/C API
- **Performance optimization** techniques
- **Parser design** and implementation

By building mini-msgspec from scratch, you'll gain practical experience with the same challenges faced by production serialization libraries.

---

## Project Structure

```
mini-msgspec/
├── README.md                     # You are here
├── LEARNING_GUIDE.md            # Complete implementation guide
├── HISTORICAL_TIMELINE.md       # Maps commits to msgspec evolution
│
├── docs/                        # Documentation
│   ├── adrs/                    # Architecture Decision Records
│   ├── comparisons/             # Mini vs Real msgspec
│   ├── diagrams/                # Visual architecture
│   ├── checkpoints/             # Learning checkpoints
│   └── references/              # msgspec references
│
├── src/mini_msgspec/            # Implementation
│   ├── _core.c                  # Main C extension
│   ├── __init__.py              # Python package
│   ├── json.py                  # JSON wrapper
│   └── msgpack.py               # MessagePack wrapper
│
├── tests/                       # Tests
│   ├── unit/                    # Unit tests
│   ├── integration/             # End-to-end tests
│   ├── benchmarks/              # Performance tests
│   └── exercises/               # Learning exercises
│
├── examples/                    # Usage examples
└── tools/                       # Development tools
    ├── visualizer/              # Visualize type trees
    └── profiler/                # Performance profiling
```

---

## Quick Start

### Prerequisites

- Python 3.9+
- GCC or Clang
- Basic understanding of C
- Familiarity with Python type hints

### Installation (when implemented)

```bash
cd mini-msgspec/src
python setup.py develop
```

### Usage Example (target API)

```python
import mini_msgspec

# Define a struct
class User(mini_msgspec.Struct):
    name: str
    age: int
    email: str | None = None

# Encode
user = User(name="Alice", age=30)
json_bytes = mini_msgspec.json.encode(user)
# b'{"name":"Alice","age":30,"email":null}'

# Decode with validation
decoded = mini_msgspec.json.decode(json_bytes, type=User)
print(decoded.name, decoded.age)  # Alice 30

# Validation errors
mini_msgspec.json.decode(b'{"name":"Bob","age":"twenty"}', type=User)
# ValidationError: Expected `int`, got `str` - at `$.age`
```

---

## Learning Path

This project is organized into **5 phases**, each building on the previous one:

### Phase 1: Foundation (Week 1-2)
**Goal**: Basic JSON encoding/decoding

- ✅ String builder utility
- ✅ JSON encoder (primitives + collections)
- ✅ JSON decoder (primitives + collections)
- ✅ Basic error handling

**What you'll learn**: Dynamic buffers, type dispatch, parsing, escaping

### Phase 2: Type System (Week 2-3)
**Goal**: Struct type and type annotations

- ✅ Type annotation parser
- ✅ Struct metaclass
- ✅ Field descriptors
- ✅ Type validation (basic)

**What you'll learn**: Python introspection, metaclasses, C struct layout, validation

### Phase 3: Advanced Validation (Week 3-4)
**Goal**: Full type validation

- ✅ Union type handling (`int | str`)
- ✅ Optional type handling (`T | None`)
- ✅ Generic collections (`list[int]`, `dict[str, int]`)
- ✅ Detailed error paths (`$.users[0].email`)

**What you'll learn**: Type theory, recursive validation, error reporting

### Phase 4: MessagePack (Week 4-5)
**Goal**: Binary serialization

- ✅ MessagePack encoder
- ✅ MessagePack decoder
- ✅ Extension types
- ✅ Binary type handling

**What you'll learn**: Binary formats, compact encoding, type tagging

### Phase 5: Optimization (Week 5+)
**Goal**: Performance improvements

- ✅ Fast integer formatting (itoa)
- ✅ Fast float parsing (atof)
- ✅ String caching
- ✅ Type validation caching

**What you'll learn**: Profiling, hot path optimization, caching strategies

---

## Documentation

### Essential Reading

1. **[LEARNING_GUIDE.md](../LEARNING_GUIDE.md)** - Comprehensive implementation guide
   - Component design details
   - Code examples
   - Performance expectations
   - Common pitfalls

2. **[HISTORICAL_TIMELINE.md](../HISTORICAL_TIMELINE.md)** - Historical context
   - Maps each commit to msgspec evolution
   - Design decisions explained
   - Performance milestones

3. **[docs/adrs/](docs/adrs/)** - Architecture Decision Records
   - Why specific approaches were chosen
   - Alternatives considered
   - Trade-offs analyzed

### Learning Resources

- **Checkpoints**: After each phase, test your understanding
- **Exercises**: Extend features, fix bugs, optimize
- **Comparisons**: See how mini differs from real msgspec
- **References**: Links to msgspec source code

---

## Development Workflow

### 1. Read the Relevant Documentation
```bash
# Before starting Phase 1:
cat ../LEARNING_GUIDE.md  # Read Phase 1 section
cat ../HISTORICAL_TIMELINE.md  # Read Phase 1 timeline
```

### 2. Implement the Feature
```bash
# Edit src/mini_msgspec/_core.c
# Follow the guide step-by-step
```

### 3. Write Tests
```bash
# Create tests/unit/test_<feature>.py
pytest tests/unit/
```

### 4. Document Your Decisions
```bash
# Create docs/adrs/XXX-<decision>.md
# Document why you made specific choices
```

### 5. Benchmark (Phase 5)
```bash
python tools/profiler/profile.py
```

---

## Commit Strategy

Each commit should:

1. **Implement one feature** from the learning guide
2. **Include tests** for that feature
3. **Update documentation** (ADR if applicable)
4. **Reference msgspec** source code
5. **Pass all existing tests**

### Commit Message Format

```
[Phase X.Y] Feature Name - Historical Context

Brief description of what this commit implements.

msgspec Reference: src/msgspec/_core.c:1000-1500
Historical Note: This mirrors msgspec v0.X.0's approach to...

Design Decisions:
- Decision 1: [rationale]
- Decision 2: [rationale]

Learning Outcomes:
1. Understand [concept]
2. See how [feature] works
```

---

## Testing

### Run All Tests
```bash
pytest tests/
```

### Run Specific Test Suite
```bash
pytest tests/unit/test_json_encoder.py
pytest tests/integration/
pytest tests/benchmarks/
```

### Code Coverage
```bash
pytest --cov=mini_msgspec tests/
```

---

## Benchmarking

Compare performance with stdlib and real msgspec:

```bash
# Run benchmarks
python tests/benchmarks/bench_json.py

# Profile specific operation
python tools/profiler/profile.py encode --format json --data examples/sample.json
```

---

## Contributing to Your Learning

As you work through this project:

1. **Document your discoveries** in `docs/learnings/`
2. **Add exercises** you found helpful to `tests/exercises/`
3. **Share insights** in `docs/comparisons/`
4. **Optimize** and document improvements

---

## Comparison with msgspec

| Feature | msgspec | mini-msgspec | Reason for Difference |
|---------|---------|--------------|----------------------|
| **Lines of Code** | ~23,000 | ~3,000-5,000 | Learning-focused simplification |
| **Formats** | JSON, MessagePack, YAML, TOML | JSON, MessagePack | Focus on core concepts |
| **Performance** | 10-80x faster than stdlib | 2-5x faster (target) | Educational, not production |
| **Type Support** | 30+ types | 10-15 types | Cover essentials |
| **Optimization** | Extensive (SIMD, caching, etc.) | Basic optimizations | Understand principles |

---

## Learning Outcomes

By completing mini-msgspec, you will:

✅ Understand how serialization libraries work internally
✅ Master Python C API fundamentals
✅ Learn performance optimization techniques
✅ Understand type systems and validation
✅ Gain experience with parser implementation
✅ Learn to read and understand complex C codebases
✅ Understand trade-offs in library design

---

## Resources

### msgspec Source Code
- Main implementation: `/home/user/msgspec/src/msgspec/_core.c`
- Number handling: `/home/user/msgspec/src/msgspec/{itoa,atof,ryu}.h`

### External Resources
- [Python C API Documentation](https://docs.python.org/3/c-api/)
- [JSON Specification (RFC 8259)](https://tools.ietf.org/html/rfc8259)
- [MessagePack Specification](https://github.com/msgpack/msgpack/blob/master/spec.md)
- [Ryu Algorithm Paper](https://dl.acm.org/doi/10.1145/3192366.3192369)

---

## FAQ

**Q: Is this production-ready?**
A: No, this is an educational project. Use the real [msgspec](https://github.com/jcrist/msgspec) for production.

**Q: How long will this take?**
A: Plan for 5-8 weeks of focused work (10-15 hours/week). Can be faster if you skip later phases.

**Q: Do I need to know C?**
A: Basic C knowledge helps, but the guide explains everything. Great way to learn C!

**Q: Should I implement all optimizations?**
A: No! Start with Phase 1-2. Add optimizations only if interested. Learning is the goal, not performance.

**Q: Can I deviate from the guide?**
A: Absolutely! The guide is a roadmap, not a rulebook. Document your alternative approaches in ADRs.

---

## Status

- [x] Documentation framework
- [ ] Phase 1: Foundation (in progress)
- [ ] Phase 2: Type System
- [ ] Phase 3: Advanced Validation
- [ ] Phase 4: MessagePack
- [ ] Phase 5: Optimization

---

## Acknowledgments

This learning project is based on [msgspec](https://github.com/jcrist/msgspec) by Jim Crist-Harif.

All design insights and architecture patterns are derived from studying the msgspec codebase. This project exists purely for educational purposes.

---

## License

This educational project follows the same license as msgspec: New BSD License.

---

**Ready to learn? Start with [LEARNING_GUIDE.md](../LEARNING_GUIDE.md)! 🚀**
