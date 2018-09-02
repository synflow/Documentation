<p>This section specifies the semantics of statements in Cx.</p>

<h2><a class="anchor" id="assert"></a>Assert</h2>

<p>Asserts that an expression is true. Useful for verification.</p>

<h2><a class="anchor" id="assignment"></a>Assignment</h2>

<p>Assign an expression to a variable or an array.</p>

<pre><code class="cx">x = 15;
y[3] = (u4) z;</code></pre>

<p>The assignment statement may be used without a target variable just to force the evaluation of an expression. This is useful to wait for an event by reading from a <code class="cx">sync</code> port, but without using the actual data. In this case, you can use an empty assignment statement as follows:</p>

<pre><code class="cx">in sync u8 msg;
void loop() {
  // ...
  // wait for data to be available on msg, and discard it
  msg.read();
  // ...
}
</code></pre>

<h2><a class="anchor" id="fence"></a>Fence</h2>

<p>Placed between statements, forces a new cycle to start.</p>

<pre><code class="cx">print("in cycle 1");
fence;
print("in cycle 2");</code></pre>

<h2><a class="anchor" id="idle"></a>Idle</h2>

<p>Do nothing for a given number of cycles.</p>

<pre><code class="cx">print("in cycle 1");
idle(3);
print("in cycle 5");</code></pre>

<h2><a class="anchor" id="print"></a>Print</h2>

<p>Prints a message/values. Useful for verification.</p>

<pre><code class="cx">print("something");</code></pre>

<h2><a class="anchor" id="return"></a>Return</h2>

<p>Returns a value. In the current version, only valid at the end of a constant function.</p>

<pre><code class="cx">return factor << 1;</code></pre>

<h2><a class="anchor" id="write"></a>Write</h2>

<p>Write to a port.</p>

<pre><code class="cx">answer.write(42);</code></pre>

<h2><a class="anchor" id="if"></a>If</h2>

<p>Conditional "if" statement.</p>

<pre><code class="cx">if (a &gt; 0) {
  print("a greater than zero");
} else if (a &lt; 0) {
  print("a smaller than zero");
} else {
  print("a equals zero");
}</code></pre>

<p>Both if and for statements have special semantics of reads in conditional called peeking. This means that you may read the data on a port in the condition of the if/loop and still have access to that data in the branch that is chosen:</p>

<pre><code class="cx">if (a.read() &lt; b.read()) { // peeks a and b
  min.write(a.read()); // does not cause a new cycle, 
}
// we're still in the same cycle
// after the if, both a and b have been read (b implicitly in this case)

b.read(); // causes the creation of a new cycle
</code></pre>

<h2><a class="anchor" id="for"></a>For</h2>

<p>For loop with initial assignment, loop condition, post-iteration assignment.</p>

<pre><code class="cx">for (u4 i = 0; i &lt; 5; i++) {
  print("one iteration"); // this message is printed 5 times in the current cycle
}

for (; cond.read(); count++) {
  print("cond is true"); // this message is printed one time per cycle until cond becomes false
}</code></pre>

<p>A <code class="cx">for</code> loop can be executed in the current cycle if the following conditions are respected:</p>
<ul>
  <li>The loop uses a local variable used solely for iteration.</li>
  <li>The loop has compile-time constant bounds.</li>
  <li>The body does not read or write ports, or contains fence or idle instructions.</li>
</ul>

<p>Otherwise, the loop takes at least one cycle per iteration.</p>

<h2><a class="anchor" id="while"></a>While</h2>

<p>While loop with loop condition.</p>

<pre><code class="cx">while (keepGoing.read()) {
  count++;
}</code></pre>