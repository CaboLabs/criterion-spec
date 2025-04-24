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

This example declares a `myVar` variable that has type `integer`. 7In general the declarations are used to define the input variables of a rule. Refer to the [Criterion Data Types](#criterion-data-types) section to see the possible types a variable can have.

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



## Logic BLocks

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
    "=": "1"
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
      "return": "555"
    }
  ],
  "else":[
    {
      "return": "890"
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
TBD
```json
{
  "||": [true, false]
}
```

#### Not
TBD

```json
{
  "!": [true, false]
}
```

#### Xor
TBD

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
TBD

```json
{
  ">": [1, 2]
}
```

#### Equals To
TBD


```json
{
  "==": [1, 2]
}
```

#### Not Equals To / Different Than
TBD

```json
{
  "!=": [1, 2]
}
```

#### Lower Or Equals
TBD

```json
{
  "<=": [1, 2]
}
```

#### Greater Or Equals
TBD

```json
{
  ">=": [1, 2]
}
```




### Arithmetic

TBD

#### Increment

Increments the value "in place" and returns it. If the input Argument is a Variable Reference, it will increment the value of that variable.

```json
{
  "++": $myNumber
}
```


#### Decrement

TBD

```json
{
  "--": $myNumber
}
```

#### Addition
TBD

```json
{
  "+": [1, 2]
}
```

#### Subtraction
TBD
```json
{
  "-": [1, 2]
}
```

#### Division
TBD
```json
{
  "/": [1, 2]
}
```

#### Multiplication
TBD

```json
{
  "*": [1, 2]
}
```
#### Modulus
TBD

```json
{
  "%": [1, 2]
}
```


### Strings

TBD

## Criterion Data Types

TBD
