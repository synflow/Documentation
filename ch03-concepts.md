This section introduces core concepts of the Cx language: how an application is organized in Cx, the type system, and the execution model.

## Organization

An application is organized as a hierarchy. At the top of the hierarchy is a [network](/documentation/networks). A network may instantiate other networks, as well as [tasks](/documentation/tasks). Tasks and networks may refer to [bundles](/documentation/bundles): a bundle is an entity that defines constants and functions.

![Organization](/images/structure/modeling.png)

### Packages

To avoid name conflicts, an entity (bundle, task, network) has a *qualified name* that is composed of the name of the package to which the entity belongs concatenated with the entity's name. A package is a list of identifiers separated by dots. Packages are hierarchical, so package "a.b.c" refines package "a.b". By convention, a package name begins either with the company name (e.g. com.acme) or organization name (org.something), followed by the name of the application, and further refined to match the organization of the application.

### Referring to another entity

Cx entities (and the constants, types, and functions they define) can be imported by other entities with import statements:

1. to import an entity, use `import com.synflow.sha256.SHACommon;`
2. to import everything an entity defines, append ".\*" to the import statement: `import com.synflow.sha256.SHACommon.*;`

### Modules

Entities are defined in modules, and there is one module per .cx file. A module begins with a package declaration, may have module-level import directives, and then may declare any number of entities.

    package com.synflow.sha256;

    // module-level import directives
    import com.synflow.sha256.SHAConstants;
    import com.synflow.sha256.LookupTable;

    /**
     *  documentation for network
     */
    network N {
      // entity-level import directives
      import com.synflow.sha256.SHALoop;
      import std.lib.SinglePortRAM;

      // description of network
    }

    /**
     *  documentation for task 1
     */
    task T1 {
      // code for task 1
    }

    /**
     *  documentation for task 2
     */
    task T2 {
      // code for task 2
    }

Note that import directives may appear at the beginning of the module (module-level import directives) or at the beginning of an entity. When imports appear in an entity, the entities imported are visible only within that entity.

## Type system

Cx has an bit-accurate type system that includes fixed-width types, custom-width types, and type definitions.

### Fixed-width types

#### Boolean type

The boolean type is declared with the `bool` keyword, it can have values `true` and `false`. When assigning a boolean variable, the value 0 is automatically converted to `false` and the value 1 to `true`.

    bool value;

#### Signed integer types

Signed integers are integers of an arbitrary length (greater or equal than two, see note on integer types below) represented in two's complement. They are declared with "i" immediately followed by the number of bits: `i3` (3-bit integer), `i6` (6-bit integer), `i128` (128-bit integer).

    i13 immSigned;

#### Unsigned integer types

Unsigned integers are integers of an arbitrary length (greater or equal than two, see note on integer types below) in natural binary representation. They are declared with "u" immediately followed by the number of bits: `u3` (3-bit unsigned integer), `u6` (6-bit unsigned integer), `u128` (128-bit unsigned integer).

    u5 phyAddr;

#### Character type

Cx defines a single character type`char` that is an unsigned 8-bit integer storing single-byte codepoints. `char` should be used to indicate that a variable or a port is used to represent characters rather than integers. If you need to store unsigned 8-bit integers that are not characters, we suggest you use a `u8` instead.

#### C-like integer types

Cx also supports C-like types:

- `short` (synonym for `i16`),
- `int` (synonym for `i32`),
- `long` (synonym for `i64`).

Like in C, you can specify the signedness by prepending `signed` or `unsigned`: `signed short` (equivalent to `i16`), `unsigned int` (equivalent to `u32`). If `signed` or `unsigned` is specified without any other qualifier (i.e. without `short`, `int`, or `long`), this implies that the `int` type is to be used to give a signed (respectively unsigned) 32-bit type.

Cx also supports other commonly found types (OpenCL):

- `ushort` (synonym for `u16`),
- `uint` (synonym for `u32`),
- `ulong` (synonym for `u64`).

#### A note on integer types

The language's integer types support at least 2-bit integers. The first reason is that a 1-bit integer is often used as a boolean, in which case it makes a lot more sense to indicate this by using a `bool`. The second reason is that signed 1-bit has a slightly disturbing range of [-1 .. 0] (remember it is in two's complement). The third reason is that 1-bit arithmetic is often more a source of errors than something actually useful, except in the case of additions with input carry. In this case, convert a boolean with the ternary operator ?: as in C:

    // if carry is true, computes a + b + 1, else computes a + b + 0
    result = a + b + (carry ? 1 : 0);

### Custom-width types

#### Custom-width integer types

Custom-width integer types are types that accept a compile-time constant expression that defines their size. All "int" variants can be given a size, in other words any type from the following set: `{signed, int, signed int, unsigned, uint, unsigned int}`. The syntax is shown in the example below.

    const int words = 1;

    signed<words * 32> a;
    int<words * 32> b;
    signed int<words * 32> c;

    unsigned<words * 32> x;
    uint<words * 32> y;
    unsigned int<words * 32> z;

The types for a, b, and c are all strictly equivalent (as are the types of x, y, z).

#### Array types

Arrays are declared with C-like syntax and semantics: *type* name[*dim1*]...[dim*n*]. Dimensions must be compile-time constant expressions.

    u128 aesKey[11]; // unidimensional array of unsigned 128-bit integers
    bool flags[3][16]; // two-dimensional array of booleans with 3 rows and 16 columns

The size of an array is the product of the size of its elements and of each dimension. In the code above, `aesKey` has a size of 1 408 bits = 128 * 11, and `flags` has a size of 48 bits = 1 (size of boolean) * 3 (first dimension) * 16 (second dimension).

### Type definitions

Type definitions allow you to add *meaning* to a type. For instance, instead of writing:

    u8 val;

you can write:

    pixel val;

To define a type, use the C-like "typedef" mechanism with any fixed-width or custom-width type:

    typedef uint<8> pixel;

You can declare typedefs in networks, tasks, and bundles.

### Type unification

Type unification defines how to unify two types to create a type that has the proper signedness and is as big as the biggest type. Unification is defined as follows:

First operandSecond operandResult typesigned<X>signed<Y>signed<max(X, Y)>unsigned<X>unsigned<Y>unsigned<max(X, Y)>signed<X>unsigned<Y>signed<max(X, Y)>unsigned<X>signed<Y>boolboolbool

Justification: unification of an unsigned integer type with a signed integer type produces a signed integer type because it makes much more sense than producing an unsigned integer type. If you multiply a signed variable, say `i3 x = -2`, with an unsigned variable, like `u6 y = 50`, you expect `i9 z = -100`, **not**`u9 z = 300`!

History: type unification used to unify two types so the resulting type was large enough to represent any value that can be represented in either type. This meant that in case of mixed signedness, for example signed<X> and unsigned<Y>, if X was less than or equal to Y, the size of the result was Y + 1 to account for one additional sign bit. In the example above (unification of `i3` and `u6`) this would have given `i7`. This worked fine, but was surprising from a user's point of view. Indeed, several arithmetic operations are typed in a conservative way to prevent overflow, and this resulted in types that were bigger than you would expect.

## Execution model

The execution model of Cx features two important aspects:

- Parallelism: all instances of a network (and their sub-instances, if any) run concurrently, at the same time.
- Determinism: a model can be simulated by running tasks in any order and always produce the same results. Variables cannot be shared between different tasks and ports cannot be written by different instances.

### Task Execution

A task may have an `setup` function that is executed once, and is used generally to perform a single action or for initializing variables, hence its name. A task also defines a `loop` function, which is executed repeatedly. If a task defines both functions, `setup` is executed once, and then `loop` will be executed repeatedly. Consider a task T:

    task T {
      void setup() {
        print("first time");
      }

      void loop() {
        print("all the time");
      }
    }

Executing an instance of the task T for four cycles will yield:

    first time
    all the time
    all the time
    all the time

### Cycle-accurate execution

A task is defined in a cycle-accurate way, i.e. a task executes cycle by cycle. Everything that can be scheduled in parallel during a cycle will be, provided that data dependencies allow it. In the following code, the two operations "a + b" and "a - b" can be scheduled in parallel in hardware:

    task T {
      properties {
        test: {
          a: [3, 5,  8, 13],
          b: [5, 8, 13, 21]
        }
      }

      in i6 a, b;
      void loop() {
        i6 x = a.read;
        i6 y = b.read;
        i7 o1 = x + y;
        i7 o2 = x - y;
        print("o1 = ", o1, " and o2 = ", o2);
      }
    }

Running this task for four cycles using the values defined by the [test property](/documentation/properties/#test) gives:

    o1 = 8 and o2 = -2
    o1 = 13 and o2 = -3
    o1 = 21 and o2 = -5
    o1 = 34 and o2 = -8

On a side note, Cx has no non-blocking assignment (we find that the little increase in expressive power is not worth the added complexity and the confusion it can cause, it would interact poorly with the rest of the language, and we already have non-blocking writes). As a result, to exchange two values you would write the following code:

    u13 tmp = a;
    a = b;
    b = tmp;

### Network Execution

A Cx model is executed by repeatedly running all instances simultaneously, one cycle at a time. This corresponds to a Discrete Event execution model in which events are clock cycles. The model supports a limited type of combinational (a.k.a. asynchronous) description that is executed within a clock cycle; combinational loops are not supported.

#### A simple example

For instance, assuming a network N with two inner [tasks](/documentation/structure/#tasks) t1 and t2:

    network N {
      t1 = new task {
        int i;
        void loop() {
          print("first (cycle ", i, ")");
          i++;
        }
      };

      t2 = new task {
        int i;
        void loop() {
          print("second (cycle ", i, ")");
          i++;
        }
      };
    }

Running this network for three cycles prints (comments starting with -- added for readability):

    -- starting first cycle
    first (cycle 0)
    second (cycle 0)
    -- starting second cycle
    first (cycle 1)
    second (cycle 1)
    -- starting third cycle
    first (cycle 2)
    second (cycle 2)

#### Scheduling algorithm

The algorithm for scheduling a network (for one cycle) is as follows:

1. execute: executes all synchronous instances for one cycle based on current values.
2. commit: commits new values produced by synchronous instances to become the current values.
3. update: updates combinational instances following the dependency chain from a synchronous instance producing data to a synchronous instance consuming data.

Let's see how this works with an instance of counter, and an inner task reading the counter value and printing it:

    network N {
      t1 = new task {
        out uint counter;
        uint count;
        void loop() {
          count++; // increments count
          counter.write(count); // writes count
        }
      };

      t2 = new task {
        void loop() {
          print("count = ", t1.counter.read);
        }
      };
    }

This network is executed as follows:

- execute task 1: increments count, and writes it to the `counter` port. Writing sets the **new value** on `counter` to 1.
- execute task 2: reads the **current value** on `counter`, which is still 0 (commit has not occurred yet), and prints it.
- commit task 1: updates the value on `counter`. Note how this occurs **after** task 2 printed `counter`'s value.
- commit task 2: nothing to commit, task 2 does not write values.
- no need to update anything.

The output of running this example is this (comments starting with -- added for readability):

    -- starting first cycle
    count = 0
    -- starting second cycle
    count = 1
    -- starting third cycle
    count = 2
    -- starting fourth cycle
    count = 3
    ...

### Combinational modeling

It is sometimes useful to declare tasks or networks that are executed in a combinational. A combinational entity executes in an "instantaneous" manner, during the same cycle.


---
```
Copyright 2014-2020 Synflow SAS

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
