## Variables

local and state variables.
initialization

An array can be given an initial value as follows: `u32 triple = {1, 2, 3};`. Also, Cx also supports initialization of char arrays from strings: `const char message[12] = "Hello world!";`.

## Functions

A function declaration is composed of the following elements:

- optionally, the `const` keyword to declare a constant function,
- a [type](/documentation/type-system/) or `void`,
- a name,
- parameters,
- a body of statements that do computations and may return a result.

A function may have parameters and define local variables, and contains a sequence of statements such as reading from ports, updating state variables, calling other functions, and writing to output ports.

Statements that modify the environment outside of the function are said to have side effects. Examples of side effects are accessing a port (reading, writing, testing availability), writing state variables, calling another function that has side effects. Side-effect free statements are any statement that does not have side effects.

There are two kinds of functions:

1. constant functions are functions without side-effects, they can only read state variables, and cannot access ports.
2. functions with side effects can modify state variables and read/write ports.

### Constant functions

Functions declared by a [bundle](/documentation/bundles) are implicitly constant (there is no need to specify a function declared in a bundle as `const`). This is by design, since a bundle does not have ports or state variables, it can only declare constant functions.

In tasks, to declare a constant function, start the declaration with the `const` keyword:

    const u9 increment(u8 x) {
      return x + 1;
    }

### Functions with side effects

Functions with side effects can only be declared in tasks. As of the current version of Cx, it is not possible to declare a function that has side effects _and_ returns a result (with a type other than `void`). As a result, functions that return a result must be declared constant in tasks. We hope to lift this restriction in a future version of the language (see [roadmap](/roadmap)).

## Ports

A port is an interface defined by a network or a task to communicate with other networks/tasks. The definition of a port includes a direction (input or output), a [synchronization](ports/#synchronization) type, a [type](/documentation/type-system/), and a name. A port declaration begins with a direction (`in` or `out`) and may declare one or more ports. If a port has the same type as the port that precedes it, its type can be omitted. For instance:

    in u16 a, b /* b has type u16 like a */, u48 c;
    out bool x;

### Synchronization

A port may have additional synchronization semantics as specified with the `sync` flag. The `sync` flag may occur before the port's type (if any) and only applies to the current port, not to following ports:

    in u16 a, sync b, u48 c;
    out sync bool x;

If you have multiple `sync` ports, you can use a multiple port declaration. A multiple port declaration begins with the synchronization type, and contains basic port declarations inside curly braces. All the ports contained in the declaration will be synchronized. The example above can be rewritten as follows:

    in u16 a, u48 c;
    sync {
      in u16 b; out bool x;
    }

#### Simple synchronization

The `sync` flag adds a synchronization signal that becomes true for one cycle each time data is written to the port. When a task reads from one or more synchronized ports, the `read` expression on these ports becomes blocking. This makes synchronized ports useful to make tasks sensitive to data presence, for instance to describe a pipeline. When reading from several synchronized ports, a task will only resume its execution if data is present **simultaneously** on all ports it is waiting on. The following code will only write a value on `o` if data is present on both `a` and `b` at the same time:

    in sync u3 a, b; out u6 o;
    void loop() {
      o.write(a.read * b.read);
    }

#### Ready synchronization

A `sync ready` port allows two tasks (producer and consumer) to communicate together at possibly different rates, automatically synchronizing each task as needed. A ready port handles management of signals that indicate when the consumer is ready and when the producer has valid data. Use a ready port in the following cases:

- have two tasks communicate at different rates,
- implement back-pressure in pipelines, for example you need to pause transmission during an inter-frame delay in Ethernet,
- interact with a FIFO,
- interact with the outside world for compatibility with existing components.

Note: ready ports are experimental, we're still figuring out some implementation details in code generation. See [this post](https://forum.synflow.com/t/thoughts-on-ready-ports-for-implementing-an-intel-8088/219) for discussion.


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
