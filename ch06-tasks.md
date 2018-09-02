<p>A task is an atomic entity that may have properties, input ports, output ports, state variables, and functions. A task may have any number of functions, as well as special functions: an initialization function called <code>setup</code> and the <code>loop</code> function that is the entry point of the task when it runs.</p>

<p>By default a task is synchronous (clock and reset are implicit), though this behavior can be changed with properties.</p>

<p>A task can have both state variables and constants. Contrary to the implicitly constant definitions in a bundle, the definition of a constant in a task requires an explicit <strong>const</strong> qualifier:</p>

<pre><code class="cx">// i is a 16-bit state variable initialized to 0
short i = 0;

// LENGTH_PRE is a 16-bit constant equal to 6
const short LENGTH_PRE = 6;
</code></pre>

<h2><a class="anchor" id="external"></a>External tasks</h2>

<p>An external task is a task with only a signature (parameters, clocks, reset, ports). Its implementation is done by the designer (in Verilog, VHDL, or another language) and must be indicated in the properties of the task.</p>

<p>When you have existing code (for example VHDL or Verilog) that you wish to call from Cx, you must declare a task with the same signature (parameters, clocks, resets, ports) as your code. This kind of task is said to have an external implementation: it has no "setup" or "loop" methods, and no code will be generated from it, but otherwise you can instantiate it and connect it to other entities just as with any other task.</p>

<p>To declare an external task, create a task with an "external" implementation:</p>

<pre><code class="cx">package com.acme;

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

  in sync uint&lt;width&gt; din; out sync uint&lt;width&gt; dout, bool ack;

}</code></pre>

<p>Parameters in your code are declared as constants in the Cx code, like <em>depth</em> and <em>width</em> in the example above. Typically these constants will be overridden when the task is instantiated: <code class="cx">new Queue({depth: 32, width: 8})</code>.</p>

<p>The implementation of the task must be in a file whose name must be specified with the <code class="cx">file</code> property (in the example, the file is <code>../../../rtl/Queue.v</code>. The entity may depend on other files, in which case these must be specified with the <code class="cx">dependencies</code> property. When exporting simulation or synthesis scripts from a network that instantiates an external task, that task's dependencies will be included in the script in the order in which they were declared and before the task itself. Paths can be absolute, or relative to the folder in which the Cx task is located.</p>

<h2><a class="anchor" id="cycle_modeling"></a>Cycle-accurate modeling</h2>

<p>Cx models are cycle-accurate in the sense that the behavior of a task can be exactly <strong>specified</strong> cycle by cycle. Because the language supports ports with synchronization, a task may be <strong>executed</strong> with increased latency, in other words the behavior specification represents the best case scenario:</p>
<ul>
	<li>data is always available from inputs</li>
	<li>there is no back-pressure from tasks later in the pipeline</li>
</ul>

<p>Statements are executed <strong>sequentially in the same cycle</strong> (like in Verilog/VHDL), until a "<strong>cycle break</strong>" occurs. A cycle break causes the current cycle to end, and begins the definition of a new cycle.</p>

<h3><a class="anchor" id="explicit_cycle_breaks"></a>Explicit cycle breaks</h3>

<p>Cycle breaks can be explicitly introduced with two instructions:</p>

<ul>
	<li><code class="cx">fence</code> ends the current cycle and introduces a new one.</li>
	<li><code class="cx">idle(n)</code> where n is a constant integer ends the current cycle, and waits for n cycles.</li>
</ul>

<p>In the example below, the task t1 defines a counter that updates every two cycles:</p>

<pre class="pre-scrollable"><code class="cx">network N {
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
}</code></pre>

<p>This network is executed as follows:</p>

<ul>
	<li>execute: (in task 1) count++; (in task 2) print("count = ", t1.counter.read);</li>
	<li>commit: no new values were written, nothing to commit.</li>
	<li>execute: (in task 1) counter.write(count); (in task 2) print("count = ", t1.counter.read);</li>
	<li>commit: updates the value on <code>counter</code> port. Note how this occurs <strong>after</strong> task 2 printed <code>counter</code>'s value.</li>
</ul>

<p>The output of running this example is this (comments starting with -- added for readability):</p>

<pre>-- starting cycle 1
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
...</pre>

<p>The task t1 is implemented with a two-state, two-edge Finite State Machine: <code class="cx">count++;</code> is executed during the first transition, and <code class="cx">counter.write(count);</code> is executed during the second transition (back to initial state).</p>

<p>Example of a task with a one-cycle setup and a two-cycle loop:</p>

<pre><code class="cx">task T {
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
</code></pre>

<img class="img-responsive" src="/images/documentation/semantics/fsm_init.png" />

<h3><a class="anchor" id="implicit_cycle_breaks"></a>Implicit cycle breaks</h3>

<p>An implicit cycle break occurs:</p>
<ol>
	<li>when you read from a port you have already read in this cycle, or</li>
	<li>when you write to a port you have already written in this cycle, or</li>
	<li>before each iteration of a sequential loop.</li>
</ol>

<p>For instance, in the following example, a first cycle is necessary to set the variable <code class="cx">t</code> to <code class="cx">0</code>, then the core of the loop is executed each cycle until <code class="cx">t == 16</code></p>

<div class="row">
<div class="col-sm-6">
<pre class="pre-scrollable"><code class="cx">for (t = 0; t &lt; 16; t++) {
  u32 m = msg.read;
  W[t] = m;
}</code></pre>
</div>
<div class="col-sm-6">
<img class="img-responsive" src="/images/documentation/semantics/fsm_loop.png" />
</div>
</div>

<h2><a class="anchor" id="ports"></a>Synchronization with ports</h2>

<p>When a task declares <a href="/documentation/declarations#ports">synchronized ports</a>, it changes the semantics of accesses to ports in the following way. A <code class="cx">read</code> to a synchronized port that has no data will block. It is possible to test if data is available on a <code class="cx">sync</code> port using the <code class="cx">available</code> property.</p>
