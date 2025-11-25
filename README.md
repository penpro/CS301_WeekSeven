# CS301 - Chapter 5: Theory of Computing (Sections 5.3-5.5)

**Focus:**  
- 5.3 Universality  
- 5.4 Computability  
- 5.5 Intractability and Complexity  

I am using this as a personal "big picture" guide: what all reasonable computers can do, what no computer can ever do, and what is technically solvable but probably not feasible.

---

## 1. Universality (5.3)

### 1.1 Models of computation

A **model of computation** is a mathematically precise way to talk about algorithms. Examples:

- Turing machines  
- Lambda calculus  
- Random access machines  
- Real programming languages like Java, C, Python  

These all describe step by step procedures for transforming input into output.

Key idea: once a model is powerful enough to simulate a Turing machine, it is **Turing complete**. At that point it can compute exactly the same set of functions as any other Turing complete system. Differences are in efficiency, not in what is possible.

---

### 1.2 Turing machines vs real computers

A **Turing machine (TM)** has:

- A finite set of states  
- A read/write head that moves along an infinite tape  
- A transition function that says "given current state and tape symbol, what do I write, which way do I move, and what state do I go to next"

Think of this as a very stripped down CPU plus RAM, with the transition function acting like microcode.

Real computers and languages look more complicated, but they can all be simulated by a Turing machine and can simulate a Turing machine in return.

---

### 1.3 Universal Turing machine (UTM)

A "normal" Turing machine is like a single fixed program: it computes one specific function. A **Universal Turing Machine** is a programmable machine:

- Input:  
  - An encoding of some Turing machine `M`  
  - An input string `w` for `M`  
- The UTM simulates the exact step by step behavior of `M` on `w`.

The tape looks like this:

```text
<description of M> # <input w>
```

where `#` is just a marker symbol.

The UTM is like a VM or interpreter: the first part of the tape is the program, the second part is the data.

#### UTM flow

```mermaid
flowchart LR
  A["Encoded TM M"] --> B["Tape separator #"]
  C["Input w for M"] --> B
  B --> D["Universal TM U"]
  D --> E["Same output M would produce on w"]
```

**Why this matters for me:**

- This is the theoretical basis for the idea of a general purpose computer.  
- "Program" vs "data" is not a hardware distinction, it is a matter of interpretation.  
- Any language that is Turing complete can in principle emulate any other.

---

### 1.4 Church-Turing thesis

The **Church-Turing thesis** is an informal claim, not a theorem:

> Anything that can be computed by a mechanical, step by step procedure can be computed by a Turing machine.

Variants:

- Every "reasonable" programming language can be simulated by a Turing machine and vice versa.  
- Any physically realizable computer can be simulated by a Turing machine, up to efficiency overhead.

For my purposes:

- If there is no Turing machine that solves a problem, then no Java, C, Python, or future computer will either.  
- If there is a Turing machine, then some program can implement that same algorithm.

---

## 2. Computability (5.4)

Now that "algorithm" is formal, we can talk about problems that **no algorithm** can solve.

### 2.1 Decidable vs undecidable

For yes/no questions (decision problems):

- A language `L` is **decidable** (or a problem is decidable) if there exists a Turing machine that:
  - Always halts on every input string
  - Accepts exactly the strings in `L` and rejects the rest

- A language is **recognizable** if there is a TM that:
  - Halts and accepts strings in `L`
  - Might loop forever on strings not in `L`

**Undecidable** means: there is no algorithm that always halts and always gives the correct yes/no answer.

For numeric functions:

- A function `f(x)` is **computable** if some Turing machine, given `x` on its tape, eventually halts with `f(x)` written on the tape.

---

### 2.2 The halting problem

**Problem:**  

> Input: a program `P` and an input `x`.  
> Question: does `P(x)` ever halt, or does it run forever?

We might want a magical method:

```java
// Hypothetical, impossible oracle
static boolean halts(String programSource, String input);
```

that returns true if `programSource` halts on `input`, and false if it loops forever.

Turing proved this cannot exist. The halting problem is **undecidable**.

#### Core proof idea: self reference

Assume (for contradiction) that `halts(P, x)` exists, always halts, and is always correct. Now define this weird program:

```java
// Pseudocode for the diagonal argument
boolean strange(String codeP) {
    if (halts(codeP, codeP)) {
        // If P(P) halts, then this loops forever
        while (true) { }
    } else {
        // If P(P) does not halt, then this halts
        return true;
    }
}
```

Now ask: what happens if we run `strange` on its own source code?

```java
strange("strange");
```

We get a paradox:

- If `halts("strange", "strange")` returns true, strange is supposed to loop forever.  
- If it returns false, strange is supposed to halt.

So in either case `halts` gives the wrong answer. That contradicts our assumption that `halts` exists and is always correct. Therefore such a method cannot exist.

#### Halting problem logic flow

```mermaid
flowchart TD
  A["Assume a perfect halts(P, x) exists"]
  A --> B["Define strange(P) using halts(P, P)"]
  B --> C["Run strange(strange)"]
  C --> D{"halts(strange, strange) ?"}
  D --> E["If true: strange must loop forever"]
  D --> F["If false: strange must halt"]
  E --> G["Contradiction either way"]
  F --> G
  G --> H["Therefore no such halts(P, x) exists"]
```

Result: there is no algorithm that can correctly decide halting behavior for all programs and inputs.

---

### 2.3 Busy beaver (a natural uncomputable function)

Imagine all Turing machines with exactly `n` states, starting on a blank tape. Among those machines that eventually halt, look at how many 1s they leave on the tape. The **busy beaver value** `BB(n)` is the maximum possible number of 1s any such machine can leave.

- `BB(1)`, `BB(2)`, `BB(3)` can be computed by brute force for small `n`.  
- For larger `n`, `BB(n)` grows faster than any computable function.

If we could compute `BB(n)` for all `n`, we could also solve halting for many machines by checking whether they exceed the busy beaver bound. Since halting is undecidable, `BB(n)` must be uncomputable too.

Takeaway: there are "naturally defined" uncomputable functions, not just artificial diagonal tricks.

---

### 2.4 Other undecidable problems (names to recognize later)

Many problems are proved undecidable by reducing the halting problem to them. Some classic examples:

- **Program properties:** any nontrivial property of what a program does on its inputs (for example "halts on all inputs" or "prints 'hello' on some input").  
- **Hilbert's tenth problem:** given a polynomial with integer coefficients, is there an integer solution?  
- **Post correspondence problem:** can you arrange given tiles so that top and bottom strings match?

These show undecidability spreads far outside "just" halting.

---

## 3. Intractability and Complexity (5.5)

Now assume a problem is decidable. Next question: is it **feasible**?

### 3.1 Computable vs tractable

- **Computable:** there exists some algorithm that always halts with the right answer.  
- **Tractable:** there exists an algorithm that runs in time that is realistic for the input sizes we care about.

In theory, any algorithm that halts is "fine." In practice, an algorithm that takes 2^N steps is useless once N gets moderately large.

---

### 3.2 Polynomial vs exponential time

Complexity classes use asymptotic time in terms of input size `N`.

- **Polynomial time:** runtime is `N^k` (or `N^k log N`, etc) for some constant `k`.  
- **Superpolynomial / exponential:** runtimes like `2^N`, `N!`, `N^(log N)`, etc.

In complexity theory:

- Polynomial time is considered "efficient" or "tractable".  
- Exponential time is considered "intractable" for large `N`.

Very rough intuition:

- A `N^3` algorithm may be slow, but doubling hardware power helps a lot.  
- A `2^N` or `N!` algorithm will blow up so fast that even a universe full of computers cannot save you for moderate N.

---

### 3.3 Classes P and NP

We mostly care about **decision problems**, but it is often easier to think in terms of **search problems**: find a solution if one exists.

#### P: problems solvable quickly

- `P` is the class of problems that can be solved in polynomial time by a deterministic machine (a "normal" algorithm).  

Examples (roughly corresponding to earlier chapters):

- Sorting an array  
- Checking if a graph has a path between two vertices  
- Solving a system of linear equations with Gaussian elimination

#### NP: problems verifiable quickly

- `NP` is the class of problems where, given a candidate solution, we can **verify** its correctness in polynomial time.

Example: subset sum:

> Input: integers `a1, a2, ..., aN` and a target `T`.  
> Question: is there a subset that sums to `T`?

- If someone hands me a subset (list of indices), I can check in O(N) that they really add to `T`.  
- Finding that subset in the first place by brute force is O(2^N).

Simple Java verification example:

```java
// Verify a claimed subset solution in O(N) time.
static boolean verifySubsetSum(int[] a, boolean[] choose, int target) {
    int sum = 0;
    for (int i = 0; i < a.length; i++) {
        if (choose[i]) sum += a[i];
    }
    return sum == target;
}
```

This shows subset sum is in NP. We do not know a polynomial time algorithm that always finds a solution.

By definition: `P` is a subset of `NP` (if you can find solutions quickly, you can verify them quickly).

---

### 3.4 Extended Church-Turing thesis (efficiency version)

The original Church-Turing thesis is about what is computable. The **extended** version says:

> Any "reasonable" computing model can simulate any other with at most polynomial slowdown.

So, "polynomial time" is robust: if a problem is in P on one reasonable machine model, it is still "feasible" on any other, up to polynomial factors.

This is why complexity theory focuses on polynomial vs exponential rather than specific constant factors or low level details.

---

### 3.5 P vs NP

The famous open problem:

> Is P equal to NP?

- If P = NP: any problem whose solutions can be checked quickly can also be solved quickly.  
- If P ≠ NP: some problems can be checked quickly but not solved quickly.

Most computer scientists strongly believe P ≠ NP, but nobody has proved it. There is a million dollar prize for a correct proof either way.

---

### 3.6 Reductions and NP-completeness

To compare difficulty of problems, we use **reductions**.

**Idea:** problem A reduces to problem B if:

- There is a polynomial time translator `f` that converts any instance of A into an instance of B.  
- Solving B on `f(instance)` gives a solution to the original A instance.

If A reduces to B and B is easy, then A is also easy.

```mermaid
flowchart LR
  A["Instance of problem A"] --> T["Poly-time transform f"]
  T --> B["Instance of problem B"]
  B --> S["Solution to B"]
  S --> R["Convert back to solution of A"]
```

**NP-complete** problems are:

1. In NP, and  
2. Every problem in NP can be reduced to them in polynomial time.

So NP-complete problems are the "hardest" problems in NP. If we find a polynomial time algorithm for one NP-complete problem, then all of NP collapses into P.

Common NP-complete examples (names only, details later in other courses):

- SAT (satisfiability of Boolean formulas)  
- 3-SAT  
- Clique  
- Vertex cover  
- Traveling salesman (decision version)  
- Subset sum, partition, knapsack (various variants)

These show up everywhere: scheduling, routing, puzzle solving, circuit design, bioinformatics, and more.

---

### 3.7 Class relationships at a glance

```mermaid
graph TD
  P["P (solvable in poly time)"]
  NP["NP (solutions verifiable in poly time)"]
  NPC["NP-complete"]
  NPH["NP-hard"]

  P --> NP
  NP --> NPC
  NPC --> NPH

  subgraph "All decision/search problems"
    P
    NP
    NPC
    NPH
  end
```

Facts:

- We know `P ⊆ NP`.  
- We know NP-complete problems are in NP and are NP-hard.  
- We do not know if `P = NP` or if there are problems in NP that are neither in P nor NP-complete.

---

## 4. Quick self check for Future Me

If I can answer these without notes, I probably remember the core of 5.3-5.5:

1. What does it mean for a model of computation to be universal?  
2. How is a Universal Turing Machine like a modern general purpose computer?  
3. Define "decidable language" and "computable function" in my own words.  
4. State the halting problem. Why is it undecidable?  
5. What is the busy beaver function and why is it uncomputable?  
6. What is the practical difference between an O(N^3) algorithm and an O(2^N) algorithm?  
7. Define P and NP. Why is every problem in P also in NP?  
8. What does it mean for a problem to be NP-complete?  
9. What would a polynomial time algorithm for an NP-complete problem imply about P vs NP?

---

## 5. Optional video reminders

If I want quick refreshers:

- **Computability, Universality (MIT OCW)** - maps the math definitions to actual programming intuition.  
- **Turing Complete (Computerphile)** - good on why "Turing completeness" is the magic threshold.  
- **Turing and the Halting Problem (Computerphile)** - walks through the halting paradox.  
- **Are There Problems That Computers Cannot Solve? (Tom Scott)** - gives the big picture of undecidability and uncomputability.

"""# CS-301 – Computability & Complexity Quiz Notes

These are reference notes for the quiz questions you saw on decidability, computability, and complexity.

---

## Q1. What does decidability refer to in computer science?

**Correct answer:**  
> A Turing machine decides a language if it always gives the correct answer and halts on every input (doesn't go into an infinite loop).

**Notes:**

- *Decidable problem* = there exists an algorithm that:
  - Takes any valid input from the problem domain.
  - Halts on every input.
  - Answers yes or no correctly.
- Decidable is stronger than just “recognizable” - recognizable machines are allowed to loop forever on some inputs, but deciders are not.

---

## Q2. What does computability refer to in computer science?

**Correct answer:**  
> A function f(x) is computable if there exists some Turing Machine such that when its tape is initialized to x, it leaves the string f(x) on the tape.

**Notes:**

- A function is *computable* if there is some Turing machine that:
  - Halts on every possible input x.
  - Outputs f(x) when given x.
- This is like “decidable,” but framed for functions instead of yes/no languages.
- Intuition: if you can write a correct program that always finishes and produces the right output, the function is computable.

---

## Q3. Turing machine and what term are interchangeable?

**Question:**  
In the theory of computing, the terms Turing machine and ____ can be used interchangeably; there may be many Turing machines for each specified task, just as we often consider ________________________________.

**Correct answer:**  
> algorithm; multiple algorithms to solve a programming problem

**Notes:**

- We often treat “Turing machine” and “algorithm” as the same idea:
  - An algorithm is an abstract procedure.
  - A Turing machine is a formal model that captures what an algorithm can do.
- Just like you can have many algorithms for sorting, you can have many Turing machines that solve the same task.

---

## Q4. What is a universal Turing machine?

**Correct answer:**  
> A Turing machine that can simulate the operation of any Turing machine on any given input.

**Notes:**

- A universal Turing machine (UTM) is a single machine that:
  - Takes as input a description of another Turing machine M plus an input x.
  - Simulates “M running on x”.
- This is the theoretical ancestor of modern general-purpose computers.

---

## Q5. What feature do modern computers share with universal Turing machines?

**Correct answer:**  
> They both use the von Neumann architecture: where computer programs and data are stored in the same main memory.

**Notes:**

- In von Neumann architecture:
  - Instructions (programs) and data sit in the same memory.
  - The machine can treat programs as data and vice versa.
- This is exactly what UTMs do - they read the description of another machine as input.

---

## Q6. What is the Church–Turing Thesis?

**Correct answer:**  
> A universal Turing Machine can perform any computation that can be described by any physically realizable computing device.

**Notes:**

- Informal statement: “Whatever can be effectively computed can be computed by a Turing machine.”
- It is not a theorem - it cannot be proved, it is a belief backed by lots of evidence.
- All reasonable models of computation invented so far (lambda calculus, register machines, real computers) are equivalent in power to Turing machines.

---

## Q7. Examples of universal models of computation (select all that apply)

**Correct choices:**

- ✅ Lambda calculus  
- ❌ Finite automata  
- ✅ Cellular automata (with a universal rule set, like rule 110 or Conway’s Life)  
- ✅ Your computer  
- ✅ Most programming languages  
- ❌ Regular expressions (in the formal language sense, not the extended versions in some languages)

**Notes:**

- *Turing complete / universal* means:
  - Can compute any function a Turing machine can compute.
- Finite automata and classical regular expressions are weaker:
  - They can only recognize regular languages, not all computable languages.
- Most real programming languages are Turing complete because they support:
  - Unbounded loops and recursion.
  - Unbounded memory usage in principle.

---

## Q8. What does unsolvability refer to?

**Correct answer:**  
> Problems where there is no algorithm (or Turing machine) to solve the problem.

**Notes:**

- An *unsolvable* (or *undecidable*) problem:
  - Has no algorithm that always halts and gives a correct yes or no answer on every input.
- This is about theoretical impossibility, not “computers are too slow right now.”

---

## Q9. What is the halting problem?

**Correct summary (you had this right):**

1. Problem statement:  
   > Given a program and its input, determine whether that program will halt (not enter an infinite loop) when run on that input.

2. Answer:  
   > It is unsolvable.

**Notes:**

- Alan Turing proved that there is no algorithm that solves the halting problem for all programs and inputs.
- Many other undecidable problems are proved by reducing from the halting problem.

---

## Q10. Why is it important to understand unsolvability?

**Correct answer:**  
> So they don’t waste time trying to solve problems that cannot be solved by any algorithm.

**Notes:**

- If a problem is fundamentally unsolvable:
  - No hardware upgrade will fix it.
  - No clever algorithm can exist.
- Knowing this saves time - you can switch to approximate, heuristic, or restricted versions instead of chasing an impossible exact solution.

---

## Q11. Examples of unsolvable problems (select all that apply)

**Correct choices:**

- ✅ Totality problem (does a given program enter into an infinite loop for any input?)  
- ✅ Program equivalence problem (do two programs compute the same result?)  
- ✅ Functional property (does a program have any nontrivial semantic property?)  
- ✅ Memory management (will a given variable ever be referenced again?)  
- ✅ Virus recognition (is a given program a virus?)  
- ❌ Shortest path problem (this is solvable in polynomial time)  
- ❌ Duplicate detection problem (also solvable - just use sorting or a set)  
- ❌ Parity problem (trivial - just check n mod 2)

**Notes:**

- Rice’s theorem: For any nontrivial semantic property of programs, the problem “does program P have property X?” is undecidable.
  - Program equivalence is one such property.
  - “Will this variable ever be used again?” is another.
  - “Is this program a virus?” is another.
- Totality problem (halts on all inputs) is undecidable too.
- Shortest path, duplicate detection, parity are all classic *solvable* problems.

---

## Q12. Why is worst-case analysis useful?

**Correct answer:**  
> Because it is often easier to reason about upper bounds on running time and is practical in situations where performance guarantees are required.

**Notes:**

- Worst case gives a guaranteed upper bound:
  - “This algorithm will never take longer than c · n² steps.”
- Useful when:
  - You need guarantees (real-time systems, SLAs, etc.).
  - Average or typical case is hard to model.

---

## Q13. Easy vs difficult problems in algorithm analysis

**Correct answer:**  
> Difficult problems often require trying all possibilities to solve them, leading to exponential running time, while easy problems do not.

**Notes:**

- Rough rule of thumb:
  - *Easy* problems - can be solved in polynomial time (like n, n log n, n²).
  - *Difficult* or *intractable* problems - best known algorithms are exponential (like 2ⁿ, n!).
- Exponential-time algorithms blow up quickly and become impractical even for moderate input sizes.

---

## Q14. What is NP in computer science?

**Correct answer:**  
> The set of all decision problems that can be solved in polynomial time on a nondeterministic Turing machine.

**Notes:**

- Equivalent definition that is often used:
  - NP = problems where “yes” answers can be verified in polynomial time by a deterministic machine, given a certificate or witness.
- “Nondeterministic Turing machine” is a mathematical tool - it is not something you can build, but it makes complexity classes easier to define.

---

## Q15. What is P in computer science?

**Correct answer:**  
> The set of all search problems that can be solved in polynomial time (essentially, all search problems that have algorithmic solutions that are guaranteed to finish in a feasible amount of time).

**Notes:**

- More standard phrasing:  
  - P = set of all decision problems solvable in polynomial time on a deterministic Turing machine.
- Intuition:
  - If a problem is in P, then there is an algorithm that always finishes “fast” (polynomial time) on every input.

---

## Q16. What does it mean for a problem to be intractable?

**Correct answer:**  
> A problem is intractable if there exists no polynomial-time algorithm to solve it.

**Notes:**

- In practice, “intractable” usually means:
  - All known algorithms are exponential or worse.
  - No polynomial-time algorithm is known and we suspect none exists.
- In the P vs NP context:
  - If P ≠ NP, then NP-complete problems are intractable.

---

## Q17. Examples of NP-complete problems (give any 3)

Some standard NP-complete problems:

- Boolean satisfiability (SAT) - given a Boolean formula, is there an assignment of true or false values that makes it true?
- 3-SAT - SAT where each clause has exactly 3 literals.
- Traveling salesperson problem (TSP, decision version) - given distances and a budget B, is there a tour visiting each city once with total cost at most B?
- Hamiltonian cycle problem - does a graph contain a cycle that visits each vertex exactly once?
- Clique problem - does a graph contain a clique of size at least k?
- Vertex cover problem - does a graph contain a vertex cover of size at most k?

**Any three distinct ones from this list are good answers.**

---

## Q18. The two possible universes from the chapter

**Correct answer:**  
> NP-complete problems are intractable (P≠NP) OR all search problems are tractable (P=NP)

**Notes:**

- These are the two big possibilities:
  1. **P ≠ NP** - what most computer scientists believe:
     - NP-complete problems are truly difficult.
     - No polynomial-time algorithm exists for them.
  2. **P = NP** - wild alternate universe:
     - Every problem whose solution can be verified quickly can also be solved quickly.
     - Most search problems would become tractable.
- We do not know which universe we are in. This is the famous P vs NP question.
"""


---

