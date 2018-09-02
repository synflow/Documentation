<p>A bundle allows tasks and networks to be <em>configurable</em>, which means their behavior can be influenced by overriding parts of the bundle at instantiation. A bundle is a stateless entity that defines types, constants, and functions that are common to several tasks and networks.</p>

<pre class="pre-scrollable"><code class="cx">package com.synflow.sha256;

/**
 * Description of the SHACommon bundle
 */
bundle SHACommon {
  // type definition
  typedef u6 addr_t;

  // constant definition
  u9 HASH_SIZE = 256;

  u32 Ch(u32 x, u32 y, u32 z) {
    return (x &amp; y) ^ (~x &amp; z);
  }

  u32 Maj(u32 x, u32 y, u32 z) {
    return (x &amp; y) ^ (x &amp; z) ^ (y &amp; z);
  }
}</code></pre>

<p>The constants, types, functions of a bundle can be referenced in other entities in three different ways. In the "SHACommon" bundle above, the "addr" type, the "HASH_SIZE" constant, and the "Ch" and "Maj" functions can be referenced as follows:</p>
<ol>
	<li>Typical, shortest way: import everything from the bundle, and reference them by their short name, like <code>addr</code> or <code>Ch</code>.</li>
	<li>Reasonably short way: import the bundle, and reference objects by their qualified name <code>SHACommon.addr</code> or <code>SHACommon.Ch</code>.</li>
	<li>Verbose way: Reference objects by their full name: <code>com.synflow.sha256.SHACommon.addr</code> or <code>com.synflow.sha256.SHACommon</code></li>
</ol>
