# Real Benchmark Results - Employee Data

## Benchmark Overview

This document contains **real, measured benchmarks** with valid syntax for all formats.

**Methodology**: All formats use their standard/optimal encoding:
- JSON: Standard `indent=2` (development) and minified (production)
- YAML: 2-space indent (standard)
- TOON: YAML-style for nested objects (per specification)
- GBLN: No whitespace (production format) and pretty-printed (development only)

---

## 100 Employee Records - Complete Comparison

### Test Dataset

- **Files**: `employees-100.*`
- **Records**: 100 employees with complete HR information
- **Complexity**: Nested structures (address object, certifications array of objects)
- **Data types**: Strings, numbers, booleans, dates, arrays, nested objects
- **Realism**: Randomized but realistic data (names, cities, skills, certifications, dates)

### Production Comparison (Wire/Cache/LLM)

| Format | Bytes | vs JSON | Type-Safe | Memory-Bounded |
|--------|-------|---------|-----------|----------------|
| **GBLN** | **49,276** ⭐ | **0.4% smaller** ⭐ | ✅ | ✅ |
| JSON (minified) | 49,466 | baseline | ❌ | ❌ |
| TOON | 53,601 | 8.4% larger | ❌ | ❌ |
| YAML | 57,701 | 16.6% larger | ❌ | ❌ |

**Critical Finding**: GBLN is **190 bytes smaller** than minified JSON (0.4%) **AND includes type safety + memory bounds**.

### Development Comparison (Pretty-Printed)

| Format | Bytes | vs JSON | Notes |
|--------|-------|---------|-------|
| TOON | 53,601 | -32% | YAML-style, 2-space indent |
| YAML | 57,701 | -27% | 2-space indent |
| **GBLN (formatted)** | **66,273** | **-17%** ⭐ | 2-space indent + type hints |
| JSON (indent=2) | 79,394 | baseline | Standard `indent=2` |

**Key Insight**: GBLN formatted is 17% smaller than JSON **while including type information**.

### The Real Story: Type Safety for Free

**JSON minified**: 49,466 bytes
- ✅ Smallest text format (without types)
- ❌ No type validation
- ❌ No memory bounds
- ❌ Runtime errors only

**GBLN** (production format): 49,276 bytes  
- ✅ **Smaller than JSON** (190 bytes / 0.4%)
- ✅ **Parse-time type validation** (`id<u32>` validates 0-4B)
- ✅ **Bounded strings** (`s32` = max 32 chars)
- ✅ **Memory predictable** (no buffer overflows)
- ✅ **Catches LLM hallucinations** at parse-time

**Conclusion**: GBLN gives you type safety + memory bounds **for free** - actually 0.4% cheaper than unsafe JSON.

**Important**: GBLN (49,276 bytes) is the actual format. Pretty-printed GBLN (66,273 bytes) is only for development/editing - never sent over wire or cached.

---

## Why GBLN is Smallest

### 1. Arrays Without Quotes/Commas

**JSON**:
```json
"skills": [
  "MongoDB",
  "DevOps",
  "Machine Learning",
  "Node.js"
]
```
= 109 characters (with formatting)

**GBLN**:
```gbln
skills<s32>[MongoDB DevOps Machine Learning Node.js]
```
= 60 characters (45% smaller!)

### 2. Field Names Without Quotes

**JSON**: `"firstName": "Alice"`  
**GBLN**: `firstName<s32>(Alice)`

JSON wastes 6 characters on quotes (`" "` around key, `" "` around value)

### 3. Booleans as Single Char

**JSON**: `"active": true,` = 16 chars  
**GBLN**: `active<b>(t)` = 13 chars

### 4. No Commas Between Fields

**JSON**: Every field ends with `,` (except last)  
**GBLN**: Whitespace-separated, no commas needed

### 5. GBLN Format Has No Structural Whitespace

**JSON minified**: Still has `,` and `:` separators  
**GBLN**: Zero whitespace except inside `()` values (the actual format)

**Note**: "GBLN formatted" or "GBLN pretty-printed" is just for humans during development - adds optional whitespace for readability.

---

## Format-Specific Analysis

### JSON (79,394 bytes readable, 49,466 bytes minified)

**Strengths**:
- Universal support
- Minified is very compact
- Fast parsers everywhere

**Weaknesses**:
- No type safety
- No memory bounds
- Quotes everywhere (overhead)
- Commas required (overhead)
- Runtime validation only

### YAML (57,701 bytes)

**Strengths**:
- Human-readable
- No quotes on simple values

**Weaknesses**:
- Indentation-sensitive
- Slow parsers
- Many gotchas (octal, booleans)
- Cannot minify (indentation = structure)
- No type safety

### TOON (53,601 bytes)

**Strengths**:
- Good for flat/uniform data
- CSV-style compression

**Weaknesses**:
- Falls back to YAML-style for nested data (this benchmark)
- No type safety
- No memory bounds
- Cannot compress further (uses indentation)

**Why TOON didn't win here**: This dataset has nested `address` objects and `certifications` arrays of objects. TOON falls back to YAML-style, losing its CSV compression advantage.

### GBLN (49,276 bytes production, 66,273 bytes formatted)

**Strengths**:
- **Smallest production format** (0.4% smaller than JSON minified)
- **Type safety included** (no extra cost!)
- **Memory bounds included**
- Arrays without quotes/commas
- Optional formatting: compact (production) or pretty-printed (development)
- Deterministic parsing (3 simple rules)

**Weaknesses**:
- New format (smaller ecosystem than JSON)
- Type hints add size when pretty-printed (development only)

---

## Real-World Context

### For Production Systems

**Bandwidth Savings**: 0.4% vs JSON minified (~insignificant)

**BUT - Added Value at Zero Cost**:
- ✅ **Type validation** - Catch errors at parse-time, not runtime
- ✅ **Memory bounds** - `s32` = max 32 chars, `u32` = 0-4B range
- ✅ **LLM validation** - Reject hallucinated data immediately
- ✅ **Deterministic** - 3 simple rules, O(1) lookahead
- ✅ **Progressive** - Types optional for prototyping

**The Real Win**: Not file size, but **reliability and safety**.

### For LLM Contexts

**Token count** (approximate via word count):

| Format | Words (wc -w) | vs JSON |
|--------|---------------|---------|
| GBLN minified | TBD | TBD |
| JSON minified | TBD | baseline |

*(To be measured with proper tokenizer)*

**Key Point**: At 100 records, file size differences are marginal. Token efficiency matters more for **large datasets** (1000+ records) or **repeated transfers**.

---

## The TOON vs GBLN Trade-Off

### When TOON Would Win

TOON excels with **flat, uniform data** (CSV-style):

```toon
employees[1000]{id,firstName,lastName,email,salary}:
  1,Alice,Johnson,alice@company.com,75000
  2,Bob,Schmidt,bob@company.com,68000
```

Schema declared once, values as CSV rows = very compact.

### When GBLN Wins (This Benchmark)

GBLN excels with **nested, complex data**:

```gbln
employees[{id<u32>(1)firstName<s32>(Alice)address{street<s64>(Main St)city<s32>(Berlin)}certifications[{name<s64>(AWS Cert)date<s16>(2020-01-01)}]}]
```

Compresses completely while preserving structure, types, and bounds.

---

## Honest Conclusions

### File Size Winner

**For nested data (this benchmark):**
1. 🥇 **GBLN** - 49,276 bytes (0.4% smaller than JSON, production format)
2. 🥈 **JSON minified** - 49,466 bytes (baseline for production)
3. 🥉 **TOON** - 53,601 bytes (8.4% larger vs JSON)

**For flat/uniform data:**
- TOON would likely win with CSV-style compression
- Not tested in this benchmark

**Important Note**: GBLN at 49,276 bytes **is** the format. Pretty-printed GBLN (66,273 bytes) is only for development.

### Real Winner: GBLN for Reliability

File size is nearly identical to JSON minified (~0.4% difference).

**GBLN's true advantages**:
- ✅ **Type safety** - Parse-time validation vs runtime crashes
- ✅ **Memory bounds** - Predictable allocation, no OOM surprises  
- ✅ **LLM validation** - Catch hallucinated data immediately
- ✅ **Deterministic** - 3 rules, O(1) lookahead, easy to implement
- ✅ **Progressive** - Types optional for prototyping

**Use GBLN when**: Reliability, type safety, and memory bounds matter (production systems, LLM outputs, constrained environments).

**Use JSON when**: Maximum ecosystem compatibility is required.

**Use TOON when**: Data is flat/uniform and token costs are critical.

**Use YAML when**: Human-editable config files (accept the gotchas).

---

## Benchmark Files

### 100 Records
- `employees-100.json` - JSON formatted (79,394 bytes, indent=2, development)
- `employees-100-minified.json` - JSON minified (49,466 bytes, production)
- `employees-100.yaml` - YAML (57,701 bytes)
- `employees-100.toon` - TOON (53,601 bytes)
- `employees-100.gbln` - **GBLN** (49,276 bytes, production format) ⭐
- `employees-100.pretty.gbln` - GBLN formatted (66,273 bytes, development only)

### 5 Records (Initial Benchmark)
- `employees.json` - JSON formatted (4,431 bytes, development)
- `employees.yaml` - YAML (3,535 bytes)
- `employees.toon` - TOON (3,326 bytes)
- `employees.gbln` - **GBLN** (3,138 bytes, production format) ⭐
- `employees.pretty.gbln` - GBLN formatted (4,757 bytes, development only)

**File Naming Convention**:
- `*.gbln` = Production format (compact, no whitespace)
- `*.pretty.gbln` = Development format (formatted for humans)
- `*.json` = Development format (indent=2)
- `*-minified.json` = Production format

All files use **valid, optimised syntax** for their respective formats.

---

**Last Updated**: 2025-01-22  
**Primary Dataset**: 100 employee records with nested address and certifications  
**Methodology**: Fair comparison using standard encoding for each format
