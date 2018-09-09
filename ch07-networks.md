Networks are entities that may have properties, input and output ports, and that can instantiate other entities, thus allowing a hierarchical modeling style. A design generally has a few top-level networks that instantiate lower-level tasks and networks, which in turn instantiate even lower-level tasks and networks, etc. Instances communicate with each other through input and output ports. Like bundles, tasks can be referred to by their short name if they are imported with an `import` directive. Example of a top-level network that instantiates the `org.crypto.Padding` and `com.synflow.sha.SHA256` tasks, and connects them together:

    package com.btccorp;

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
    }

A network can also declare and instantiate inner tasks. An inner task is an anonymous task defined by a network. An inner task is defined and used by exactly one instance, and exists solely within the network in which it is declared. Inner tasks also have direct access to other instances' ports. All of this makes inner tasks very useful for describing small tasks, as well as tasks that have a high correlation and that are better understood if defined next to one another. The example above could be rewritten as follows:

    package com.btccorp;

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
    }

## Connecting ports

This section describes how to connect ports of different instances together.

Within a network, ports are either readable or writable:

- a readable port is a port that can be read by an instance: input ports of the enclosing network and output ports of other instances of the network.
- a writable port is a port that can be written to by an instance: output ports of the enclosing network and input ports of other instances of the network.

A readable port may have several readers, in which case its data is broadcast to the readers (fan-out), and may be left unconnected if it is unused. In contrast, a writable port must have one and exactly one producer.

### Connect statements

Readable and writable ports can be connected together using `reads` and `writes` declarations. Contrary to `read` and `write`, these declarations specify a continuous assignment from one or more ports to one or more other ports:

    network Expander() {
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
    }

Using `instance.reads` means that `instance` will read from the given readable ports, and `instance.writes` means that `instance` will write to the given writable ports. `this` refers to the surrounding network.

Because writable ports can be connected only once, `reads` iterates over unconnected writable ports of its target (input ports if it is an instance or output ports if it is the enclosing network). The set of unconnected writable ports for each instance is updated each time a writable port of an instance is connected (either with reads or writes). The consequence is that reads and writes are ordered, and changing the order may affect how instances are connected.

### Connecting ports in an inner task

An inner task has access to its own ports (like a normal task) and to readable and writable ports (like connect statements). A simple example is shown below:

    network N {
      in u8 addr; out u12 data;
      ctrl = new task {
        void loop() {
          ram.address.write(addr.read << 1);
          idle(1); // wait for data to be retrieved
          data.write(ram.q.read);
        }
      };

      ram = new SinglePortRAM(8, 12, 0);
    }

The "ctrl" task reads an address directly from the network's `addr` input port, shifts it by one position to the left, and writes it to the ram's `address` input port. The task waits for one cycle, and then reads the data from the ram's `q` output port, and writes it to the network's `data` output port.

In a network that contains both inner tasks and connect statements, first ports of inner tasks are connected, and then connect statements are taken into account.

## Instantiation

Instantiation is the process by which a network materializes an entity (task or network) into an instance that is part of that network. Each instance is its own specific, concrete copy of the original entity, in the sense that any two instances of the same entity do not share anything. Specific means that any parameter of the original entity is replaced by a constant value in the instance.

Examples:

    inst = new Entity();
    ram = new SinglePortRAM({depth: 8, width: 12});
