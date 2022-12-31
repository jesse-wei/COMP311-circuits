# COMP311 circuits

Note: The images in this README are transparent with black text, so it's hard to read on dark mode. I recommend using light mode or dark dimmed (Settings > Appearance > Light default or Dark dimmed).

## Table of contents

- [Adder-subtractor](#adder-subtractor-4-bit) ⭐
- [D flip flop](#d-flip-flop-rising-edge)
- [D latch](#d-latch-positive)
- [Decoder](#decoder-s--2)
- [DeMUX](#demux-s--1)
- [Encoder](#encoder-s--2)
- [Full adder](#full-adder) ⭐
- [Half adder](#half-adder)
- [Inverter clock](#inverter-clock)
- [MUX](#mux)

Designs are from slides made by Professors Brent Munsell and Montek Singh.

## Digital

Use [Digital](https://github.com/hneemann/Digital), an awesome circuit creation & simulation program made by [hneemann](https://github.com/hneemann/), to open and run simulations on `.dig` files. All of the circuit designs are in the [circuits](https://github.com/jesse-wei/COMP311-circuits/tree/main/circuits) folder, and I highly recommend that you play around with the designs.

Here's the download link for Digital (from the Digital repo above). This is not my software.

[![Download](Download.svg)](https://github.com/hneemann/Digital/releases/latest/download/Digital.zip)

For advice on running the program, see the [wiki](https://github.com/jesse-wei/COMP311-circuits/wiki/How-to-run-Digital). Here's [documentation](https://github.com/hneemann/Digital/releases/download/v0.29/Doc_English.pdf) from the [Digital repo](https://github.com/hneemann/Digital). You can also access it by clicking the `Help` button in Digital.

# Definitions

Here are some definitions and explanations of these common circuit designs.

I describe specific cases, such as a DeMUX with 1 select bit and a decoder with 2 select bits. It's easier to understand a specific case than the general case. You can easily generalize to $N$ bits yourself once you understand the specific case.

I also recommend understanding the behavior of a circuit by understanding the high-level circuit schematics and high-level behavioral specifications before looking at the truth table. This makes it easier to understand the truth table.

## Adder-subtractor (4-bit)

This and the [full adder](#full-adder) are my favorite circuit designs 🙂

This section requires understanding of the [full adder](#full-adder) component. If you don't know what a full adder does, then read through that section first.

![Adder-subtractor schematic](/img/Adder-subtractor-schematic.png)

![Adder-subtractor (4-bit)](/img/Adder-subtractor-4-bit.png)

### Inputs

- `A[3:0]`
- `B[3:0]`

### Options

- `Sub`

### Outputs

- `Sum[3:0]`

### Flags

- `FlagC`
  - C means "carry"
- `FlagZ`
  - Z means "zero"

### Behavior (pseudocode)

```c
bool FlagC, FlagZ;
int temp[5];                      // Store the carry out bit
int Sum[4];                       // But this is the actual Sum register
if (Sub)
  temp[4:0] = A[3:0] - B[3:0];    // Two's complement
else
  temp[4:0] = A[3:0] + B[3:0];
Sum[3:0] = temp[3:0];
FlagC = (temp[4] == 1);
FlagZ = NOR(Sum[3], Sum[2], Sum[1], Sum[0]);
```

- We can use a single circuit to do both addition and subtraction 🤯
- `temp[4]` doesn't actually exist since we assume our registers are 4-bit. It represents the carry out of the most significant bit position.
- Note that *in the circuit and corresponding pseudocode*, `FlagC`, `FlagZ`, and `Sum[3:0]` are determined in exactly the same way for both addition and subtraction.
    - But while tracing through a SAP program, there is an unsigned comparison table we can use as a shortcut for determining flags ***after `A-B` only***, NOT `A+B`.

### Behavior (details)

- See [Full adder](#full-adder) if you're confused about how the full adder components work.
- For this example, let's assume our registers are 4-bit.

#### Addition

- With `Sub=0`, inputting `A = 5 = 0b0101` and `B = 6 = 0b0110` results in `S = 11 = 0b1011`, as you would expect.
- ![5+6](/img/Adder-subtractor-5plus6.png)
  - Works by ripple-carry addition. See [Full adder](#full-adder) if confused.
- What about `A = B = 8 = 0b1000`? $8+8$ *should* equal $16$, right?
- ![8+8](/img/Adder-subtractor-8plus8.png)
  - Note that the most significant bit position produced a carry out.
  - If you think of the carry out as `S[4]`, then the circuit computes the correct result `S = 16 = 0b10000`. But the Sum register is 4-bit, so the Sum register stores `S[3:0] = 0b0000 = 0` in actuality.
    - With 4 bit registers, `8 + 8 = 0`.
  - Since 16 exceeds the limit of a 4 bit register, there is overflow. We store this information by setting `FlagC=1`.
  - Also, since all Sum bits are 0 (even though $8+8 \neq 0$), `FlagZ=1`.
  - This is the same behavior as the following C code.
    - Rather, C behaves the same as the assembly it is compiled to.

```c
#include <stdint.h>
#include <stdio.h>

int main() {
  uint8_t num = 255;
  printf("8-bit unsigned num is %d\n", num);
  printf("after adding 1, num is %d\n", ++num);
  printf("num is %s 0\n", num ? "not equal to" : "equal to");
  return 0;
}
```

```
8-bit unsigned num is 255
after adding 1, num is 0
num is equal to 0
```

#### Subtraction

- How does an adder circuit do subtraction?
- Two's complement 😈
  - $A - B = A + (-B) = A + \sim B + 1$
- When `Sub=1`,
  - inverse the bits of B by applying a bitwise $\oplus 1$.
    - See the `XOR` explanation in [Full adder](#full-adder) if this doesn't make sense.
  - add the 1 by feeding 1 into $C_{in}$ of the least significant bit position.
    - This is why the LSB can't be a half adder (without a $C_{in}$ input) if we want to do subtraction.
- Otherwise, when `Sub=0`,
  - preserve (don't change) the B bits by applying a bitwise $\oplus 0$.
    - Again, see the `XOR` explanation in [Full adder](#full-adder) if this doesn't make sense.
  - feed 0 into $C_{in}$ of the least significant bit position, doing nothing.
  - That is, when `Sub=0`, normal addition will occur, as described [above](#addition).
- With `Sub=1`, inputting `A = 5 = 0b0101` and `B = 6 = 0b0110` results in `S = -1 = 0b1111`.
- ![5-6](/img/Adder-subtractor-5-6.png)
- What about `A = B = 8 = 0b1000` (i.e. the numbers we're subtracting have the same value)?
- ![8-8](/img/Adder-subtractor-8-8.png)
  - All Sum bits are 0, as we might expect, but why is `FlagC=1`?
    - What's going on under the hood is `0b1000 - 0b1000 = 0b1000 + 0b0111 + 1`.
    - But note that `0b1000 + 0b0111 = 0b1111`.
      - If we add a number to its own inverse, then the result is always all `1`'s.
    - Then when `A == B`, the carry in `1` from 2's complement will always ripple through to the MSB carry out position and set all Sum bits to 0 along the way.
      - This sets `FlagC=1` and `FlagZ=1`.
    - Unlike the addition example where `FlagC` could be seen as `S[4]` (which isn't stored in the 4-bit Sum register!), it doesn't make any sense to think of `FlagC` as `S[4]` here.

### Flags

- Why do we need the flags? Well, if you're in COMP311, the SAP instructions `JC` and `JZ` are a huge part of the class and will be the bane of your existence if you don't understand these concepts.
- But beyond that, programs evaluate boolean expressions by checking flags resulting from ALU subtraction, just as we're about to do here.
- Again, note that in the adder-subtractor, `FlagC` and `FlagZ` are determined the same way for both addition and subtraction since they occur in the same circuit.
  - `FlagC` is 1 if the MSB produced a carry out, 0 otherwise.
  - `FlagZ` $=\overline{S_3+S_2+S_1+S_0}$
    - `NOR` all Sum bits, however many there are.
- But there are some shortcuts that we can use to determine flags set by `A-B`. Just note that these shortcuts ***DO NOT APPLY*** to `A+B`.

#### Subtraction

- It should make intuitive sense that when we want to compare two numbers, we should subtract them. If $A\lt B$, then $A-B\lt 0$. If they're equal, then $A-B=0$. I leave figuring out the third case as an exercise for the reader.
- We have an **unsigned** comparison table that evaluates these conditions using `FlagC` and `FlagZ` **but only after an ALU subtraction `A-B`, NOT ALU addition `A+B`**.
  - Again, note that `FlagC = MSB produced carry` and `FlagZ = big NOR(Sum bits)` from the circuit hold for both addition and subtraction. But the unsigned comparison table holds only for `A-B`.

##### Unsigned comparison table for ALU subtraction

| Condition | Symbol | Equation            |
| :---------: | :------: | :-------------------: |
| `EQ`      | $==$   | $Z$                 |
| `NE`      | $\neq$ | $\sim Z$            |
| `LTU`     | $\lt$  | $\sim C$            |
| `LEU`     | $\leq$ | $\sim C + Z$        |
| `GEU`     | $\geq$ | $C$                 |
| `GTU`     | $\gt$  | $\sim (\sim C + Z)$ |

***This table works for A-B, NOT A+B***

- Evaluate the relationships between `A` and `B` ***before*** `A-B` occurs (since in SAP, `A` will be assigned the result of `A-B`) to determine what the flags will be.
- For example, `5-6` will result in `C=0` since $5<6$. `Z=0`, clearly.
- `8-8` results in `Z=1` because $8==8$, and `C=1` also because $8\geq 8$.
- `Z` is 1 if the two numbers are equal.
- The easiest way to determine `C` is by evaluating $A \geq B$. The easiest equation involving `C` goes with $\geq$.
- Some explanations of the table:
  - It should be obvious that $==$ corresponds to $Z$.
  - $\neq$ is the negation of $==$, so $\neq$ corresponds to $\sim Z$.
  - I'm unsure how to rigorously prove that $\geq$ corresponds to $C$, so let's simply accept that for now. TODO: Prove this statement.
  - Given the above, we know $\lt$ is the negation of $\geq$, so $\lt$ corresponds to $\sim C$.
  - $\leq \iff (\lt \vee ==)$, so $\leq$ corresponds to $\sim C + Z$.
  - Finally, $\gt$ is the negation of $\leq$, so $\gt$ corresponds to $\sim (\sim C + Z)$.

#### Addition

- There is no comparison table for determining `FlagC` and `FlagZ` after `A+B`. As mentioned above, determining the relationship between $A$ and $B$ requires subtraction.
  - Using the comparison table to determine flags after `A+B` is wrong.
- For addition, think about how the flags are determined in the circuit.
- `FlagC` is 1 if there is overflow ***after*** the addition. That's all.
  - If the registers are 4-bit but the result is 5-bit, then `FlagC=1`.
- `FlagZ` is 1 if `big NOR(Sum bits)=1`. That is, all Sum bits are 0.
  - This can occur after overflow.
  - For example, `15+1 = 0b1111 + 0b0001 = 0b10000`. This sets `FlagZ=1`.
  - But `15+2 = 0b1111 + 0b0010 = 0b10001` sets `FlagZ=0`.

### How to evaluate flags easily in a SAP program

|Operation|When to check conditions|`FlagC`|`FlagZ`|
|:---:|:---:|:---:|:---:|
|$+$|After `A+B`|`1` if Carry bit is 1, else `0`|`1` if `big NOR(Sum bits)=1`, else `0`|
|$-$|Before `A-B`|`1` if $A\geq B$, else `0`|`1` if $A==B$, else `0`|

## D flip flop (rising edge)

![D flip flop schematic](/img/D-flip-flop-schematic.png)

![D flip flop (rising edge)](/img/D-flip-flop-rising.png)

### Inputs

- $D$
- `CLK`

### Output

- $Q$

### Behavior

- Q follows D on a rising clock edge, else holds previous value.
- For a falling edge D flip flop, Q follows D on a falling clock edge.

### Behavior (pseudocode)

```verilog
always_ff @(posedge clock) begin
  // Q is assigned D on every rising clock edge
  Q <= D;
end
```

### Truth table

| `CLK`                                    | $D$ | $Q_p$ | $Q$ |
| :----------------------------------------: | :---: | :-----: | :---: |
| Rising edge                              | 0   | x     | 0   |
| Rising edge                              | 1   | x     | 1   |
| Falling edge, active high, or active low | x   | 0     | 0   |
| Falling edge, active high, or active low | x   | 1     | 1   |

### Misc

- Use two latches, one positive and one negative, to contruct a D flip flop.
- ![D flip flop two latches](/img/D-flip-flop-construction-schematic.png)
- The order of the latches determines whether the D flip flop is enabled by a rising or falling edge.
- This uses the "escapement" strategy.
- ![Escapement](/img/D-flip-flop-escapement-schematic.png)

## D latch (positive)

![D latch schematic](/img/D-latch-schematic.png)

![D latch (positive)](/img/D-latch-positive.png)

### Inputs

- $D$
- $Q_{in}$
- $G$

### Output

- $Q_{out}$

### Behavior

- Q follows D whenever (i.e. active high) G=1, else holds previous value.

### Behavior (pseudocode)

```verilog
always @ (G) begin
  // Q is assigned D as long as G is high
  Q <= D;
end
```

### Truth table

<!-- | Inputs |  | | Output |
| --- | --- | --- | --- | -->

| $G$ | $D$ | $Q_{in}$ | $Q_{out}$ |
| --- | --- | -------- | --------- |
| 0   | x   | 0        | 0         |
| 0   | x   | 1        | 1         |
| 1   | 0   | x        | 0         |
| 1   | 1   | x        | 1         |

## Decoder (S = 2)

![Decoder schematic](/img/Decoder-schematic.png)

![Decoder (S = 2)](/img/Decoder(s%3D2).png)

### Input

- $S_1S_0$
  - 2-bit input

### Output

- $A_0A_1A_2A_3$
  - 4-bit one-hot output
    - One-hot means one bit is `1`, and all others are `0`.
    - For example, `0001`, `0010`, `0100`, and `1000` are one-hot, and the other 12 4-bit binary values are not one-hot.

### Behavior

- This circuit "unfolds" or "decompresses" a binary number.
- Each possible output is <span style="color:green">one-hot</span>.
- The binary number $S_1S_0$ selects which bit is hot.
- For example, for the input `01`, the output has a 1 only in the `1`$^{\text{th}}$ bit, $A_1$. The rest of the $A$ bits are 0.
- This has the opposite behavior of an encoder.
- However, looking at the circuit designs, the design of a decoder is most similar to that of a [DeMUX](#demux-s--1).

<!-- ### Behavior (pseudocode)

```python
def decoder(s1, s0):
  S = 0b
  A = [0, 0, 0, 0]
  A[int(S)] = 1
  return A
``` -->

### Truth table

<!-- |Input||Output||||
|---|---|---|---|---|---| -->

| $S_1$ | $S_0$ | $A_0$ | $A_1$ | $A_2$ | $A_3$ |
| :-----: | :-----: | :-----: | :-----: | :-----: | :-----: |
| 0     | 0     | 1     | 0     | 0     | 0     |
| 0     | 1     | 0     | 1     | 0     | 0     |
| 1     | 0     | 0     | 0     | 1     | 0     |
| 1     | 1     | 0     | 0     | 0     | 1     |

- This truth table is different (only in naming) from the one given in Brent's slides.

## DeMUX (S = 1)

![DeMUX schematic](/img/DeMUX-schematic.png)

![DeMUX (S = 1)](/img/DEMUX(s%3D1).png)

### Inputs

- $Y$
- $S$

### Outputs

- $A$
- $B$

### Behavior

- Use S to select which output (A or B) becomes the input Y.
- S acts like a switch determining which input is switched to the output.
- The design of a DeMUX is very similar to that of a [Decoder](#decoder-s--2).

### Behavior (pseudocode)

```c
if (S)
  A = Y;
else
  B = Y;
```

### Truth table

<!-- |Inputs|Outputs|
|---|---| -->

| $S$ | $Y$ | $A$ | $B$ |
| :---: | :---: | :---: | :---: |
| 0   | 0   | 0   | 0   |
| 0   | 1   | 0   | 1   |
| 1   | 0   | 0   | 0   |
| 1   | 1   | 1   | 0   |

## Encoder (S = 2)

![Encoder schematic](/img/Encoder-schematic.png)

![Encoder (S = 2)](/img/Encoder(s%3D2).png)

### Input

- $A_3A_2A_1A_0$
  - 4-bit input
  - The input should be one-hot (this term is defined in the [Decoder](#decoder-s--2) section).
  - If not, then the output is `xx`.

### Output

- $S_0S_1$
  - 2-bit output

### Behavior

- This circuit compresses a <span style="color:green">one-hot</span> input.
- Given a <span style="color:green">one-hot</span> input, the output is a binary number that tells you which bit was the hot one.
    - For example, looking at the third row where $A_3A_2A_1A_0=\texttt{0b0100}$, because the $2^{\text{nd}}$ bit is hot, the output $S_0S_1=\texttt{0b10}$ is $2$.
- This is the opposite behavior of a decoder.

### Truth table

<!-- |Input||||||Output|
|---|---|---|---|---|---|---| -->

| $A_3$ | $A_2$ | $A_1$ | $A_0$ | $S_0$ | $S_1$ |
| :-----: | :-----: | :-----: | :-----: | :-----: | :-----: |
| 0     | 0     | 0     | 1     | 0     | 0     |
| 0     | 0     | 1     | 0     | 0     | 1     |
| 0     | 1     | 0     | 0     | 1     | 0     |
| 1     | 0     | 0     | 0     | 1     | 1     |
|all|other|4-bit|inputs|x|x|

- All inputs that aren't <span style="color:green">one-hot</span> (for the above 4-to-2 encoder with 4-bit inputs, there are 12 possible inputs that aren't one-hot) result in $S_0S_1=\text{xx}$ because the encoder does not care about inputs that aren't one-hot. The output doesn't matter for inputs that aren't one-hot.
- This truth table is different (only in naming) from the one given in Brent's slides.

## Full adder

This and the [adder-subtractor](#adder-subtractor-4-bit), which uses full adders, are my favorite circuit designs 🙂

![Full adder schematic](/img/Full-adder-schematic.png)

![Full adder](/img/Full-Adder.png)

[Full adder (only AND/OR/NOT) from slides](/img/Full-Adder-from-slides.png)

### Inputs

- $A$
- $B$
- $C_{\text{in}}$
  - Carry in

### Outputs

- $S$
  - Sum
- $C_{\text{out}}$
  - Carry out

### Behavior (abstract)

- The following describes how we intuitively know what the outputs of the circuit should be based on how vertical addition of two numbers works, which we're all very comfortable with.
  - Of course, the circuit doesn't have any notion of that. It follows the boolean equations given below.
- Consider adding two binary numbers `A=0b1011` and `B=0b1001` vertically.

$$
\begin{align*}
& \quad \quad \quad C_{in/out}\\
&1011 \quad A\\
+&1001 \quad B\\
\hline
1&0100 \quad S
\end{align*}
$$

- Addition in each column is done by a full adder.
  - Addition in the rightmost column _may_ be done by a half adder, though the adder-subtractor circuit requires a full adder for all columns.
  - $C_{out}$ of any column is fed into the $C_{in}$ of the next column (one to the left).
    - Except the leftmost column, which doesn't have a full adder to feed into. Its $C_{out}$ is the C flag of the ALU.
- If 1 or 3 of $\{A, B, C_{in}\}$ are 1, then S is 1.
- Otherwise, if 0 or 2 of them are 1, then S is 0.
- Use a mod operation to represent this: `S = (A + B + Cin) % 2`
- $C_{out}$ is 1 if at least 2 of $\{A, B, C_{in}\}$ are 1.

### Behavior (boolean equations)

This is some magic.

$$S = A \oplus B \oplus C_{in}$$

$$C_{out} = AB + C_{in}(A \oplus B)$$

#### $S$ equation explanations

- The XOR ( $\oplus$ ) operation can be understood as addition modulo 2.

| $A$ | $B$ | $A \oplus B$ | $(A + B) \bmod{2}$ |
| :---: | :---: | :----------: | :----------------: |
| 0   | 0   |      0       |         0          |
| 0   | 1   |      1       |         1          |
| 1   | 0   |      1       |         1          |
| 1   | 1   |      0       |         0          |

- Since XOR is addition modulo 2, the $S$ equation matches up with the perhaps more intuitive `S = (A + B + Cin) % 2`
- Also note that $x \oplus 0 = x$ and $x \oplus 1 = \overline{x}$. You can see this from the above truth table, treating $A$ as $x$ and $B$ as $0 \text{ or } 1$, or vice versa.
  - That is, $\oplus 0$ does nothing, whereas $\oplus 1$ negates.
  - This matches with our understanding that adding 0 shouldn't change $S$, whereas adding 1 should toggle $S$.
  - So for the [adder-subtractor](#adder-subtractor-4-bit), `XOR` can be used to inverse the bits of `B` ( $b_i \oplus 1$ ) for two's complement subtraction or do nothing ( $b_i \oplus 0$ ) for addition.

#### $C_{out}$ equation explanation

- $C_{out}$ is 1 iff $\left(\text{A and B are both 1}\right) \text{OR} \left(C_{in} \text{ and either one of A or B is 1}\right)$
  - This should be intuitive
- Why use $A \oplus B$ instead of $A + B$ in the equation?
- Although the $C_{out}$ truth table wouldn't change (i.e. this change is functionally correct), notice that the $S$ equation also contains $A \oplus B$.
  - We can use this one XOR gate in both the $S$ and $C_{out}$ parts of the circuit to save a gate! 🤯

### Misc

- There is a full adder design that doesn't use `XOR` gates, but that implementation is pretty mid. The equations involving `XOR` are magical.

## Half Adder

![Half adder schematic](/img/Half-adder-schematic.png)

![Half adder circuit](/img/Half-Adder.png)

[Half adder (only AND/OR/NOT) from slides](/img/Half-Adder-from-slides.png)

### Inputs

* $A$
* $B$

### Outputs

* $S$
* $C_{\text{o}}$

### Behavior (truth table)

| $A$ | $B$ | $S$ | $C_{\text{o}}$ |
| :---: | :---: | :----------: | :----------------: |
| 0   | 0   |      0       |         0          |
| 0   | 1   |      1       |         0          |
| 1   | 0   |      1       |         0          |
| 1   | 1   |      0       |         1          |

### Behavior (boolean equations)

$$S = A \oplus B = A \overline{B} + \overline{A}B$$

$$C_o = AB$$


### Usage

- The only difference between this and the full adder is that there is no $C_\text{in}$.
- The rightmost column of addition never has a carry in, so in a ripple-carry adder design (similar to [adder-subtractor](#adder-subtractor-4-bit)), the rightmost full adder *can* be replaced by a half adder.
- However, doing so prevents the ripple-carry circuit from being used for two's complement subtraction since the $+1$ from two's complement is normally fed into $C_\text{in}$ of the full adder in the LSB position.

## Inverter clock

![Inverter clock](/img/Inverter-clock.png)

*Note*: When running this circuit, you will get an error "Logic seems to oscillate," which is the intended behavior for this circuit lol

- The simplest implementation of a clock involves simply wiring up, in a loop, an _odd_ number of NOT gates.
  - What would happen if there's an even number of NOT gates?
  - _Hint: That circuit design wouldn't cause an error during simulation._

### Input

- None

### Output

- Connect a wire to any part of the circuit.

### Behavior

- This functions as a clock because each NOT gate (and all gates) have $t_{pd}$, propagation delay.
  - In the real world, $t_{pd}$ depends on how the inverter was manufactured and other factors, so this is not a good implementation.
- Any wire from the circuit oscillates between 0 and 1.

### Truth table

101010101010101010101010101010101010...

## MUX

Perhaps the most important and useful circuit in this document.

![MUX schematic](/img/MUX-schematic.png)

![MUX](/img/MUX(s%3D1).png)

### Inputs

- `S`
- `A`
- `B`

### Output

- `Y`

### Behavior


```c
if (S)
  Y = B;
else
  Y = A;
```

A MUX with more select bits would just be a long chain of `else if`'s (of course, the final `else if` can be an `else`) or a `case` statement.

### Truth table

| S   | A   | B   | Y   |
| :---: | :---: | :---: | :---: |
| 0   | 0   | 0   | 0   |
| 0   | 0   | 1   | 0   |
| 0   | 1   | 0   | 1   |
| 0   | 1   | 1   | 1   |
| 1   | 0   | 0   | 0   |
| 1   | 0   | 1   | 1   |
| 1   | 1   | 0   | 0   |
| 1   | 1   | 1   | 1   |

I highly recommend understanding MUX in terms of its behavior, especially the trapezoidal circuit representation. You shouldn't need to use this truth table.
