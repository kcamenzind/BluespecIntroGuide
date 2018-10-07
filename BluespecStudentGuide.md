# Bluespec User Guide (6.004)

## Overview

This document is a beginner's guide to Bluespec. It is intended for 6.004 students, so it is structured in the order that the class is taught. It will cover the basic language syntax, data types, and program structure. It is not a complete reference for the Bluespec language, and is therefore missing many features and may simplify others.

I've listed some other resoures to learn Bluespec below:

* [Bluespec Manual](http://www.bluespec.com/forum/download.php?id=157): A comprehensive guide of the language.
* [Bluespec Reference Card](http://www.bluespec.com/forum/download.php?id=98): A short document (3 pages) good for looking up syntax.
* [Online tutorial](http://wiki.bluespec.com/Home): An online walk-through of some of the common features of Bluespec, along with examples.

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

Semicolons are needed after any expression. They are not needed, however, after **begin** and **end** keywords.

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

The keywords `begin` and `end` are how we can lump multiple statements into one statement, mostly in conditionals and loops. For example, to assign two variables in an if statement, we need to write:

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

Bluespec has both built-in data types and user-defined data types. It's important to remember that no matter what data type you're using in your code, when it's compiled to hardware everything is just stored as bits. What the types do is allow you to not have to worry about the underlying structure of those bits.

#### Literals

When we write programs, we often have to assign hard-coded numeric values to variables. It's best practice to write all Bluespec literals with explicit sizes, but it is possible to write both sized and unsized literals. Unsized literals are most useful when you want set a variable (or some section of a variable) to all 0's or all 1's, or when using Integers (since the Integer type is unsized anyway, more on that below).

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

We need to be careful when using '0 or '1 or the compiler will be unhappy. Only use these when both the size of the variable is defined, and the size of the space we're filling with 0's or 1's is unambiguous.

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

There are some built-in types that will be explained in the sequential section.

#### User-defined types

##### Type synonyms

You can rename types with the following syntax:

`typedef OldType NewType`

For example, you may want to rename Bit#(8) as Byte.

`typedef Bit#(8) Byte`

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

##### Enums

Enums are how we can define custom types that are defined by the compiler as bits, but we don't explicitly have to understand how they translate into bits. For example, if you want to define a type Color, which can take values Red, Green, Yellow and Blue, we can write

```bluespec
typedef enum Color { Red, Green, Yellow, Blue } deriving (Bits, Eq); 
```

Deriving Bits means that the values Red, Green, Yellow and Blue will be automatically assigned an underlying representation in bits. Deriving Eq means that the equality operator is derived for the type as well, so if you have a Color variable, you can check if it's Red or Green or Yellow or Blue using the == comparator. You will generally want to include both of these in your enum declarations.

#### Type conversions

##### Converting between numeric types, Integers, and Bits

To extract the Integer value of a numeric type, use `Integer i = valueOf(n)`, where n is the numeric type.

To extract convert an Integer to a Bit#(n) value, use `Bit#(n) x = fromInteger(i)`, where i is an Integer.

You can chain these together to store the value of a numeric type into a Bit#(n) as follows:

```bluespec
Bit#(m) x = fromInteger(valueOf(n));
```

(Note: n and m could be the same value, they're just different numeric type names to illustrate that they don't have to be the same value.)

Lastly, you can extract the size of a Type (in bits) by using SizeOf. SizeOf returns a numeric type, which then you can then further convert into an Integer or Bit# depending on your use case. For example:

```bluespec
Bit#(3) x = 0; // Size of x is 3 bits
Integer i = valueOf(SizeOf(x)); // i = 3, converted from numeric type -> Integer
```

##### Converting between Bits and other types

If a type is represented by Bits, then we can convert between these types and their bit representations. Examples of this are Int, UInt, and any user-defined type deriving Bits.

To convert from Bit to any other type, use unpack:

```bluespec
Bit#(3) x = 3'b101;     // x is the binary value 101
UInt#(3) y = unpack(x); // y = 5, the UInt represented by the bits 101
Int#(3) z = unpack(x);  // z = -3, the Int represented by the bits 101
```

To convert from any type to Bit, use pack:

```bluespec
typedef enum Color { Red, Green, Blue, Yellow } deriving (Bits, Eq);

Color red = Red;
Color yellow = Yellow;

Bit#(2) x = pack(red);    // x = 2'b00, the binary representation of Red
Bit#(2) y = pack(yellow); // y = 2'b11, the binary representation of Yellow

```

#### Working with Bits

##### Indexing Bits

Bits are stored as a string of bits, indexed from the MSB to LSB. For example, if you have a `Bit#(4) x = 4'b1010`, then x[0] = 0 (the LSB), and x[4] = 1 (the MSB). For a `Bit#(n)`, we can index it from 0 to n-1.

To access a parameterized Bit, we can use the numeric type -> Integer conversion discussed above. For example:

```bluespec
Bit#(n) x_param = 1; // x = 1, has n-1 leading zeros
Bit#(1) x_msb = x_param[valueOf(n)-1]; // x_msb is the top bit of x_param
```

We can also take slices of Bits. For example:

```bluespec
Bit#(4) x = 4'b1010;
Bit#(4) y = x;      // y = 4'b1010 
Bit#(4) z = x[3:0]; // z = 4'b1010
Bit#(3) lower_bits = x[2:0]; // lower_bits = 3'b010;
Bit#(3) upper_bits = x[3:1]; // upper_bits = 3'b101;
```

Note that with bit indexing, it's generally recommended to use constants as the indices. There are cases (particularly in for loops) where it's ok to extract bits with a variable, and other cases where's it's allowed but extremely inefficient. Bit slicing with a non-constant variable is illegal.

Some examples:

```bluespec
Integer fixed_i = 2;   // Value of fixed_i known at compile time
Bit#(2) dynamic_i = 3; // Value of dynamic_i not known at compile time
Bit#(4) x = 4'b1001;

// Indexing
Bit#(1) a = x[fixed_i];   // OK, fixed_i is a fixed value
Bit#(1) b = x[fixed_i-1]; // OK, fixed_i-1 is a fixed value
Bit#(1) c = x[dynamic_i]; // OK but slightly inefficient, dynamic_i isn't a fixed value

// Slicing
Bit#(2) d = x[i:i-1];                 // OK, fixed-size and fixed-value slice
Bit#(2) e = x[dynamic_i:dynamic_i-1]; // OK but inefficient, fixed-size but non-fixed-value slice
Bit#(2) f = x[fixed_i:0];             // ONLY OK if i=1 to guarantee sizes match
Bit#(2) g = x[dynamic_i:0];           // NOT OK, cannot guarantee that sizes will match
``` 

##### Concatenating Bits

We can combine strings of bits into longer strings of bits. To concatenate two Bits, use the following notation:

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

Sometimes it can be useful to add arbitrary numbers of 0's or 1's to the beginning or end (usually beginning) of Bits to change the size but retain the numeric value. The first way that we can do this is by concatenating our original number with '1 or '0, which represent "as many 0's or 1's as needed to fill the specified width".

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

### Operators

There are several built-in operators for built-in Bluespec types.

#### Bitwise operators

Bitwise operators operate bit-by-bit on numbers. If the operator takes two arguments, so `a = b OP c`, then this is equivalent to writing `a[i] = b[i] OP c[i]` for every i from 0 to n-1. (Notice that a, b, and c must all be the same size.)

&: bitwise-AND
|: bitwise-OR
^: bitwise-XOR
~: bitwise-NOT

```bluespec
Bit#(4) a = 4'b0011;
Bit#(4) b = 4'b0101;
Bit#(4) c = a & b; // c = 4'b0001;
Bit#(4) d = a | b; // d = 4'b0111;
Bit#(4) e = a ^ b; // e = 4'b0110;
Bit#(4) f = ~a;    // f = 4'b1100;
```

#### Logical operators

Each of these bitwise-operators has a logical equivalent. Logical operators perform the same operations as the bitwise operators, but instead of performing them bit-by-bit, it evaluates the entire variables as 0 if all the bits are 0, and 1 otherwise.

&&: logical AND
||: logical OR
! : logical NOT

```bluespec
Bit#(4) a = 4'b0101;
Bit#(4) b = 4'b0000;

Bit#(1) d = a || b; // c = 1 since a != 0
Bit#(1) e = a && b; // c = 0 since b == 0
Bit#(1) f = !a;     // f = 0 since a != 0
Bit#(1) g = !b;     // g = 1 singe b == 0
```

#### Ternary operator

The ternary statement mimics the behaviour of a multiplexer, and is shorthand for an if-else statement. The expression `(cond) ? val1 : val2` evaluates to val1 if cond==True, and val2 if cond==False. The cond must evaluate to the Boolean (not Bit) type.

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

TODO: Figure out the actual rules for arithmetic when numbers are different sizes or signed vs unsigned

`a + b` : Addition

`a - b` : Subtraction

`a * b` : Multiplication

`a / b` : Division

`a % b` : Modulus

`a << b` : Left shift

`a >> b` : Right shift

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

Variables must be assigned values to be used. Generally, you will include an initial value in its declaration. If you don't, then you should always make sure taht the variable is assigned a value before it's used.

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
y = f(var2); // var2 doesn't have a value if cond=False !
```

#### let

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

#### Assignments are blocking

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

A function can return exactly 1 value. If you want to return multiple values, then you can pack them into a user-defined type and extract the separate values from the fields of the return value.

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

This says that inputs `a` and `b` will be Bits of size n, c is a Bit of size 1, and the function will return a Bit of size n+1. (If you don't remember how the TAdd# function works, refer to the section on numeric type operations.) 

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

You can add for loops to your program! You can do so with the following syntax:

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

One thing to note is that loops are unrolled at compile-time. This means that what this above function actually does is the following:

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

Another thing to note is the use of the Integer type in the for loop. We use Integers because they're unsized so we don't have to worry about if we're using enough bits. At the same time, since the loop is unrolled, it's ok to use an Integer because i won't ever actually change values in the compiled circuit, it just becomes a hard-coded constant for each iteration of the loop.

We generally want to keep the bounds of our for loop to a constant, because otherwise our circuit has to unroll every possible iteration of the for loop and put a mux on every iteration deciding whether that iteration is the final one or not. In code:

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
// Don't need to have a default else statement.
```

##### Case

The case statement is a shorthand way of writing long if/else blocks. The syntax is as follows:

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
    val1: xval1;    // x = xval1 if (switch==val1)
    val2: xval2;    // x = xval2 if (switch==val2)
    default: xval3; // x = xval3 if (switch!=val1) && (switch!=val2)
endcase;            // Note the semicolon here
```

#### Return statements

You can have return statements anywhere in your function. However, note that you must have a return statement, so there cannot be a possible path where your function will not return a value.

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

// Also ok to have a default return statement
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

TODO: with the compiler is this actually true, can it not figure out that it can just mux the input? 

## Sequential Circuits

Up to this point, we've only talked about writing code to generate circuits that have no concept of time or state. We'll now take a look at how we can use Bluespec to describe sequential circuits, which are cycle-driven (by an implicit clock--we're going to skip in this guide talking about designs that use multiple clocks) and can store state across cycles.

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

##### Value Methods

Value methods do not alter the internal state of the circuit, so there's no enable signal because nothing in the circuit needs to change when the method is called. Instead, value methods just output some value generated in the circuit. Value methods are declared as follows:

```bluespec
method ReturnType valueMethodName(ArgType1 arg1, ArgType2 arg2...); // Can have 0 or more args
```

##### ActionValue Methods

ActionValue methods both alter the internal state of the circuit and return a value from the circuit. This means there is both an enable signal, and an output (return) value. ActionValue methods are declared as follows:

```bluespec
method ActionValue#(ReturnType) avMethodName(ArgType1 arg1, ArgType2 arg2...); // Can have 0 or more args
```

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

Internal state should be instantiated at the beginning of a module. We need to declare the internal state just like we would any variable in a function, but to initialize the value, we use a new operator, `<-`, to actually create an instantiation of the module.

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

`let y = x;` is equivalent to `let y = x._read();`
`x <= 2'b00;` is equivalent to `x._write(2'b00);`

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

and the end value of `y` will be 0, not 1, because writes to `x` don't happen until the end of the cycle, while reads happen at the beginning of the cycle. Note that `y` has to be an intermediate wire, not a register, because we're using `=` assignment, which isn't valid for registers. If `y` was a register and we wanted to read the value of `x` into `y`, we would need to do:

```bluespec
x <= 1; // write 1 to x
y <= x; // read x into y
``` 

Note: In this second example, the old value of `x` will not appear in `y` until the end of the cycle, since this operation is a write to the register `y`! So if we were to read from `y` on the next line, it would still return the old value of `y`.

##### Vectors

Sometimes we want to declare an array of registers of the same size. For example, if we have a buffer of length n, we need an array of n registers to store our data. Bluespec has another built-in type, Vector, that we can use for this purpose, that has the following declaration:

```bluespec
Vector#(n, ElementType);
```
where n is the number of elements in the array, and ElementType is the type of elements in the array.

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

#### Methods and Rules

Methods and rules are how we define the combinatorial logic that decides when/how to change the internal state of the sequential circuit. Methods are how the outside world gives the circuit inputs and reads outputs. Rules, on the other hand, are invisible to the outside world and describe the rest of the combinational logic in the circuit.

Methods and rules consist of method calls to their internal modules, function calls, and assignments of temporary variables (wires). Both rules and methods are atomic, which means that either all or none of their actions are executed, where "actions" are calls to internal modules or writes to internal state. 

guards, implicit and explicit
no double-reads or double-writes

##### Methods

Methods are written as follows:

```bluespec
method MethodType methodName(ArgType1 arg1, ... ) if (guard);
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

Rules are similar to methods in that they are a collection of method calls, function calls, and use of temporary variables. However, they do not take inputs or generate outputs, and they do not interact with the outside world. Instead, they defined how the sequential circuit is continuously updating its internal state. While methods only execute when they are called, rules execute all the time when they can.

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

## Additional Topics (coming soon)

#### Program Structure (scoping, visibility, file structure, etc.)
#### Rule conflicts and scheduling
#### Testbenches
#### Synthesis / synthesize keyword
#### Debugging / Common Error Messages
#### Display statements and other system tasks/functions
#### Provisos
#### Recursion
#### Don't care values
#### Maybe values
#### Case matches
