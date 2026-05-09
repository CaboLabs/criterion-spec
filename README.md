# Criterion Rule Specification

Criterion is a Generic Rule Engine. This is the public specification of the rule format.

## Introduction

The Criterion Rule Format is expressed in JSON, and includes two main sections: input and logic. The input is the declaration of the Variables that are needed by the rule's logic. The logic is a sequence of Blocks that define what the rule does.

```json
{
  "name": "Sample Rule",
  "input": [
    {
      "var": "input1",
      "type": "string"
    },
    {
      "var": "input2",
      "type": "integer"
    }
  ],
  "logic": [
    ...
  ]
}
```


## Basic

Criterion Rules have three basic operations: variable declaration, value assignment and the combination of both in the same operation, which we call declare-assign or dec-assign.

### Variable Declaration

This example declares a `myVar` variable that has type `integer`. In general the declarations are used to define the input variables of a rule. Refer to the [Criterion Data Types](#criterion-data-types) section to see the possible types a variable can have.

```json
{
  "var": "myVar",
  "type": "integer"
}
```


### Arguments

The values that can be assigned or used as function parameters are called Argument. Arguments can have four types:

1. Literal: represents a simple constant value of a certain type like the integer `2` or the string `"hellow world"`.
2. Value Reference: represents the reference to an existing variable by its name, and always starts with `$`, for instance `$myVar`.
3. Return Literal: represents the value returned by a function, and which can be assigned to a variable.
4. Data Source Literal: represents a value that is extracted and retrieved from a Data Source (JSON, XML, etc).


### Value Assignment

As mentioned in the previous section, the Criterion Rule Format defines four types of values (called Arguments) that can be assigned to variables: Literal, Value Reference, Return Literal and Data Source Literal. The following examples show how to assign each type of Argument.

#### Literal Value Assignment

Literals are constant values given "inline", like numbers and strings. In this example, `2` is a Literal assigned to variable `myVar`. Note the `$` prefix indicates that is a reference to an existing (declared) variable.

```json
{
  "$myVar": 2
}
```

#### Value Reference Assignment

A Value Reference is a reference to an existing variable and represents the access to that variable's value. In this example we are assigning the value of variable `anotherVar` to the variable `myVar`.

```json
{
  "$myVar": "$anotherVar"
}
```

#### Return Literal Assignment

A Return Literal represents a value that is returned by calling a function. In this example the function is `+`, the normal arithmetic addition, that adds two values: the value of the variable `anotherVar` and the literal `2`.

```json
{
  "$myVar": {
    "+": ["$anotherVar", 2]
  }
}
```

#### Data Source Literal Assignment

The Data Source Literal represents a value that is extracted from a Data Source, like a JSON or XML document.

In this example, we reference the "patient" [Data Source](#data-source-block), and extract an exact data point using JSON Path, the patient's gender, and assign that to the patient_sex` variable:

```json
{
  "$patient_sex": {
    "source": "@patient",
    "extract": {
      "jsonpath": [
        "$.gender"
      ]
    },
    "aggregate": "first",
    "transform": "noop"
  }
}
```


### Declare Assignment

This operation is the combination of Variable Declaration and Value Assignment. The values that can be assigned are [Arguments](#arguments) as mentioned on the previous sections.

#### Dec-Assign Literal

Note for the assignment, the key `=` is used. Then the other two keys are exactly the same to a Variable Declaration (`var` and `type`).

```json
{
  "var": "myVar",
  "type": "integer",
   "=": 2
}
```

#### Dec-Assign Value Reference

This example declares the variable `myVar` and assigns the value of `anotherVar` to it.

```json
{
  "var": "myVar",
  "type": "integer",
  "=": "$anotherVar"
}
```

#### Dec-Assign Return Literal

This example declares the variable `myVar` and assigns the value returned by the `+` function to it.

```json
{
  "var": "myVar",
  "type": "integer",
  "=": {
    "+": ["$anotherVar", 2]
  }
}
```

#### Dec-Assign Data Source Literal

This example extracts the value form the `patient` Data Source, and assigns it to the variable `sex`.

```json
{
  "var": "sex",
  "type": "string",
  "=": {
    "source": "@patient",
    "extract": {
      "jsonpath": [
        "$.gender"
      ]
    },
    "aggregate": "first",
    "transform": "noop"
  }
}
```



## Logic Blocks

### Return Block

The Return Block is used to return a value from the rule, and stop the rule execution. It's like the `return` keyword from most programming languages. The types of values that can be returned are: literal, value reference, return literal and data source literal.


#### Return Block Literal

In this example, the rule would return a boolean value `true`.

```json
{
  "return": true
}
```

#### Return Block Value Reference

In this example the rule would return the current value of `anotherVar`.

```json
{
  "return": "$anotherVar"
}

```

#### Return Block Return Literal

```json
{
  "return": {
    "<": ["$value1", 1000]
  }
}
```

#### Return Block Data Source Literal

This example returns a value extracted from the `patient` Data Source.

```json
{
  "return": {
    "source": "@patient",
    "extract": {
      "jsonpath": [
        "$.gender"
      ]
    },
    "aggregate": "first",
    "transform": "noop"
  }
}
```

### Block List

Defines a list of blocks that can be used by the rule or by other blocks. For instance the rule's `logic` section is a Block List. Another example is the If-Else's `then` and `else` blocks are Block Lists. In this example we have a Block List with two blocks: a Dec-Assign and a Return block.

```json
[
  {
    "var": "a",
    "type": "integer",
    "=": 1
  },
  {
    "return": "$a"
  }
]
```

### If-Else Block

This is a flow-control block, that checks a boolean condition, and when the condition evaluates to `true`, the blocks in the `then` Block List are executed, otherwise the bloks in the `else` Block List are executed.

```json
{
  "if": "$is_pending",
  "then": [
    {
      "return": 555
    }
  ],
  "else":[
    {
      "return": 890
    }
  ]
}
```

### Data Source Block

A Data Source Block describes the access to an external resource throw some kind of Data Access (HTTP, FILE, DB, etc.) to retrieve and cache some type of data (JSON, XML, HTML, CSV, etc.) that can be then accessed via a Data Source Literal to extract values from the Data Source.

In this example, we are retrieving a JSON document via an HTTP GET request, and call that Data Source "patient". Check the (Data Source Literal Assignment)[data-source-literal-assignment] section to check how to access this Data Source. For instance, there could be two Data Source Literals that extract the patient's sex and date of birth from the same Data Source.

```json
{
  "source": "patient",
  "type": "JSON",
  "access": {
    "type": "http",
    "url": "https://my-fhir-server/Patient/1234",
    "method": "GET"
  }
}
```



## Functions

Functions can receive 0..N input parameters, which will be a type of [Argument](#arguments) and will return a Literal Argument. In general, the result of a Function will be assigned to a Variable or used as input for another Function.


### Logic

Logic functions are boolean functions (return true or false), and have boolean [Arguments](#arguments) as input.

#### And

The result is `true` if both input [Arguments](#arguments) are `true`, otherwise the result will be `false`. In this example, we show the `and` function being called using boolean Literal Arguments `true` and `false`, though any of the inputs can be any type of Argument (Literal, Variable Reference, Return Literal or Data Source Literal).

```json
{
  "&&": [true, false]
}
```

#### Or

Returns `true` if at least one of the two input Arguments is `true`, otherwise returns `false`.

```json
{
  "||": [true, false]
}
```

#### Not

Returns `true` if the input Argument is `false`, and `false` if the input Argument is `true`. Takes exactly one boolean Argument.

```json
{
  "!": true
}
```

#### Xor

Returns `true` if exactly one of the two input Arguments is `true`. If both are `true` or both are `false`, returns `false`.

```json
{
  "xor": [true, false]
}
```




### Comparison

The input values for the comparison should be `comparable`, that is:

1. Are of the same data type (can't compare a number to a string).
2. Have an order relationship.

Note that all comparison functions are boolean functions (return `true` or `false`).

#### Lower Than

This function will return `true` if the value represented by the first Argument is lower than the value represented by the second Argument, otherwise it will return `false`.

```json
{
  "<": [1, 2]
}
```

#### Greater Than

Returns `true` if the value of the first Argument is greater than the value of the second Argument, otherwise returns `false`.

```json
{
  ">": [1, 2]
}
```

#### Equals To

Returns `true` if both Arguments represent the same value. Both Arguments must be of the same type.

```json
{
  "==": [1, 2]
}
```

#### Not Equals To / Different Than

Returns `true` if the Arguments have different values or are of different types, otherwise returns `false`.

```json
{
  "!=": [1, 2]
}
```

#### Lower Or Equals

Returns `true` if the value of the first Argument is lower than or equal to the value of the second Argument, otherwise returns `false`.

```json
{
  "<=": [1, 2]
}
```

#### Greater Or Equals

Returns `true` if the value of the first Argument is greater than or equal to the value of the second Argument, otherwise returns `false`.

```json
{
  ">=": [1, 2]
}
```




### Arithmetic

Arithmetic functions operate on numeric Arguments (`integer` and `decimal`). When both inputs are `integer` the result is `integer`, except for Division which always returns `decimal`. If either input is `decimal`, the result is `decimal`.

> **Precision note:** Numbers with a decimal point are represented as the native `decimal` type of the host platform (e.g. `BigDecimal` in Java, `Decimal` in Python), not as IEEE 754 floating-point. This guarantees exact decimal arithmetic and avoids precision loss.

#### Increment

Increments the value "in place" and returns it. If the input Argument is a Variable Reference, it will increment the value of that variable.

```json
{
  "++": "$myNumber"
}
```


#### Decrement

Decrements the value "in place" and returns it. If the input Argument is a Variable Reference, it will decrement the value of that variable.

```json
{
  "--": "$myNumber"
}
```

#### Addition

Returns the sum of two numeric Arguments.

```json
{
  "+": [1, 2]
}
```

#### Subtraction

Returns the result of subtracting the second Argument from the first.

```json
{
  "-": [1, 2]
}
```

#### Division

Returns the result of dividing the first Argument by the second. Always returns `decimal`. The second Argument must not be zero.

```json
{
  "/": [1, 2]
}
```

#### Multiplication

Returns the product of two numeric Arguments.

```json
{
  "*": [1, 2]
}
```
#### Modulus

Returns the remainder of dividing the first Argument by the second. Both Arguments must be `integer`.

```json
{
  "%": [1, 2]
}
```


### Strings

String functions operate on `string` Arguments. Unless otherwise noted, they return a `string`.

#### Concat

Returns a new string by joining two or more string Arguments together.

```json
{
  "concat": ["$firstName", " ", "$lastName"]
}
```

#### Length

Returns the number of characters in a string as an `integer`.

```json
{
  "length": "$description"
}
```

#### Trim

Returns the string with leading and trailing whitespace removed.

```json
{
  "trim": "$rawInput"
}
```

#### ToUpper

Returns the string with all characters converted to uppercase.

```json
{
  "toUpper": "$code"
}
```

#### ToLower

Returns the string with all characters converted to lowercase.

```json
{
  "toLower": "$email"
}
```

#### Contains

Returns `true` if the first string Argument contains the second string Argument as a substring, otherwise returns `false`.

```json
{
  "contains": ["$body", "error"]
}
```

#### StartsWith

Returns `true` if the first string Argument starts with the second string Argument, otherwise returns `false`.

```json
{
  "startsWith": ["$url", "https"]
}
```

#### EndsWith

Returns `true` if the first string Argument ends with the second string Argument, otherwise returns `false`.

```json
{
  "endsWith": ["$filename", ".json"]
}
```

#### Substring

Returns a portion of the string starting at the index given by the second Argument and ending before the index given by the third Argument. Indices are zero-based.

```json
{
  "substring": ["$text", 0, 5]
}
```

#### Replace

Returns a new string with all occurrences of the second Argument replaced by the third Argument.

```json
{
  "replace": ["$template", "{name}", "$userName"]
}
```

## Criterion Data Types

Every variable and function result in Criterion has one of the following types. The type determines what values a variable can hold and what operations are valid on it.

| Type | Description | JSON literal example |
|---|---|---|
| `boolean` | A truth value | `true` or `false` |
| `integer` | A whole number, positive or negative | `42`, `-7` |
| `decimal` | A number with a decimal part, represented with exact precision | `3.14`, `-0.5` |
| `string` | A sequence of characters | `"hello"` |
| `date` | A calendar date, encoded as an ISO 8601 string | `"2024-01-15"` |
| `datetime` | A date and time with timezone, encoded as ISO 8601 | `"2024-01-15T10:30:00Z"` |
| `array` | An ordered collection of values of the same type | `[1, 2, 3]` |
| `object` | An untyped key-value map | `{"key": "value"}` |

Declaration examples:

```json
{ "var": "active",     "type": "boolean"  }
{ "var": "age",        "type": "integer"  }
{ "var": "ratio",      "type": "decimal"  }
{ "var": "name",       "type": "string"   }
{ "var": "dob",        "type": "date"     }
{ "var": "created_at", "type": "datetime" }
{ "var": "scores",     "type": "array",   "items": "integer" }
{ "var": "payload",    "type": "object"   }
```

Arrays require an `items` field that specifies the type of each element. The `object` type is schema-less and can hold any key-value structure.


---

## Candidate Features for v1

The following features are proposed for inclusion in v1 of the specification. Each item includes a description and example. Items marked with an open decision need a choice before the spec can be finalised.

---

### Rule Metadata

Add `id`, `description`, and `version` fields to the top-level rule object to support tooling, registries, and versioning.

**Open decision:** Which fields are required vs optional? Recommendation: `name` required, others optional.

```json
{
  "id": "eligibility-check-us-v1",
  "name": "US Eligibility Check",
  "description": "Returns true if the person is 18 or older and located in the US",
  "version": "1.0.0",
  "input": [
    { "var": "age", "type": "integer" },
    { "var": "country", "type": "string" }
  ],
  "logic": [...]
}
```

---

### Output Declaration

Add an `output` field to the top-level rule to declare the return type. Gives consumers and implementors a type contract for what the rule returns.

**Open decision:** Required or optional?

```json
{
  "name": "Is Eligible",
  "input": [
    { "var": "age", "type": "integer" }
  ],
  "output": { "type": "boolean" },
  "logic": [
    { "return": { ">=": ["$age", 18] } }
  ]
}
```

---

### Else-If / Chained Conditions

Two options for handling chained conditions without deeply nested if-else blocks.

**Open decision:** Add explicit `elseif` key (Option A), or document nested `if` inside `else` as the canonical pattern (Option B, no new syntax)?

**Option A — `elseif` key:**

```json
{
  "if": { "<": ["$score", 50] },
  "then": [{ "return": "fail" }],
  "elseif": [
    {
      "condition": { "<": ["$score", 70] },
      "then": [{ "return": "pass" }]
    }
  ],
  "else": [{ "return": "distinction" }]
}
```

**Option B — nested `if` inside `else` (no new syntax):**

```json
{
  "if": { "<": ["$score", 50] },
  "then": [{ "return": "fail" }],
  "else": [
    {
      "if": { "<": ["$score", 70] },
      "then": [{ "return": "pass" }],
      "else": [{ "return": "distinction" }]
    }
  ]
}
```

---

### Iteration Blocks

#### forEach

Iterates over each element of an array variable, binding the current element to a named variable for use inside the `do` block.

```json
{
  "forEach": "$items",
  "as": "item",
  "do": [
    { "$total": { "+": ["$total", "$item"] } }
  ]
}
```

#### while

Repeats the `do` block while the boolean condition is `true`.

```json
{
  "while": { "<": ["$count", 10] },
  "do": [
    { "$count": { "++": "$count" } }
  ]
}
```

---

### Action Blocks

Data Source Blocks are read-only. Action Blocks send or write data out.

**Open decision:** Scope v1 to HTTP only, or include DB and FILE?

#### HTTP Action

```json
{
  "action": "http",
  "method": "POST",
  "url": "https://api.example.com/notifications",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "$authToken"
  },
  "body": "$payload",
  "result": "$httpResponse"
}
```

#### DB Action

```json
{
  "action": "db",
  "operation": "insert",
  "connection": "main",
  "table": "audit_log",
  "values": {
    "rule_id": "$ruleId",
    "outcome": "$result"
  },
  "result": "$insertedId"
}
```

---

### Variable Scope Rules

Variables declared inside `then`, `else`, or `do` blocks are local to that block. Variables declared in the top-level `logic` block are accessible from all nested blocks.

```json
{
  "if": "$isAdmin",
  "then": [
    { "var": "level", "type": "integer", "=": 10 }
  ],
  "else": [
    { "var": "level", "type": "integer", "=": 1 }
  ]
}
```

`level` in `then` and `level` in `else` are separate variables in separate scopes.

---

### Additional Data Source Access Types

Currently only HTTP GET is specified. Proposed additional access types:

#### FILE

```json
{
  "source": "config",
  "type": "JSON",
  "access": {
    "type": "file",
    "path": "/etc/rules/config.json"
  }
}
```

#### DB (read query)

```json
{
  "source": "patient_record",
  "type": "object",
  "access": {
    "type": "db",
    "connection": "main",
    "query": "SELECT * FROM patients WHERE id = ?",
    "params": ["$patientId"]
  }
}
```

---

### Data Source Literal — `aggregate` and `transform` Values

The valid values for `aggregate` and `transform` are not yet enumerated.

**`aggregate`** — controls how to reduce multiple extraction results:

| Value | Meaning |
|---|---|
| `first` | Use the first match |
| `last` | Use the last match |
| `all` | Return all matches as an `array` |
| `count` | Return the number of matches as `integer` |
| `sum` | Sum all numeric matches |
| `min` | Minimum of numeric matches |
| `max` | Maximum of numeric matches |

**`transform`** — post-extraction value conversion:

| Value | Meaning |
|---|---|
| `noop` | No transformation, use value as-is |
| `toString` | Convert to `string` |
| `toInt` | Parse as `integer` |
| `toDecimal` | Parse as `decimal` |
| `toBoolean` | Parse as `boolean` |
| `trim` | Remove leading/trailing whitespace |
| `toLower` | Convert to lowercase |
| `toUpper` | Convert to uppercase |

Example using non-default options:

```json
{
  "$totalScore": {
    "source": "@results",
    "extract": { "jsonpath": ["$.scores[*]"] },
    "aggregate": "sum",
    "transform": "noop"
  }
}
```

---

### jsonpath — Single vs Multi-Path

Currently `jsonpath` is always an array, but the multi-path semantics (try each path in order, use first non-null) are not defined.

**Open decision:** Support multi-path array (try each path as a fallback), or simplify to a single string?

**Multi-path (try each in order, use first non-null result):**

```json
{
  "$sex": {
    "source": "@patient",
    "extract": {
      "jsonpath": ["$.gender", "$.sex", "$.administrativeGender"]
    },
    "aggregate": "first",
    "transform": "noop"
  }
}
```

**Single-path (plain string, simpler):**

```json
{
  "$sex": {
    "source": "@patient",
    "extract": {
      "jsonpath": "$.gender"
    },
    "aggregate": "first",
    "transform": "noop"
  }
}
```

---

### Null / Missing Value Handling

Define behaviour when a variable is unassigned or an extraction returns no match.

Proposal: failed extraction returns `null`. Operations on `null` produce a runtime error unless a `default` is specified.

**`default` field on Data Source Literal:**

```json
{
  "$age": {
    "source": "@patient",
    "extract": { "jsonpath": ["$.age"] },
    "aggregate": "first",
    "transform": "toInt",
    "default": 0
  }
}
```

**`isNull` function:**

```json
{
  "if": { "isNull": "$age" },
  "then": [{ "return": false }],
  "else": [{ "return": { ">=": ["$age", 18] } }]
}
```

---

### Error Handling

**Open decision:** Rule-level error propagation only (simpler, recommended for v1), or explicit try/catch blocks?

**Option A — error propagation (no new syntax):** Rule execution stops and returns a structured error object to the caller. Implementors handle it at the host level.

Error object shape:

```json
{
  "error": {
    "code": "DATA_SOURCE_UNAVAILABLE",
    "message": "HTTP GET to patient endpoint returned 503",
    "source": "patient"
  }
}
```

**Option B — try/catch block:**

```json
{
  "try": [
    {
      "source": "patient",
      "type": "JSON",
      "access": { "type": "http", "url": "https://my-fhir-server/Patient/1234", "method": "GET" }
    },
    { "$sex": { "source": "@patient", "extract": { "jsonpath": ["$.gender"] }, "aggregate": "first", "transform": "noop" } }
  ],
  "catch": [
    { "return": false }
  ]
}
```

---

### Variable Naming Rules

Proposed rules for valid variable names:

- Pattern: `[a-zA-Z_][a-zA-Z0-9_]*`
- Case-sensitive: `myVar` and `MyVar` are distinct variables
- Maximum length: 64 characters
- The `$` prefix is a reference marker, not part of the name itself

Valid: `age`, `patient_id`, `isActive`, `_temp`, `x1`

Invalid: `1count` (starts with digit), `my-var` (hyphen not allowed), `my var` (space not allowed)

---

### Type Coercion Rules

**Open decision:** Strict typing (no implicit coercion) or permissive (automatic widening)?

Recommendation for v1: strict typing. No implicit coercion. Assigning a `decimal` to an `integer` variable is a runtime error. Use explicit `transform` in Data Source Literals or string conversion functions for type changes.

```json
{ "var": "age", "type": "integer", "=": 3.14 }
```

The above must produce a runtime error, not silently truncate to `3`.
