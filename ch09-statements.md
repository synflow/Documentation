This section specifies the semantics of statements in Cx.

## Assert

Asserts that an expression is true. Useful for verification.

## Assignment

Assign an expression to a variable or an array.

    x = 15;
    y[3] = (u4) z;

The assignment statement may be used without a target variable just to force the evaluation of an expression. This is useful to wait for an event by reading from a `sync` port, but without using the actual data. In this case, you can use an empty assignment statement as follows:

    in sync u8 msg;
    void loop() {
      // ...
      // wait for data to be available on msg, and discard it
      msg.read();
      // ...
    }

## Fence

Placed between statements, forces a new cycle to start.

    print("in cycle 1");
    fence;
    print("in cycle 2");

## Idle

Do nothing for a given number of cycles.

    print("in cycle 1");
    idle(3);
    print("in cycle 5");

## Print

Prints a message/values. Useful for verification.

    print("something");

## Return

Returns a value. In the current version, only valid at the end of a constant function.

    return factor << 1;

## Write

Write to a port.

    answer.write(42);

## If

Conditional "if" statement.

    if (a > 0) {
      print("a greater than zero");
    } else if (a < 0) {
      print("a smaller than zero");
    } else {
      print("a equals zero");
    }

Both if and for statements have special semantics of reads in conditional called peeking. This means that you may read the data on a port in the condition of the if/loop and still have access to that data in the branch that is chosen:

    if (a.read() < b.read()) { // peeks a and b
      min.write(a.read()); // does not cause a new cycle,
    }
    // we're still in the same cycle
    // after the if, both a and b have been read (b implicitly in this case)

    b.read(); // causes the creation of a new cycle

## For

For loop with initial assignment, loop condition, post-iteration assignment.

    for (u4 i = 0; i < 5; i++) {
      print("one iteration"); // this message is printed 5 times in the current cycle
    }

    for (; cond.read(); count++) {
      print("cond is true"); // this message is printed one time per cycle until cond becomes false
    }

A `for` loop can be executed in the current cycle if the following conditions are respected:

- The loop uses a local variable used solely for iteration.
- The loop has compile-time constant bounds.
- The body does not read or write ports, or contains fence or idle instructions.

Otherwise, the loop takes at least one cycle per iteration.

## While

While loop with loop condition.

    while (keepGoing.read()) {
      count++;
    }
