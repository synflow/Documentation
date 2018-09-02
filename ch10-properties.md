<p>Properties are arbitrary data associated with a task or network that are written in a simple syntax (related to Javascript/JSON).</p>

<h2>Definition of properties</h2>

<h3>Syntax</h3>
<p>Properties can describe three types of elements:</p>
<ol>
	<li>A primitive can be a string (surrounded by single or double quotes), a number (float or integer, supporting the full Cx syntax with prefixes 0b and 0x, and "_" digit separator), a boolean (<strong>true</strong> or <strong>false</strong>), or the <strong>null</strong> literal.</li>
	<li>An array is surrounded by square brackets <strong>[</strong> and <strong>]</strong>, and contains elements separated with commas.</li>
	<li>An object is surrounded with curly brackets <strong>{</strong> and <strong>}</strong>, and contains key-value pairs separated with commas, where the key is an identifier, and the value is an element.</li>
</ol>
<p>That's all there is to it.</p>

<p>For example, this is an object with two properties "clocks" and "reset", associated respectively with an array and an object:</p>
<pre><code class="cx">{
  clocks: ["din", "dout"],
  reset: {type: "synchronous", active: "high"}
}</code></pre>

<h3>Declaration</h3>

<p>Properties are defined in a task or network with the <strong>properties</strong> keyword followed by an object. Properties can also be given to an instance as its first argument, for example to associate clocks and reset signals.</p>

<h2><a class="anchor" id="clocks-decl"></a>Clocks declaration</h2>

<p>To specify the clocks of a task or network, use the <em>clocks</em> key. The value is an array of clock names:</p>

<pre><code class="cx">properties {
  clocks: ["din", "dout"]
}</code></pre>

<p>The value of <em>clocks</em> must be an array, and the clock names must be valid identifiers. To declare a single clock, you can use the <em>clock</em> property (without 's') as follows: <code class="cx">clock: "clk"</code> is a synonym for <code class="cx">clocks: ["clk"]</code>. To indicate the task/network has no clocks, you can either write <code class="cx">clock: null</code> or equivalently <code class="cx">clocks: []</code>.</p>

<p>A preferable way of declaring an entity with no clocks is to use the <a href="properties/#type">type</a> property to indicate that the task/network is combinational.</p>

<h3>Implicit value</h3>

<p>In the absence of the <em>clocks</em> property, the following clock is defined: <code class="cx">clocks: ["clock"]</code>. This defines a single clock named "clock".</p>

<h2><a class="anchor" id="clocks-assoc"></a>Clocks association</h2>

<p>In most cases, when a network <a href="/documentation/instantiation/">instantiates</a> an entity (task or network), clocks are associated with clocks declared by the entity implicitly. Here is the list of cases where no explicit clocks association is required:</p>
<ul>
	<li>the entity declares no clocks (it is combinational). The number of clocks declared by the network does not matter.</li>
	<li>the entity and the network declares the same number of clocks. Clocks are associated together in the following manner: the first clock of the network is associated with the first clock of the entity, and so on.</li>
	<li>the network has only one clock, and the entity declares multiple clocks. The network's clock is associated multiple times, once with each of the entity's clocks.</li>
</ul>

<h3>Explicit association</h3>

<p>In short, explicit association is necessary each time a network declares more clocks than a synchronous entity it instantiates. This is done with a <code>clocks</code> property specified at the instantiation site, with a value that can be:</p>

<ol>
	<li>an object whose keys are the entity's clock names (in this case the clock names are those declared by the DualPortRAM entity):
<pre><code class="cx">ram = new DualPortRAM({
    clocks: {rd_clock: "clk_recv", wr_clock: "clk_send"},
    /* other properties */
    depth: 8, width: 12
  });</code></pre></li>
	<li>or an array of clock names:
<pre><code class="cx">ram = new DualPortRAM({
    clocks: ["clk_recv", "clk_send"],
    /* other properties */
    depth: 8, width: 12
    });</code></pre></li>
</ol>

<h3>Clock association with inner tasks</h3>

<p>This corner case arises when you use an inner task within a network that declares multiple clocks. Whether this is a good idea or not is debatable, that said how do you associate clocks in this case? Unlike instances of classical entities, an instance of an inner task accepts no arguments and no properties. The trick in this case is that the task's properties are used both for declaration and instantiation, as follows:</p>

<pre><code class="cx">network N {
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
}</code></pre>

<p>In this case, note that the entities that will be generated with use as clock names clk_a and clk_b respectively.</p>

<h2><a class="anchor" id="reset"></a>Reset</h2>

<p>To specify the reset properties of a task or network, use the <em>reset</em> key. The value is either <strong>null</strong> (to specify that the task/network has no reset), or an object with properties <em>type</em>, <em>active</em>, and <em>name</em>:</p>

<pre><code class="cx">properties {
  reset: {type: "synchronous", active: "high", name: "reset_p"}
}</code></pre>

<p>Valid values for <em>type</em> are "asynchronous" and "synchronous". <em>active</em> can be either "high" or "low". <em>name</em> is the reset signal's name.</p>

<h3>Implicit values</h3>

<p>In the absence of a <em>reset</em> key, the following reset is defined:</p>

<pre><code class="cx">properties {
  reset: {type: "asynchronous", active: "low", name: "reset_n"}
}</code></pre>

<p>In the absence of the <em>name</em> key, if the reset is active low, the name is "reset_n", else if it is active high, the name is simply "reset".</p>

<h3>Use with instances</h3>

<p>When passing properties to an instance, the <em>reset</em> key can be associated with the name of a signal that should be used as a reset for the instance (in this example, the "ready" signal will be used as SimpleTask's reset):</p>

<pre><code class="cx">new SimpleTask({reset: "ready"})</code></pre>

<h2><a class="anchor" id="type"></a>Type</h2>

<p>To define a task or network as "combinational", use the <em>type</em> key with the value "combinational":</p>

<pre><code class="cx">properties {
  type: "combinational"
}</code></pre>

<p>The "combinational" value is equivalent to the following properties (in other words no clock and no reset): <code class="cx">{clocks: [], reset: null}</code></p>

<h2><a class="anchor" id="test"></a>Test</h2>

<p>The <em>test</em> property allows you to test a task or a network by specifying stimulus/expected values for ports cycle by cycle. The value of <em>test</em> is an object that associates each port name with an array that contains one value per cycle. For example, to test a Run-Length Encoding task whose ports are <code class="cx">in sync u8 data, out sync value, sync u15 count;</code>, you can write the following properties:</p>

<pre><code class="cx">properties {
  test: {
    data:  [    6, 5,    5, 4,    4,    4, 3,    3,    3,    3, 2 ],
    value: [ null, 6, null, 5, null, null, 4, null, null, null, 3 ],
    count: [ null, 1, null, 2, null, null, 3, null, null, null, 4 ]
  }
}</code></pre>

<p>Values must be of the same type as the port, for example you can use <code class="cx">true</code> and <code class="cx">false</code> for <code class="cx">bool</code> ports. When no value is expected on a port in a cycle, use <code class="cx">null</code>.</p>

<p>This test suite is interpreted as follows:</p>
<ol>
	<li>on cycle 0, write 6 to port <code class="cx">data</code>, and expect no data on ports <code class="cx">value</code> and <code class="cx">count</code>. It is an error if a <code class="cx">sync</code> port is written to when it is not supposed to.</li>
	<li>on cycle 1, write 5 to port <code class="cx">data</code>, and expect 6 to be written to <code class="cx">value</code> and 1 to be written to <code class="cx">count</code>.</li>
</ol>

<p>We use a classic "black box" approach where a testbench instantiates the task, henceforth known as a Design Under Test (DUT). In this approach, the testbench checks values outside of the DUT - <strong>after</strong> the DUT has written them. This occurs one cycle later (as part of good coding style, all output ports are registered in the RTL code generated from Cx), as can be seen on the waveform:</p>

<img src="/images/documentation/properties/properties_test.png" alt="properties_test" class="img-responsive center-block" />

<p>Test values are specified cycle by cycle to help verify synchronization issues. After all a <code class="cx">sync</code> port can wait for data, so technically you could allow the same thing for tests, and never specify <code class="cx">null</code> values. As a matter of fact, that is what we used to do, and the problem was that a test would not test the synchronization behavior. We realized when we switched to truly cycle-accurate tests that some automated unit tests had synchronization issues that had been left undetected before.</p>

<p>The stimulus/expected values are in the format designed and used by <a href="http://www.adacsys.com/">Adacsys</a>.</p>

<h2><a class="anchor" id="impl"></a>Implementation</h2>

<p>The <em>implementation</em> key allows the designer to tell the code generator what to produce from a Cx task or network. This property is meant for entities declared in Cx and implemented in Verilog or VHDL (or any other language or format, really). The value is an object that can define a type, file and dependencies:</p>

<pre><code class="cx">properties {
  implementation: {
    type: "external",
    file: "../../top.v",
    dependencies: ["../../entity_a.v", "../../entity_b.v"]
  }
}</code></pre>

<p>The <em>type</em> property supports two values: "builtin" (reserved for entities defined by the compiler) and "external" (for entities that you define in Verilog or VHDL). The file that implements the task declaring these properties is specified with the <em>file</em> property.</p>

<p>In a future version, "implementation" could be extended to embed code in Cx, for example:</p>

<pre><code class="cx">properties {
  implementation: {
    vhdl: {s0_s1: "a &lt;= a xor b;"},
    verilog: {s0_s1: "a = a ^ b;"}
  }
}</code></pre>
