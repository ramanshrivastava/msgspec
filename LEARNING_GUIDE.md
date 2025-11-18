# Mini-msgspec: A Reasoning-Based Learning Framework

## Quick Reference

### What's in msgspec

| Component | Lines of Code | Complexity |
|-----------|---------------|------------|
| **Total Code** | ~23,000 lines C | High |
| **Main Components** | 6 core systems | Advanced |
| **Serialization Formats** | 4 (JSON, MessagePack, YAML, TOML) | Varied |
| **Type Validation** | Zero-cost schema validation | Advanced |
| **Optimization Techniques** | 10+ (caching, SIMD, fast paths) | Expert |

### What Your Mini Version Should Have

| Component | Target Size | Core Features |
|-----------|-------------|---------------|
| **Target Size** | 3,000-5,000 lines C | Achievable |
| **Formats** | 1-2 (JSON + MessagePack) | Focused |
| **Basic Types** | 5-6 (int, str, list, dict, bool, null) | Essential |
| **Struct System** | Simple structs (no inheritance) | Learnable |
| **Validation** | Basic type checking | Practical |
| **Optimization** | Reference implementation first | Iterative |

---

## Architecture Overview

```
Python Type Annotations
         ↓
   TYPE ANALYZER (500-800 lines)
   ├─ Parse type hints
   ├─ Build type tree
   └─ Generate validators
         ↓
   STRUCT SYSTEM (800-1200 lines)
   ├─ Struct metaclass
   ├─ Field descriptors
   └─ Instance creation
         ↓
   ENCODERS (1000-1500 lines)
   ├─ JSON Encoder
   │  ├─ Type dispatch
   │  ├─ String escaping
   │  └─ Number formatting
   └─ MessagePack Encoder
      ├─ Type tagging
      ├─ Binary packing
      └─ Extension types
         ↓
   DECODERS (1000-1500 lines)
   ├─ JSON Decoder
   │  ├─ Tokenizer
   │  ├─ Parser
   │  └─ Validator
   └─ MessagePack Decoder
      ├─ Binary unpacker
      ├─ Type reader
      └─ Validator
         ↓
   NUMBER HANDLING (300-500 lines)
   ├─ Fast integer parsing
   ├─ Float parsing (atof)
   └─ Float formatting (ryu)
         ↓
   Results (Serialized/Deserialized data)
```

---

## Component Design Details

### 1. TYPE ANALYZER (500-800 lines)

**Purpose**: Parse Python type annotations → type tree for validation

**Types to recognize:**
- **Primitives**: `int`, `str`, `float`, `bool`, `None`
- **Collections**: `list[T]`, `dict[K, V]`, `set[T]`, `tuple[T, ...]`
- **Unions**: `int | str`, `Optional[T]`
- **Structs**: Custom `msgspec.Struct` classes
- **Literals**: `Literal[1, 2, 3]`, `Literal["a", "b"]`

**Simplified Implementation:**
- Use Python's `typing` module introspection
- Build recursive type tree
- No complex generic variance handling

**Key data structure:**
```c
typedef enum {
    TYPE_INT,
    TYPE_STR,
    TYPE_FLOAT,
    TYPE_BOOL,
    TYPE_NONE,
    TYPE_LIST,
    TYPE_DICT,
    TYPE_STRUCT,
    TYPE_UNION,
    // ... more types
} TypeKind;

typedef struct TypeNode {
    TypeKind kind;
    union {
        struct {
            struct TypeNode *item_type;  // For list[T]
        } list;
        struct {
            struct TypeNode *key_type;    // For dict[K, V]
            struct TypeNode *value_type;
        } dict;
        struct {
            PyObject *struct_class;       // For Struct types
        } struct_type;
        struct {
            struct TypeNode **options;    // For Union types
            int n_options;
        } union_type;
    } details;
} TypeNode;
```

**Key functions:**
- `parse_type_annotation(PyObject *annotation) → TypeNode*`
- `validate_value(TypeNode *type, PyObject *value) → bool`

**Reference**: `/home/user/msgspec/src/msgspec/_core.c:2747` (TypeNode)

---

### 2. STRUCT SYSTEM (800-1200 lines)

**Purpose**: Fast alternative to `dataclasses` with C-level speed

**Features to implement:**
- Struct class definition
- Field declarations with defaults
- `__init__`, `__repr__`, `__eq__`
- Fast attribute access (use slots)

**Core structure:**
```c
typedef struct {
    PyObject *name;        // Field name
    TypeNode *type;        // Field type
    PyObject *default_val; // Default value (or NODEFAULT)
    Py_ssize_t offset;     // Offset in struct instance
} FieldInfo;

typedef struct {
    PyTypeObject base;     // Extends PyType
    FieldInfo *fields;     // Array of fields
    Py_ssize_t n_fields;   // Number of fields
    // ... other metadata
} StructMetaObject;

typedef struct {
    PyObject_HEAD
    // Fields stored inline using __slots__
    PyObject *field_values[];
} StructObject;
```

**Example Python usage:**
```python
class User(msgspec.Struct):
    name: str
    age: int
    email: str | None = None

# Compiles to efficient C struct
```

**Key functions:**
- `struct_new()` - Create struct instance
- `struct_get_field()` - Fast field access
- `struct_set_field()` - Fast field assignment

**Simplified approach:**
- Use Python's C API `tp_members` for field access
- No inheritance initially (Phase 1)
- No `__post_init__` hooks (add in Phase 2)

**Reference**: `/home/user/msgspec/src/msgspec/_core.c:2972` (StructInfo)

---

### 3. JSON ENCODER (500-800 lines)

**Purpose**: Convert Python objects → JSON bytes

**Encoding strategy:**
- Recursive type dispatch
- String builder for output
- Special handling for common types

**Core algorithm:**
```c
typedef struct {
    char *buffer;      // Output buffer
    size_t size;       // Current size
    size_t capacity;   // Allocated capacity
} StrBuilder;

int encode_json(PyObject *obj, StrBuilder *out) {
    if (obj == Py_None) {
        strbuilder_append(out, "null", 4);
    }
    else if (PyBool_Check(obj)) {
        if (obj == Py_True)
            strbuilder_append(out, "true", 4);
        else
            strbuilder_append(out, "false", 5);
    }
    else if (PyLong_Check(obj)) {
        // Fast integer formatting
        long val = PyLong_AsLong(obj);
        char buf[32];
        int len = itoa(val, buf);
        strbuilder_append(out, buf, len);
    }
    else if (PyFloat_Check(obj)) {
        // Fast float formatting (ryu algorithm)
        double val = PyFloat_AS_DOUBLE(obj);
        char buf[32];
        int len = ryu_d2s(val, buf);
        strbuilder_append(out, buf, len);
    }
    else if (PyUnicode_Check(obj)) {
        // String with escaping
        encode_json_string(obj, out);
    }
    else if (PyList_Check(obj)) {
        // Recursive list encoding
        strbuilder_append(out, "[", 1);
        for (Py_ssize_t i = 0; i < PyList_GET_SIZE(obj); i++) {
            if (i > 0) strbuilder_append(out, ",", 1);
            encode_json(PyList_GET_ITEM(obj, i), out);
        }
        strbuilder_append(out, "]", 1);
    }
    else if (PyDict_Check(obj)) {
        // Recursive dict encoding
        // ...
    }
    else if (Struct_Check(obj)) {
        // Struct encoding (encode as object)
        // ...
    }
    else {
        PyErr_Format(PyExc_TypeError, "Unsupported type: %s",
                     Py_TYPE(obj)->tp_name);
        return -1;
    }
    return 0;
}
```

**String escaping (critical for security):**
```c
void encode_json_string(PyObject *str, StrBuilder *out) {
    Py_ssize_t len;
    const char *data = PyUnicode_AsUTF8AndSize(str, &len);

    strbuilder_append(out, "\"", 1);
    for (Py_ssize_t i = 0; i < len; i++) {
        char c = data[i];
        switch (c) {
            case '"':  strbuilder_append(out, "\\\"", 2); break;
            case '\\': strbuilder_append(out, "\\\\", 2); break;
            case '\n': strbuilder_append(out, "\\n", 2); break;
            case '\r': strbuilder_append(out, "\\r", 2); break;
            case '\t': strbuilder_append(out, "\\t", 2); break;
            default:
                if (c < 32) {
                    // Unicode escape \u00XX
                    char buf[7];
                    snprintf(buf, 7, "\\u%04x", c);
                    strbuilder_append(out, buf, 6);
                } else {
                    strbuilder_append(out, &c, 1);
                }
        }
    }
    strbuilder_append(out, "\"", 1);
}
```

**Reference**: `/home/user/msgspec/src/msgspec/_core.c:10477` (JSON Encoder)

---

### 4. JSON DECODER (800-1200 lines)

**Purpose**: Parse JSON bytes → Python objects (with validation)

**Tokenization approach:**
- Single-pass parser (no separate tokenizer)
- Recursive descent
- Type-driven validation

**Core structure:**
```c
typedef struct {
    const char *input;     // Input buffer
    const char *pos;       // Current position
    const char *end;       // End of input
    TypeNode *expected_type; // Expected type (for validation)
    PathNode *path;        // Current path (for error messages)
} JSONDecoder;

PyObject* decode_json(JSONDecoder *self) {
    // Skip whitespace
    skip_whitespace(self);

    char c = peek_char(self);

    if (c == 'n') {
        return parse_null(self);
    }
    else if (c == 't' || c == 'f') {
        return parse_bool(self);
    }
    else if (c == '"') {
        return parse_string(self);
    }
    else if (c == '[') {
        return parse_array(self);
    }
    else if (c == '{') {
        return parse_object(self);
    }
    else if (c == '-' || is_digit(c)) {
        return parse_number(self);
    }
    else {
        return parse_error(self, "Unexpected character");
    }
}
```

**Number parsing (fast path):**
```c
PyObject* parse_number(JSONDecoder *self) {
    const char *start = self->pos;
    const char *p = start;

    // Sign
    bool negative = false;
    if (*p == '-') {
        negative = true;
        p++;
    }

    // Integer part
    if (!is_digit(*p)) {
        return parse_error(self, "Expected digit");
    }

    int64_t int_val = 0;
    bool is_int = true;

    while (is_digit(*p)) {
        int digit = *p - '0';
        // Check overflow
        if (int_val > (INT64_MAX - digit) / 10) {
            is_int = false;
            break;
        }
        int_val = int_val * 10 + digit;
        p++;
    }

    // Check for decimal point
    if (*p == '.' || *p == 'e' || *p == 'E') {
        is_int = false;
    }

    if (is_int) {
        self->pos = p;
        return PyLong_FromLongLong(negative ? -int_val : int_val);
    }
    else {
        // Fall back to float parsing
        return parse_float(self, start);
    }
}
```

**Type validation during parsing:**
```c
PyObject* decode_with_type(JSONDecoder *self, TypeNode *type) {
    PyObject *obj = decode_json(self);
    if (obj == NULL) return NULL;

    // Validate type
    if (!validate_value(type, obj)) {
        PyErr_Format(PyExc_ValidationError,
                     "Expected `%s`, got `%s` - at `%s`",
                     type_name(type),
                     Py_TYPE(obj)->tp_name,
                     path_str(self->path));
        Py_DECREF(obj);
        return NULL;
    }

    return obj;
}
```

**Reference**: `/home/user/msgspec/src/msgspec/_core.c:16678` (JSON Decoder)

---

### 5. MESSAGEPACK ENCODER/DECODER (600-1000 lines)

**Purpose**: Binary serialization format (more compact than JSON)

**MessagePack format basics:**
- Type tags (1 byte) + data
- Variable-length integers
- Binary strings
- Extension types

**Type tags:**
```c
#define MP_NIL      0xc0
#define MP_FALSE    0xc2
#define MP_TRUE     0xc3
#define MP_UINT8    0xcc
#define MP_UINT16   0xcd
#define MP_UINT32   0xce
#define MP_UINT64   0xcf
#define MP_INT8     0xd0
#define MP_INT16    0xd1
#define MP_INT32    0xd2
#define MP_INT64    0xd3
#define MP_FLOAT32  0xca
#define MP_FLOAT64  0xcb
#define MP_STR8     0xd9
#define MP_ARRAY16  0xdc
#define MP_MAP16    0xde
```

**Encoding example:**
```c
int encode_msgpack_int(int64_t val, StrBuilder *out) {
    if (val >= 0) {
        if (val < 128) {
            // Positive fixint (0-127)
            char c = (char)val;
            strbuilder_append(out, &c, 1);
        }
        else if (val <= UINT8_MAX) {
            char buf[2] = {MP_UINT8, (char)val};
            strbuilder_append(out, buf, 2);
        }
        else if (val <= UINT16_MAX) {
            char buf[3] = {MP_UINT16};
            _msgspec_store16(buf + 1, val);
            strbuilder_append(out, buf, 3);
        }
        // ... more cases
    }
    else {
        // Negative integers
        // ...
    }
}
```

**Decoding example:**
```c
PyObject* decode_msgpack(MPDecoder *self) {
    uint8_t tag = read_u8(self);

    if (tag <= 0x7f) {
        // Positive fixint
        return PyLong_FromLong(tag);
    }
    else if (tag >= 0xe0) {
        // Negative fixint
        return PyLong_FromLong((int8_t)tag);
    }
    else if (tag == MP_NIL) {
        Py_RETURN_NONE;
    }
    else if (tag == MP_TRUE) {
        Py_RETURN_TRUE;
    }
    // ... more cases
}
```

**Reference**: `/home/user/msgspec/src/msgspec/_core.c:9439` (MessagePack)

---

### 6. NUMBER HANDLING (300-500 lines)

**Three critical algorithms:**

#### A. Integer to String (itoa.h)

**Why it matters:** Standard `sprintf` is slow for integers

**Fast approach:**
```c
static const char digit_pairs[200] = {
    '0','0','0','1','0','2','0','3','0','4','0','5','0','6','0','7','0','8','0','9',
    '1','0','1','1','1','2','1','3','1','4','1','5','1','6','1','7','1','8','1','9',
    // ... up to '9','9'
};

int itoa_u64(uint64_t val, char *buf) {
    char temp[20];
    char *p = temp + 20;

    while (val >= 100) {
        uint64_t q = val / 100;
        uint64_t r = val % 100;
        p -= 2;
        memcpy(p, &digit_pairs[r * 2], 2);
        val = q;
    }

    if (val >= 10) {
        p -= 2;
        memcpy(p, &digit_pairs[val * 2], 2);
    } else {
        *--p = '0' + val;
    }

    int len = temp + 20 - p;
    memcpy(buf, p, len);
    return len;
}
```

**Reference**: `/home/user/msgspec/src/msgspec/itoa.h`

#### B. String to Float (atof.h)

**Why it matters:** Standard `strtod` is slow and locale-dependent

**Simplified implementation for MVP:**
```c
double simple_atof(const char *str, const char *end) {
    double sign = 1.0;
    double value = 0.0;

    // Parse sign
    if (*str == '-') {
        sign = -1.0;
        str++;
    }

    // Parse integer part
    while (str < end && is_digit(*str)) {
        value = value * 10.0 + (*str - '0');
        str++;
    }

    // Parse decimal part
    if (str < end && *str == '.') {
        str++;
        double scale = 0.1;
        while (str < end && is_digit(*str)) {
            value += (*str - '0') * scale;
            scale *= 0.1;
            str++;
        }
    }

    // Parse exponent (e/E)
    // ... (simplified for MVP)

    return sign * value;
}
```

**Reference**: `/home/user/msgspec/src/msgspec/atof.h` (~439 lines, highly optimized)

#### C. Float to String (ryu.h)

**Why it matters:** Standard `sprintf("%.17g")` is slow

**Ryu algorithm** (Simplified explanation):
- Converts float → shortest decimal representation
- Guarantees round-trip accuracy
- 5-10x faster than `sprintf`

**For MVP:** Use simplified version or standard library initially

**Reference**: `/home/user/msgspec/src/msgspec/ryu.h` (~995 lines)

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1-2, ~1200 lines)

**Goal**: Basic JSON encoding/decoding without validation

**Components:**
1. ✅ String builder utility
2. ✅ JSON encoder (primitives + collections)
3. ✅ JSON decoder (primitives + collections)
4. ✅ Basic error handling

**Test:**
```python
import mini_msgspec

# Encode
data = {"name": "Alice", "age": 30, "scores": [95, 87, 92]}
json_bytes = mini_msgspec.json.encode(data)
print(json_bytes)  # b'{"name":"Alice","age":30,"scores":[95,87,92]}'

# Decode
decoded = mini_msgspec.json.decode(json_bytes)
print(decoded)  # {'name': 'Alice', 'age': 30, 'scores': [95, 87, 92]}
```

**Commit structure:**
| Commit | Feature | Lines | Learning Focus |
|--------|---------|-------|----------------|
| 1.1 | Project setup + string builder | 150 | Dynamic buffers |
| 1.2 | JSON encoder (primitives) | 250 | Type dispatch, escaping |
| 1.3 | JSON encoder (collections) | 200 | Recursive encoding |
| 1.4 | JSON decoder (primitives) | 300 | Parsing, number conversion |
| 1.5 | JSON decoder (collections) | 250 | Recursive parsing |
| 1.6 | Error handling | 150 | Error propagation |

---

### Phase 2: Type System (Week 2-3, +1000 lines)

**Goal**: Struct type and basic type annotations

**Components:**
1. ✅ Type annotation parser
2. ✅ Struct metaclass
3. ✅ Field descriptors
4. ✅ Type validation (basic)

**Test:**
```python
class User(mini_msgspec.Struct):
    name: str
    age: int

user = User(name="Bob", age=25)
json_bytes = mini_msgspec.json.encode(user)
# b'{"name":"Bob","age":25}'

decoded = mini_msgspec.json.decode(json_bytes, type=User)
print(decoded.name, decoded.age)  # Bob 25

# Validation error
mini_msgspec.json.decode(b'{"name":"Bob","age":"twenty"}', type=User)
# ValidationError: Expected `int`, got `str` - at `$.age`
```

**Commit structure:**
| Commit | Feature | Lines | Learning Focus |
|--------|---------|-------|----------------|
| 2.1 | TypeNode structure | 200 | Type representation |
| 2.2 | Type annotation parser | 300 | Introspection |
| 2.3 | Struct metaclass | 350 | Metaclass programming |
| 2.4 | Struct instance creation | 250 | Fast object creation |
| 2.5 | Basic validation | 200 | Type checking |

---

### Phase 3: Advanced Validation (Week 3-4, +800 lines)

**Goal**: Full type validation including unions, optionals

**Components:**
1. ✅ Union type handling
2. ✅ Optional type handling
3. ✅ List/Dict type validation
4. ✅ Detailed error paths

**Test:**
```python
class Config(mini_msgspec.Struct):
    port: int
    host: str | None = None
    options: dict[str, int] = {}

# Valid
mini_msgspec.json.decode(
    b'{"port":8080,"host":"localhost","options":{"timeout":30}}',
    type=Config
)

# Invalid
mini_msgspec.json.decode(
    b'{"port":8080,"options":{"timeout":"30"}}',
    type=Config
)
# ValidationError: Expected `int`, got `str` - at `$.options.timeout`
```

---

### Phase 4: MessagePack Support (Week 4-5, +1000 lines)

**Goal**: Binary serialization format

**Components:**
1. ✅ MessagePack encoder
2. ✅ MessagePack decoder
3. ✅ Extension types
4. ✅ Binary type handling

**Test:**
```python
data = {"name": "Alice", "data": b"\x00\x01\x02"}
mp_bytes = mini_msgspec.msgpack.encode(data)
decoded = mini_msgspec.msgpack.decode(mp_bytes)
```

---

### Phase 5: Optimization (Week 5-6, +500 lines)

**Goal**: Performance improvements

**Optimizations:**
1. ✅ Fast integer formatting (itoa)
2. ✅ Fast float parsing (simplified atof)
3. ✅ String interning/caching
4. ✅ Type validation caching

**Benchmarking:**
```python
# Compare with standard library json
import json
import time

data = [{"name": f"user{i}", "age": i} for i in range(1000)]

# Standard json
t0 = time.time()
for _ in range(100):
    json.dumps(data)
t1 = time.time()
print(f"json: {t1-t0:.3f}s")

# mini_msgspec
t0 = time.time()
for _ in range(100):
    mini_msgspec.json.encode(data)
t1 = time.time()
print(f"mini_msgspec: {t1-t0:.3f}s")
```

---

## Key Design Decisions for Mini Version

### 1. Parsing Approach

| Approach | msgspec Uses | Recommendation |
|----------|--------------|----------------|
| JSON | Single-pass recursive descent | ✅ Same (simple & fast) |
| MessagePack | Binary reader with type dispatch | ✅ Same (natural fit) |

### 2. Type Validation Strategy

| Approach | msgspec Uses | Recommendation |
|----------|--------------|----------------|
| Validation timing | During decoding | ✅ Same (zero-cost) |
| Type cache | Cache compiled validators | ⚠️ Phase 2 (start simple) |
| Error reporting | Full JSON path | ✅ Same (better UX) |

### 3. Struct Implementation

| Feature | msgspec | Recommendation |
|---------|---------|----------------|
| Storage | C struct with slots | ✅ Same (performance) |
| Inheritance | Supports single inheritance | ❌ Skip initially |
| Defaults | Per-field defaults | ✅ Same (essential) |
| Validators | Custom validators | ❌ Skip (advanced) |

### 4. Number Handling

| Component | msgspec | Recommendation |
|-----------|---------|----------------|
| Integer parsing | Custom (fast path) | ⚠️ Start with `strtoll`, optimize later |
| Float parsing | Custom atof | ⚠️ Start with `strtod`, optimize later |
| Integer formatting | Custom itoa | ✅ Implement early (simple & big win) |
| Float formatting | Ryu algorithm | ❌ Use `snprintf` initially |

### 5. Memory Management

| Feature | msgspec | Recommendation |
|---------|---------|----------------|
| Reference counting | Python's GC | ✅ Same (use Py_INCREF/DECREF) |
| String builder | Doubling growth | ✅ Same (standard approach) |
| Object pooling | For some types | ❌ Skip (premature optimization) |

---

## File Organization for Mini-msgspec

```
mini-msgspec/
├── README.md                     # Project overview
├── LEARNING_GUIDE.md            # This file
├── HISTORICAL_TIMELINE.md       # Maps to msgspec evolution
│
├── docs/
│   ├── adrs/                    # Architecture Decision Records
│   │   ├── 001-json-encoder-design.md
│   │   ├── 002-struct-system.md
│   │   ├── 003-type-validation.md
│   │   └── ...
│   │
│   ├── comparisons/             # Mini vs Real msgspec
│   │   ├── json-encoder-comparison.md
│   │   ├── struct-comparison.md
│   │   └── performance-comparison.md
│   │
│   ├── diagrams/                # Visual architecture
│   │   ├── encoding-pipeline.svg
│   │   ├── type-tree.svg
│   │   └── validation-flow.svg
│   │
│   ├── checkpoints/             # Learning checkpoints
│   │   ├── phase1-checkpoint.md
│   │   ├── phase2-checkpoint.md
│   │   └── ...
│   │
│   └── references/              # msgspec references
│       ├── core-sections.md     # Map of _core.c sections
│       ├── optimization-techniques.md
│       └── papers.md
│
├── src/
│   ├── mini_msgspec/
│   │   ├── __init__.py
│   │   ├── _core.c              # Main C extension
│   │   │   ├── Type system
│   │   │   ├── JSON encoder/decoder
│   │   │   ├── MessagePack encoder/decoder
│   │   │   └── Utilities
│   │   │
│   │   ├── json.py              # Python wrapper
│   │   ├── msgpack.py           # Python wrapper
│   │   └── structs.py           # Python Struct interface
│   │
│   ├── include/
│   │   ├── common.h             # Common definitions
│   │   ├── itoa.h               # Integer formatting
│   │   ├── atof.h               # Float parsing (Phase 2)
│   │   └── types.h              # Type system
│   │
│   └── README.md                # Build instructions
│
├── tests/
│   ├── unit/                    # Unit tests per module
│   │   ├── test_json_encoder.py
│   │   ├── test_json_decoder.py
│   │   ├── test_msgpack.py
│   │   ├── test_struct.py
│   │   └── test_validation.py
│   │
│   ├── integration/             # End-to-end tests
│   │   ├── test_roundtrip.py
│   │   ├── test_validation_errors.py
│   │   └── test_compatibility.py
│   │
│   ├── benchmarks/              # Performance tests
│   │   ├── bench_json.py
│   │   ├── bench_msgpack.py
│   │   └── compare_stdlib.py
│   │
│   └── exercises/               # Learning exercises
│       ├── 01-add-type.md
│       ├── 02-optimize-string.md
│       └── 03-custom-validator.md
│
├── examples/                    # Usage examples
│   ├── 01-basic-json.py
│   ├── 02-structs.py
│   ├── 03-validation.py
│   └── 04-msgpack.py
│
├── tools/
│   ├── visualizer/              # Visualize type trees
│   │   └── show_types.py
│   │
│   └── profiler/                # Performance profiling
│       └── profile.py
│
├── pyproject.toml               # Build configuration
└── setup.py                     # Build script
```

---

## Testing Strategy

### Unit Tests per Component

**1. JSON Encoder Tests:**
```python
def test_encode_primitives():
    assert encode(None) == b'null'
    assert encode(True) == b'true'
    assert encode(42) == b'42'
    assert encode("hello") == b'"hello"'

def test_encode_escaping():
    assert encode("quote\"") == b'"quote\\""'
    assert encode("newline\n") == b'"newline\\n"'

def test_encode_collections():
    assert encode([1, 2, 3]) == b'[1,2,3]'
    assert encode({"a": 1}) == b'{"a":1}'
```

**2. JSON Decoder Tests:**
```python
def test_decode_primitives():
    assert decode(b'null') is None
    assert decode(b'true') is True
    assert decode(b'42') == 42
    assert decode(b'"hello"') == "hello"

def test_decode_numbers():
    assert decode(b'0') == 0
    assert decode(b'-123') == -123
    assert decode(b'3.14') == 3.14
    assert decode(b'1e10') == 1e10
```

**3. Struct Tests:**
```python
def test_struct_creation():
    class Point(Struct):
        x: int
        y: int

    p = Point(x=1, y=2)
    assert p.x == 1
    assert p.y == 2

def test_struct_defaults():
    class User(Struct):
        name: str
        age: int = 0

    u = User(name="Alice")
    assert u.age == 0
```

**4. Validation Tests:**
```python
def test_type_validation():
    class User(Struct):
        age: int

    # Valid
    decode(b'{"age":25}', type=User)

    # Invalid
    with pytest.raises(ValidationError, match="Expected `int`, got `str`"):
        decode(b'{"age":"25"}', type=User)
```

---

## Performance Expectations

| Operation | Python json | msgspec | mini-msgspec (target) |
|-----------|-------------|---------|----------------------|
| Encode dict (1000 items) | 100 ms | 5 ms (20x faster) | 20 ms (5x faster) |
| Decode JSON (1000 items) | 120 ms | 8 ms (15x faster) | 30 ms (4x faster) |
| Struct creation | 50 μs (dataclass) | 3 μs (17x faster) | 10 μs (5x faster) |

**Focus:**
1. **Correctness first** - Pass all test cases
2. **Clarity second** - Readable, well-documented code
3. **Performance third** - Optimize hot paths only

---

## Common Pitfalls to Avoid

### 1. JSON Parsing
- ✅ **DO**: Handle all escape sequences (`\"`, `\\`, `\n`, `\t`, `\uXXXX`)
- ❌ **DON'T**: Forget to validate UTF-8 sequences
- ✅ **DO**: Check for number overflow
- ❌ **DON'T**: Use `atof` directly (locale issues)

### 2. Type Validation
- ✅ **DO**: Provide detailed error paths (`$.users[0].email`)
- ❌ **DON'T**: Just say "validation failed"
- ✅ **DO**: Handle Union types correctly (try each option)
- ❌ **DON'T**: Assume type annotations are always valid

### 3. Memory Management
- ✅ **DO**: Use `Py_INCREF`/`Py_DECREF` consistently
- ❌ **DON'T**: Return borrowed references without incrementing
- ✅ **DO**: Clean up on error paths
- ❌ **DON'T**: Leak memory when exceptions occur

### 4. Struct System
- ✅ **DO**: Use `__slots__` for memory efficiency
- ❌ **DON'T**: Allow arbitrary attribute assignment
- ✅ **DO**: Validate field types during `__init__`
- ❌ **DON'T**: Silently coerce types

### 5. String Handling
- ✅ **DO**: Handle Unicode correctly (UTF-8)
- ❌ **DON'T**: Assume ASCII everywhere
- ✅ **DO**: Escape JSON strings properly
- ❌ **DON'T**: Introduce XSS vulnerabilities

---

## References

### msgspec Source Files

**Main implementation:**
- `/home/user/msgspec/src/msgspec/_core.c` (22,725 lines)
  - Lines 303-450: Utilities (hash, cache, endian)
  - Lines 2747-3500: Type system (TypeNode, StructInfo)
  - Lines 9439-10477: Encoders (JSON, MessagePack)
  - Lines 14771-16678: MessagePack Decoder
  - Lines 16678-19719: JSON Decoder
  - Lines 19719-20498: to_builtins
  - Lines 20498-22164: convert

**Number handling:**
- `/home/user/msgspec/src/msgspec/itoa.h` (190 lines) - Integer to ASCII
- `/home/user/msgspec/src/msgspec/atof.h` (439 lines) - ASCII to float
- `/home/user/msgspec/src/msgspec/ryu.h` (995 lines) - Float to string (Ryu)

**Constants:**
- `/home/user/msgspec/src/msgspec/atof_consts.h` (675 lines) - Lookup tables
- `/home/user/msgspec/src/msgspec/common.h` (23 lines) - Common macros

### Academic Papers & Resources

**Algorithms:**
- Ryu: Fast Float-to-String Conversion (Ulf Adams, 2018)
  - https://dl.acm.org/doi/10.1145/3192366.3192369

**Formats:**
- JSON Specification (RFC 8259)
  - https://tools.ietf.org/html/rfc8259
- MessagePack Specification
  - https://github.com/msgpack/msgpack/blob/master/spec.md

**Python C API:**
- Python/C API Reference Manual
  - https://docs.python.org/3/c-api/index.html

---

## Next Steps

Ready to start building? Here's the recommended path:

### Immediate (Commit 1.1):
1. ✅ Set up project structure
2. ✅ Write ADR-001: Project Overview & Goals
3. ✅ Implement string builder utility
4. ✅ Write tests for string builder
5. ✅ Document design decisions

### This Week (Phase 1):
- Complete JSON encoder (primitives)
- Complete JSON decoder (primitives)
- Add collection support
- Write comprehensive tests
- Document learnings

### This Month (Phases 1-2):
- Finish JSON encoder/decoder
- Implement Struct system
- Add basic type validation
- Compare performance with stdlib

---

## Learning Outcomes

By completing this project, you will deeply understand:

1. **Serialization Formats**
   - JSON structure and parsing
   - MessagePack binary encoding
   - Trade-offs between text and binary formats

2. **Type Systems**
   - Python's type annotation system
   - Runtime type validation
   - Generic types and type variables

3. **Python C Extensions**
   - Python/C API fundamentals
   - Memory management (reference counting)
   - Creating Python types in C
   - Performance optimization techniques

4. **Parsing Techniques**
   - Recursive descent parsing
   - Number parsing algorithms
   - Error recovery and reporting

5. **Performance Engineering**
   - Profiling and benchmarking
   - Hot path optimization
   - Memory allocation strategies
   - Cache-friendly data structures

---

## Questions?

As you work through this project, consider:

1. **Why is msgspec so fast?**
   - What specific optimizations make the biggest difference?
   - Where does stdlib JSON spend most of its time?

2. **How does zero-cost validation work?**
   - Why is validation "free" during decoding?
   - What's the trade-off compared to validating after?

3. **What makes Struct faster than dataclass?**
   - How do slots reduce memory usage?
   - Why is C-level field access faster?

4. **When should you use JSON vs MessagePack?**
   - What are the size differences?
   - What about encoding/decoding speed?

Document your discoveries in the `/docs/learnings/` directory!

---

**Ready to begin? Let's build mini-msgspec! 🚀**
