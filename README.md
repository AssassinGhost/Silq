# Silq

Silq is a new high-level programming language for quantum computing with a strong static type system, developed at ETH Zürich.

Overview of Key Concepts
In the following, we provide an overview of the key concepts of Silq.
GitHub

Background
We assume a basic background in quantum computation. Concretely, the reader should be familiar with the following concepts:

Concept	Short Explanation
State of quantum bit (qubit)	
φ
=
γ
0
|
0
⟩
+
γ
1
|
1
⟩
, for 
γ
0
,
γ
1
∈
C
(Computational) basis states	States not in superposition, such as 
|
0
⟩
,
|
1
⟩
Tensor product	Used to represent states consisting of multiple qubits, e.g. 
|
0
⟩
⊗
|
0
⟩
=
|
0
⟩
|
0
⟩
Entangled states	States than cannot be written as a tensor product of two states. For example, 
1
√
2
|
0
⟩
|
0
⟩
+
1
√
2
|
1
⟩
|
1
⟩
Measurement	Measuring state 
φ
=
∑
1
b
=
0
γ
b
|
b
⟩
 yields 
γ
b
|
b
⟩
 with probability 
|
|
γ
b
|
|
2
 (we do not normalize the result).
Lifting classical computations to quantum inputs	Any function 
f
:
A
→
B
 can be made reversible (and thus realized on a quantum computer) by running the (unitary) operation 
U
f
|
a
⟩
|
b
⟩
↦
|
a
⟩
|
b
⊕
f
(
a
)
⟩
 on input 
|
a
⟩
|
0
⟩
.
No-cloning theorem	We cannot physically realize an operation of the form 
φ
↦
φ
⊗
φ
.
Basic Features
We first introduce some basic features of Silq.

Basic Feature: Variable Assignment
The following snippet applies the Hadamard transform H to a qubit x, and assigns the name y to the result.

1
y:=H(x);
Compiling this snippet to a circuit yields:

Variable assignment

Here, we have named the wire into H as x, and the resulting wire as y. A possible state before applying H is 
|
0
⟩
x
, where subscript x indicates that variable x stores 0. Line y:=H(x) would then result in state 
1
√
2
n
(
|
0
⟩
y
+
|
1
⟩
y
)
.

To emphasize that the outcome of H still refers to the same qubit x, we may write:

1
x:=H(x);
This line consumes the original x, but names the output x again.

Basic Feature: Conditionals
The following snippet performs a controlled application of the Hadamard transform.

1
2
3
if x {     // controlled on x,
  y:=H(y); // apply H to y
}
Compiling this snippet to a circuit yields

Simple conditional

Grover's Algorithm in Silq
We illustrate Silq and its benefits on Grover’s algorithm, a widely known quantum search algorithm. For a given function 
f
:
{
−
2
n
−
1
,
.
.
.
,
2
n
−
1
−
1
}
→
{
0
,
1
}
 from n-bit integers to booleans, it finds the input 
w
⋆
 for which 
f
(
w
⋆
)
=
1
. For simplicity, we only discuss the case where 
w
⋆
 is unique, i.e., where 
f
(
v
)
=
0
 for all 
v
≠
w
⋆
.

We first show the implementation, and then walk through its key features.

Grover's algorithm in Silq

Comparison to Circuits
For readers familiar with quantum circuits, we next show a possible implementation of grover in terms of circuits. While it does not follow the standard presentation of Grover’s algorithm, it implements the same behavior. To avoid notational clutter, the circuit’s wires are unnamed.

Grover's algorithm as a circuit

Generic Parameter and Classical Types
The first argument of grover is a generic parameter n, used to parametrize the input length of f. It has type !ℕ, which indicates classical natural numbers of arbitrary size. Here, annotation ! indicates n is classically known, i.e., it is in a basis state (not in superposition), and we can manipulate it classically.

Silq requires generic parameters to be classical, allowing their use in parameterizing types. While Silq supports quantum values of the fixed-size quantum integer types int[n] (containing n-bit integers) and uint[n] (containing n-bit unsigned integers), it disallows quantum values of type ℤ (containing all integers) or ℕ (containing all natural numbers), as the latter require a dynamic-length representation.

Note the function body also requires n to be classical. Otherwise, we could not determine the number of loop iterations without measuring nIterations, which would collapse the state. Further, the parameter f is also annotated as classical. This enables us to use f liberally, i.e., like a normal variable on a classical computer. Concretely, we can call f multiple times as well as silently drop it from the context at the end of grover.

Qfree Functions
The type of f is annotated as qfree which indicates f neither introduces nor destroys superpositions. In particular, if f takes as input a basis state, then it will return a basis state. A good example of a qfree function is the bit-flip gate X, which maps 
∑
1
v
=
0
γ
v
|
v
⟩
 to 
∑
1
v
=
0
γ
v
|
1
−
v
⟩
.

Constant Parameters for Qfree Functions
Note that while X is qfree, it does not preserve its argument, i.e., it consumes its input. In contrast, the argument of f is annotated as const, indicating that f preserves it.

Concretely, consider state 
∑
v
γ
v
φ
v
|
v
⟩
x
 where 
φ
v
 captures the (possibly entangled) remainder of the state. Then, running y := f(x) on this state, where f is qfree, yields 
∑
v
γ
v
φ
v
⊗
|
v
⟩
x
⊗
|
f
(
v
)
⟩
y
. The resulting state preserves the original summand expressions and augments them with an additional value 
|
f
(
v
)
⟩
y
.

Function parameters not annotated as const are not accessible after calling the function, that is, the function consumes them. For example, groverDiff consumes its argument (see top-right box in the above code). Hence, the call in Line 8 consumes cand, transforms it, and writes the result into a new variable with the same name cand. Similarly, measure in Line 10 consumes cand by measuring it.

Input State
The state of the system after Line 1 is 
ψ
1
, where 
f
 and 
n
 denote the value of the variables. Next, Line 2 initializes variable nIterations, yielding state 
ψ
2
.

Superpositions
Lines 3-4 result in state 
ψ
4
, where cand holds the equal superposition of all n-bit integers in 
{
−
2
n
−
1
,
.
.
.
,
2
n
−
1
−
1
}
. To this end, Line 4 updates the i-th bit of cand by applying the Hadamard transform H to it.

Loops
The loop in Line 6 runs nIterations times, which is only possible as the latter is classical. Each loop iteration increases the coefficient of 
|
w
⋆
⟩
, thus increasing the probability of measuring 
w
⋆
 in the end (Line 10). We now discuss the first loop iteration (
k
=
0
). It starts from state 
ψ
6
,
0
 which introduces variable k. For convenience, it also splits the superposition into 
w
⋆
 and all other values.

Conditionals
Line 7 runs phase(π) if f(cand) returns true. As phase(π) flips the sign of coefficients, Line 7 changes the coefficient of 
|
w
⋆
⟩
 from 
1
√
2
n
 to 
−
1
√
2
n
.

In the circuit shown previously, we physically realize this by (i) evaluating f(cand) using 
U
f
, (ii) applying phase(π) using a 
Z
 gate, and (iii) uncomputing f(cand) using 
U
†
f
. Uncomputing f(cand) is critical, as we will discuss shortly.

Grover’s Diffusion Operator
Completing the explanations of our example, Line 8 applies Grover’s diffusion operator to cand. groverDiff increases the coefficient of solution 
w
⋆
, obtaining 
∣
∣
∣
∣
γ
+
w
⋆
∣
∣
∣
∣
>
∣
∣
∣
∣
1
√
2
n
∣
∣
∣
∣
 and decreases the coefficient of non-solutions 
v
≠
w
⋆
, obtaining 
∣
∣
∣
∣
γ
−
v
∣
∣
∣
∣
<
∣
∣
∣
∣
1
√
2
n
∣
∣
∣
∣
. After one loop iteration, this results in state 
ψ
8
,
0
. Repeated iterations of the loop, Lines 6-9, further increase the coefficient of 
w
⋆
, until it is approximately 
1
. Thus, measuring cand in Line 10 returns 
w
⋆
 with high probability.

Uncomputation
While the behavior of Silq’s conditionals and the resulting state 
ψ
7
,
0
 is intuitive, achieving it physically requires uncomputation of f(cand) at the end of Line 7. Concretely, before this uncomputation, the state contains the value of f(cand), stored in a temporary variable  
f(cand)
––––––––– :

ψ
′
7
,
0
=
ψ
2
⊗
(
∑
v
≠
w
⋆
 
1
√
2
n
|
v
⟩
cand
|
0
⟩
f(cand)
––––––––– 
−
1
√
2
n
|
w
⋆
⟩
cand
|
1
⟩
f(cand)
––––––––– 
)
⊗
|
0
⟩
k
.
Now, dropping  
f(cand)
–––––––––  (i.e., removing it from consideration) is physically equivalent to measuring it. Concretely, measuring and dropping  
f(cand)
–––––––––  results in one of the following states, collapsing 
ψ
′
7
,
0
:

ψ
′
7
,
0
,
0
=
ψ
2
⊗
∑
v
≠
w
⋆
 
1
√
2
n
|
v
⟩
cand
⊗
|
0
⟩
k
or
ψ
′
7
,
0
,
1
=
ψ
2
⊗
1
√
2
n
|
w
⋆
⟩
cand
⊗
|
0
⟩
k
.
In this case, as the probability of obtaining 
ψ
′
7
,
0
,
1
 is 
1
2
n
, grover returns the correct result 
w
⋆
 with probability 
1
2
n
. Hence, without uncomputation of  
f(cand)
––––––––– , grover degrades to random guessing.

As a consequence, to prevent an undesired implicit measurement, we must uncompute  
f(cand)
––––––––– , i.e., modify its state to be unentangled with all other qubits in the system. In our case, correct uncomputation yields intuitive semantics that simply forget the state of  
f(cand)
–––––––––  in 
ψ
′
7
,
0
, thus obtaining 
ψ
7
,
0
.

Uncomputation Pitfalls
Various pitfalls can lead to uncomputation issues. Programmers may

forget to use it altogether,
attempt to uncompute variables that are not actually uncomputable
use wrong operations for uncomputation, or
rely on variables no longer available to the uncomputation.
Automatic Uncomputation
In contrast, the type system of Silq ensures that uncomputation of  
f(cand)
–––––––––  is possible, as (i) f is qfree and (ii) cand is not modified in Line 7 and hence we can temporarily interpret it as a const.

In general, Silq supports automatic uncomputation for expressions which are lifted, i.e., consist of qfree functions only depending on const variables. We note that variables may be temporarily viewed as const, as in Line 7 of Grover’s algorithm.

Beyond subexpressions, Silq’s type system also enables uncomputation of variables, which allows overwriting or silently forgetting quantum variables in a wide range of scenarios and without unintuitive side-effects.

We stress that Silq not only guarantees safe uncomputation, but also leads to cleaner code that captures the programmer’s intent.

Preventing Errors
In the following, we demonstrate how the type system of Silq rejects programs with unintuitive semantics, or which are physically unrealizable.

Invalid Measurements
Silq prevents implicit measurements, as illustrated in the following examples.

Implicit Measurement
1
2
3
4
def implicitMeas[n:!ℕ](x:uint[n]){
  y := x % 2;
  return y;
} // parameter 'x' is not consumed
The above function tries to forget variable x, which cannot be uncomputed. As this would induce an implicit measurement, Silq rejects this code.

We note that if parameter x is constant, it need not be consumed:

1
2
3
4
def unconsumedConst[n:!ℕ](const x:uint[n]){
  y := x % 2;
  return y;
} // no error
This is because Silq’s type system ensures that callers of unconsumedConst either use const arguments, or arguments that can be uncomputed.

Conditioned Measurement
1
2
3
4
5
6
def condMeas(const c:𝔹,x:𝔹){
  if c{
    x:= measure(x);
  }
  return x;
} // cannot call function 'measure[𝔹]' in 'mfree' context
Function condMeas (above) tries to apply a measurement, conditioned on quantum variable c. Silq rejects this program, as the then-branch requires a physical action and we cannot determine whether or not we need to carry out the physical action without measuring the condition. However, changing the type of c to !𝔹 would fix this error, as conditional measurement is possible if c is classical:

1
2
3
4
5
6
7
def classCondMeas(const c:!𝔹,x:𝔹){
  if c{
    // `:𝔹` interprets the measurement result as a quantum value
    x:= measure(x):𝔹;
  }
  return x;
} // no error
We note that Silq would also detect this error if measurement was hidden in a passed function, as this function would not be mfree, i.e., free of measurements:

1
2
3
4
5
6
7
def hiddenCondMeas(f:𝔹!→𝔹,const c:𝔹,x:𝔹){
  if c{
    x:= f(x);
    // error: cannot call function 'f' in 'mfree' context
  }
  return x;
}
Reverse Measurement
1
2
3
def revMeas(){
  return reverse(measure);
} // reversed function must be mfree
The expression reverse(f) returns the inverse of function f. As we cannot invert a measurement (this would violate quantum mechanics), reverse only operates on mfree functions. Thus, Silq rejects function revMeas (above).

We note that while reverse is guaranteed to return the inverse of f, using the latter is unsafe if f is not surjective. For example, calling the function returned by reverse(dup) is only safe when both its arguments are equal:

1
2
3
4
5
6
7
8
9
10
11
12
13
def useReverseSafe():𝔹{
  x:=H(0:𝔹);
  y:=dup(x); // 1/√̅2 (|00⟩+|11⟩)
  reverse(dup[𝔹])(x,y); // uncomputes y
  return x; // 1/√̅2 (|0⟩+|1⟩)
}

def useReverseUnsafe():𝔹{
  x:=H(0:𝔹);
  y:=H(0:𝔹); // 1/2 (|00⟩+|01⟩+|10⟩+|11⟩)
  reverse(dup[𝔹])(x,y); // UNDEFINED behavior, since dup cannot produce the above state
  return x;
}
As reverse(dup) is generally useful to provide unsafe uncomputation, we introduce the (unsafe) shorthand forget(e1=e2), which works analogously.

Using Consumed Variable
1
2
3
4
def useConsumed(x:𝔹){
  y := H(x);
  return (x,y); // undefined identifier x
}
Function useConsumed tries to access x after it was consumed by H. As x is no longer available after it was consumed, Silq rejects this code.

We note that if x is constant, Silq allows us to use it even after it was consumed:

1
2
3
4
def duplicateConst(const x:𝔹){
  y := H(x);
  return (x,y); // no error
}
The interpretation in terms of circuits is as follows, where we do not consume x directly, but explicitly duplicate it first.

Duplicate Constant

Thus, Silq allows silent duplication of const variables, but not of non-const, non-classical variables. This is reasonable because all duplicates of constant variables are either consumed (as above), or can be uncomputed.

Impossible Uncomputation
In the following, we show two functions rejected by Silq as automatic uncomputation of subexpressions is impossible or unintuitive.

Not Constant
1
2
3
4
5
def nonConst(y:𝔹){
  if X(y) { // X consumes y
    phase(π);
  } 
} // non-'lifted' quantum expression must be consumed
While function nonConst consumes y in X(y), automatic uncomputation (implemented by reversing X) would re-introduce y. Thus, while we could allow nonConst in principle, Silq disallows it to prevent this confusing re-introduction of y.

However, marking y as const would clarify that y should remain in the context, meaning the resulting program would be accepted. As discussed, if y is const, X consumes a duplicate of y, thus leaving the original y unchanged.

1
2
3
4
5
def signFlipOf0(const y:𝔹){
  if X(y) { // X consumes a copy of y
    phase(π);
  }
} // no error
Not Qfree
1
2
3
4
5
6
def nonQfree(const y:𝔹,z:𝔹){
  if H(y) {
    z := X(z);
  }
  return z;
} // non-'lifted' quantum expression must be consumed
While function nonQfree uses a constant input y, automatic uncomputation does not work in this case, intuitively because H may introduce additional states into the superposition that cannot be uncomputed in the end (this can be seen by a straight-forward computation). To prevent this case, Silq only supports uncomputing qfree expressions.

Of course, umcomputation can always be made explicit by reverse or forget, at the cost of losing safety.
