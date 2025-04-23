# Criterion Rule Specification

Criterion is a Generic Rule Engine. This is the public specification of the rule format.

## Introduction

The Criterion Rule Format is expressed in JSON, and includes two main sections: input and logic. The input is the declaration of the Variables that are needed by the rule's logic. The logic is a sequence of Blocks that define what the rule does.

```json
TODO: example
```


## Basic

Criterion Rules have three basic operations: variable declaration, value assignment and the combination of both in the same operation, which we call declare-assign or dec-assign.

### Variable Declaration

This example declares a `myVar` variable that has type `integer`. In general the declarations are used to define the input variables of a rule. Refer to the [Crtierion Data Types](#criterion-data-types) section to see the possible types a variable can have.

```json
{
  "var": "myVar",
  "type": "integer"
}
```

### Value Assignment

The Criterion Rule Format defines four types of values that can be assigned to variables: Literal, Value Reference, Return Literal and Data Source Literal.

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

TBD



### Declare Assignment

This operation is the combination of Variable Declaration and Value Assignment. The values assigned can be of the four types mentioned in the previous section: Literal, Value Reference, Return Literal and Data Source Literal.

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

TBD



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

TBD
```json

```


### List Block

Defines a list of blocks that can be used by the rule or by other blocks. For instance the rule's `logic` section is a List Block. Another example is the If-Else's `then` and `else` blocks are List Blocks.

```json
[
  block1,
  block2,
  ...
]
```

### If-Else Block

TBD

### Data Source Block

TBD



## Functions

TBD

### Logic

TBD

### Arithmetic

TBD

### Strings

TBD

## Criterion Data Types

TBD
