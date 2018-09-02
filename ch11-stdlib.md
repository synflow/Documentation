<p>Cx defines built-in tasks that are presented below.</p>

<h2><a class="anchor" id="std.fifo.SynchronousFIFO"></a>Synchronous FIFO</h2>

<p>A synchronous FIFO is used to store data between two entities that have different production/consumption rates.</p>

<pre><code class="cx">package std.fifo;

task SynchronousFIFO {
  const int size, width;

  sync ready {
    in unsigned&lt;width&gt; din;
    out unsigned&lt;width&gt; dout;
  }
}</code></pre>

<p>Definition of the parameters:</p>
<ul>
	<li><code>size</code>: required, represents the size of the FIFO in number of elements.</li>
	<li><code>width</code>: required, represents the width of data (read and written) in bits.</li>
</ul>

<h3>Usage</h3>

<p>A synchronous FIFO is connected between two tasks: a producer and a consumer.</p>
<ul>
  <li>The producer writes data to the <code>din</code> port unless the FIFO is full; when that happens, the FIFO indicates that the <code>din</code> port is no longer ready to accept any data, and remains that way until the FIFO is no longer full.</li>
  <li>The FIFO writes data to its <code>dout</code> port when the consumer is ready and when the FIFO is not empty. When the consumer is ready and the FIFO is empty, the consumer will be blocked until the FIFO is not empty. When the consumer is no longer accepting data, <code>dout</code> becomes "not ready", and the FIFO stops sending data until it becomes ready again.</li>
</ul>

<h2><a class="anchor" id="std.lib.SynchronizerFF"></a>Synchronizer Flip-Flop</h2>

<p>A synchronizer flip-flop is used for clock domain crossing of a 1-bit boolean signal. <code>din</code> is in the <code>din_clock</code> domain, and <code>dout</code> is in the <code>dout_clock</code> domain. This synchronizer is simply a shift-register (clocked by <code>dout_clock</code>) whose number of stages is given by the <code>stages</code> parameter. The number of stages may be increased for high-speed applications and/or hardened (fault-tolerant) hardware. Use a smaller number of stages at your own risk.</p>

<pre><code class="cx">package std.lib;

task SynchronizerFF {
  properties { clocks: ["din_clock", "dout_clock"] }
  const int stages = 2;
  in bool din; out bool dout;
}</code></pre>

<h2><a class="anchor" id="std.lib.SynchronizerMux"></a>Synchronizer Mux</h2>

<p>A synchronizer mux is used for clock domain crossing of a N-bit signal. Like for the synchronizer flip-flop, <code>din</code> is in the <code>din_clock</code> domain, and <code>dout</code> is in the <code>dout_clock</code> domain. The <code>width</code> parameter controls the size of the <code>din</code> and <code>dout</code> ports, and the <code>stages</code> parameter has the same meaning as in the synchronizer flip-flop.</p>

<p>The synchronizer mux works as follows:</p>
<ol>
	<li>another task writes a new value to <code>din</code>, which is declared as a <code class="cx">sync</code> port.</li>
	<li>the synchronization signal crosses from the input clock domain to the output clock domain using the synchronizer flip-flop shown before.</li>
	<li>during this time, the value must be kept stable (no new value must be written to the input port!).</li>
	<li>when the synchronization signal arrives in the output clock domain, the value is sampled and written to the <code>dout</code> output port.</li>
</ol>

<pre><code class="cx">package std.lib;

task SynchronizerMux {
  properties { clocks: ["din_clock", "dout_clock"] }
  const int width = 16, stages = 2;
  in sync unsigned&lt;width&gt; din; out unsigned&lt;width&gt; dout;
}</code></pre>

<h2><a class="anchor" id="std.mem.SinglePortRAM"></a>Single-port RAM</h2>

<p>The single-port RAM has the following signature:</p>

<pre><code class="cx">package std.mem;

task SinglePortRAM {
  properties { reset: null } // no reset, single default clock

  const int size, width, depth = sizeof(size - 1);
  const bool writeShiftMode = false, addOutputRegister = false;

  in unsigned&lt;depth&gt; address, sync unsigned&lt;width&gt; data; out unsigned&lt;width&gt; q;
}</code></pre>

<p>Definition of the parameters:</p>
<ul>
	<li><code>size</code>: required, represents the size of the RAM in number of elements. The total size in bits of the RAM is <code>size * width</code>.</li>
	<li><code>width</code>: required, represents the width of data (read and written) in bits.</li>
  <li><code>writeShiftMode</code>: optional, <code class="cx">false</code> by default. Governs how the RAM behaves when writing a value to the RAM. By default (<code class="cx">writeShiftMode == false</code>), the RAM will output the new value on its output port <code>q</code>. When <code class="cx">writeShiftMode == true</code>, the RAM will instead "shift" the memory cell at the given address and output the <em>previous</em> value on the <code>q</code> output port. This is known as "Read before Write" or "Read during Write: old data" on FPGA.</li>
	<li><code>addOutputRegister</code>: optional, <code class="cx">false</code> by default. Adds an output register, effectively delaying the value available on the <code>q</code> output port by one cycle. When using RAM on FPGA this can improve performance (higher frequency) and make routing easier.</li>
</ul>

<p>Definition of the ports:</p>
<ul>
	<li><code>address</code>: input port that specifies the address at which data is to be read or written. The type of this port depends on <code>depth</code>, which is the number of bits needed to represent the maximum address based on the RAM's size. For example, a RAM with size = 1024 can use addresses from 0 to 1023, and therefore <code>depth = sizeof(1023) = 10</code>.</li>
	<li><code>data</code>: synchronized input port, acting as a combined write enable/value. When a value is available on this port (when another entity writes a value to it), the RAM operates in write mode, and writes the value at the address present on the <code>address</code> port. When no data is available on this port, the RAM operates in read mode.</li>
	<li><code>q</code>: output port that contains the value read/written at the address given by the <code>address</code> port at the previous cycle.</li>
</ul>

<h3>Usage</h3>

<p>It can be instantiated as follows:</p>

<pre><code class="cx">ram = new std.mem.SinglePortRAM({size: 32, width: 128});</code></pre>
or using the short form:
<pre><code class="cx">network N {
  import std.mem.SinglePortRAM;

  ram = new SinglePortRAM({size: 32, width: 128});
}</code></pre>

<p>This defines a RAM of 32 elements * 128 bit each = 4Kbits.</p>

<p>Example usage:</p>
<pre><code class="cx">network N {
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
}</code></pre>

<p>In this example, note how the <code>ctrl</code> task issues two consecutive reads, and uses a <code class="cx">fence</code>. This is because a read to the RAM has a latency of 1 cycle in this case. Issuing two reads in two consecutive cycles reduces the overall latency from 6 cycles to 4. We need the <code class="cx">fence</code> to force the start of a new cycle (cycle 3) so that we read the proper value on <code>q</code>.</p>

<h2><a class="anchor" id="std.mem.DualPortRAM"></a>Dual-port RAM</h2>

<p>A dual-port RAM can issue two reads or two writes or one read and one write simultaneously. Cx defines a dual-port RAM with the following signature:</p>

<pre><code class="cx">package std.mem;

task DualPortRAM {
  properties { reset: null, clocks: ["clock_a", "clock_b"] }
  const int size, width, depth = sizeof(size - 1);
  in uint&lt;depth&gt; address_a, sync uint&lt;width&gt; data_a; out uint&lt;width&gt; q_a;
  in uint&lt;depth&gt; address_b, sync uint&lt;width&gt; data_b; out uint&lt;width&gt; q_b;
}</code></pre>

<p>The parameters have the exact same meaning as <a href="instantiation/stdlib/#std.mem.SinglePortRAM">SinglePortRAM</a>'s parameters.</p>

<p>The task has no reset, and defines two clocks <code>clock_a</code> and <code>clock_b</code>.</p>

<p>Each group of ports (ports ending with <code>_a</code> and ports ending with <code>_b</code>) has the same meaning as the SinglePortRAM ports. Ports in group a are relative to clock_a, and ports in group b are relative to clock_b.</p>

<h2><a class="anchor" id="std.mem.PseudoDualPortRAM"></a>Pseudo Dual-port RAM</h2>

<p>A pseudo dual-port RAM is a trade-off between a single-port RAM (less area, lower throughput) and a dual-port RAM (more area, higher throughput). Like a dual-port RAM, it has two address ports and can issue one read and one write simultaneously, and like a single-port RAM it cannot issue two reads or two writes at the same time. Cx defines a pseudo dual-port RAM with the following signature:</p>

<pre><code class="cx">package std.mem;

task PseudoDualPortRAM {
  properties { reset: null, clocks: ["rd_clock", "wr_clock"] }
  const int size, width, depth = sizeof(size - 1);
  in uint&lt;depth&gt; rd_address, wr_address, sync uint&lt;width&gt; data; out uint&lt;width&gt; q;
}</code></pre>

<p>The parameters have the exact same meaning as the parameters of <a href="instantiation/stdlib/#std.mem.SinglePortRAM">SinglePortRAM</a> and <a href="instantiation/stdlib/#std.mem.DualPortRAM">DualPortRAM</a>.</p>

<p>The task has no reset, and defines two clocks <code>rd_clock</code> and <code>wr_clock</code>.</p>

<p>Definition of the ports:</p>
<ul>
	<li><code>rd_address</code>: input port that specifies the address at which data is to be read. Relative to <code>rd_clock</code>.</li>
	<li><code>wr_address</code>: input port that specifies the address at which data is to be written. Relative to <code>wr_clock</code>.</li>
	<li><code>data</code>: synchronized input port, acting as a combined write enable/value. When a value is available on this port, the RAM operates in write mode, and writes the value at the address present on the <code>address</code> port. Relative to <code>wr_clock</code>.</li>
	<li><code>q</code>: output port that contains the value read at the address given by the <code>rd_address</code> port at the previous cycle. Relative to <code>rd_clock</code>.</li>
</ul>

<p>Usage is quite similar to <a href="instantiation/stdlib/#std.mem.SinglePortRAM">SinglePortRAM</a>, except that reads and writes can occur simultaneously and may be freely interleaved.</p>
