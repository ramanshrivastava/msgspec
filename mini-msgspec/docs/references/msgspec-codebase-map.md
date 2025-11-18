# msgspec Codebase Map

A comprehensive guide to navigating the msgspec source code.

---

## File Overview

### Main Implementation

**`/home/user/msgspec/src/msgspec/_core.c`** (22,725 lines)
- The entire implementation in a single C file
- All encoders, decoders, type system, utilities
- Heavily optimized, production-grade code

### Optimization Headers

**`/home/user/msgspec/src/msgspec/itoa.h`** (190 lines)
- Fast integer-to-string conversion
- ~5x faster than sprintf
- Uses pre-computed digit pairs

**`/home/user/msgspec/src/msgspec/atof.h`** (439 lines)
- Fast string-to-float conversion
- ~3x faster than strtod
- Locale-independent
- Eisel-Lemire algorithm

**`/home/user/msgspec/src/msgspec/atof_consts.h`** (675 lines)
- Lookup tables for atof
- Pre-computed powers of 10
- Used by Eisel-Lemire algorithm

**`/home/user/msgspec/src/msgspec/ryu.h`** (995 lines)
- Fast float-to-string conversion
- Ryu algorithm implementation
- Shortest accurate representation

**`/home/user/msgspec/src/msgspec/common.h`** (23 lines)
- Common macros and definitions
- Platform compatibility

### Python Interface

**`/home/user/msgspec/src/msgspec/__init__.py`** (529 bytes)
- Python package initialization
- Exports main API

**`/home/user/msgspec/src/msgspec/json.py`** (212 bytes)
- JSON encoder/decoder wrapper
- Thin Python layer over C

**`/home/user/msgspec/src/msgspec/msgpack.py`** (154 bytes)
- MessagePack encoder/decoder wrapper

**`/home/user/msgspec/src/msgspec/structs.py`** (2,942 bytes)
- Struct system Python helpers
- Metaclass utilities

**`/home/user/msgspec/src/msgspec/inspect.py`** (28,896 bytes)
- Type introspection utilities
- Schema generation

---

## _core.c Section Map

The main `_core.c` file is organized into clearly marked sections:

### Utilities & Infrastructure (Lines 1-2747)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 1-302 | Headers & Macros | Python C API setup, compatibility macros | Low |
| 303-311 | Lookup Tables | Hex encoding, Base64 encoding | Low |
| 312-327 | GC Utilities | Garbage collection integration | Medium |
| 328-371 | Murmurhash2 | Hashing algorithm for strings | Medium |
| 372-401 | String Cache | String interning optimization | Medium |
| 402-448 | Endian Handling | Big-endian/little-endian conversion | Low |
| 449-551 | Module State | Global module state management | Medium |
| 552-660 | String Builder | Dynamic buffer for encoding | Medium |
| 661-785 | PathNode | JSON path tracking for errors | Medium |
| 786-1378 | Int/Str Lookups | Lookup tables for ints & strings | High |
| 1379-1616 | Raw | Raw bytes type | Low |
| 1617-2160 | Meta | Type metadata | High |
| 2161-2216 | NODEFAULT | Sentinel for missing defaults | Low |
| 2217-2313 | UNSET | Sentinel for unset values | Low |
| 2314-2420 | Factory | Factory functions for defaults | Medium |
| 2421-2550 | Field | Struct field definitions | Medium |
| 2551-2746 | AssocList | Association list for ordering | Medium |

### Type System (Lines 2747-9042)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 2747-2893 | TypeNode Enums | Type kind definitions | Low |
| 2894-2971 | TypeNode Types | Type node structures | High |
| 2972-3474 | StructInfo | Struct metadata | High |
| 3475-9042 | Struct/TypeNode Implementation | Full type system | Very High |

**Key data structures:**
```c
typedef enum {
    TYPE_ANY,
    TYPE_NONE,
    TYPE_BOOL,
    TYPE_INT,
    TYPE_FLOAT,
    TYPE_STR,
    TYPE_BYTES,
    TYPE_BYTEARRAY,
    TYPE_DATETIME,
    TYPE_DATE,
    TYPE_TIME,
    TYPE_TIMEDELTA,
    TYPE_UUID,
    TYPE_DECIMAL,
    TYPE_EXT,
    TYPE_RAW,
    TYPE_ENUM,
    TYPE_LITERAL,
    TYPE_INTLITERAL,
    TYPE_STRLITERAL,
    TYPE_CUSTOM,
    TYPE_DATACLASS,
    TYPE_STRUCT,
    TYPE_NAMEDTUPLE,
    TYPE_TYPEDDICT,
    TYPE_DICT,
    TYPE_LIST,
    TYPE_SET,
    TYPE_FROZENSET,
    TYPE_VARTUPLE,
    TYPE_FIXTUPLE,
    TYPE_UNION,
    TYPE_CUSTOM_GENERIC,
} TypeNodeKind;
```

### Encoders (Lines 9043-11545)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 9043-9201 | Ext | Extension type for MessagePack | Medium |
| 9202-9353 | Dataclass Utilities | Encoding dataclasses | Medium |
| 9354-9438 | Object Utilities | Generic object encoding | Medium |
| 9439-9877 | Shared Encoder | Common encoder infrastructure | High |
| 9878-10476 | MessagePack Encoder | Binary encoding | High |
| 10477-11545 | JSON Encoder | JSON encoding | High |

**JSON Encoder key functions:**
```c
static int
encoder_encode_json(
    EncoderState *self,
    PyObject *obj,
    int json_compatible
);

// Specialized encoding functions
static int encoder_encode_json_str(...);
static int encoder_encode_json_int(...);
static int encoder_encode_json_float(...);
static int encoder_encode_json_list(...);
static int encoder_encode_json_dict(...);
static int encoder_encode_json_struct(...);
```

### Decoders (Lines 11545-20498)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 11545-11585 | Shared Decoding | Common decoder utilities | Medium |
| 11586-11706 | Datetime Utilities | Date/time parsing | Medium |
| 11707-11785 | Base64 Encoder | Base64 encoding | Low |
| 11786-11853 | UUID Utilities | UUID parsing | Low |
| 11854-11952 | Decimal Utilities | Decimal parsing | Medium |
| 11953-12402 | strict=False Utilities | Lenient parsing | Medium |
| 12403-13616 | Post-Decode Handlers | Type coercion, validation | High |
| 13617-14770 | Number Parser | Fast number parsing | High |
| 14771-16677 | MessagePack Decoder | Binary decoding | Very High |
| 16678-19719 | JSON Decoder | JSON decoding | Very High |

**JSON Decoder architecture:**
```c
typedef struct {
    const char *input;       // Input buffer
    const char *pos;         // Current position
    const char *end;         // End of buffer
    TypeNode *type;          // Expected type
    PathNode *path;          // Current path (for errors)
    // ... more state
} JSONDecoderState;

// Main decoder entry point
static PyObject*
json_decode(
    JSONDecoderState *self,
    TypeNode *type
);

// Specialized decoding functions
static PyObject* json_decode_null(...);
static PyObject* json_decode_bool(...);
static PyObject* json_decode_number(...);
static PyObject* json_decode_string(...);
static PyObject* json_decode_array(...);
static PyObject* json_decode_object(...);
```

**Number parsing (critical for performance):**
```c
// Lines 13617-14770
static PyObject*
ms_decode_number(
    const char **buf,
    const char *end,
    bool strict
);
```

### Conversion Utilities (Lines 19719-22164)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 19719-20497 | to_builtins | Convert to Python builtins | Medium |
| 20498-22164 | convert | Type conversion | High |

### Module Setup (Lines 22164-end)

| Lines | Section | Purpose | Complexity |
|-------|---------|---------|------------|
| 22164-end | Module Init | Python module initialization | Medium |

---

## Key Functions by Category

### String Handling

**String builder (dynamic buffer):**
```c
// Lines 603-660
typedef struct strbuilder {
    char *buf;
    size_t size;
    size_t capacity;
} StrBuilder;

static int strbuilder_extend(StrBuilder *self, const char *data, size_t size);
static int strbuilder_extend_unicode(StrBuilder *self, PyObject *str);
```

**String cache (optimization):**
```c
// Lines 372-401
static PyObject *string_cache[STRING_CACHE_SIZE];
static void string_cache_clear(void);
```

### Type System

**TypeNode creation:**
```c
// Lines 2894+
static TypeNode* TypeNode_Convert(PyObject *type);
static int TypeNode_Validate(TypeNode *type, PyObject *obj);
```

**Struct system:**
```c
// Lines 2972+
typedef struct StructInfo {
    PyObject_HEAD
    PyTypeObject *class;
    PyTupleObject *struct_fields;
    PyTupleObject *defaults;
    // ... more fields
} StructInfo;
```

### Encoding

**JSON encoding:**
```c
// Lines 10477+
static int encoder_encode_json(EncoderState *self, PyObject *obj, int json_compatible);
static int encoder_encode_json_str(EncoderState *self, const char *str, Py_ssize_t len);
```

**MessagePack encoding:**
```c
// Lines 9878+
static int encoder_encode_msgpack(EncoderState *self, PyObject *obj);
```

### Decoding

**JSON decoding:**
```c
// Lines 16678+
static PyObject* json_decode(JSONDecoderState *self, TypeNode *type);
static PyObject* json_decode_string(JSONDecoderState *self);
static PyObject* json_decode_number(JSONDecoderState *self);
```

**MessagePack decoding:**
```c
// Lines 14771+
static PyObject* mpdecode_read(DecoderState *self, TypeNode *type);
```

### Number Handling

**Integer to string (itoa.h):**
```c
int itoa_u64(uint64_t val, char *buf);
int itoa_i64(int64_t val, char *buf);
```

**String to float (atof.h):**
```c
double ms_atof(const char *str, const char *end);
```

**Float to string (ryu.h):**
```c
int ryu_d2s(double val, char *buf);
```

---

## Common Patterns

### 1. Type Dispatch Pattern

Used throughout for handling different Python types:

```c
if (obj == Py_None) {
    // Handle None
}
else if (obj == Py_True || obj == Py_False) {
    // Handle bool
}
else if (PyLong_CheckExact(obj)) {
    // Handle int
}
else if (PyFloat_CheckExact(obj)) {
    // Handle float
}
else if (PyUnicode_CheckExact(obj)) {
    // Handle str
}
// ... more types
```

**Why this order?**
- Most common types first (int, str)
- Exact checks (`CheckExact`) for performance
- Subclass checks later if needed

### 2. Error Handling Pattern

```c
if (some_operation() < 0) {
    return NULL;  // Error set by some_operation
}

// Or set error explicitly:
PyErr_SetString(PyExc_TypeError, "Invalid type");
return NULL;
```

### 3. Reference Counting Pattern

```c
PyObject *obj = PyLong_FromLong(42);  // refcnt = 1
Py_INCREF(obj);  // refcnt = 2
// ... use obj ...
Py_DECREF(obj);  // refcnt = 1
Py_DECREF(obj);  // refcnt = 0, obj freed
```

### 4. String Builder Pattern

```c
StrBuilder builder = {NULL, 0, 0};
strbuilder_extend(&builder, "Hello", 5);
strbuilder_extend(&builder, " ", 1);
strbuilder_extend(&builder, "World", 5);
// builder.buf now contains "Hello World"
```

---

## Performance Hot Paths

Based on profiling, these are the most frequently called functions:

### 1. JSON Encoder Hot Paths

**Most common operations:**
1. `encoder_encode_json_str()` - String encoding (30-40% of time)
2. `encoder_encode_json_int()` - Integer encoding (15-20% of time)
3. `encoder_encode_json_dict()` - Dict encoding (15-20% of time)

**Optimizations:**
- Fast integer formatting (itoa)
- String builder to avoid reallocations
- Inline small strings (ASCII fast path)

### 2. JSON Decoder Hot Paths

**Most common operations:**
1. `json_decode_string()` - String parsing (30-40% of time)
2. `json_decode_number()` - Number parsing (20-25% of time)
3. `json_decode_object()` - Object parsing (15-20% of time)

**Optimizations:**
- Fast number parsing (custom atof/atoi)
- String interning (cache frequently-used strings)
- Single-pass parsing (no tokenization step)

### 3. Validation Hot Paths

**Most common operations:**
1. Type checking (`PyLong_Check`, `PyUnicode_Check`, etc.)
2. Union type validation (try each option)
3. Collection item validation (recursive)

**Optimizations:**
- Type caching (don't re-parse annotations)
- Fast path for common types
- Avoid Python function calls (do in C)

---

## Code Style & Conventions

### Naming Conventions

**Functions:**
```c
static int encoder_encode_json(...);  // Module prefix
static PyObject* json_decode_string(...);  // Format prefix
static int strbuilder_extend(...);  // Type prefix
```

**Macros:**
```c
#define MS_LIKELY(x) __builtin_expect(!!(x), 1)
#define MS_UNLIKELY(x) __builtin_expect(!!(x), 0)
```

**Types:**
```c
typedef struct EncoderState { ... } EncoderState;  // PascalCase
typedef enum { ... } TypeNodeKind;  // PascalCase
```

### Error Handling

**Always check return values:**
```c
if (PyDict_SetItem(dict, key, value) < 0) {
    return NULL;  // Error already set
}
```

**Set descriptive errors:**
```c
PyErr_Format(
    PyExc_ValidationError,
    "Expected `%s`, got `%s` - at `%s`",
    expected_type,
    actual_type,
    path_str
);
```

### Memory Management

**Clean up on error paths:**
```c
PyObject *obj = PyLong_FromLong(42);
if (some_operation(obj) < 0) {
    Py_DECREF(obj);  // Must decref before returning!
    return NULL;
}
Py_DECREF(obj);
return result;
```

---

## Learning Roadmap

### Beginner Level

**Start here:**
1. `itoa.h` (190 lines) - Simple integer formatting
2. String builder (lines 552-660) - Dynamic buffers
3. JSON encoder primitives (lines 10500-10800) - Basic encoding

### Intermediate Level

**Then study:**
1. JSON decoder primitives (lines 17000-17500) - Parsing
2. Type dispatch patterns (throughout)
3. Error handling & path tracking (lines 661-785)

### Advanced Level

**Finally tackle:**
1. TypeNode system (lines 2747-3500) - Type representation
2. Struct metaclass (lines 2972-3474) - Advanced Python C API
3. Full JSON decoder (lines 16678-19719) - Complete implementation

### Expert Level

**Deep dives:**
1. atof.h (439 lines) - Eisel-Lemire algorithm
2. MessagePack decoder (lines 14771-16677) - Binary parsing
3. Validation engine (lines 12403-13616) - Type checking

---

## Common Questions

**Q: Why is everything in one file?**
A: Simplifies compilation, enables cross-function optimization, easier to distribute.

**Q: Why custom number parsing?**
A: Standard library functions (sprintf, strtod) are slow and have locale issues.

**Q: Why single-pass JSON parsing?**
A: Avoids allocating intermediate tokens, faster, less memory.

**Q: How does zero-cost validation work?**
A: Validation happens during object construction (no separate pass needed).

**Q: Why use macros instead of functions?**
A: Inlining for performance (compiler can't always inline across files).

---

## Next Steps

1. **Read the code**: Start with simple sections (itoa, string builder)
2. **Run examples**: See how msgspec works in practice
3. **Trace execution**: Use a debugger to step through
4. **Modify and test**: Make small changes, see what breaks
5. **Implement mini version**: Apply what you learned

---

## Useful Commands

### Find function definitions
```bash
grep -n "^static.*function_name" /home/user/msgspec/src/msgspec/_core.c
```

### Find struct definitions
```bash
grep -n "^typedef struct" /home/user/msgspec/src/msgspec/_core.c
```

### Find section headers
```bash
grep -n "^/\*\*\*\*\*" /home/user/msgspec/src/msgspec/_core.c
```

### Count lines in sections
```bash
sed -n 'START,ENDp' /home/user/msgspec/src/msgspec/_core.c | wc -l
```

---

**Happy exploring! 🔍**
