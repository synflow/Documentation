<p>This section specifies the semantics (evaluation rules and typing rules) of expressions. Two key points of Cx expressions are:</p>
<ul>
  <li><p>Expressions cannot overflow. The type of an expression is guaranteed to be big enough to accomodate its result so that <strong>evaluating an expression never produces an overflow</strong>.</p></li>
  <li><p>Expressions can only have limited side effects: they can read ports (possibly causing the introduction of a new execution rule), but <strong>they cannot modify local or state variables</strong>. Assignment and increment are statements, not expressions.</p></li>
</ul>

<h2><a class="anchor" id="evaluation"></a>Evaluation of expressions</h2>

<p>The evaluation of an expression is done as follows:</p>

<ul>
	<li>Resize each operand as appropriate: for comparison operators, operands are resized to <code>unify(T1, T2)</code>. For arithmetic operators, operands are resized to the size of the result to ensure no overflow can happen. Truncation drops high-order bits regardless of signedness. Extension adds high-order bits: signed numbers are sign-extended, unsigned numbers are zero-extended.</li>
	<li>In case of mixed signedness, unsigned operands are converted to signed numbers. This ensures correct results for operators in which sign matter, most notably multiplication and arithmetic right shift.</li>
	<li>Evaluate the resulting expression.</li>
</ul>

<p>Justification: unsigned operands must be resized <em>first</em> and then converted to signed, otherwise the result would be invalid. The type of the multiplication of a signed variable, say <code class="cx">i7 x = -50</code>, with an unsigned variable, like <code class="cx">u3 y = 5</code> is <code class="cx">i10</code>. Resizing both operands to the result type gives <code class="cx">i10 x = -50</code> and <code class="cx">u10 y = 5</code> (zero extended from 3 to 10 bits), but converting and then resizing gives: <code class="cx">i10 x = -50</code> and <code class="cx">i10 y = -3</code> (sign extended from conversion of y as signed <code class="cx">i3 y = -3</code>), which returns the wrong result (150 instead of -250).</p>

<h2><a class="anchor" id="ternary"></a>Ternary operator</h2>

<p>The ternary operator <code>?:</code> evaluates its first operand (condition), and if it is true, it returns its second operand, else it returns its third operand.</p>

<pre><code class="cx">cond ? e1 : e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>Cond type</th>
        <th>First operand type</th>
        <th>Second operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>bool</td>
        <td>T1 = (signed|unsigned) int&lt;size&gt;</td>
        <td>T2 = (signed|unsigned) int&lt;size&gt;</td>
        <td>unify(T1, T2)</td>
      </tr>
    </tbody>
  </table>
</div>

<p>Cx has a ternary operator <code>?:</code> that can be used to write a conditional expression in a concise manner. The condition must have type <code>bool</code>, and the result of an expression <code>cond ? e1 : e2</code> is <strong>max</strong>(<strong>type</strong>(e1), <strong>type</strong>(e2)). If <strong>min</strong> is not defined for the (e1, e2) tuple, the expression is not valid.</p>

<h2><a class="anchor" id="conditional"></a>Boolean conditional operators</h2>

<pre><code class="cx">e1 || e2</code></pre>

<pre><code class="cx">e1 &amp;&amp; e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>Operand types</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>bool</td>
        <td>bool</td>
      </tr>
      <tr>
        <td>bool</td>
        <td>bool</td>
      </tr>
    </tbody>
  </table>
</div>

<h2><a class="anchor" id="bitwise"></a>Bitwise operators</h2>

<pre><code class="cx">e1 | e2</code></pre>

<pre><code class="cx">e1 ^ e2</code></pre>

<pre><code class="cx">e1 &amp; e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>First operand type</th>
        <th>Second operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>T1</td>
        <td>T2</td>
        <td>unify(T1, T2)</td>
      </tr>
    </tbody>
  </table>
</div>

<p>In the case of the <code>&amp;</code> operator, the size of the result type is set to the minimum possible: size = min(size(T1), size(T2)).</p>

<h2><a class="anchor" id="equality"></a>Equality operators</h2>

<pre><code class="cx">e1 == e2</code></pre>

<pre><code class="cx">e1 != e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>First operand type</th>
        <th>Second operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>T1</td>
        <td>T2</td>
        <td>bool</td>
      </tr>
    </tbody>
  </table>
</div>

<h2><a class="anchor" id="relational"></a>Relational operators</h2>

<pre><code class="cx">e1 &lt; e2</code></pre>

<pre><code class="cx">e1 &lt;= e2</code></pre>

<pre><code class="cx">e1 &gt; e2</code></pre>

<pre><code class="cx">e1 &gt;= e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>First operand type</th>
        <th>Second operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>T1</td>
        <td>T2</td>
        <td>bool</td>
      </tr>
    </tbody>
  </table>
</div>

<h2><a class="anchor" id="shift"></a>Shift operators</h2>

<pre><code class="cx">e1 &lt;&lt; e2</code></pre>

<pre><code class="cx">e1 &gt;&gt; e2</code></pre>

<h2><a class="anchor" id="additive"></a>Additive operators</h2>

<pre><code class="cx">e1 + e2</code></pre>

<pre><code class="cx">e1 - e2</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>First operand type</th>
        <th>Second operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>T1</td>
        <td>T2</td>
        <td>T&lt;max(size(T1), size(T2)) + 1&gt; where T = unify(T1, T2)</td>
      </tr>
    </tbody>
  </table>
</div>

<p>Justification for typing: the additional bit is necessary to avoid overflow, suppose you add <code class="cx">u3 x = 6</code> and <code class="cx">u2 y = 2</code>, the result <code class="cx">8</code> is an <code class="cx">u4</code>.</p>

<h2><a class="anchor" id="multiplicative"></a>Multiplicative operators</h2>

<pre><code class="cx">e1 * e2</code></pre>

<pre><code class="cx">e1 / e2</code></pre>

<pre><code class="cx">e1 % e2</code></pre>

<h2><a class="anchor" id="unary"></a>Unary operators</h2>

<h3>Bitwise complement</h3>

<p>The bitwise complement operator <code>~</code> returns the one's complement of its integer operand, in other words it inverts every bit.</p>

<pre><code class="cx">~e</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>Operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>T = (signed|unsigned) int&lt;size&gt;</td>
        <td>T (same as operand)</td>
      </tr>
    </tbody>
  </table>
</div>

<p>Justification for typing: the type of the result is the same as the type of its input because this is a bitwise operation.</p>

<h3>Logical complement</h3>

<p>The logical complement operator <code>!</code> returns the opposite of its boolean operand.</li>

<pre><code class="cx">!e</code></pre>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>Operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>bool</td>
        <td>bool</td>
      </tr>
    </tbody>
  </table>
</div>

<h3>Unary minus</h3>

<p>The unary minus operator <code>-</code> negates its operand.</li>

<pre><code class="cx">-e</code></pre>

<p>The type of a unary minus expression depends on whether the operand is a constant or not. If the operand is a constant, the expression is evaluated and typed as a <a href="/documentation/expressions#literals">literal</a>.</p>

<div class="table-responsive">
  <table class="table table-condensed">
    <thead>
      <tr>
        <th>Operand type</th>
        <th>Result type</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>operand is a compile-time constant</td>
        <td>type of expression evaluated as a literal</td>
      </tr>
      <tr>
        <td>otherwise, (signed|unsigned) int&lt;size&gt;</td>
        <td>int&lt;size + 1&gt;</td>
      </tr>
    </tbody>
  </table>
</div>

<p>Justification for typing: if operand is unsigned, result is always signed, and the worst case requires an additional bit. For example <code class="cx">-x</code> where <code class="cx">x = 3</code> (highest bound of type <code class="cx">u2</code>), the expression evaluates to <code class="cx">-3</code>, which has type <code class="cx">i3</code>. If operand is signed, it is not possible to know whether the result will be unsigned or signed, and therefore signed is the most conservative assumption. The worst case if operand is signed is the lowest bound of a signed type (<code class="cx">x = -4</code> has type <code class="cx">i3</code>), which when negated becomes <code class="cx">4</code>; which using a signed type translates to <code class="cx">i4</code>.</p>

<p>Consequence in practice: if you care about the signedness of your expression, and you only have unsigned operands and would like an unsigned result, use <code class="cx">a - b * c</code> (unsigned subtract and multiply) rather than <code class="cx">- b * c + a</code> (unary minus makes <code class="cx">-b</code> signed).

<h3><code class="cx">sizeof</code> operator</h3>

<p>The sizeof operator <code>sizeof</code> returns the size as a number of bits needed to represent its compile-time constant operand.</p>

<pre><code class="cx">uint&lt;sizeof(7)&gt; temp; // temp is 3 bits wide
return sizeof(256); // returns 9
</code></pre>

<h2><a class="anchor" id="cast"></a>Cast expression</h2>

<pre><code class="cx">(typ) e</code></pre>

<h2><a class="anchor" id="variable"></a>Variable expression</h2>

<h3>Array access</h3>

<pre><code class="cx">var[index1]...[indexN]</code></pre>

<h3>Port access</h3>

<pre><code class="cx">var.available()</code></pre>

<pre><code class="cx">var.read()</code></pre>

In a task, data is obtained from an input port with <strong>read</strong>, and data is produced to an output port with <strong>write</strong>. Input/output ports (tristates) are forbidden: input ports can only be read, and output ports can only be written. An example of using ports is shown below:
<pre><code class="cx">void loop() {
  u32 temp = op1.read() + op2.read();
  result.write(temp);
}</code></pre>
Note that the <em>temp</em> variable is a local variable, and as such does not create logic. We can describe the same thing in a more concise way, because (1) parentheses after read are optional, and (2) the write method accepts any expression:
<pre><code class="cx">void loop() {
  result.write(op1.read + op2.read);
}</code></pre>
Any given port can be accessed (read or written) at most once per cycle. Reading twice from the same port, or writing twice to the same port, will automatically create a new cycle:
<pre><code class="cx">void loop() {
  result.write(op1.read + op2.read); // cycle 1
  result.write(bigOp.read); // writing to result again: cycle 2
}</code></pre>

<h3>Function call</h3>

<pre><code class="cx">func(e1, ..., eN)</code></pre>

<h2><a class="anchor" id="literals"></a>Literals expression</h2>

<h3>Boolean literals</h3>

<pre><code class="cx">true</code></pre>

<pre><code class="cx">false</code></pre>

<h3>Character literals</h3>

<pre><code class="cx">'a'</code></pre>

<h3>Integer literals</h3>

<p>Integer of any size written in base 2, 10, 16, optionally with number separators to improve clarity:</p>

<pre><code class="cx">-1
42
0b10_10_10
0xC0FFEE
0x794389801297897498324987234098213</code></pre>

<p>The type of a positive number is unsigned, and its size is the number of bits of the number. The type of a negative number is signed, and its size is the number of bits of the number plus one for the sign bit.</p>

<p><em>Technically</em>, an integer literal can only be written in the source as a positive number. A negative number such as <code class="cx">-1</code> is actually the unary minus operator <code class="cx">-</code> applied to the positive integer literal <code class="cx">1</code>. Why? Like in many other languages, this removes an ambiguity in the grammar which otherwise would cause an expression like <code class="cx">a-1</code> to cause syntax errors because it would be interpreted as <code class="cx">a</code> followed by <code class="cx">-1</code>, which doesn't make sense.</p>

<h3>String literals</h3>

<pre><code class="cx">"clock"</code></pre>
