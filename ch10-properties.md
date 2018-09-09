Properties are arbitrary data associated with a task or network that are written in a simple syntax (related to Javascript/JSON).

## Definition of properties

### Syntax

Properties can describe three types of elements:

1. A primitive can be a string (surrounded by single or double quotes), a number (float or integer, supporting the full Cx syntax with prefixes 0b and 0x, and "\_" digit separator), a boolean (**true** or **false**), or the **null** literal.
2. An array is surrounded by square brackets **[** and **]**, and contains elements separated with commas.
3. An object is surrounded with curly brackets **{** and **}**, and contains key-value pairs separated with commas, where the key is an identifier, and the value is an element.

That's all there is to it.

For example, this is an object with two properties "clocks" and "reset", associated respectively with an array and an object:

    {
      clocks: ["din", "dout"],
      reset: {type: "synchronous", active: "high"}
    }

### Declaration

Properties are defined in a task or network with the **properties** keyword followed by an object. Properties can also be given to an instance as its first argument, for example to associate clocks and reset signals.

## Clocks declaration

To specify the clocks of a task or network, use the _clocks_ key. The value is an array of clock names:

    properties {
      clocks: ["din", "dout"]
    }

The value of _clocks_ must be an array, and the clock names must be valid identifiers. To declare a single clock, you can use the _clock_ property (without 's') as follows: `clock: "clk"` is a synonym for `clocks: ["clk"]`. To indicate the task/network has no clocks, you can either write `clock: null` or equivalently `clocks: []`.

A preferable way of declaring an entity with no clocks is to use the [type](properties/#type) property to indicate that the task/network is combinational.

### Implicit value

In the absence of the _clocks_ property, the following clock is defined: `clocks: ["clock"]`. This defines a single clock named "clock".

## Clocks association

In most cases, when a network [instantiates](/documentation/instantiation/) an entity (task or network), clocks are associated with clocks declared by the entity implicitly. Here is the list of cases where no explicit clocks association is required:

- the entity declares no clocks (it is combinational). The number of clocks declared by the network does not matter.
- the entity and the network declares the same number of clocks. Clocks are associated together in the following manner: the first clock of the network is associated with the first clock of the entity, and so on.
- the network has only one clock, and the entity declares multiple clocks. The network's clock is associated multiple times, once with each of the entity's clocks.

### Explicit association

In short, explicit association is necessary each time a network declares more clocks than a synchronous entity it instantiates. This is done with a `clocks` property specified at the instantiation site, with a value that can be:

1. an object whose keys are the entity's clock names (in this case the clock names are those declared by the DualPortRAM entity):

   ram = new DualPortRAM({
   clocks: {rd*clock: "clk_recv", wr_clock: "clk_send"},
   /* other properties \_/
   depth: 8, width: 12
   });

2. or an array of clock names:

   ram = new DualPortRAM({
   clocks: ["clk_recv", "clk_send"],
   /_ other properties _/
   depth: 8, width: 12
   });

### Clock association with inner tasks

This corner case arises when you use an inner task within a network that declares multiple clocks. Whether this is a good idea or not is debatable, that said how do you associate clocks in this case? Unlike instances of classical entities, an instance of an inner task accepts no arguments and no properties. The trick in this case is that the task's properties are used both for declaration and instantiation, as follows:

    network N {
      properties {
        clocks: ["clk_a", "clk_b"]
      }

      inner_a = new task {
        properties { clock: "clk_a" }
        void loop() {
          print("inner a running");
        }
      };

      inner_b = new task {
        properties { clock: "clk_b" }
        void loop() {
          print("inner b running");
        }
      };
    }

In this case, note that the entities that will be generated with use as clock names clk_a and clk_b respectively.

## Reset

To specify the reset properties of a task or network, use the _reset_ key. The value is either **null** (to specify that the task/network has no reset), or an object with properties _type_, _active_, and _name_:

    properties {
      reset: {type: "synchronous", active: "high", name: "reset_p"}
    }

Valid values for _type_ are "asynchronous" and "synchronous". _active_ can be either "high" or "low". _name_ is the reset signal's name.

### Implicit values

In the absence of a _reset_ key, the following reset is defined:

    properties {
      reset: {type: "asynchronous", active: "low", name: "reset_n"}
    }

In the absence of the _name_ key, if the reset is active low, the name is "reset_n", else if it is active high, the name is simply "reset".

### Use with instances

When passing properties to an instance, the _reset_ key can be associated with the name of a signal that should be used as a reset for the instance (in this example, the "ready" signal will be used as SimpleTask's reset):

    new SimpleTask({reset: "ready"})

## Type

To define a task or network as "combinational", use the _type_ key with the value "combinational":

    properties {
      type: "combinational"
    }

The "combinational" value is equivalent to the following properties (in other words no clock and no reset): `{clocks: [], reset: null}`

## Test

The _test_ property allows you to test a task or a network by specifying stimulus/expected values for ports cycle by cycle. The value of _test_ is an object that associates each port name with an array that contains one value per cycle. For example, to test a Run-Length Encoding task whose ports are `in sync u8 data, out sync value, sync u15 count;`, you can write the following properties:

    properties {
      test: {
        data:  [    6, 5,    5, 4,    4,    4, 3,    3,    3,    3, 2 ],
        value: [ null, 6, null, 5, null, null, 4, null, null, null, 3 ],
        count: [ null, 1, null, 2, null, null, 3, null, null, null, 4 ]
      }
    }

Values must be of the same type as the port, for example you can use `true` and `false` for `bool` ports. When no value is expected on a port in a cycle, use `null`.

This test suite is interpreted as follows:

1. on cycle 0, write 6 to port `data`, and expect no data on ports `value` and `count`. It is an error if a `sync` port is written to when it is not supposed to.
2. on cycle 1, write 5 to port `data`, and expect 6 to be written to `value` and 1 to be written to `count`.

We use a classic "black box" approach where a testbench instantiates the task, henceforth known as a Design Under Test (DUT). In this approach, the testbench checks values outside of the DUT - **after** the DUT has written them. This occurs one cycle later (as part of good coding style, all output ports are registered in the RTL code generated from Cx), as can be seen on the waveform:
![Test properties](/images/documentation/properties/properties_test.png)
Test values are specified cycle by cycle to help verify synchronization issues. After all a `sync` port can wait for data, so technically you could allow the same thing for tests, and never specify `null` values. As a matter of fact, that is what we used to do, and the problem was that a test would not test the synchronization behavior. We realized when we switched to truly cycle-accurate tests that some automated unit tests had synchronization issues that had been left undetected before.

The stimulus/expected values are in the format designed and used by [Adacsys](http://www.adacsys.com/).

## Implementation

The _implementation_ key allows the designer to tell the code generator what to produce from a Cx task or network. This property is meant for entities declared in Cx and implemented in Verilog or VHDL (or any other language or format, really). The value is an object that can define a type, file and dependencies:

    properties {
      implementation: {
        type: "external",
        file: "../../top.v",
        dependencies: ["../../entity_a.v", "../../entity_b.v"]
      }
    }

The _type_ property supports two values: "builtin" (reserved for entities defined by the compiler) and "external" (for entities that you define in Verilog or VHDL). The file that implements the task declaring these properties is specified with the _file_ property.

In a future version, "implementation" could be extended to embed code in Cx, for example:

    properties {
      implementation: {
        vhdl: {s0_s1: "a <= a xor b;"},
        verilog: {s0_s1: "a = a ^ b;"}
      }
    }
