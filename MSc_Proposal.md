# MSc Thesis Proposal: Deductive Verification in Lean 4 with LLM Assistance

**Supervisors:** Jorge Sousa Pinto and Maria João Frade
**Proposal written with assistance from Mistral Vibe (Lean agent)**

---

## Abstract

This thesis explores the use of Lean 4 as a proof assistant for deductive verification, focusing on two classes of systems: functional programs and distributed algorithms. It will investigate how Mistral's Lean agent (Vibe/Leanstral) can assist with formalizing specifications and proving safety properties. The work begins with functional programs (e.g., sorting algorithms such as insertion sort and merge sort) to build familiarity with Lean 4 and Leanstral, then advances to distributed algorithms (using examples such as Chang-Roberts leader election and Paxos consensus) to evaluate the agent's effectiveness in proving invariant-based safety properties. Additionally, we explore the translation of TLA+ and Why3 specifications to Lean 4.

---

## 1. Motivation

Formal verification remains challenging despite advances in proof assistants and automated reasoning. Traditional approaches include:

- **Model checking** (TLA+, Spin): Exhaustive state exploration, limited by state space explosion
- **Deductive verification** (TLA+, Coq, Isabelle): Mathematical proofs, requires significant expertise
- **SMT solvers** (Z3, CVC5): Automated but limited to decidable fragments

Lean 4 offers a powerful middle ground: a proof assistant with dependent types, tactics, and a growing ecosystem (mathlib4). When combined with LLM-based agents, it has the potential to make formal verification more accessible.

**Research gap:** There is no systematic study of Lean 4 + LLM collaboration for formal verification, particularly one that starts with simpler functional programs and progresses to more complex distributed algorithms.

**Novelty:** This thesis systematically evaluates LLM assistance in formal verification, using a progressive approach from functional programs to distributed algorithms, and explores specification translation from TLA+ and Why3.

---

## 2. Background

### 2.1 Systems Under Study

This thesis will explore two classes of systems:

**1. Functional Programs (Initial Exploration)**
- Examples: Sorting algorithms (insertion sort, merge sort), list operations, tree traversals
- Goal: Build familiarity with Lean 4, Leanstral, and inductive proof techniques
- Properties: Correctness, termination, preservation of invariants (e.g., sorted order)

**2. Distributed Algorithms (Primary Focus)**
- Examples: Chang-Roberts leader election, Paxos consensus
- Goal: Evaluate LLM assistance for more complex reasoning
- Properties: Safety invariants (unique leader, value agreement)

### 2.2 Distributed Algorithms and Safety Properties

For distributed algorithms, we focus on **safety properties** proven via inductive invariants.

| Algorithm (Example) | Safety Property | State Machine Model |
|---------------------|----------------|---------------------|
| Chang-Roberts | Max ID elected as leader | Process states: idle, participating, elected, leader |
| Paxos | At most one value chosen | Phases: prepare, promise, accept |

**Safety vs. Liveness:**
- Safety: "Nothing bad ever happens" (invariant-based, e.g., at most one leader)
- Liveness: "Something good eventually happens" (temporal, e.g., leader eventually elected)

This thesis focuses on **safety properties only**, proven via inductive invariants.

### 2.3 TLA+ Specifications

TLA+ defines algorithms via:
```tla
CONSTANT N
VARIABLE leader
init == leader = [i \in Processes |-> FALSE]
typeInvariant == \E i \in Processes: leader[i] => (\A j \in Processes: ~leader[j] \/ j = i)
```

Key features relevant to our work:
- **State machines**: Next-state relation
- **Invariants**: Predicates over states
- **Action properties**: Safety as invariant preservation

### 2.4 Why3 Specifications

Why3 uses first-order logic with:
- **Program specifications**: `requires`, `ensures`, `invariant`
- **Theories**: `int`, `real`, `array`, `list`
- **Proof obligations**: Generated from annotations

Example:
```why
predicate is_max (a: int) (s: set int) =
  forall x. x \in s -> x <= a
```

### 2.5 Lean 4 for Verification

| Feature | Purpose | Relevance |
|---------|---------|-----------|
| Inductive types | Model state machines | Define process states, message types, list structures |
| Dependent types | Precise invariants | `Ring.inv_unique_leader : \u2200 r, Reachable r r' \u2192 ...` |
| Tactics | Proof automation | `omega`, `simp`, `apply`, `decide`, `induction` |
| `sorry` | Placeholder proofs | Identify gaps for LLM assistance |
| `Prop` | Logical propositions | Safety properties as propositions |
| Structures | Algorithm state | `Ring`, `Process`, `Message`, `List` |

**Mathlib4 libraries:**
- `Mathlib.Data.Nat.Basic`: Natural number operations
- `Mathlib.Logic.Basic`: Propositional logic
- `Mathlib.Order.Basic`: Ordering relations (\u2264, <)
- `Mathlib.Algebra.Group.Defs`: Group structures
- `Mathlib.Data.List.Basic`: List operations (for functional programs)

### 2.6 LLM Agents for Formal Verification

| Tool | Capability | Relevance |
|------|------------|-----------|
| Vibe (Mistral) | Lean-specific agent | Lean code generation, proof assistance |
| Leanstral | LSP integration | Context-aware code completion |
| `lake build` | Project building | Verification of formal models |

Additionally, we will evaluate whether **Leanstral is capable of generating appropriate inductive invariants** for both functional and distributed algorithms, which is a key challenge in formal verification.

### 2.7 Related Work: Lean 4 for Formal Verification

The **Veil** repository (https://github.com/leanprover-community/veil) is a Lean 4 framework specifically dedicated to distributed systems (DS) proofs. It provides tools and patterns for formalizing distributed algorithms in Lean. This thesis will build upon and compare with the approaches used in Veil.

Additionally, the **verse-lab/Lentil** repository (https://github.com/verse-lab/lentil) provides another framework for formal verification in Lean 4, focusing on distributed systems and protocols. We will consider its approaches and patterns in our evaluation.

**Note on multi-student participation:** If more than one student is interested in this proposal, it may fork into two distinct but related theses. One possible split would have one thesis focusing specifically on **converting Why3 developments** (both functional programs and distributed algorithms) to Lean 4, while the other maintains the broader scope including TLA+ translation and LLM evaluation.

---

## 3. Methodology

### 3.1 Progressive Approach

The thesis follows a **two-phase approach**:

**Phase 1: Functional Programs**
- Focus on simpler, well-understood algorithms
- Examples: Insertion sort, merge sort, binary search, list operations
- Goals:
  - Build familiarity with Lean 4 syntax and tactics
  - Understand Leanstral's capabilities and limitations
  - Develop proof patterns for inductive invariants
  - Establish baseline metrics for LLM assistance

**Phase 2: Distributed Algorithms**
- Apply lessons from Phase 1 to more complex systems
- Examples: Chang-Roberts leader election, Paxos consensus
- Goals:
  - Evaluate LLM assistance for state-machine reasoning
  - Test invariant discovery and proof construction
  - Compare with manual proof construction

### 3.2 Workflow

```
Functional/Distributed specs (natural language or pseudo-code)
       \u2193 (manual + LLM-assisted)
   Lean 4 formalization
       \u2193
   Safety property specification
       \u2193
   Proof attempt (tactics + LLM)
       \u2193
   Evaluation: % proved, time, iterations
```

### 3.3 Case Studies

**Functional Programs:**
| Algorithm | Complexity | Proof Focus |
|-----------|------------|------------|
| Insertion Sort | O(n^2) | Preserves sorted invariant |
| Merge Sort | O(n log n) | Divide-and-conquer correctness |
| Binary Search | O(log n) | Index bounds, termination |

**Distributed Algorithms:**
| Algorithm | Complexity | Expected Difficulty |
|-----------|------------|---------------------|
| Chang-Roberts | ~100 lines | Medium (state propagation) |
| Paxos | ~300 lines | Hard (multi-phase) |

### 3.4 LLM Integration Strategies

1. **Prompt engineering**: How to ask Leanstral for specific proof steps
2. **Proof decomposition**: Break complex proofs into LLM-manageable chunks
3. **Invariant discovery**: Can LLM suggest invariants from natural language specs?
4. **Error recovery**: How LLM handles proof failures (`sorry` gaps)

### 3.5 Verification Framework

For each system, we will:

1. Define the **state machine** or **function specification**
2. Formalize **safety invariants** (sorted order, unique leader, etc.)
3. Define a **step relation** and/or **inductive structure**
4. Prove that **invariants are preserved**
5. **Evaluate** LLM assistance at each stage

### 3.6 Translation from TLA+ and Why3

- **TLA+**: Extract state variables, next-state relations, and invariants; map to Lean structures and inductive predicates
- **Why3**: Convert pre/post conditions to Lean theorems; map Why3 theories to Mathlib4 libraries
- **Examples**: Both Chang-Roberts and Paxos will be translated from TLA+ specifications; functional programs will be translated from Why3 specifications where applicable

### 3.7 Evaluation Metrics

| Metric | Definition | Measurement |
|--------|------------|-------------|
| Proof completion | % of theorems fully proved | Count with/without LLM |
| Time to proof | Wall-clock time | Timer per theorem |
| Iterations | Number of LLM prompts per proof | Prompt count |
| Quality | Proof correctness | Formal verification |
| Novel invariants | LLM-suggested invariants | Manual review |
| Learning curve | Time to reach proficiency | Phase 1 vs Phase 2 comparison |

---

## 4. Expected Contributions

### 4.1 Technical Contributions

1. **Lean 4 formalizations** of functional programs and distributed algorithms with complete safety proofs
2. **Translation guidelines** from TLA+ and Why3 specifications to Lean 4
3. **Invariant library** for common patterns in both functional and distributed systems
4. **LLM proof strategies** catalog: what works, what doesn't, for different classes of problems

### 4.2 Methodological Contributions

1. **Progressive learning framework** for formal verification with LLM assistance
2. **Best practices** for human-LLM collaboration in proof construction
3. **Limitations analysis**: Where LLMs succeed/fail in formal reasoning across different problem classes

### 4.3 Deliverables

| Deliverable | Format | Audience |
|-------------|--------|----------|
| Thesis document | PDF | Academic |
| Lean 4 codebase | GitHub repo | Researchers, practitioners |
| Translation examples (TLA+, Why3 \u2192 Lean 4) | Markdown docs | Users of TLA+/Why3 |
| Evaluation data | CSV/JSON | Reproducibility |
| Leanstral usage guide | Markdown | LLM-assisted verification community |

---

## 5. Timeline

| Phase | Duration | Activities | Deliverables |
|-------|----------|------------|--------------|
| Literature Review | 1 month | Study Lean 4, TLA+, Why3, formal verification | Annotated bibliography |
| Lean 4 Setup | 1 month | Mathlib4, lake, LSP, Leanstral integration | Working environment |
| Functional Programs (Phase 1) | 2 months | Sorting algorithms: model, invariants, proofs | Formalized algorithms, LLM baseline |
| Distributed Algorithms (Phase 2) | 3 months | Chang-Roberts, Paxos: model, invariants, proofs | Formalized algorithms, LLM evaluation |
| TLA+ Translation | 1 month | Chang-Roberts, Paxos from TLA+ | Translation patterns, evaluation |
| Why3 Translation | 1 month | Functional examples from Why3 | Translation patterns, evaluation |
| LLM Evaluation & Analysis | 1 month | Systematic testing, metrics, comparison | Evaluation results |
| Writing | 2 months | Thesis composition, revisions | Final thesis |
| **Total** | **11 months** | | |

---

## 6. References

### Formal Methods
- Lamport, L. (2002). *Specifying Systems: The TLA+ Language and Tools for Hardware and Software Engineering*. Addison-Wesley.
- Cousot, P. (1990). Hoare Logic. *ACM Transactions on Programming Languages and Systems*.
- Nelson, G. (1981). Operational Semantics. *ACM Computing Surveys*.

### Lean 4
- Ullrich, S. et al. (2019). *The Lean 4 Theorem Prover*. https://arxiv.org/abs/1912.01885
- Mathlib4 documentation: https://leanprover-community.github.io/
- Lean 4 Language Reference: https://lean-lang.org/

### Lean 4 for Distributed Systems
- Veil repository: https://github.com/leanprover-community/veil
- verse-lab/Lentil repository: https://github.com/verse-lab/lentil

### Distributed Algorithms
- Lynch, N. (1996). *Distributed Algorithms*. Morgan Kaufmann.
- Chang, E. & Roberts, E. (1979). An Improved Algorithm for Decentralized Extrema-Finding in Circular Configurations. *SIAM Journal on Computing*.
- Lamport, L. (1989). The Part-Time Parliament. *ACM Transactions on Computer Systems*.

### Functional Programs
- Cormen, T. et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. (For sorting algorithms and correctness proofs)

### LLM for Code
- Mistral AI documentation: https://docs.mistral.ai/
- Vibe CLI documentation: https://mistral.ai/news/vibe-code/
- Leanstral announcement: https://mistral.ai/news/leanstral-1-5/
