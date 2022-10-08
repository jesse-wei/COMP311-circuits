# COMP311 circuits

Some circuits from COMP311 (Computer Organization) at [UNC](https://unc.edu). Designs are from Professors Brent Munsell and Montek Singh.

Use [Digital](https://github.com/hneemann/Digital), a really great circuit creation & simulation program made by [hneemann](https://github.com/hneemann/), to open and run simulations on `.dig` files.

For convenience, here is the download link for Digital (from the Digital repo above). This is not my software.

[![Download](distribution/Download.svg)](https://github.com/hneemann/Digital/releases/latest/download/Digital.zip)

To see how to run the program and other documentation, click the link to Digital's repo above.

# Definitions

Here are some definitions and explanations of these circuits.

I describe specific cases, such as a DeMUX with 1 select bit and a decoder with 2 select bits. I believe it's easier to understand a specific case than the general case. You can easily generalize to $N$ bits yourself once you understand the specific case.

## Table of contents

* [D flip flop](#d-flip-flop-rising-edge)
* [D latch](#d-latch-positive)
* [Decoder](#decoder-s--2)
* [DeMUX](#demux-s--1)
* [Encoder](#encoder-s--2)
* [Full Adder](#full-adder)



## D flip flop (rising edge)

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

This and the adder-subtractor, which uses full adders, are my favorite circuit designs 🙂

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
  * $0 + 0 = 0 \equiv 0 \pmod{2}$
  * $0 + 1 = 1 \equiv 1 \pmod{2}$
  * $1 + 0 = 1 \equiv 1 \pmod{2}$
  * $1 + 1 = 2 \equiv 0 \pmod{2}$
  * Note that the rightmost numbers are the same as those in the truth table of XOR.
  * Since XOR is addition modulo 2, the $S$ equation matches up with the perhaps more intuitive `S = (A + B + Cin) % 2`
* Also note that $x \oplus 0 = x$ and $x \oplus 1 = \overline{x}$.
* This matches up with our understanding that adding 0 shouldn't change $S$, whereas adding 1 should toggle $S$.

#### $C_{out}$ equation explanation

* $C_{out}$ is 1 iff $\left(\text{A and B are both 1}\right) \text{OR} \left(C_{in} \text{ and either one of A or B is 1}\right)$
  * This should be intuitive
* Why use $A \oplus B$ instead of $A + B$ in the equation?
* Although the $C_{out}$ truth table wouldn't change (i.e. this change is functionally correct), notice that the $S$ equation also contains $A \oplus B$.
  * We can use this one XOR gate in both the $S$ and $C_{out}$ parts of the circuit to save a gate! 🤯

### Truth table

* Unnecessary 🙂
