Cx defines built-in tasks that are presented below.

## Synchronous FIFO

A synchronous FIFO is used to store data between two entities that have different production/consumption rates.

    package std.fifo;

    task SynchronousFIFO {
      const int size, width;

      sync ready {
        in unsigned<width> din;
        out unsigned<width> dout;
      }
    }

Definition of the parameters:

- `size`: required, represents the size of the FIFO in number of elements.
- `width`: required, represents the width of data (read and written) in bits.

### Usage

A synchronous FIFO is connected between two tasks: a producer and a consumer.

- The producer writes data to the `din` port unless the FIFO is full; when that happens, the FIFO indicates that the `din` port is no longer ready to accept any data, and remains that way until the FIFO is no longer full.
- The FIFO writes data to its `dout` port when the consumer is ready and when the FIFO is not empty. When the consumer is ready and the FIFO is empty, the consumer will be blocked until the FIFO is not empty. When the consumer is no longer accepting data, `dout` becomes "not ready", and the FIFO stops sending data until it becomes ready again.

## Synchronizer Flip-Flop

A synchronizer flip-flop is used for clock domain crossing of a 1-bit boolean signal. `din` is in the `din_clock` domain, and `dout` is in the `dout_clock` domain. This synchronizer is simply a shift-register (clocked by `dout_clock`) whose number of stages is given by the `stages` parameter. The number of stages may be increased for high-speed applications and/or hardened (fault-tolerant) hardware. Use a smaller number of stages at your own risk.

    package std.lib;

    task SynchronizerFF {
      properties { clocks: ["din_clock", "dout_clock"] }
      const int stages = 2;
      in bool din; out bool dout;
    }

## Synchronizer Mux

A synchronizer mux is used for clock domain crossing of a N-bit signal. Like for the synchronizer flip-flop, `din` is in the `din_clock` domain, and `dout` is in the `dout_clock` domain. The `width` parameter controls the size of the `din` and `dout` ports, and the `stages` parameter has the same meaning as in the synchronizer flip-flop.

The synchronizer mux works as follows:

1. another task writes a new value to `din`, which is declared as a `sync` port.
2. the synchronization signal crosses from the input clock domain to the output clock domain using the synchronizer flip-flop shown before.
3. during this time, the value must be kept stable (no new value must be written to the input port!).
4. when the synchronization signal arrives in the output clock domain, the value is sampled and written to the `dout` output port.

   package std.lib;

   task SynchronizerMux {
   properties { clocks: ["din_clock", "dout_clock"] }
   const int width = 16, stages = 2;
   in sync unsigned<width> din; out unsigned<width> dout;
   }

## Single-port RAM

The single-port RAM has the following signature:

    package std.mem;

    task SinglePortRAM {
      properties { reset: null } // no reset, single default clock

      const int size, width, depth = sizeof(size - 1);
      const bool writeShiftMode = false, addOutputRegister = false;

      in unsigned<depth> address, sync unsigned<width> data; out unsigned<width> q;
    }

Definition of the parameters:

- `size`: required, represents the size of the RAM in number of elements. The total size in bits of the RAM is `size * width`.
- `width`: required, represents the width of data (read and written) in bits.
- `writeShiftMode`: optional, `false` by default. Governs how the RAM behaves when writing a value to the RAM. By default (`writeShiftMode == false`), the RAM will output the new value on its output port `q`. When `writeShiftMode == true`, the RAM will instead "shift" the memory cell at the given address and output the _previous_ value on the `q` output port. This is known as "Read before Write" or "Read during Write: old data" on FPGA.
- `addOutputRegister`: optional, `false` by default. Adds an output register, effectively delaying the value available on the `q` output port by one cycle. When using RAM on FPGA this can improve performance (higher frequency) and make routing easier.

Definition of the ports:

- `address`: input port that specifies the address at which data is to be read or written. The type of this port depends on `depth`, which is the number of bits needed to represent the maximum address based on the RAM's size. For example, a RAM with size = 1024 can use addresses from 0 to 1023, and therefore `depth = sizeof(1023) = 10`.
- `data`: synchronized input port, acting as a combined write enable/value. When a value is available on this port (when another entity writes a value to it), the RAM operates in write mode, and writes the value at the address present on the `address` port. When no data is available on this port, the RAM operates in read mode.
- `q`: output port that contains the value read/written at the address given by the `address` port at the previous cycle.

### Usage

It can be instantiated as follows:

    ram = new std.mem.SinglePortRAM({size: 32, width: 128});

or using the short form:

    network N {
      import std.mem.SinglePortRAM;

      ram = new SinglePortRAM({size: 32, width: 128});
    }

This defines a RAM of 32 elements \* 128 bit each = 4Kbits.

Example usage:

    network N {
      ram = new std.mem.SinglePortRAM({size: 32, width: 128});
      ctrl = new task {
        void loop() {
          // first write
          ram.address.write(8);
          ram.data.write(13);

          // second write
          ram.address.write(21);
          ram.data.write(34);

          // issue two reads
          ram.address.write(8); // cycle 1
          ram.address.write(21); // cycle 2
          fence;
          print("read @8 = ", ram.q.read()); // cycle 3
          print("read @21 = ", ram.q.read()); // cycle 4
        }
      };
    }

In this example, note how the `ctrl` task issues two consecutive reads, and uses a `fence`. This is because a read to the RAM has a latency of 1 cycle in this case. Issuing two reads in two consecutive cycles reduces the overall latency from 6 cycles to 4. We need the `fence` to force the start of a new cycle (cycle 3) so that we read the proper value on `q`.

## Dual-port RAM

A dual-port RAM can issue two reads or two writes or one read and one write simultaneously. Cx defines a dual-port RAM with the following signature:

    package std.mem;

    task DualPortRAM {
      properties { reset: null, clocks: ["clock_a", "clock_b"] }
      const int size, width, depth = sizeof(size - 1);
      in uint<depth> address_a, sync uint<width> data_a; out uint<width> q_a;
      in uint<depth> address_b, sync uint<width> data_b; out uint<width> q_b;
    }

The parameters have the exact same meaning as [SinglePortRAM](instantiation/stdlib/#std.mem.SinglePortRAM)'s parameters.

The task has no reset, and defines two clocks `clock_a` and `clock_b`.

Each group of ports (ports ending with `_a` and ports ending with `_b`) has the same meaning as the SinglePortRAM ports. Ports in group a are relative to clock_a, and ports in group b are relative to clock_b.

## Pseudo Dual-port RAM

A pseudo dual-port RAM is a trade-off between a single-port RAM (less area, lower throughput) and a dual-port RAM (more area, higher throughput). Like a dual-port RAM, it has two address ports and can issue one read and one write simultaneously, and like a single-port RAM it cannot issue two reads or two writes at the same time. Cx defines a pseudo dual-port RAM with the following signature:

    package std.mem;

    task PseudoDualPortRAM {
      properties { reset: null, clocks: ["rd_clock", "wr_clock"] }
      const int size, width, depth = sizeof(size - 1);
      in uint<depth> rd_address, wr_address, sync uint<width> data; out uint<width> q;
    }

The parameters have the exact same meaning as the parameters of [SinglePortRAM](instantiation/stdlib/#std.mem.SinglePortRAM) and [DualPortRAM](instantiation/stdlib/#std.mem.DualPortRAM).

The task has no reset, and defines two clocks `rd_clock` and `wr_clock`.

Definition of the ports:

- `rd_address`: input port that specifies the address at which data is to be read. Relative to `rd_clock`.
- `wr_address`: input port that specifies the address at which data is to be written. Relative to `wr_clock`.
- `data`: synchronized input port, acting as a combined write enable/value. When a value is available on this port, the RAM operates in write mode, and writes the value at the address present on the `address` port. Relative to `wr_clock`.
- `q`: output port that contains the value read at the address given by the `rd_address` port at the previous cycle. Relative to `rd_clock`.

Usage is quite similar to [SinglePortRAM](instantiation/stdlib/#std.mem.SinglePortRAM), except that reads and writes can occur simultaneously and may be freely interleaved.


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
