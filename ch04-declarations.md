<h2><a class="anchor" id="variables"></a>Variables</h2>

local and state variables.
initialization

<p>An array can be given an initial value as follows: <code class="cx">u32 triple = {1, 2, 3};</code>. Also, Cx also supports initialization of char arrays from strings: <code class="cx">const char message[12] = "Hello world!";</code>.</p>

<h2><a class="anchor" id="functions"></a>Functions</h2>

<p>A function declaration is composed of the following elements:</p>
<ul>
	<li>optionally, the <code class="cx">const</code> keyword to declare a constant function,</li>
	<li>a <a href="/documentation/type-system/">type</a> or <code class="cx">void</code>,</li>
	<li>a name,</li>
	<li>parameters,</li>
	<li>a body of statements that do computations and may return a result.</li>
</ul>

<p>A function may have parameters and define local variables, and contains a sequence of statements such as reading from ports, updating state variables, calling other functions, and writing to output ports.</p>

<p>Statements that modify the environment outside of the function are said to have side effects. Examples of side effects are accessing a port (reading, writing, testing availability), writing state variables, calling another function that has side effects. Side-effect free statements are any statement that does not have side effects.</p>

<p>There are two kinds of functions:</p>
<ol>
	<li>constant functions are functions without side-effects, they can only read state variables, and cannot access ports.</li>
	<li>functions with side effects can modify state variables and read/write ports.</li>
</ol>

<h3>Constant functions</h3>

<p>Functions declared by a <a href="/documentation/bundles">bundle</a> are implicitly constant (there is no need to specify a function declared in a bundle as <code class="cx">const</code>). This is by design, since a bundle does not have ports or state variables, it can only declare constant functions.</p>

<p>In tasks, to declare a constant function, start the declaration with the <code class="cx">const</code> keyword:</p>
<pre><code class="cx">const u9 increment(u8 x) {
  return x + 1;
}</code></pre>

<h3>Functions with side effects</h3>

<p>Functions with side effects can only be declared in tasks. As of the current version of Cx, it is not possible to declare a function that has side effects <em>and</em> returns a result (with a type other than <code class="cx">void</code>). As a result, functions that return a result must be declared constant in tasks. We hope to lift this restriction in a future version of the language (see <a href="/roadmap">roadmap</a>).</p>

<h2><a class="anchor" id="ports"></a>Ports</h2>

<p>A port is an interface defined by a network or a task to communicate with other networks/tasks. The definition of a port includes a direction (input or output), a <a title="Synchronization" href="ports/#synchronization">synchronization</a> type, a <a title="Type" href="/documentation/type-system/">type</a>, and a name. A port declaration begins with a direction (<code class="cx">in</code> or <code class="cx">out</code>) and may declare one or more ports. If a port has the same type as the port that precedes it, its type can be omitted. For instance:</p>

<pre><code class="cx">in u16 a, b /* b has type u16 like a */, u48 c;
out bool x;</code></pre>

<h3>Synchronization</h3>

<p>A port may have additional synchronization semantics as specified with the <code class="cx">sync</code> flag. The <code class="cx">sync</code> flag may occur before the port's type (if any) and only applies to the current port, not to following ports:</p>

<pre><code class="cx">in u16 a, sync b, u48 c;
out sync bool x;</code></pre>

<p>If you have multiple <code class="cx">sync</code> ports, you can use a multiple port declaration. A multiple port declaration begins with the synchronization type, and contains basic port declarations inside curly braces. All the ports contained in the declaration will be synchronized. The example above can be rewritten as follows:</p>

<pre><code class="cx">in u16 a, u48 c;
sync {
  in u16 b; out bool x;
}</code></pre>

<h4>Simple synchronization</h4>

<p>The <code class="cx">sync</code> flag adds a synchronization signal that becomes true for one cycle each time data is written to the port. When a task reads from one or more synchronized ports, the <code class="cx">read</code> expression on these ports becomes blocking. This makes synchronized ports useful to make tasks sensitive to data presence, for instance to describe a pipeline. When reading from several synchronized ports, a task will only resume its execution if data is present <strong>simultaneously</strong> on all ports it is waiting on. The following code will only write a value on <code>o</code> if data is present on both <code>a</code> and <code>b</code> at the same time:</p>

<pre><code class="cx">in sync u3 a, b; out u6 o;
void loop() {
  o.write(a.read * b.read);
}
</code></pre>

<h4>Ready synchronization</h4>

<p>A <code class="cx">sync ready</code> port allows two tasks (producer and consumer) to communicate together at possibly different rates, automatically synchronizing each task as needed. A ready port handles management of signals that indicate when the consumer is ready and when the producer has valid data. Use a ready port in the following cases:</p>
<ul>
	<li>have two tasks communicate at different rates,</li>
	<li>implement back-pressure in pipelines, for example you need to pause transmission during an inter-frame delay in Ethernet,</li>
	<li>interact with a FIFO,</li>
	<li>interact with the outside world for compatibility with existing components.</li>
</ul>

<p>Note: ready ports are experimental, we're still figuring out some implementation details in code generation. See <a href="https://forum.synflow.com/t/thoughts-on-ready-ports-for-implementing-an-intel-8088/219">this post</a> for discussion.</p>
