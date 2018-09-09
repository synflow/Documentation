# Summary

- [Foreword](README.md)
- [Introduction](ch01-introduction.md)

  - [How hardware works](ch01-introduction.md#how-hardware-works)
  - [The Cx programming language](ch01-introduction.md#the-cx-programming-language)
  - [Programming model](ch01-introduction.md#programming-model)
  - [Mapping Cx to digital hardware](ch01-introduction.md#mapping-cx-to-digital-hardware)

- [Meet the Synflow IDE](ch02-synflow.md)

  - [Project Structure](ch02-synflow.md#project-structure)
  - [User Interface](ch02-synflow.md#user-interface)
  - [Build System](ch02-synflow.md#build-system)
  - [Debug Tool](ch02-synflow.md#debug-tool)

- [Concepts](ch03-concepts.md)

  - [Organization](ch03-concepts.md#organization)
  - [Type system](ch03-concepts.md#type-system)
  - [Execution model](ch03-concepts.md#execution-model)

- [Declarations](ch04-declarations.md)

  - [Variables](ch04-declarations.md#variables)
  - [Functions](ch04-declarations.md#functions)
  - [Ports](ch04-declarations.md#ports)

- [Bundles](ch05-bundles.md)
- [Tasks](ch06-tasks.md)

  - [External tasks](ch06-tasks.md#external-tasks)
  - [Cycle-accurate modeling](ch06-tasks.md#cycle-accurate-modeling)
  - [Synchronization with ports](ch06-tasks.md#synchronization-with-ports)

- [Networks](ch07-networks.md)

  - [Connecting ports](ch07-networks.md#connecting-ports)
  - [Instantiation](ch07-networks.md#instantiation)

- [Expressions](ch08-expressions.md)

  - [Evaluation of expressions](ch08-expressions.md#evaluation-of-expressions)
  - [Ternary operator](ch08-expressions.md#ternary-operator)
  - [Boolean conditional operators](ch08-expressions.md#boolean-conditional-operators)
  - [Bitwise operators](ch08-expressions.md#bitwise-operators)
  - [Equality operators](ch08-expressions.md#equality-operators)
  - [Relational operators](ch08-expressions.md#relational-operators)
  - [Shift operators](ch08-expressions.md#shift-operators)
  - [Additive operators](ch08-expressions.md#additive-operators)
  - [Multiplicative operators](ch08-expressions.md#multiplicative-operators)
  - [Unary operators](ch08-expressions.md#unary-operators)
  - [Cast expression](ch08-expressions.md#cast-expression)
  - [Variable expression](ch08-expressions.md#variable-expression)
  - [Literals expression](ch08-expressions.md#literals-expression)

- [Statements](ch09-statements.md)

  - [Assert](ch09-statements.md#assert)
  - [Assignment](ch09-statements.md#assignment)
  - [Fence](ch09-statements.md#fence)
  - [Idle](ch09-statements.md#idle)
  - [Print](ch09-statements.md#print)
  - [Return](ch09-statements.md#return)
  - [Write](ch09-statements.md#write)
  - [If](ch09-statements.md#if)
  - [For](ch09-statements.md#for)
  - [While](ch09-statements.md#while)

* [Properties](ch10-properties.md)

  - [Definition of properties](ch10-properties.md#definition-of-properties)
  - [Clocks declaration](ch10-properties.md#clocks-declaration)
  - [Clocks association](ch10-properties.md#clocks-association)
  - [Reset](ch10-properties.md#reset)
  - [Type](ch10-properties.md#type)
  - [Test](ch10-properties.md#test)
  - [Implementation](ch10-properties.md#implementation)

* [Standard library](ch11-stdlib.md)

  - [Synchronous FIFO](ch11-stdlib.md#synchronous-fifo)
  - [Synchronizer Flip-Flop](ch11-stdlib.md#synchronizer-flip-flop)
  - [Synchronizer Mux](ch11-stdlib.md#synchronizer-mux)
  - [Single-port RAM](ch11-stdlib.md#single-port-ram)
  - [Dual-port RAM](ch11-stdlib.md#dual-port-ram)
  - [Pseudo Dual-port RAM](ch11-stdlib.md#pseudo-dual-port-ram)
