# COMP311 circuits

Some circuits from COMP311 (Computer Organization) at [UNC](https://unc.edu). Designs are from Professors Brent Munsell and Montek Singh.

Use [Digital](https://github.com/hneemann/Digital), a really great circuit creation & simulation program made by [hneemann](https://github.com/hneemann/), to open and run simulations on `.dig` files.

For convenience, here is the download link for Digital (from the Digital repo above). This is not my software.

[![Download](Download.svg)](https://github.com/hneemann/Digital/releases/latest/download/Digital.zip)

To see my advice on running the program, go to the [wiki](https://github.com/jesse-wei/COMP311-circuits/wiki/How-to-run-Digital). For documentation, go to the [Digital repo](https://github.com/hneemann/Digital) or click on the `Help` button in Digital.

# Definitions

Here are some definitions and explanations of these common circuit designs.

I describe specific cases, such as a DeMUX with 1 select bit and a decoder with 2 select bits. It's easier to understand a specific case than the general case. You can easily generalize to $N$ bits yourself once you understand the specific case.

## Table of contents

* [Adder-subtractor](#adder-subtractor-4-bit)
* [D flip flop](#d-flip-flop-rising-edge)
* [D latch](#d-latch-positive)
* [Decoder](#decoder-s--2)
* [DeMUX](#demux-s--1)
* [Encoder](#encoder-s--2)
* [Full adder](#full-adder)
* [Inverter clock](#inverter-clock)

## Adder-subtractor (4-bit)

This and the [full adder](#full-adder) are my favorite circuit designs 🙂

![Adder-subtractor (4-bit)](/circuits/Adder-subtractor-4-bit.png)

### Inputs

* `A[3:0]`
* `B[3:0]`

### Options

* `Sub`

### Outputs

* `Sum[3:0]`

### Flags

* `FlagC`
* `FlagZ`

### Behavior (basic)

```c
if (Sub)
  Sum[3:0] = A[3:0] - B[3:0]
else
  Sum[3:0] = A[3:0] + B[3:0]
```

* We can use a single circuit to do both addition and subtraction 🤯

### Behavior (detailed)

* See [Full adder](#full-adder) if you're confused about how the full adder components work.
* For this example, let's assume our registers are 4-bit.

#### Addition

* With `Sub=0`, inputting `A = 5 = 0b0101` and `B = 6 = 0b0110` results in `S = 11 = 0b1011`, as you would expect.
  * ![5+6](/circuits/Adder-subtractor-5%2B6.png)
  * Works by ripple-carry addition. See [Full adder](#full-adder) if confused.
* What about `A = 8 = 0b1000` and `B = 8 = 0b1000`?
  * ![8+8](/circuits/Adder-subtractor-8%2B8.png)
  * Note that the most significant bit position produced a carry out.
  * If you think of the carry out as `S[4]`, then the circuit does output the correct result `S = 16 = 0b10000`. But the Sum register is 4-bit, so it stores only `S[3:0] = 0b0000 = 0`.
  * In other words, since 16 exceeds the limit of a 4 bit register, there is overflow. We store this information by setting `FlagC=1`.
  * Also, since all Sum bits are 0 (even though `8+8 != 0`), `FlagZ=1`.

#### Subtraction

* How does an adding circuit do subtraction?
* 2's complement 😈
  * `A - B = A + (-B) = A + ~B + 1`
* When `Sub=1`, 
  * inverse the bits of B by applying a bitwise `XOR` with 1.
    * See the `XOR` explanation in [Full adder](#full-adder) if this doesn't make sense.
  * add the 1 by feeding 1 into $C_{in}$ of the least significant bit position.
* With `Sub=1`, inputting `A = 5 = 0b0101` and `B = 6 = 0b0110` results in `S = -1 = 0b1111`.
  * ![5-6](/circuits/Adder-subtractor-5-6.png)
* What about `A = 8 = 0b1000`

### Truth table

* Unnecessary

## D flip flop (rising edge)

![D flip flop (rising edge)](/circuits/D-flip-flop-rising.png)

### Inputs

* $D$
* `CLK`

### Output

* $Q$

### Behavior

* Q follows D on a rising clock edge, else holds previous value.
* For a falling edge D flip flop, Q follows D on a falling clock edge.

### Truth table

| `CLK` | $D$ | $Q_p$ | $Q$|
| --- | --- | --- | --- |
| Rising edge | 0 | x | 0 |
| Rising edge | 1 | x | 1 |
|Falling edge, active high, or active low | x | 0 | 0 |
|Falling edge, active high, or active low | x | 1 | 1 |

### Misc

* Use two latches, one positive and one negative, to contruct a D flip flop.
* The order of the latches determines whether the D flip flop is enabled by a rising or falling edge.
* This uses the "escapement" strategy.

## D latch (positive)

![D latch (positive)](/circuits/D-latch-positive.png)

### Inputs

* $D$
* $Q_{in}$
* $G$

### Output

* $Q_{out}$

### Behavior

* Q follows D whenever (i.e. active high) G=1, else holds previous value.

### Truth table

<!-- | Inputs |  | | Output |
| --- | --- | --- | --- | -->

| $G$ | $D$ | $Q_{in}$ | $Q_{out}$ |
| --- | --- | --- | --- |
| 0 | x | 0 | 0|
| 0 | x| 1 | 1 |
|1|0|x|0
|1|1|x|1



## Decoder (S = 2)

![Decoder (S = 2)](/circuits/Decoder(s%3D2).png)

### Input

* $S_1S_0$
  * 2-bit input

### Output

* $A_0A_1A_2A_3$
  * 4-bit output

### Behavior

* This circuit "unfolds" or "decompresses" a binary number.
* Each possible output is <span style="color:green">one-hot</span>.
    <!-- * One of $\{A_3, A_2, A_1, A_0\}$ is 1, and all others are 0. -->
* The binary number $S_1S_0$ selects which bit is hot.
* For example, for the input `01`, the output has a 1 only in the `1`$^{\text{th}}$ bit, $A_1$. The rest of the $A$ bits are 0.
* This has the opposite behavior of an encoder.

### Truth table

<!-- |Input||Output||||
|---|---|---|---|---|---| -->

|$S_1$|$S_0$|$A_0$|$A_1$|$A_2$|$A_3$|
|---|---|---|---|---|---|
|0|0|1|0|0|0
|0|1|0|1|0|0
|1|0|0|0|1|0
|1|1|0|0|0|1

* This truth table is different (only in naming) from the one given in Brent's slides.

## DeMUX (S = 1)

![DeMUX (S = 1)](/circuits/DEMUX(s%3D1).png)

### Inputs

* $Y$
* $S$

### Outputs

* $A$
* $B$

### Behavior

* Use S to select which output (A or B) becomes the input Y.

* S acts like a switch determining which input is switched to the output.

### Truth table

<!-- |Inputs|Outputs|
|---|---| -->

|$S$|$Y$|$A$|$B$|
|---|---|---|---|
|0|0|0|0|
|0|1|0|1|
|1|0|0|0|
|1|1|1|0|

## Encoder (S = 2)

![Encoder (S = 2)](/circuits/Encoder(s%3D2).png)

### Input

* $A_3A_2A_1A_0$
  * 4-bit input

### Output

* $S_0S_1$
  * 2-bit output

### Behavior

* This circuit compresses a <span style="color:green">one-hot</span> binary number.
* Given a <span style="color:green">one-hot</span> input, the output is a binary number that tells you which bit was the hot one.
* This has the opposite behavior of a decoder.

### Truth table

<!-- |Input||||||Output|
|---|---|---|---|---|---|---| -->

|$A_3$|$A_2$|$A_1$|$A_0$|$S_0$|$S_1$|
|---|---|---|---|---|---|
|0|0|0|1|0|0|
|0|0|1|0|0|1|
|0|1|0|0|1|0|
|1|0|0|0|1|1|

* All input combinations that aren't <span style="color:green">one-hot</span> (for the above 4-to-2 encoder, there are 12 omitted rows) result in $S_0S_1=\text{xx}$ because the encoder does not care about inputs that aren't one-hot. The output doesn't matter for inputs that aren't one-hot.

## Full adder

This and the [adder-subtractor](#adder-subtractor-4-bit), which uses full adders, are my favorite circuit designs 🙂

![Full adder](/circuits/Full-Adder.png)


### Inputs

* $A$
* $B$
* $C_{in}$
  * Carry in

### Outputs

* $S$
  * Sum
* $C_{out}$
  * Carry out

### Behavior (abstract)

* The following describes how we intuitively know what the outputs of the circuit should be based on how vertical addition of two numbers works, which we're all very comfortable with.
  * Of course, the circuit doesn't have any notion of that. It follows the boolean equations given below.
* Consider adding two binary numbers `A=0b1011` and `B=0b1001` vertically.

$$\begin{align*}
& \quad \quad \quad C_{in/out}\\
&1011 \quad A\\
+&1001 \quad B\\
\hline
1&0100 \quad S
\end{align*}$$

<!-- TODO: add adder subtractor circuit and move this behavior description into it since it's not needed for just one full adder. Or just leave it here and for an adder-subtractor, say see the specification of a full adder -->

* Addition in each column is done by a full adder.
  * Addition in the rightmost column *may* be done by a half adder, though the adder-subtractor circuit requires a full adder for all columns.
  * $C_{out}$ of any column is fed into the $C_{in}$ of the next column (one to the left).
    * Except the leftmost column, which doesn't have a full adder to feed into. Its $C_{out}$ is the C flag of the ALU.
* If 1 or 3 of $\{A, B, C_{in}\}$ are 1, then S is 1.
* Otherwise, if 0 or 2 of them are 1, then S is 0.
* Use a mod operation to represent this: `S = (A + B + Cin) % 2`
* $C_{out}$ is 1 if at least 2 of $\{A, B, C_{in}\}$ are 1.

### Behavior (boolean equations)

This is some magic.

$$S = A \oplus B \oplus C_{in}$$

$$C_{out} = AB + C_{in}(A \oplus B)$$

#### $S$ equation explanations

* The XOR ( $\oplus$ ) operation can be understood as addition modulo 2.

|$A$|$B$|$A \oplus B$|$(A + B) \bmod{2}$|
|---|---|:---:|:---:|
|0|0|0|0|
|0|1|1|1|
|1|0|1|1|
|1|1|0|0|

* Since XOR is addition modulo 2, the $S$ equation matches up with the perhaps more intuitive `S = (A + B + Cin) % 2`
* Also note that $x \oplus 0 = x$ and $x \oplus 1 = \overline{x}$. You can see this from the above truth table, treating $A$ as $x$ and $B$ as $0 \text{ or } 1$, or vice versa.
    * That is, $\oplus 0$ does nothing, whereas $\oplus 1$ negates.
    * This matches with our understanding that adding 0 shouldn't change $S$, whereas adding 1 should toggle $S$.
    * Also, for the [adder-subtractor](#adder-subtractor-4-bit), `XOR` can be used to inverse bits ($\oplus 1$) or do nothing ($\oplus 0$).

#### $C_{out}$ equation explanation

* $C_{out}$ is 1 iff $\left(\text{A and B are both 1}\right) \text{OR} \left(C_{in} \text{ and either one of A or B is 1}\right)$
  * This should be intuitive
* Why use $A \oplus B$ instead of $A + B$ in the equation?
* Although the $C_{out}$ truth table wouldn't change (i.e. this change is functionally correct), notice that the $S$ equation also contains $A \oplus B$.
  * We can use this one XOR gate in both the $S$ and $C_{out}$ parts of the circuit to save a gate! 🤯

### Truth table

* Unnecessary

### Misc

* There is a full adder design that doesn't use `XOR` gates, but that implementation is pretty mid. The equations involving `XOR` are magical.

## Inverter clock

![Inverter clock](/circuits/Inverter-clock.png)

* The simplest implementation of a clock involves simply wiring up, in a loop, an *odd* number of NOT gates.
  * What would happen if there's an even number of NOT gates?
  * *Hint: That circuit design wouldn't cause an error during simulation.*

### Input

* None

### Output

* Connect a wire to any part of the circuit.

### Behavior

* This functions as a clock because each NOT gate (and all gates) have $t_{pd}$, propagation delay.
  * In the real world, $t_{pd}$ depends on how the inverter was manufactured and other factors, so this is not a good implementation.
* Any wire from the circuit oscillates between 0 and 1.

### Truth table

101010101010101010101010101010101010...
