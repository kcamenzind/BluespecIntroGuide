# Intro Bluespec User Guide

- [Overview](#overview)
- [Bluespec Syntax](#bluespec-syntax)
  * [Capitalization](#capitalization)
  * [Whitespace and comments](#whitespace-and-comments)
  * [Semicolons and blocks](#semicolons-and-blocks)
- [Bluespec Variables, Types, and Operators](#bluespec-variables-types-and-operators)
  * [Data Types](#data-types)
    + [Literals](#literals)
    + [Predefined Types](#predefined-types)
    + [User-defined types](#user-defined-types)
    + [Type conversions](#type-conversions)
    + [Working with Bits](#working-with-bits)
  * [Operators](#operators)
    + [Bitwise operators](#bitwise-operators)
    + [Logical operators](#logical-operators)
    + [Ternary operator](#ternary-operator)
    + [Arithmetic operators](#arithmetic-operators)
    + [Numeric type operators](#numeric-type-operators)
- [Combinational Circuits](#combinational-circuits)
  * [Variables](#variables)
    + [Variable declaration](#variable-declaration)
    + [Variable assignment](#variable-assignment)
    + [let keyword](#let-keyword)
    + [Order of execution](#order-of-execution)
  * [Function structure](#function-structure)
    + [Function Declaration](#function-declaration)
    + [Parameterization](#parameterization)
    + [Higher-level programming constructs](#higher-level-programming-constructs)
    + [Return statements](#return-statements)
    + [Note on Calling Functions](#note-on-calling-functions)
- [Sequential Circuits](#sequential-circuits)
  * [Interfaces](#interfaces)
    + [Method Types](#method-types)
    + [Empty Interface](#empty-interface)
  * [Modules](#modules)
    + [Module Declaration](#module-declaration)
    + [Internal State](#internal-state)
    + [Methods and Rules](#methods-and-rules)
- [Additional Topics](#additional-topics)
  * [Maybe values](#maybe-values)
  * [Case matches](#case-matches)


## Overview

This document is an introductory guide to learning Bluespec. It's intended for 6.004 students, and is structured mostly in the order that the class is taught. It will cover the basic language syntax, data types, and how to write combinational circuits (functions) and sequential circuits (modules).

This isn't a complete (or official!) reference for the Bluespec language, so it's missing a lot of advanced topics and may have mistakes. If you do notice any mistakes or want to contribute content, feel free to open a pull request or issue, or just shoot me an email at kcamenzi@mit.edu.

I've listed some other resoures to learn Bluespec below:

* [Bluespec Manual](http://www.bluespec.com/forum/download.php?id=157): A comprehensive guide of the language.
* [Bluespec Reference Card](http://www.bluespec.com/forum/download.php?id=98): A short document (3 pages) good for looking up syntax.
* [Online tutorial](http://wiki.bluespec.com/Home): An online walk-through of some of the common features of Bluespec, along with examples.

If you just want to briefly review Bluespec syntax or quickly look up a particular piece of syntax, you may want to consult the [Quick Reference](BluespecQuickReference.md).

## Bluespec Syntax

### Capitalization

Capitalization is important in Bluespec, and your program will not compile if you do not follow the capitalization conventions. The required capitalization of the first letter is as follows:

**Foo**: Type names, Typeclass names, Interface names, Enum labels, 
Tagged union labels, Package names

**foo**: bit, int, module names, instance names, all variables, 
all type variables, rule names

### Whitespace and comments

Whitespace can be used freely in Bluespec!

Comments are treated as whitespace, and can either be one-line comments:

`// Your comment here`

or multiline comments
```bluespec
/* You can
write comments
across however many
lines! */
```

### Semicolons and blocks

Semicolons are needed after any expression. They are not needed, however, after **begin** and **end** keywords. Here are some examples for your reference:

```bluespec
// Needs semicolon
x = 5;

// Needs semicolon
if (y) x = 5;

// Whitespace doesn't matter, this is still 1 expression = 1 semicolon
if (y)
    x = 5;

// Semicolons not needed after keywords begin and end
if (y) begin
    x = 5;
end

// For loops work similarly to if statements
for (Integer i = 0; i < max; i = i + 1) begin
    do_something();
    do_something_else();
end

// Function declarations need semicolons, end statements do not.
function ReturnType fnName(Type var1, Type var2);
    some_stuff();
endfunction

// Module declarations need semicolons, endmodule does not
module mkMyModule();
    // State declarations also need semicolons
    Reg#(Bit#(n)) myReg <- mkRegU;

    // Same thing with rules
    rule doSomething;
        do_some_stuff();
    endrule

    // And with methods!
    method Type myMethod();
        do_some_other_stuff();
    endmethod
endmodule

```

The keywords `begin` and `end` are how we can lump multiple statements into one statement, mostly used in conditionals, loops, and case statements. For example, to assign two variables in an if statement, we need to write:

```bluespec
// Correct syntax
if (cond) begin
    x = 1;
    y = 2;
end

// Incorrect syntax, y is not part of the conditional
// and will always be assigned 2
if (cond)
    x = 1;
    y = 2;
``` 

## Bluespec Variables, Types, and Operators

### Data Types

Bluespec has both built-in data types and user-defined data types. No matter what data type you're using in your code, when it gets synthesized to hardware everything is just stored as bits. However, types allow us to focus on the value of our variables rather than how they'll translate into bits.

#### Literals

When we write programs, we often have to assign hard-coded numeric values to variables. It's good practice to write all Bluespec literals with explicit sizes, but it is possible to write both sized and unsized literals. Unsized literals are most useful when you want set a variable (or some section of a variable) to all 0's or all 1's, or when using Integers (since the Integer type is unsized anyway, more on that below).

**Sized literals:**
```bluespec
4'd10       // Decimal value 10, stored in 4 bits

4'b1010     // Decimal value 10, stored in 4 bits
8'b00001010 // Decimal value 10, stored in 8 bits
8'b1010     // Decimal value 10, stored in 8 bits

4'ha        // Hex value 10, stored in 4 bits
8'h0a       // Hex value 10, stored in 8 bits
8'h0A       // Capitalization of hex digits doesn't matter
``` 

**Unsized literals:**
```bluespec
10  // Decimal value 10, unsized
0   // Decimal value 0, unsized
1   // Decimal value 1, unsized
'0  // Enough 0's to fill the needed width
'1  // Enough 1's to fill the needed width

```

We need to be careful when using `'0` or `'1` or the compiler will be unhappy. Only use these when both the size of the variable is defined, and the size of the space we're filling with 0's or 1's is unambiguous.

#### Predefined Types

Almost all variables in Bluespec represent, in the eventual circuit, some number of bits. The exception to this is the Integer type, which is only used in static elaboration. This means that you cannot have Integer be the type of your inputs or outputs; it's exclusively used by the compiler, for example, as a loop variable.

Here are some of Bluespec's built-in types:

```bluespec
Bit#(n)  // n bits
Int#(n)  // n bits, interpreted as a signed number
UInt#(n) // n bits, interpreted as an unsigned number
Bool     // True or False (1 bit)
Integer  // unsized number, only used in static elaboration
```

##### Tuples

Tuples are built-in types that are made up of other types that you specify. For example, `Tuple2#(Bit#(1), Bit#(2))` is the type of a 2-tuple that contains a `Bit#(1)` and a `Bit#(2)`.

Tuples can be constructed with the special functions `tuple2`, `tuple3`, and so on:

```bluespec
Tuple2#(Bit#(1), Bit#(2)) pair = tuple2(1, 0);
```

To access individual elements of tuples, you can use the special functions `tpl_1`, `tpl_2`, and so on. For example `tpl_1(pair)` gets the first element from the tuple we constructed above, which would be 1. You can also use pattern-matching to get all values from a tuple at once:

```bluespec
match {.a, .b} = pair;
```

##### Other Types

There are some built-in types that will be explained in the sequential section.

#### User-defined types

##### Type synonyms

You can give types new names with the following syntax:

```bluespec
typedef OldType NewType;
```

For example, you may want to rename `Bit#(8)` as `Byte`.

```bluespec
typedef Bit#(8) Byte;
```

After this, you can write `Byte` instead of `Bit#(8)`. (You can also keep using the old `Bit#(8)` name.)

##### Structs

You can also define your own types by defining a new type that is made up of other types. The syntax is:

```bluespec
typedef struct {
    OldType1 member1;
    OldType2 member2;
} NewType;
```

You can instantiate this variable and access its fields as follows:

```bluespec
OldType1 m1 = 2'b00;
OldType2 m2 = some_value;

// Declare the variable
NewType myNewVar = NewType{member1: m1, member2: m2};

// Read a field
OldType2 m2_copy = myNewVar.m2;

// Set a field
myNewVar.m1 = 2'b11;
```

Similar to tuples, you can also get all fields from a struct with pattern matching as follows:

```bluespec
match tagged NewType {m1: .myM1, m2: .myM2} = myNewVar;
// you can use variables myM1 and myM2 here
```

##### Enums

Enums are how we can define custom types that are defined by the compiler as bits, but we don't explicitly have to understand how they translate into bits. For example, if you want to define a type Color, which can take values Red, Green, Yellow and Blue, we can write

```bluespec
typedef enum Color { Red, Green, Yellow, Blue } deriving (Bits, Eq);
```

Deriving `Bits` means that the values `Red`, `Green`, `Yellow` and `Blue` will be automatically assigned an underlying representation in bits. Deriving `Eq` means that the equality operator is derived for the type as well, so if you have a `Color` variable, you can check if it's `Red` or `Green` or `Yellow` or `Blue` using the `==` comparator. You will generally want to include both of these in your enum declarations.

#### Type conversions

##### Converting between numeric types, Integers, and Bits

To extract the Integer value of a numeric type, use `Integer i = valueOf(n)`, where `n` is the numeric type.

To convert an Integer to a `Bit#(n)` value, use `Bit#(n) x = fromInteger(i)`, where `i` is an `Integer`.

You can chain these together to store the value of a numeric type into a `Bit#(n)` as follows:

```bluespec
Bit#(m) x = fromInteger(valueOf(n));
```

(Note: `n` and `m` could be the same value, they're just different numeric type names to illustrate that they don't have to be the same value.)

Lastly, you can extract the size of a Type (in bits) by using `SizeOf`. `SizeOf` returns a numeric type, which then you can then further convert into an `Integer` or `Bit#` depending on your use case. For example:

```bluespec
Bit#(3) x = 0; // Size of x is 3 bits
Integer i = valueOf(SizeOf(x)); // i = 3, converted from numeric type -> Integer
```

##### Converting between Bits and other types

If a type is represented by `Bits`, then we can convert between these types and their bit representations. Examples of this are `Int`, `UInt`, and any user-defined type deriving `Bits`.

To convert from `Bit` to any other type, use `unpack`:

```bluespec
Bit#(3) x = 3'b101;     // x is the binary value 101
UInt#(3) y = unpack(x); // y = 5, the UInt represented by the bits 101
Int#(3) z = unpack(x);  // z = -3, the Int represented by the bits 101
```

To convert from any type to `Bit`, use `pack`:

```bluespec
typedef enum Color { Red, Green, Blue, Yellow } deriving (Bits, Eq);

Color red = Red;
Color yellow = Yellow;

Bit#(2) x = pack(red);    // x = 2'b00, the binary representation of Red
Bit#(2) y = pack(yellow); // y = 2'b11, the binary representation of Yellow
```

#### Working with Bits

##### Indexing Bits

`Bit#`s are stored as a string of bits, indexed from the least significant bit (LSB) to the most significant bit (MSB). That is, more significant bits correspond to higher indices. Note that this is **backwards** from what you might expect based on how binary literals are normally written, since the literals go from most significant bits to least significant bits! (It's done this way so that `x[i]` in a binary number corresponds nicely to 2<sup>i</sup>.) For example, if you have a `Bit#(4) x = 4'b1010`, then `x[0] = 0` (the LSB or rightmost bit), `x[1] = 1`, `x[2] = 0`, and `x[3] = 1` (the MSB or leftmost bit). In general, for a `Bit#(n)`, we can index it from `0` to `n-1`. When you index a `Bit#`, you get a `Bit#(1)`.

To access a parameterized Bit, we can use the numeric type -> `Integer` conversion discussed above. For example:

```bluespec
Bit#(n) x_param = 1; // x = 1, has n-1 leading zeros
Bit#(1) x_msb = x_param[valueOf(n)-1]; // x_msb is the top bit of x_param
```

We can also take slices of `Bit#`s with the syntax `x[hi:lo]`. This means to get the bits from `hi` to `lo` **inclusive**, and the first index `hi` should be greater than or equal to the second index `lo`. The result will have type `Bit#(hi - lo + 1)` (if you do the math, you'll see that `hi - lo + 1` is just the number of bits sliced). For example:

```bluespec
Bit#(4) x = 4'b1010;
Bit#(4) y = x;      // y = 4'b1010
Bit#(4) z = x[3:0]; // z = 4'b1010
Bit#(3) lower_bits = x[2:0]; // lower_bits = { x[2], x[1], x[0] } = 3'b010;
Bit#(3) upper_bits = x[3:1]; // upper_bits = { x[3], x[2], x[1] } = 3'b101;
```

Note that with bit indexing, it's generally recommended to use constants as the indices (or `Integer`s, since they're elaborated to constants at compile-time). It's generally ok to extract bits with a variable, but requires more hardware.

Some examples:

```bluespec
Integer fixed_i = 2;   // Value of fixed_i known at compile time because it's an Integer
Bit#(2) dynamic_i = 3; // Value of dynamic_i not known at compile time because it's a Bit#
Bit#(4) x = 4'b1001;

// Indexing
Bit#(1) a = x[fixed_i];   // OK, fixed_i is a fixed value
Bit#(1) b = x[fixed_i-1]; // OK, fixed_i-1 is a fixed value
Bit#(1) c = x[dynamic_i]; // OK but inefficient, dynamic_i isn't a fixed value

// Slicing
Bit#(2) d = x[i:i-1];                 // OK, fixed-size and fixed-value slice
Bit#(2) e = x[dynamic_i:dynamic_i-1]; // OK but inefficient, fixed-size but non-fixed-value slice
Bit#(2) f = x[fixed_i:0];             // ONLY OK if i=1 to guarantee sizes match
Bit#(2) g = x[dynamic_i:0];           // BAD, no guarantee that sizes will match
```

When indexing with dynamic values, there's also always the danger of the indexing out of range. For example, if you have a `Bit#(3)`, you have to index with at least 2 bits to cover values `2'b00`, `2'b01`, and `2'b10`. However, if the index takes the value `2'b11`, then this is out of the range of the bit string.

*Additional Note on Bit slicing*

While it's legal to use operators in the expressions for indexing (for example, `x[i-1:i-2]`), the compiler doesn't type check on slices with operators. In that example, it would be unable to determine the size of the slice. (In fact, even if you wrote `x[3-1:0]`, it would be unable to determine the size of the slice.) So be particularly careful that you match your slice width to the assigned variable widths. If there is a size mismatch, the compiler will truncate the left-most bits, or pad on the left with 0's.

##### Concatenating Bits

We can combine strings of bits into longer strings of bits. To concatenate two (or more) `Bit#`s, surround them with curly braces `{ }` and separate by commas, as in the following notation:

```bluespec
Bit#(2) a = 2'b11;
Bit#(3) b = 3'b001;

// Concatenation
Bit#(5) c = { x, y }; // c = 5'b11001;

// It's ok to use slices
Bit#(4) d = { x, y[2:1] }; // d = 4'b1100;

// It's ok to write bits explicitly
Bit#(5) e = { 1'b0, x, 2'b00 }; // e = 5'b01100;
```

##### Extending Bits

Sometimes it can be useful to add arbitrary numbers of 0's or 1's to the beginning or end (usually beginning) of Bits to change the size but retain the numeric value. The first way that we can do this is by concatenating our original number with `'1` or `'0`, which represent "as many 0's or 1's as needed to fill the specified width".

For example:

```bluespec
Bit#(4) x = 4'b1001;

// Extend x to 6 bits
Bit#(6) a = { '0, x }; // a = 6'b001001;
Bit#(7) b = { '0, x }; // b = 7'b0001001;
Bit#(7) c = { '1, x }; // c = 7'b1111001;
Bit#(7) d = { x, '0 }; // d = 7'b1001000;
Bit#(7) e = { x, '1 }; // e = 7'b1001111;

Bit#(7) f = { '0, x, '1 }; // NOT ALLOWED, not clear how many 0's vs 1's to fill
```

We can also accomplish the functionality through two built-in Bluespec functions, zeroExtend and signExtend.

* `zeroExtend(x)`: equivalent to `{ '0, x }`
* `signExtend(x)`:
    * equivalent to `{ '0, x }` if the MSB of x is 0
    * equivalent to `{ '1, x }` if the MSB of x is 1

##### Truncating Bits

We can truncate bits by explicitly indexing the number of bits that we want. However, we can also use the built-in truncate function. For example:

```bluespec
Bit#(5) x = 5'b10011;
Bit#(4) y = truncate(x); // y = 4'b0011
Bit#(2) z = truncate(x); // z = 2'b11
```

##### Other Bit Functions

Bluespec has several more functions for working with bits, but it's unlikely you will need them.

```bluespec
function Bit#(1) parity(Bit#(n) v); // even or odd number of 1's
function Bit#(n) reverseBits(Bit#(n) v);
function UInt#(lgn1) countOnes(Bit#(n) bin) provisos (...); // number of 1's
function UInt#(lgn1) countZerosMSB(Bit#(n) bin) provisos (...); // number of 0's from MSB until first 1
function UInt#(lgn1) countZerosLSB(Bit#(n) bin) provisos (...); // number of 0's from LSB until first 1
function Bit#(n) truncateLSB(Bit#(m) x) provisos (...); // truncate from LSB
```

### Operators

There are several built-in operators for built-in Bluespec types.

#### Bitwise operators

Bitwise operators operate bit-by-bit on numbers, including `Bit#(n)`, `Int#(n)`, and `UInt#(n)`. If the operator takes two arguments, so `a = b OP c`, then this is equivalent to writing `a[i] = b[i] OP c[i]` for every i from 0 to n-1. (Notice that a, b, and c must all be the same size.)

- `&`: bitwise-AND
- `|`: bitwise-OR
- `^`: bitwise-XOR
- `~`: bitwise-NOT

```bluespec
Bit#(4) a = 4'b0011;
Bit#(4) b = 4'b0101;
Bit#(4) c = a & b; // c = 4'b0001;
Bit#(4) d = a | b; // d = 4'b0111;
Bit#(4) e = a ^ b; // e = 4'b0110;
Bit#(4) f = ~a;    // f = 4'b1100;
```

You cannot use bitwise operators on booleans; booleans have their own logical operators.

#### Logical operators

The AND, OR, and NOT bitwise operators have logical equivalents. Logical operators perform the same operations as the bitwise operators, but they take two boolean arguments (`True` or `False`) and produce a boolean result.

- `&&`: logical AND
- `||`: logical OR
- `!`: logical NOT

```bluespec
Bool a = True;
Bool b = False;

Bool d = a || b; // d = True since a == True
Bool e = a && b; // e = False since b == False
Bool f = !a;     // f = False since a != False
```

There is no separate logical XOR operator, but the not-equals operator `!=` has the exact behavior of logical XOR.

#### Ternary operator

The ternary statement mimics the behaviour of a multiplexer, and is shorthand for an if-else statement. The expression `(cond) ? val1 : val2` evaluates to `val1` if `cond==True`, and `val2` if `cond==False`. The cond must evaluate to the `Bool` (**not `Bit#(1)`**) type.

Example:
```bluespec
Bit#(1) s = 1'b0;
Bit#(2) a = 2'b11;
Bit#(2) b = 2'b01;

Bit#(2) x = (s==1'b0) ? a : b; // x = a since s==1'b0
Bit#(2) y = (s==1'b1) ? a : b; // y = b since s!=1'b1
```

#### Arithmetic operators

You can use arithmetic operators on many types. If the operator does different things depending on whether the number is signed or unsigned, then you probably want to specify the value explicitly as an Int or UInt.

`a + b` : Addition

`a - b` : Subtraction

`a * b` : Multiplication

`a / b` : Division

`a % b` : Modulus

`a << b` : Left shift

`a >> b` : Right shift

TODO: Describe the rules for what happens when there are size mismatches, sign mismatches, etc.

##### Comparators

`a <= b` : Less than or equal to

`a < b`  : Less than

`a >= b` : Greater than or equal to

`a > b ` : Greater than

`a == b` : Equals

`a != b` : Not equals


#### Numeric type operators

When we have parameterized types, we sometimes want to define variable widths based on some function of the the parameter width. There are built-in functions for doing basic arithemetic operations on numeric types.

```bluespec
Bit#(n) x; Bit#(m) y; // x is n bits wide, y is m bits wide

Bit#(TAdd#(n, m)) a; // a is n + m bits wide
Bit#(TAdd#(n, 1)) b; // b is n + 1 bits wide
Bit#(TSub#(n, 1)) c; // c is n - 1 bits wide
Bit#(TLog#(n))    d; // d is log(n) bits wide (ceiling)
Bit#(TExp#(n))    e; // e is 2^n bits wide
Bit#(TMul#(n, m)) f; // f is n*m bits wide
Bit#(TDiv#(n, m)) g; // g is n/m bits wide 

Bit#(n+1) h;            // ILLEGAL: Cannot use + operator with numeric type n
Bit#(valueOf(n) + 1) i; // ILLEGAL: Bit parameter needs to be a type, not an Integer
```

## Combinational Circuits

Now we know how to define types and variables and write basic expressions. The next question is, how we can put these expressions together into code that actually does something?

In Bluespec, we can create representations of combinational (stateless and unclocked) circuits by writing functions. The function inputs are the inputs to the combinational circuit, the function outputs are the outputs from the combinational circuit, and the body of the function describes the combinational logic that converts the inputs to outputs. This section describes the basic components needed to write Bluespec functions.

### Variables

#### Variable declaration

Variables are declared in code as follows:

`TypeName variableName;`

Variables must be declared in the function definition, or within the function. You cannot declare a global variable in a file, as there is no such thing as a "global variable" in hardware, since the function itself should encapsulate an entire combinational circuit. (This isn't entirely true once we move onto sequential circuits, but more on that later.)

This also means that the scope of a declared variable is only within the function that it is declared in.

#### Variable assignment

Variables must be assigned values to be used. Generally, you will include an initial value in its declaration. If you don't, then you should always make sure that the variable is assigned a value before it's used.

```bluespec
TypeName variableName = initialValue; // Initializes variableName to initialValue

// Alternate way of assigning initalValue
TypeName var1;
if (cond1) var1 = init1;
else var1 = init2;

// BAD
TypeName var2;
if (cond) var2 = init1;
...
y = f(var2); // var2 doesn't have a value if cond=False
```

#### let keyword

It's good practice to explicitly declare your variable sizes. However, you can also allow the compiler to infer the type instead of writing it explicitly, by using the keyword `let` instead of declaring a variable type.

For example

```bluespec
Bit#(5) a = 0;
let b = a;           // b will be a Bit#(5)
let c = { 1'b0, a }; // c will be a Bit#(6) since we added a bit to a
let d = 2'b11;       // d will be a Bit#(2)

// INVALID
let e = { '0, a }; // size of e can't be determined since '0 is unsized
let f = 1;         // size of f can't be determined since 1 is unsized

```

#### Order of execution

One thing to note is that in these functions, statements execute like they would in many other programming languages: top to bottom. Re-assigning a variable another value will update its value for and only for future statements.

```bluespec
Bit#(2) x = 2'b10; // x = 2'b10
Bit#(2) y = x;     // y = 2'b10
x = 2'b11;         // x = 2'b11, y = 2'b10
Bit#(2) z = x;     // z = 2'b11
```

### Function structure

Now that we know how to write variables in our function, let's talk about how the function is actually structured. A function consists of a declaration, variables, body, return value, and potentially some parameterization.

#### Function Declaration

A function is declared as followed:

```bluespec
function ReturnType functionName(ArgType1 argName1, ArgType2 argnName2, ... , ArgTypeN argNameN);
    // Body of function here
endfunction
```

A function can return exactly 1 value. If you want to return multiple values, then you can pack them into a [tuple](#tuples) or a user-defined [struct](#structs) and extract the separate values from the values or fields of the return value.

`function` and `endfunction` are Bluespec keywords that define the beginning and end of the function declaration.

You can pass in any number of arguments to a function, including 0. The names you give to the arguments are the variable names for accessing the input values in the body of the function. Again, the scope of these variables is only inside the function.

Below is an example declaration of a 4-bit adder function. The function take two 4-bit numbers (`a` and `b`) and a carry-in 1-bit value (`c`), and returns a 5-bit number that is equal to a+b+c.

```bluespec
function Bit#(5) add4(Bit#(4) a, Bit#(4) b, Bit#(1) c);
    // body
endfunction
```

#### Parameterization

It's possible that you want to write multiple functions that do the exact same thing, but for different Bit widths. For example, in the adder example above, you might want to have an add2, add4, add8, and add16 function. Instead of rewriting the function for every Bit width, we can often generalize it by parameterizing the function, and then specifying when we pass in arguments to the function what value we want the parameter to take.

We parameterize the function by replacing certain numeric types with variables. For our adder example, we could generalize it by writing

```bluespec
function Bit#(TAdd#(n,1)) addN(Bit#(n) a, Bit#(n) b, Bit#(1) c);
    // body
endfunction
```

This says that inputs `a` and `b` will be Bits of size n, c is a Bit of size 1, and the function will return a Bit of size n+1. (If you don't remember how the `TAdd#` function works, refer to the section on [numeric type operators](#numeric-type-operators).)

You can then actually call your function by just passing in arguments of compatible bit widths. This can be done in several ways, as shown below.

```bluespec
// Here are your variables to add
Bit#(4) a, b;
Bit#(1) c;

...
... // assume variables are initialized to values somewhere :)
...

// You can call the general function as long sizeOf(a) = sizeOf(b) = sizeOf(sum) - 1
Bit#(5) sum = addN(a, b, c);

// The compiler can infer the size of the return type from the size of the inputs
let sum = addN(a, b, c);

// Alternatively, you can declare a specifically parameterized function
function Bit#(5) add4(Bit#(4) a, Bit#(4) b, Bit#(1) c);
    return addN(a, b, c);
endfunction

// Can now call the specific add4 function
Bit#(5) sum = add4(a, b, c);

// Again, the compiler can determine the size of the return type
let sum = add4(a, b, c);

// INVALID: The first two arguments need to be the same width
let sum = addN(a, c, c);

// INVALID: the return type needs to be 1 bit wider than the arguments
Bit#(6) sum = addN(a, b, c);
```

#### Higher-level programming constructs

##### For loops

You can add for loops to your program! Syntax is as follows:

```bluespec
// General syntax
for (Type iter_val = initial_val; cond; iter_val = f(iter_val)) begin
    // Stuff to do in for loop.
    // Loop will continue if cond==True, and will apply f(iter_val) at the end of every loop cycle
end

// Example for loop. Will initialize i to 0, and then execute as long as i < 10,
// with i incrementing at the end of every loop execution.
for (Integer i = 0; i < 5; i = i + 1) begin
    count = count + i;
end 

// Example for loop in a parameterized function (where n is a numeric type).
for (Integer i = 0; i < valueOf(n); i = i + 1) begin
    // Do something
end
```

One thing to note is that loops are *unrolled at compile-time*. This means that what the second for-loop above actually does is the following:

```bluespec
i = 0;              // i = 0
count = count + i;
i = i + 1;          // i = 1
count = count + i;
i = i + 1;          // i = 2
count = count + i;
i = i + 1;          // i = 3
count = count + i;
i = i + 1;          // i = 4
count = count + i;
```

Another thing to note is the use of the `Integer` type in the for loop. We use `Integer`s because they're unsized so we don't have to worry about if we're using enough bits. At the same time, since the loop is unrolled, it's ok to use an Integer because `i` won't ever actually change values in the compiled circuit, it just becomes a hard-coded constant for each iteration of the loop.

We generally want to keep the bounds of our `for` loop to a constant, because otherwise our circuit has to unroll every possible iteration of the for loop and put a mux on every iteration deciding whether that iteration is the final one or not. In code:

```bluespec
Bit#(5) max = get_max(); // value of max is unknown at compile time

// BAD: Compiler has to unroll 2^5 loops and then dynamically
// decide after which iteration to take the value of res
for (Integer i = 0; i < max; i = i + 1)
    res = f(res);
end
```

##### If-else statements

If-else statements are just like any other language.

```bluespec
if (cond1) begin
    // Will execute if cond1==True
end else if (cond2) begin
    // Will execute if cond1==False and cond2==True
end else begin
    // Will execute if cond1==False and cond2==False
end

// We can also one-line these statements if only one action needs to happen.
if (cond1) doSomething;
else if (cond2) doSomethingElse;
// Default else statement is optional.
```

##### Case

The `case` statement is a shorthand way of writing long `if`/`else` blocks. The syntax is as follows:

```bluespec
Type switch = some_val;

// This case conditionally executes statements based on which value switch matches.
// The default value executes if no other value is matches, and is not always needed.
case (switch)
    val1: do1();    // do1() executes if (switch==val1)
    val2: begin     // do2() and do3() execute if (switch==val2)
        do2();
        do3();
    end
    default: do4(); // do4() executes if (switch!=val1) && (switch!=val2)
endcase

// This case conditionally sets to a value based on which value switch matches.
let x = case (switch)
    val1: xval1;           // x = xval1 if (switch==val1)
    val2: xval2;           // x = xval2 if (switch==val2)
    val3: (xval1 + xval2); // Need to wrap multi-term expressions in parentheses
    default: xval3;        // x = xval3 if (switch!=val1) && (switch!=val2)
endcase;                   // Note the semicolon here
```

#### Return statements

`return` statements specify the return value of your function. You can only have return statements at the very end of your function (there can't be any statements after them, not even other return statements), although they can be at the end of branches in an if-else conditional. In addition, you must have a return statement at the end of every path of execution, so there cannot be a possible path where your function will not return a value.

```bluespec
// Best to put your return value at the end.
function ReturnType fnName(args...);
    ReturnType res;
    if (cond1) res = val1;
    else res = val2;
    return res;
endfunction

// Also ok to return from every branch of an if-else
function ReturnType fnName(args...);
    if (cond1) return val1;
    else return val2;
endfunction

// BAD: the return statements in the if/else come before the return statement at the very end
// (This is easy to fix by just adding `else` before `return val3;`)
function ReturnType fnName(args...);
    if (cond1) return val1;
    else if (cond2) return val2;
    return val3;
endfunction

// BAD: if cond1==False and cond2==False, no return statement
function ReturnType fnName(args...);
    if (cond1) return val1;
    else if (cond2) return val2;
endfunction
```

#### Note on Calling Functions

You can call a function from within another function, or from within a module's rule or method (to be explained in the next function). Every time you write a function call, it generates a new instance of that combinational circuit; there's no sharing of an instance of a function across separate calls. This means if you have the following code, generating an instance of `myOtherFunc` will have in it two instances of `myFunc`.

```bluespec
function ReturnType1 myFunc(ArgType arg1);
    // Some stuff
endfunction

function ReturnType myOtherFunc();
    if (cond1) myFunc(val1);
    else myFunc(val2);
endfunction
```

## Sequential Circuits

Up to this point, we've only talked about writing code to generate circuits that have no concept of time or state. We'll now take a look at how we can use Bluespec to describe sequential circuits, which are cycle-driven (by an implicit clock—we're going to skip in this guide talking about designs that use multiple clocks) and can store state across cycles.

### Interfaces

Interfaces define the inputs and outputs to class of sequential circuits. An interface consists of 1 or more method declarations, where each method defines a subset of the inputs and outputs to the circuit, and has a specific function. The exact implementation of the function is not defined in the interface, it will be defined later by a module (talked about in the next section) that implements the interface.

```bluespec
// Basic interface declaration
interface InterfaceName;
    method MethodType method1name(ArgType1 arg1, ArgType2 arg2 ... );
    method MethodType method2name(ArgType3 arg1, ArgType4 arg2 ... );
    ...
    method MethodType methodNname(); // methods don't have to have inputs
endinterface
```

You can also parameterize interfaces as shown below. This can be a parameterization similar to function we've seen where we want to use the interface for varying Bit widths, but can also be used for things like FIFOs where you want to have an interface that describes all FIFOs regardless of the data type stored in it.

```bluespec
// Parameterized interface declaration
interface ParamInterfaceName#(type typeName); // type is a keyword, typeName is your name for the type
    method MethodType regularMethodName;
    method MethodType#(typeName) paramMethodName; // Pass the type parameter into methods as needed
endinterface
```

#### Method Types

There are three different types of methods. The type of method defines some implicit inputs/outputs to the sequential circuit, specifically whether there is an enable signal and whether there is a return value (output).

All methods have an implicit *ready* signal. This signal tells the outside world when the sequential circuit is in a valid state for the method to be called. Some circuits may have their ready signals always set to True, but that's unrelated to the interface, so more on that later.

##### Action Methods

Action methods alter the internal state of the circuit, which means that there is an enable signal. When the method is called, the enable signal will go high, which tells the circuit to change its state based on its current state and the method inputs. An Action method is declared as follows:

```bluespec
method Action actionMethodName(ArgType1 arg1, ArgType2 arg2...); // Can have 0 or more args
```

You can call such methods as:

```bluespec
module.actionMethodName(arg1, arg2, ...);
```

##### Value Methods

Value methods do not alter the internal state of the circuit, so there's no enable signal because nothing in the circuit needs to change when the method is called. Instead, value methods just output some value generated in the circuit. Value methods are declared as follows:

```bluespec
method ReturnType valueMethodName(ArgType1 arg1, ArgType2 arg2...); // Can have 0 or more args
```

You can call such methods and just use the return value in an expression or assign it to a variable:

```bluespec
ReturnType r = module.valueMethodName(arg1, arg2, ...);
```

##### ActionValue Methods

ActionValue methods both alter the internal state of the circuit and return a value from the circuit. This means there is both an enable signal, and an output (return) value. ActionValue methods are declared as follows:

```bluespec
method ActionValue#(ReturnType) avMethodName(ArgType1 arg1, ArgType2 arg2...); // Can have 0 or more args
```

You can call such methods with the same syntax, but to use the return value, you must use the single arrow operator `<-`:

```bluespec
ReturnType r <- module.avMethodName(arg1, arg2, ...);
```

**If you write `=` instead of `<-`, `r` will still have the special type `ActionValue#(ReturnType)`,** which you can't perform computations on like `ReturnType`.

#### Empty Interface

An interface with no methods is built into Bluespec, and is called `Empty`. This is useful for creating top-level modules and testbenches.

### Modules

Modules are implementation of interfaces, and so they are how we actually define how the sequential circuit works. Modules have three components:

* Internal state (registers)
* Methods (inputs and outputs)
* Rules (internal logic)

Again, we're going to only talk about sequential circuits that use one clock domain, so the clock is implicit and we can think of modules on a timestep basis. What this means is that on every clock cycle (or timestep), the internal state and inputs are read, and then some actions are conditionally executed, and the some new values are conditionally written back to the internal state. Then, on the next timestep, the same thing repeats, using the new state and new inputs.

#### Module Declaration

A module declaration and implementation follows the following structure. Note that the name of a module is always prefixed by mk, which stands for "make".

```bluespec
// Basic module declaration
module mkModuleName(InterfaceName);
    // Internal state here

    // Rules here

    // Methods here
endmodule
```

If our module or interface includes parameterizations, here are alternate module declarations:

```bluespec
// Interface is parameterized, module is not. For example, if the module implements
// a specific parameterization of the interface.
module mkModuleName(InterfaceName#(InterfaceParamType));

// Interface and module are both parameterized. Often ModuleParamType and InterfaceParamType
// will be the same. For example, a parameterized FIFO module that can be instantiated
// to store any data type.
module mkModuleName#(ModuleParamType) (Interface#(InterfaceParamType));

// Module is parameterized, interface is not. For example, a non-parameterizable interface,
// but the module that implements it includes a FIFO with parameterizable depth.
module mkModuleName#(ModuleParamType) (Interface);
```

#### Internal State

Any module instantiated within a module is considered internal state, since every sequential module has internal state. The most basic unit of internal state in Bluespec is the register (a built-in Bluespec module), which only consists of two methods, read and write, and stores whatever values are written to it. However, the module can have any collection of registers, vectors of registers, or other modules as internal state.

##### Instantiating Internal State

Internal state should be instantiated at the beginning of a module. We need to declare the internal state just like we would any variable in a function, but to initialize the value, we use a new operator, the left arrow `<-`, to actually create an instantiation of the module.

```bluespec
Reg#(Bit#(1)) myReg <- mkRegU();            // Creates a register storing 1 bit, undefined initial value
Reg#(Bool) myRegFlag <- mkReg(False);       // Creates a register storing a Bool, initialized to False
Reg#(Bit#(4)) myRegValue <- mkReg(4'b1001); // Creates a register storing 4 bits, initialzed to 4'b1001

ModuleName myModule <- mkModuleName();      // Creates an instance of the module ModuleName
```

The initial values stored in registers are the reset values for the registers, and are only relevant when you first instantiate the circuit. As soon as you write to the register, the initial value becomes irrelevant. For registers that store data, we often can just not specify an initial value (as this results in less hardware). Sometimes, however, we need to define an initial state so that our circuit starts up correctly. For example, if we have an FSM that uses a `busy` flag, and the `start` method can't be called while `busy=True`, then we need to make sure that `busy` is initialized to `False`.

##### Registers

The most basic module in Bluespec is the register: `Reg#(Type)`. A register can hold any type in the Bits class (including user-defined types deriving Bits, etc.) and we can instantiate any number of registers in our module.

A register has only two methods: `_read` and `_write`. Since it's such a commonly used module, however, there's a shorthand for these two methods. If we have a 2-bit register `x`:

- `let y = x;` is equivalent to `let y = x._read();`
- `x <= 2'b00;` is equivalent to `x._write(2'b00);` Note that writing to a register uses the double arrow `<=`, which is distinct from the single arrow `<-` used for instantiating modules (above) or calling `ActionValue#` methods (below).

When you read from a register, it returns the value of the data stored in the register. When you write data to a register, the new data value does not appear until the next cycle. So if a 1-bit register `x` is currently `0`, and in some rule/method (explained later) we have:

```bluespec
x <= 1; // write 1 to x
y = x;  // read x into y
```

this is the same as 

```bluespec
x._write(1);
y = x._read();
```

and the end value of `y` will be 0, not 1, because writes to `x` don't happen until the end of the cycle, while reads happen at the beginning of the cycle. Note that `y` has to be a variable (corresponding to an intermediate wire), not a register, because we're using `=` assignment, which isn't valid for registers. If `y` was a register and we wanted to read the value of `x` into `y`, we would need to do:

```bluespec
x <= 1; // write 1 to x
y <= x; // read x into y
```

Note: In this second example, **the old value of `x` will not appear in `y` until the end of the cycle**, since this operation is a write to the register `y`! So if we were to read from `y` on the next line, it would still return the old value of `y`.

`ConfigReg`s are a small variant that behave just like normal registers, except that they don't enforce reads to be scheduled before writes. This does *not* mean that reads will see the value written by writes! All reads will still see old values. Import them with `import ConfigReg :: *;` and create them with `mkConfigReg` or `mkConfigRegU`.

##### Vectors

Sometimes we want to declare an array of registers of the same size. For example, if we have a buffer of length n, we need an array of n registers to store our data. Bluespec has another built-in type, `Vector`, that we can use for this purpose, that has the following declaration:

```bluespec
Vector#(n, ElementType);
```
where `n` is the number of elements in the array, and `ElementType` is the type of elements in the array.

If we want to actually instantiate a Vector of Registers, we would do so as follows:

```bluespec
// Instantiate a 5-element Vector of n-bit registers with uninitialized values
Vector#(5, Reg#(Bit#(n))) myVec1 <- replicateM(mkRegU());

// Instantiate an n-element Vector of 5-bit registers initialized to all 0's
Vector#(n, Reg#(Bit#(5))) myVec2 <- replicateM(mkReg(0));
```

Note: To use Vectors, you have to import the Vector package by adding the following line to the top of your file:

```bluespec
import Vector :: * ;
```

##### FIFO Queues

There are other, more complex modules that can be used to store internal state. A FIFO (first-in-first-out) queue stores some amount of data. A producer can enqueue (`enq`) data, putting it into the queue, and a consumer can dequeue (`deq`) data, taking it out of the queue; this can happen in the same cycle or in different cycles. The consumer always dequeues data in the same order that the producer produces it, hence first-in-first-out. FIFOs are useful for flexibly storing data between pipeline stages.

```bluespec
interface FIFO#(type a);
    method Action  enq (a x);
    method Action  deq;
    method a       first; // data that was enqueued the earliest
    method Action  clear;
endinterface;
```

You must `import FIFO :: *;` to use FIFOs.

```bluespec
FIFO#(datatype) fifo <- mkFIFO;
```

To enqueue data you will usually write:

```bluespec
fifo.enq(data);
```

To dequeue data you will usually write:

```bluespec
let data = fifo.first;
fifo.deq;
```

But you don't need to call both methods. You can choose to call just `fifo.first` to examine the data at the front of the queue, or just `fifo.deq;` to dequeue something and get rid of it.

There are also "FIFOFs", which are just like FIFOs except that they also have methods to explicitly determine if they are (not) full or empty: `notFull` and `notEmpty` methods, which return `Bool`. You should `import FIFOF :: *;` to use them.

```bluespec
FIFOF#(datatype) fifof <- mkFIFOF;
```

If you want to specify exactly how large your FIFO or FIFOF should be, You can call `mkSizedFIFO` or `mkSizedFIFOF` with a positive integer argument.

```bluespec
FIFOF#(datatype) fifof <- mkSizedFIFOF(3);
```

Finally, there are a variety of more specialized FIFOs/FIFOFs if you `import SpecialFIFOs :: *;`. The most likely ones to be used:

-   A *pipeline FIFO* (`mkPipelineFIFO` or `mkPipelineFIFOF`, which is size 1) is a FIFO where you can enqueue into a full FIFO if you also dequeue from it in the same cycle. It forces dequeueing to happen before enqueueing in each cycle.
-   A *bypass FIFO* (`mkBypassFIFO` or `mkBypassFIFOF`, which is size 1) is a FIFO where you can dequeue from an empty FIFO if you also enqueue into it in the same cycle. It forces enqueueing to happen before dequeueing in each cycle.

If you are curious about these FIFOs' implementation or need to customize them, you can look at the Bluespec source in `$BLUESPECDIR/BSVSource/Misc` directory. (Here `$BLUESPECDIR` is an environment variable. You can type `cd $BLUESPEDIR/BSVSource/Misc` in a terminal to go to that directory.)

##### Wires

A basic but less 6.004-relevant module are "wires", which are modules with a value that can be written in a cycle and then have the value read out later in that cycle (so reads are constrained to be scheduled later than writes). The most primitive is the `RWire` (created with `mkRWire`) module supports a `wset` action and a `wget` method, where `wget` returns a `Maybe#` value that is valid only if it was written earlier in the cycle. The `Wire` interface module supports `_read` and `_write`, so it can be operated on with the same syntax as a register, and has more variants:

- `mkWire` or `mkUnsafeWare` produces a `Wire` in which reads are implicitly guarded on whether a write occurred earlier. (`mkUnsafeWire` allows the write and read to be in the same rule, but `mkWire` does not.)
- `mkBypassWire` produces a `Wire` with no implicit guard; the compiler warns if the wire is not written in every cycle.
- `mkDWire(defaultValue)` produces a `Wire` with no implicit guard. Reading from this wire is always valid and will read the default value if no writes occurred.

##### Ephemeral History Registers

Ephemeral history registers, or EHRs, are basically registers that can be read/written several times in a cycle such that writes can be observed by later reads. In recent Bluespec versions, they can be found under the name `CReg`, for "concurrent register". The syntax to create one looks like:

```bluespec
Reg#(datatype) regs[3] <- mkCReg(3, defaultval);
```

You can now read and write to `regs[0]`, `regs[1]`, and `regs[2]`. Of course, the number 3 can be changed and the rules are similar to the above, but only small integers (up to 5?) are supported.

The rules are:

- Reads to `regs[i]` must happen before writes to the same `regs[i]`. The individual registers behave like normal registers in this regard.
- All the writes must happen in order: if `i < j`, then writes to `regs[i]` must happen before writes to `regs[j]`. The last of these writes that occurs becomes the value of the `CReg` at the start of the next cycle.
- Writes must come before, and are seen by, later reads: if `i < j`, then writes to `regs[i]` must happen before reads from `regs[j]`, and the last of all values written to `regs[i]` for `i < j` will be read by `regs[j]` (or, if no such write occurred, then the register's value at the start of the cycle will be read.)
- Note, however, that if you don't write `reg[i]`, say, then there's no conflict between `reg[i]` and `reg[i+1]`.

6.004 students may also be provided with an implementation called `Ehr`. Consult lecture slides/notes on usage.

#### Methods and Rules

Methods and rules are how we define the combinatorial logic that decides when/how to change the internal state of the sequential circuit. Methods are how the outside world gives the circuit inputs and reads outputs. Rules, on the other hand, are invisible to the outside world and describe the rest of the combinational logic in the circuit.

Methods and rules consist of method calls to their internal modules, function calls, and assignments of temporary variables (wires). Both rules and methods are atomic, which means that either all or none of their actions are executed, where "actions" are calls to internal modules or writes to internal state.

##### Methods

Methods are defined as follows:

```bluespec
method ReturnType methodName(ArgType1 arg1, ...) if (guard);
    statement1;
    statement2;
    ...
    statementN;
endmethod
```

A method can only be executed if `guard=True`. This is an explicit guard, and usually depends on the internal state of the module. If the method is executed, then statement1 through statementN will all be executed. Otherwise, none of them will be executed.

If these statements include calls to internal modules, then they can also generate implicit guards. Take the following example. The `start` method in mkTwoModules only has one guard, `!busy`. However, since `mod1.start` and `mod2.start` also have guards, and can only execute when their guards are true, `mkTwoModules.start` can only execute if `mod1.busy=False` and `mod2.busy=False`. Since mkTwoModules doesn't have any visibility into the internal implementation of mkMyModule, we can't write them explicitly in mkTwoModules; they're instead implicit and get generated later by the compiler.

```bluespec
// Module that does something
module mkMyModule(IfcType);
    // Some internal state
    ...
    Reg#(Bool) busy <- mkReg(False);

    method Action start() if (!busy);
        doStuff();
        ...
    endmethod

    ...

endmodule

// Module that instantiates two MyModules
module mkTwoModules(IfcType);
    MyModule mod1 <- mkMyModule;
    MyModule mod2 <- mkMyModule;
    Reg#(Bool) busy <- mkReg(False);

    ...

    method Action start() if (!busy);
        mod1.start();
        mod2.start();
        ...
    endmethod

    ...

endmodule
```

##### Rules

Rules are similar to methods in that they are a collection of method calls, function calls, and use of temporary variables. However, they do not take inputs or generate outputs, and they do not interact with the outside world. Instead, they define how the sequential circuit is continuously updating its internal state. While methods only execute when they are called, rules execute all the time when they can.

A rule is implemented as follows:

```bluespec
rule ruleName if (guard); // The word `if` is optional for rules
    statement1;
    statement2;
    ...
    statementN;
endrule
```

Just like methods, a rule has implicit guards. If any statement is a method call with guards, then any guards on that method call are an implicit guard on this rule. If any implicit or explicit guard is false, then no statements will execute, otherwise all statements will execute.

##### Avoiding Double Writes

Since everything in a rule or method happens on the same cycle, we have to make sure that we don't try to double write to a register. Examples of double writes are:

```bluespec
// BAD: Double write
method Action doubleWrite;
    x <= 1;
    x <= 0;
endmethod

// BAD: Conditional double write
method Action condDoubleWrite;
    x <= 1;
    if (y) x <= 0;
endmethod

// OK: Two exclusive writes
method Action condExclusiveWrite;
    if (y) x <= 1;
    else x <= 0;
endmethod;
```

We also can't call conflicting methods in the same cycle. This includes double calling to the same method, or calling two methods that both write to the same register. For example:

```bluespec
module mkSubmodule;
    // Internal state
    ...
    Reg#(Bit#(1)) x <- mkRegU;

    method Action writeValueA;
        x <= valA;
    endmethod

    method Action writeValueB;
        x <= valB;
    endmethod
endmodule

module mkMyModule;
    // Internal state
    ...
    Submodule submod <- mkSubmodule;

    // BAD: If this method executes, it would cause a double write
    // to the register submod.x
    method Action doSomething;
        submod.writeValueA();
        submod.writeValueB();
    endmethod
endmodule
```

##### Scheduling

**Scheduling** concerns how the Bluespec compiler determines which rules will fire in each cycle. Generally, in every cycle, Bluespec will try to fire every rule whose guard is True, in some order. If it can't do that, which could happen if two rules both interact with the same registers or conflicting methods of the same module, Bluespec will issue a warning. No matter what, each rule will execute at most once each cycle.

Some constraints from basic modules:

-   For a normal register, all reads (including e.g. in the guards of rules) must be scheduled before all writes in each cycle.
-   For a normal FIFO queue, only one rule can `enq` and only one rule can `deq` each cycle, but the two could happen in either order. `first` must happen before `deq`. In a pipeline FIFO, `deq` must come before `enq`. In a bypass FIFO, `enq` must come before `deq`.

If the `-show-schedule` flag is passed to Bluespec, which it should be in 6.004 makefiles, you can see the generated schedule of rules in the `.sched` file. There are also some *scheduling attributes* that you can write before rules to affect their scheduling. They are rather advanced but can be useful to make sure that methods are fired under the conditions you expect them to, and scheduled in the order you expect them to. Consult the Bluespec reference guide for more information.

```bluespec
(* fire_when_enabled *) // the immediately following rule *must* fire if its guard is enabled. If the compiler can't make this happen, it errors.
(* no_implicit_conditions *) // The immediately following rule must not have any implicit guards, caused by calling a method with a guard. That is, it must be able to fire if its guard is enabled.
(* descending_urgency = "rule1, rule2, rule3" *) // rule1 is more urgent than rule2, which is more urgent than rule3, etc.; which means that if the guard of multiple of these rules is enabled and they conflict, the earlier (more urgent) rules will fire
(* execution_order = "rule1, rule2, rule3" *) // in each cycle, rule1 should be scheduled before rule2, which should be scheduled before rule3. If this can't happen, the compiler will consider them to conflict, even if they could have executed in the other order without this attribute.
(* mutually_exclusive = "rule1, rule2, rule3" *) // Tells the compiler that these rules' guards are mutually exclusive, even if Bluespec can't determine it. Bluespec will insert code so that there will be an error if this fails during runtime simulation.
(* conflict_free = "rule1, rule2, rule3" *) // Tells the compiler that these rules are conflict-free, i.e. they will never call conflicting methods when running, even if Bluespec can't determine it. Bluespec will insert code so that there will be an error if this fails during runtime simulation.
(* preempts = "rule1, rule2" *) // Tells the compiler that if rule1 fires, rule2 must not fire; equivalent to forcing the two rules to conflict and then annotating with descending_urgency.
```

## Additional Topics

#### Maybe values

Given a type `Type`, you can create a type called `Maybe#(Type)`. Values of the type `Maybe#(Type)` could either be `Valid` and contain a value of type `Type`, or be `Invalid` (and not contain anything --- there's exactly one possible `Invalid` value). The syntax for a valid `Maybe` value is `tagged Valid value` and the syntax for the invalid `Maybe` value is `tagged Invalid`. For example, here are all possible values of the type `Maybe#(Bit#(2))`:

```bluespec
Maybe#(Bit#(2)) invalid = tagged Invalid;
Maybe#(Bit#(2)) valid00 = tagged Valid 2'b00;
Maybe#(Bit#(2)) valid01 = tagged Valid 2'b01;
Maybe#(Bit#(2)) valid10 = tagged Valid 2'b10;
Maybe#(Bit#(2)) valid11 = tagged Valid 2'b11;
```

To use a `Maybe` value, you might want to use the built-in functions `fromMaybe` and `isValid`.

-   If `defaultVal` is a value of some type `Type` and `maybeVal` is a value of type `Maybe#(Type)`, then `fromMaybe(defaultVal, maybeVal)` returns the value inside `maybeVal` if `maybeVal` is Valid, and `defaultVal` if `maybeVal` is invalid.
-   `isValid(maybeVal)` returns `True` if `maybeVal` is Valid and `False` if `maybeVal` is Invalid.

However, the most generally useful way of handling a `Maybe` value is to use a case matching statement or expression.

#### Case matches

```bluespec
Maybe#(Bit#(2)) foo = // ...

case (foo) matches
    tagged Valid .x:
        // foo is Valid and x is the value of type Bit#(2) inside foo
    tagged Invalid:
        // foo is Invalid
endcase
```

## TODO

#### Program Structure (scoping, visibility, file structure, etc.)
#### Rule conflicts and scheduling
#### Testbenches
#### Synthesis / synthesize keyword
#### Debugging / Common Error Messages

Most of Bluespec's error messages have line and column numbers, so it can often help you track down the error sooner if you enable line numbers on your text editor.

##### Bluespec doesn't respond for a long time

Check if you have Internet access inside the VM and that you're not on MIT GUEST, since Bluespec needs Internet access to check out a license and run.

##### "Type error ... Expected type ... Inferred type ..."

That means that Bluespec wanted some expression to be a particular ("expected") type, but the expression was a different ("inferred") type, so Bluespec couldn't compile the expression. Try to figure out why they are different and what you can do to both sides to make them the same. Common possible type errors:

-   **The two types are `Bit#(n)` and `Bit#(m)` for different numbers `n` and `m`, or one of the types is `Bit#(n)` and the other is `Integer`**: Remember that most of Bluespec's bitwise and arithmetic operators only operate between two operands of the same number of bits, or between two `Integer`s. If you have two `Bit` types of different lengths, you may want to `extend`/`zeroExtend`/`signExtend`, `truncate`, or slice one or both of them so they match. If you have an `Integer` (in particular, the result of calling `valueOf` on a numeric type variable), you can call `fromInteger` on it to turn it into an arbitrary `Bit#(n)`.

-   **One of the types is `Bool` and the other is some `Bit#(n)`**: Remember that `Bool`s and `Bit#(1)`s are different types, and that Bluespec's boolean and bitwise operators are different.

    -   For `Bool`s, you use `&&` `||` and `!`.
    -   For `Bit#(n)`, you use `&` `|` and `~`.

    You can convert a `Bit#(1) b` to a `Bool` with `b == 1` and you can convert a `Bool b` to a `Bit#(1)` with `b ? 1 : 0`.

##### "The numeric types ... could not be shown to be equal"

This usually arises because you are using type variables in some way that only works if they are equal to some fixed type or to each other. For example, if you try to assign a value of type `Bit#(m)` to a variable of type `Bit#(2)` where `m` is an actual type variable, Bluespec will complain that it doesn't know if `m` equals `2`. You may be able to resolve this by extending or truncating.

One particular reason you might encounter this error is if you're trying to write a recursive function with a base case depending on a type variable. For example, in order to reverse the bits in a sequence, you might try to write a recursive function like this:

```bluespec
function Bit#(w) myReverseBits(Bit#(w) bits);
    if (valueOf(w) == 1)
        return bits[0];
    else begin
        Bit#(TSub#(w, 1)) rest = bits[valueOf(w)-1:1];
        return {bits[0], myReverseBits(rest)};
    end
endfunction
```

Unfortunately Bluespec doesn't work this way: when compiling it will not treat the condition `valueOf(w) == 1` specially, and it will still require both branches of the if/else to match the claimed return value. That is, even if `w > 1` and you know the top branch of the if/else will not be taken, Bluespec will still require that the return value from the top branch (which is `Bit#(1)`) match the return type (which is `Bit#(w)`) of the function, and it will complain that it can't show that `w` equals `1`. You should probably just try to write functions like this iteratively. (For this particular use case, Bluespec has a built-in `reverseBits` function that returns a reversed copy of the bits of a `Bit#(n)`.)

##### "The provisos for this expression are too general"

This error means you are using type variables in some more complicated way that Bluespec doesn't have enough information to see will work, most commonly numeric type variables. A "proviso" is some kind of constraint on the type variables that has to be satisfied in order for your code to make sense. For example, if you try to assign a value of type `Bit#(TAdd#(m, 1))` to a variable of type `Bit#(n)` where `m` and `n` are actual different type variables, this is only possible if `m + 1` equals `n`, and Bluespec will complain that it wants a proviso that translates to `m + 1 == n`.

Some example errors:

-   **"`The following provisos are needed: Add#(w, 1, w)`"**: The proviso `Add#(w, 1, w)` mwans that Bluespec wants `w + 1 = w` to be true. Obviously, this is mathematically impossible, but unfortunately Bluespec is not smart enough to figure this out. It typically means you are trying to assign a value of type `Bit#(TAdd#(w, 1))` to a variable of type `Bit#(w)` or vice versa. The way around is usually the same as when you have a type error between `Bit#(m)` and `Bit#(n)` for actual numbers `m` and `n`; you should try to extend or truncate.

-   **"`The following provisos are needed: Add#(a__, 1, w)`"**: If there's a variable that ends in two underscores, it's usually a made-up name internal to Bluespec. In this case this proviso just means that Bluespec thinks `w` has to be greater than or equal to 1. In this case you may actually want to add this proviso to your function; see the section below on provisos (TODO).

#### Display statements and other system tasks/functions

The `$display` statement (formally a "system task") is useful for debugging. It prints a string. (Of course this only happens during simulation of the circuit, not in a real circuit that would be synthesized.)

```bluespec
$display("Hello!");
```

Actually, `$display` is like `printf` if you've ever encountered it in C. If you have a number `n`, you could print it using a `%` format specifier like this:

```bluespec
$display("n is %d in decimal", n);
$display("n is %b in binary", n);
$display("n is %o in octal", n);
$display("n is %x in hexadecimal", n);
```

You can use multiple format specifiers for multiple numbers, like if you have another number:

```bluespec
$display("n is %d and m is %d", n, m);
```

You could also just display the number directly:

```bluespec
$display(n);
```

Note that `$display` prints a newline after the string you give it. If you don't want that, you can use `$write` instead, with the same syntax.

If you have a more complicated structure, though, you probably won't be happy with just displaying it directly. Instead, you should make the structure derive `FShow` and call `fshow` on it to get a nice format:

```bluespec
typedef struct {
    Bit#(32) a;
    Bit#(32) b;
} Foo deriving (Bits, Eq, FShow);

// later

Foo foo = Foo { a: 1, b: 2 };
$display(fshow(foo));
```

This will print something like `Foo { a: 'h00000001, b: 'h00000002 }` instead of just a garbage hex string.

TODO: `fshow` returns a `Fmt` object. I'm not sure how this works with `$display` exactly, but if you want to display a message next to it you can write:

```bluespec
$display($format("foo is ") + fshow(foo));
```

#### Provisos
#### Recursion
#### Don't care values
#### Tagged Unions
