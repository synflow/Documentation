This section specifies the semantics (evaluation rules and typing rules) of expressions. Two key points of Cx expressions are:

-

Expressions cannot overflow. The type of an expression is guaranteed to be big enough to accomodate its result so that **evaluating an expression never produces an overflow**.

-

Expressions can only have limited side effects: they can read ports (possibly causing the introduction of a new execution rule), but **they cannot modify local or state variables**. Assignment and increment are statements, not expressions.

## Evaluation of expressions

The evaluation of an expression is done as follows:

- Resize each operand as appropriate: for comparison operators, operands are resized to `unify(T1, T2)`. For arithmetic operators, operands are resized to the size of the result to ensure no overflow can happen. Truncation drops high-order bits regardless of signedness. Extension adds high-order bits: signed numbers are sign-extended, unsigned numbers are zero-extended.
- In case of mixed signedness, unsigned operands are converted to signed numbers. This ensures correct results for operators in which sign matter, most notably multiplication and arithmetic right shift.
- Evaluate the resulting expression.

Justification: unsigned operands must be resized _first_ and then converted to signed, otherwise the result would be invalid. The type of the multiplication of a signed variable, say `i7 x = -50`, with an unsigned variable, like `u3 y = 5` is `i10`. Resizing both operands to the result type gives `i10 x = -50` and `u10 y = 5` (zero extended from 3 to 10 bits), but converting and then resizing gives: `i10 x = -50` and `i10 y = -3` (sign extended from conversion of y as signed `i3 y = -3`), which returns the wrong result (150 instead of -250).

## Ternary operator

The ternary operator `?:` evaluates its first operand (condition), and if it is true, it returns its second operand, else it returns its third operand.

    cond ? e1 : e2

Cond typeFirst operand typeSecond operand typeResult typeboolT1 = (signed|unsigned) int<size>T2 = (signed|unsigned) int<size>unify(T1, T2)

Cx has a ternary operator `?:` that can be used to write a conditional expression in a concise manner. The condition must have type `bool`, and the result of an expression `cond ? e1 : e2` is **max**(**type**(e1), **type**(e2)). If **min** is not defined for the (e1, e2) tuple, the expression is not valid.

## Boolean conditional operators

    e1 || e2

    e1 && e2

Operand typesResult typeboolboolboolbool

## Bitwise operators

    e1 | e2

    e1 ^ e2

    e1 & e2

First operand typeSecond operand typeResult typeT1T2unify(T1, T2)

In the case of the `&` operator, the size of the result type is set to the minimum possible: size = min(size(T1), size(T2)).

## Equality operators

    e1 == e2

    e1 != e2

First operand typeSecond operand typeResult typeT1T2bool

## Relational operators

    e1 < e2

    e1 <= e2

    e1 > e2

    e1 >= e2

First operand typeSecond operand typeResult typeT1T2bool

## Shift operators

    e1 << e2

    e1 >> e2

## Additive operators

    e1 + e2

    e1 - e2

First operand typeSecond operand typeResult typeT1T2T<max(size(T1), size(T2)) + 1> where T = unify(T1, T2)

Justification for typing: the additional bit is necessary to avoid overflow, suppose you add `u3 x = 6` and `u2 y = 2`, the result `8` is an `u4`.

## Multiplicative operators

    e1 * e2

    e1 / e2

    e1 % e2

## Unary operators

### Bitwise complement

The bitwise complement operator `~` returns the one's complement of its integer operand, in other words it inverts every bit.

    ~e

Operand typeResult typeT = (signed|unsigned) int<size>T (same as operand)

Justification for typing: the type of the result is the same as the type of its input because this is a bitwise operation.

### Logical complement

The logical complement operator `!` returns the opposite of its boolean operand.
!e

Operand typeResult typeboolbool

### Unary minus

The unary minus operator `-` negates its operand.
-e

The type of a unary minus expression depends on whether the operand is a constant or not. If the operand is a constant, the expression is evaluated and typed as a [literal](/documentation/expressions#literals).

Operand typeResult typeoperand is a compile-time constanttype of expression evaluated as a literalotherwise, (signed|unsigned) int<size>int<size + 1>

Justification for typing: if operand is unsigned, result is always signed, and the worst case requires an additional bit. For example `-x` where `x = 3` (highest bound of type `u2`), the expression evaluates to `-3`, which has type `i3`. If operand is signed, it is not possible to know whether the result will be unsigned or signed, and therefore signed is the most conservative assumption. The worst case if operand is signed is the lowest bound of a signed type (`x = -4` has type `i3`), which when negated becomes `4`; which using a signed type translates to `i4`.

Consequence in practice: if you care about the signedness of your expression, and you only have unsigned operands and would like an unsigned result, use `a - b * c` (unsigned subtract and multiply) rather than `- b * c + a` (unary minus makes `-b` signed).

### `sizeof` operator

The sizeof operator `sizeof` returns the size as a number of bits needed to represent its compile-time constant operand.

    uint<sizeof(7)> temp; // temp is 3 bits wide
    return sizeof(256); // returns 9

## Cast expression

    (typ) e

## Variable expression

### Array access

    var[index1]...[indexN]

### Port access

    var.available()

    var.read()

In a task, data is obtained from an input port with **read**, and data is produced to an output port with **write**. Input/output ports (tristates) are forbidden: input ports can only be read, and output ports can only be written. An example of using ports is shown below:

    void loop() {
      u32 temp = op1.read() + op2.read();
      result.write(temp);
    }

Note that the _temp_ variable is a local variable, and as such does not create logic. We can describe the same thing in a more concise way, because (1) parentheses after read are optional, and (2) the write method accepts any expression:

    void loop() {
      result.write(op1.read + op2.read);
    }

Any given port can be accessed (read or written) at most once per cycle. Reading twice from the same port, or writing twice to the same port, will automatically create a new cycle:

    void loop() {
      result.write(op1.read + op2.read); // cycle 1
      result.write(bigOp.read); // writing to result again: cycle 2
    }

### Function call

    func(e1, ..., eN)

## Literals expression

### Boolean literals

    true

    false

### Character literals

    'a'

### Integer literals

Integer of any size written in base 2, 10, 16, optionally with number separators to improve clarity:

    -1
    42
    0b10_10_10
    0xC0FFEE
    0x794389801297897498324987234098213

The type of a positive number is unsigned, and its size is the number of bits of the number. The type of a negative number is signed, and its size is the number of bits of the number plus one for the sign bit.

_Technically_, an integer literal can only be written in the source as a positive number. A negative number such as `-1` is actually the unary minus operator `-` applied to the positive integer literal `1`. Why? Like in many other languages, this removes an ambiguity in the grammar which otherwise would cause an expression like `a-1` to cause syntax errors because it would be interpreted as `a` followed by `-1`, which doesn't make sense.

### String literals

    "clock"


---
```
Copyright 2014-2020 Synflow SAS

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
