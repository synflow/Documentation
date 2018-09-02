<p>Networks are entities that may have properties, input and output ports, and that can instantiate other entities, thus allowing a hierarchical modeling style. A design generally has a few top-level networks that instantiate lower-level tasks and networks, which in turn instantiate even lower-level tasks and networks, etc. Instances communicate with each other through input and output ports. Like bundles, tasks can be referred to by their short name if they are imported with an <code class="cx">import</code> directive. Example of a top-level network that instantiates the <code>org.crypto.Padding</code> and <code>com.synflow.sha.SHA256</code> tasks, and connects them together:</p>
<pre class="pre-scrollable"><code class="cx">package com.btccorp;

import com.synflow.sha.SHA256;
import org.crypto.Padding;

network TopBitcoin {
  in u32 msg; out u256 hash;

  pad = new Padding();
  inst1 = new SHA256(); // instantiates a first SHA-256
  inst2 = new SHA256(); // instantiates a second SHA-256

  // connects instances together
  pad.reads(msg); // pad reads from this network's input port msg
  inst1.reads(pad.padded_msg); // i1 reads a padded message
  inst2.reads(inst1.hashed); // i2 reads the hash from i1
  inst2.writes(hash); // and writes to this network's output port hash
}</code></pre>

<p>A network can also declare and instantiate inner tasks. An inner task is an anonymous task defined by a network. An inner task is defined and used by exactly one instance, and exists solely within the network in which it is declared. Inner tasks also have direct access to other instances' ports. All of this makes inner tasks very useful for describing small tasks, as well as tasks that have a high correlation and that are better understood if defined next to one another. The example above could be rewritten as follows:</p>
<pre class="pre-scrollable"><code class="cx">package com.btccorp;

network TopBitcoin {
  import com.synflow.sha.SHA256, org.crypto.Padding;

  in u32 msg; out u256 hash;

  pad = new task { // instead of: new Padding();
    void loop() {
      u32 m = msg.read; // reads from network
      // pad message "m" and put padded message into "padded"
      // ...
      inst1.msg_i.write(padded); // writes padded message to i1
    }
  }; // note the "end of instantiation" semicolon

  inst1 = new SHA256(); // instantiates a first SHA-256
  inst2 = new SHA256(); // instantiates a second SHA-256

  // connects instances together (pad is already connected)
  inst2.reads(inst1.hashed); // i2 reads the hash from i1
  inst2.writes(hash); // and writes to this network's output port hash
}</code></pre>

<h2><a class="anchor" id="connection"></a>Connecting ports</h2>

This section describes how to connect ports of different instances together.

Within a network, ports are either readable or writable:
<ul>
	<li>a readable port is a port that can be read by an instance: input ports of the enclosing network and output ports of other instances of the network.</li>
	<li>a writable port is a port that can be written to by an instance: output ports of the enclosing network and input ports of other instances of the network.</li>
</ul>

A readable port may have several readers, in which case its data is broadcast to the readers (fan-out), and may be left unconnected if it is unused. In contrast, a writable port must have one and exactly one producer.

<h3>Connect statements</h3>

Readable and writable ports can be connected together using <code class="cx">reads</code> and <code class="cx">writes</code> declarations. Contrary to <code class="cx">read</code> and <code class="cx">write</code>, these declarations specify a continuous assignment from one or more ports to one or more other ports:
<pre><code class="cx">network Expander() {
  import com.synflow.crypto.AesConstants.*;

  sync {
    in word key;
    out address_t addr, state_t dout;
  }

  subrot = new SubRot();
  subrot.reads(loadKey.toRotSub);
  subrot.writes(loadKey.fromRotSub); // fromRotSub is the first writable port of LoadKey

  loadKey = new LoadKey();
  loadKey.reads(key); // connects the second writable port of LoadKey
  this.reads(loadKey.w_addr, loadKey.w_data);
}</code></pre>

Using <code class="cx">instance.reads</code> means that <code class="cx">instance</code> will read from the given readable ports, and <code class="cx">instance.writes</code> means that <code class="cx">instance</code> will write to the given writable ports. <code class="cx">this</code> refers to the surrounding network.

Because writable ports can be connected only once, <code class="cx">reads</code> iterates over unconnected writable ports of its target (input ports if it is an instance or output ports if it is the enclosing network). The set of unconnected writable ports for each instance is updated each time a writable port of an instance is connected (either with reads or writes). The consequence is that reads and writes are ordered, and changing the order may affect how instances are connected.

<h3>Connecting ports in an inner task</h3>

An inner task has access to its own ports (like a normal task) and to readable and writable ports (like connect statements). A simple example is shown below:

<pre><code class="cx">network N {
  in u8 addr; out u12 data;
  ctrl = new task {
    void loop() {
      ram.address.write(addr.read &lt;&lt; 1);
      idle(1); // wait for data to be retrieved
      data.write(ram.q.read);
    }
  };

  ram = new SinglePortRAM(8, 12, 0);
}</code></pre>

The "ctrl" task reads an address directly from the network's <code>addr</code> input port, shifts it by one position to the left, and writes it to the ram's <code>address</code> input port. The task waits for one cycle, and then reads the data from the ram's <code>q</code> output port, and writes it to the network's <code>data</code> output port.

In a network that contains both inner tasks and connect statements, first ports of inner tasks are connected, and then connect statements are taken into account.

<h2>Instantiation</h2>

<p>Instantiation is the process by which a network materializes an entity (task or network) into an instance that is part of that network. Each instance is its own specific, concrete copy of the original entity, in the sense that any two instances of the same entity do not share anything. Specific means that any parameter of the original entity is replaced by a constant value in the instance.</p>

<p>Examples:</p>
<pre><code class="cx">inst = new Entity();
ram = new SinglePortRAM({depth: 8, width: 12});</code></pre>
