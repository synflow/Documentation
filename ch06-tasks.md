A task is an atomic entity that may have properties, input ports, output ports, state variables, and functions. A task may have any number of functions, as well as special functions: an initialization function called `setup` and the `loop` function that is the entry point of the task when it runs.

By default a task is synchronous (clock and reset are implicit), though this behavior can be changed with properties.

A task can have both state variables and constants. Contrary to the implicitly constant definitions in a bundle, the definition of a constant in a task requires an explicit **const** qualifier:

    // i is a 16-bit state variable initialized to 0
    short i = 0;

    // LENGTH_PRE is a 16-bit constant equal to 6
    const short LENGTH_PRE = 6;

## External tasks

An external task is a task with only a signature (parameters, clocks, reset, ports). Its implementation is done by the designer (in Verilog, VHDL, or another language) and must be indicated in the properties of the task.

When you have existing code (for example VHDL or Verilog) that you wish to call from Cx, you must declare a task with the same signature (parameters, clocks, resets, ports) as your code. This kind of task is said to have an external implementation: it has no "setup" or "loop" methods, and no code will be generated from it, but otherwise you can instantiate it and connect it to other entities just as with any other task.

To declare an external task, create a task with an "external" implementation:

    package com.acme;

    task Queue {
      properties {
        clocks: ["din_clock", "dout_clock"],
        implementation: {
          type: "external",
          file: "../../../rtl/Queue.v",
          dependencies: ["../../../rtl/Ram.v"]
        }
      }

      const int depth = 2, width = 16;

      in sync uint<width> din; out sync uint<width> dout, bool ack;

    }

Parameters in your code are declared as constants in the Cx code, like _depth_ and _width_ in the example above. Typically these constants will be overridden when the task is instantiated: `new Queue({depth: 32, width: 8})`.

The implementation of the task must be in a file whose name must be specified with the `file` property (in the example, the file is `../../../rtl/Queue.v`. The entity may depend on other files, in which case these must be specified with the `dependencies` property. When exporting simulation or synthesis scripts from a network that instantiates an external task, that task's dependencies will be included in the script in the order in which they were declared and before the task itself. Paths can be absolute, or relative to the folder in which the Cx task is located.

## Cycle-accurate modeling

Cx models are cycle-accurate in the sense that the behavior of a task can be exactly **specified** cycle by cycle. Because the language supports ports with synchronization, a task may be **executed** with increased latency, in other words the behavior specification represents the best case scenario:

- data is always available from inputs
- there is no back-pressure from tasks later in the pipeline

Statements are executed **sequentially in the same cycle** (like in Verilog/VHDL), until a "**cycle break**" occurs. A cycle break causes the current cycle to end, and begins the definition of a new cycle.

### Explicit cycle breaks

Cycle breaks can be explicitly introduced with two instructions:

- `fence` ends the current cycle and introduces a new one.
- `idle(n)` where n is a constant integer ends the current cycle, and waits for n cycles.

In the example below, the task t1 defines a counter that updates every two cycles:

    network N {
      t1 = new task {
        out uint counter;
        uint count;
        void loop() {
          count++; // increments count
          fence;
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

- execute: (in task 1) count++; (in task 2) print("count = ", t1.counter.read);
- commit: no new values were written, nothing to commit.
- execute: (in task 1) counter.write(count); (in task 2) print("count = ", t1.counter.read);
- commit: updates the value on `counter` port. Note how this occurs **after** task 2 printed `counter`'s value.

The output of running this example is this (comments starting with -- added for readability):

    -- starting cycle 1
    count = 0 // count++ (= 1), task 2 prints current counter (= 0)
    -- starting cycle 2
    count = 0 // counter.write(count), task 2 prints current counter (= 0)
    -- starting cycle 3
    count = 1 // count++ (= 2), task 2 prints current counter (= 1)
    -- starting cycle 4
    count = 1 // counter.write(count), task 2 prints current counter (= 1)
    -- starting cycle 5
    count = 2 // count++ (= 3), task 2 prints current counter (= 2)
    -- starting cycle 6
    count = 2 // counter.write(count), task 2 prints current counter (= 2)
    ...

The task t1 is implemented with a two-state, two-edge Finite State Machine: `count++;` is executed during the first transition, and `counter.write(count);` is executed during the second transition (back to initial state).

Example of a task with a one-cycle setup and a two-cycle loop:

    task T {
      out short num;
      void setup() {
        print("init cycle");
      }

      void loop() {
        print("loop cycle 1");
        fence;
        print("loop cycle 2");
      }
    }

![Implicit cycle breaks](/images/semantics/fsm_init.png)

### Implicit cycle breaks

An implicit cycle break occurs:

1. when you read from a port you have already read in this cycle, or
2. when you write to a port you have already written in this cycle, or
3. before each iteration of a sequential loop.

For instance, in the following example, a first cycle is necessary to set the variable `t` to `0`, then the core of the loop is executed each cycle until `t == 16`

    for (t = 0; t < 16; t++) {
      u32 m = msg.read;
      W[t] = m;
    }

![FSM loop](/images/semantics/fsm_loop.png)

## Synchronization with ports

When a task declares [synchronized ports](/documentation/declarations#ports), it changes the semantics of accesses to ports in the following way. A `read` to a synchronized port that has no data will block. It is possible to test if data is available on a `sync` port using the `available` property.
