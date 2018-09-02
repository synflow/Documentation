Designing Hardware is the process of creating chips. The most well-known example of chip is the processor, which powers your computer, phone, or tablet. Formally speaking, a chip is a kind of electronic component that contains an integrated circuit (IC) dedicated/optimized for a given application or set of applications. ICs are implemented with transistors, and a leading-edge complex digital IC contains billions of transistors.

![Example of chip on a board](/images/introduction/chip.png "Example of chip on a board")

**Digital** hardware manipulates data (bits) expressed with discrete boolean values: true ("1") and false ("0"). These values correspond to voltage levels, false is zero volts (ground), and true is a voltage that is technology-dependent (for example 3.3V). **Analog** hardware, by opposition, represents signals with continuous values: the range of a signal is mapped to the range of the voltage.

## How hardware works

You can watch a good introduction to hardware design with an FPGA on the video below.

Digital hardware is designed with *synchronous logic*. Synchronous logic consists of (many) registers,  combinational circuits, and combinatorial circuits. A register is a tiny memory cell holding one bit, and its value is updated at the beginning of each clock **cycle**, which is known as a rising edge (transition from 0 to 1). Therefore, a higher clock speed means that registers can update their value more frequently, which is why processor vendors were always seeking to increase that speed (until it could no longer increase because of excessive thermal dissipation, but that's another story).

![What a clock signal looks like.](/images/introduction/clock.png "What a clock signal looks like")

The next value of a register is a function of the current values of registers, and that function is described with combinational logic. Combinational logic is based on boolean operations (and, or, not) and can describe complex operations such as addition and multiplication by composing these boolean operations. In hardware, boolean operations are implemented with gates, and going through gates takes a small, but not negligible time (propagation delay). One challenge of hardware design is thus making sure that combinational logic take less time to complete than the duration of a cycle. Otherwise the new value may arrive at the register too late, *after* the beginning of the *next* clock period, and this may cause the circuit to not work properly.

![Overview of synchronous digital hardware](/images/introduction/logic.png "Overview of synchronous digital hardware")

## The Cx programming language

Cx is a **programming language for hardware design**. It offers a tremendous productivity improvement for developers, makers, and designers who want to program FPGAs or design ASICs. Indeed, Cx enables programers to write programs to be implemented as digital hardware rather than to describe the behavior of circuit using blocking and non-blocking assignment, state-machines, registers, and signal strengths (see Register Transfer Languages such as [Verilog](https://en.wikipedia.org/wiki/Verilog)).

Cx programs are structured as a set of entities connected together and executed concurrently, because this is the easiest and most natural way to describe parallelism that corresponds to how hardware behaves. An entity is either an atomic sequential task, or a network that instantiates other entities.

### Tasks

A task is a sequential process that may declare two special functions `setup` and `loop` (like Arduino). The `setup` function, if defined, is called *once* when the task starts (after reset of the chip on which the task executes). Then the `loop` function is called and as its name suggests, it loops endlessly: once the function completes, it starts again. A task may define any number of functions that can be called from `setup` and `loop`.

    task MinimalTask {
    
      void setup() {
        // (optional) setup code that runs once after reset
      }
    
      void loop() {
        // main code that executes repeatedly
      }
    
    }

A task can declare state variables: a state variable may have an initial value (otherwise it is initialized to zero) and is used to store values during the execution of the task. State variables are *private*: no other entity than the task declaring a state variable can access it. Tasks communicate together though *ports*, reading data from ports and writing data to ports. The following task is a timer that fires every 10,000 invocations, writing `true` to the boolean output port `done`.

    task Timer {
      out bool done;
    
      const uint BOUND = 10_000;
      unsigned int count;
    
      void loop() {
        if (count == BOUND - 1) {
          count = 0;
          done.write(true);
        } else {
          count++;
          done.write(false);
        }
      }
    
    }

### Networks

A network is an entity that may have properties, input and output ports, and that can instantiate other entities (thus allowing a hierarchical modeling style). An application (or hardware design) generally has a few top-level networks that instantiate lower-level tasks and networks, which in turn instantiate even lower-level tasks and networks, etc. Instances communicate with each other through input and output ports.

    network HelloWorld {
      out u8 seg;
    
      wordToDisplay = new WordToDisplay();
    
      driverSegment = new DriverSegment();
      driverSegment.reads(wordToDisplay.character);
    
      this.reads(driverSegment.seg);
    }
    

A network can instantiate as many lower-level tasks and networks as required. The network window offers a view of the current network.

![The network view](/images/introduction/helloView.png "The network view")

## Programming model

The programming model behind Cx is inspired by [Kahn process networks](http://en.wikipedia.org/wiki/Kahn_process_networks) (KPN) and its lesser-known extension *dataflow process networks* (DPN). KPN defines sequential processes that communicate with FIFO (First In First Out) queues with blocking reads and non-blocking writes. DPN extends KPN by (1) allowing a process to test the presence of data, and (2) modeling a process with **discrete non-interruptible execution rules**, such that invoking a task is equivalent to repeatedly picking an appropriate execution rule and applying it. DPN is a good candidate for modeling hardware, but describing a process as a sequence of distinct steps is tedious to say the least, and as a matter of fact this is *how hardware designers have been designing hardware for decades*, surely we can do better.

![A process network with four tasks](/images/introduction/process-network.png "A process network with four tasks")

A task is described in Cx as a sequential process, and the compiler transforms that description to a series of execution rules. Kahn and dataflow process networks use infinite FIFOs between processes, which may be fine for studying properties of programs, but for actual implementation it is overkill and impractical. For communications Cx supports several kinds of ports that offer different expressiveness/performance tradeoffs:

- default ports are *bare metal* (no overhead) with non-blocking reads and non-blocking writes,
- *synchronized* ports (`sync`) are ports that have blocking reads and that support testing if data is available on the port. If data is present on the port, it must be consumed immediately or is discarded.
- synchronous ports with ready information (`sync ready`) are synchronized ports that also have blocking writes and that support testing whether a write would block.
- synchronized ports with acknowledgement (`sync ack`) are synchronized ports that ensure data integrity: writing to a port can only succeed if previous data has been acknowledged.

As stated above, **an execution rule is non-interruptible** so how can reads or writes be blocking? When a task is invoked, either an appropriate rule can be found whose required conditions are met (such as the presence of data), and it executes without interruption; or no rule is picked. Thus when we say that reads or writes may **block**, what it really means is that if we have a rule that needs data from a port and this port has no data, then that rule's execution will be deferred. There is no requirement as to when this may happen, and so rules are regularly evaluated to know which one can be executed.

## Mapping Cx to digital hardware

Each Cx task is mapped to an independent hardware entity/module. Entities are specialized using the values they are given when instantiated. The compiler handles the transformation from the sequential Cx code to a finite state machine in hardware. To this end, the compiler walks through the code, inlines function calls, splits sequential code by creating a new execution rule when implicit breaks occur (such as reading twice from the same port), flattens control flow (for instance conditionals containing loop statements), creates parallel execution rules for non breaking if statements, etc.

An execution rule corresponds exactly to one hardware cycle, which is pretty convenient because hardware operates in a *cycle-accurate* manner. This mapping maximizes productivity and performance, since it makes it possible to know in advance how a piece of Cx code will be scheduled in hardware. That said, in a future version of the language, we'd like to support properties to give the compiler more leeway to schedule operations differently when it would be beneficial.
