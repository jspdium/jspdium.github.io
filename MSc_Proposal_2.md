# MSc Thesis Proposal: Deductive Verification of Ephemeral Trust for Secure CI/CD Runners

**Supervisors:** Jorge Sousa Pinto and Simão Melo de Sousa (UAlgarve)

**Proposal written with assistance from Mistral Vibe (Lean agent)**

This proposal is in the scope of the BRINGTRUST project.

---

## Abstract

This thesis proposes a formally verified model for hardened ephemeral CI/CD runner execution using **deductive verification** with Why3 or Lean 4. Unlike existing model-checking approaches, we will use proof assistants to establish formal guarantees for attestation-bound trust establishment, immutable policy enforcement, ephemeral secret leasing, and fail-closed revocation semantics. The work models trusted execution as a lifecycle-oriented authorization system whose correctness depends on preserving invariants throughout execution evolution. We will also explore the use of Leanstral as an auxiliary tool for proof construction and code generation.

---

## 1. Motivation

Continuous Integration and Continuous Deployment (CI/CD) infrastructures are security-critical components of modern software delivery pipelines. Contemporary CI/CD systems routinely process sensitive assets including source code, deployment credentials, signing material, and release artifacts. Compromised CI/CD infrastructures enable malicious artifact injection, credential exfiltration, dependency manipulation, and downstream software supply-chain compromise.

While frameworks like SLSA and in-toto improve provenance assurance and workflow integrity, CI/CD runners are still commonly trusted once execution begins, retaining broad access without continuously re-evaluating authorization validity. This limitation is particularly relevant in ephemeral cloud-native infrastructures, where runners are dynamically provisioned under asynchronous orchestration.

This thesis addresses these challenges through **deductive verification** of a lifecycle-oriented formal model, providing mathematical proofs of security properties rather than exhaustive state-space exploration.

---

## 2. Background

### 2.1 CI/CD Runner Security and Ephemeral Execution

CI/CD runners execute workflow tasks such as compilation, testing, artifact generation, dependency retrieval, and deployment, while processing sensitive assets including source code, credentials, signing material, and software artifacts. Modern platforms increasingly adopt ephemeral execution models in which isolated runners are provisioned for individual jobs and destroyed after completion, reducing persistence and cross-workload contamination risks.

### 2.2 Trusted Execution and Remote Attestation

Trusted Execution Environments (TEEs) and confidential computing platforms provide mechanisms for protecting workload integrity and confidentiality within partially trusted infrastructures through hardware-enforced isolation and remote attestation. Within CI/CD environments, these technologies enable attestation-bound authorization and secret release.

### 2.3 Deductive Verification Approach

This thesis uses **deductive verification** rather than model checking. The key difference:

| Approach | Method | Tools | Advantages | Limitations |
|----------|--------|-------|------------|-------------|
| Model Checking | Exhaustive state exploration | TLA+, TLC, Spin | Automated, complete for finite state | State space explosion |
| Deductive Verification | Mathematical proof | Why3, Lean 4, Coq | Scalable, compositional | Requires proof construction |

We will use either **Why3** or **Lean 4** to construct formal proofs of safety properties.

### 2.4 Why3 for Deductive Verification

Why3 is a deduction-based verification platform that:
- Uses first-order logic with theories for integers, arrays, lists, etc.
- Generates proof obligations from program annotations (`requires`, `ensures`, `invariant`)
- Supports multiple backends (Alt-Ergo, Z3, CVC5, E, etc.)
- Provides a library of verified data structures and algorithms

Example specification:
```why
predicate is_fresh (token: token) (session: session) =
  token.expiry > session.start_time

lemma fresh_token_preserved (token: token) (session: session) :
  is_fresh token session ->
  is_fresh token (session with start_time = session.start_time + 1)
```

### 2.5 Lean 4 for Deductive Verification

Lean 4 is a powerful proof assistant that:
- Uses dependent type theory for precise specifications
- Provides tactics for semi-automated reasoning (`omega`, `simp`, `apply`, `induction`)
- Has a growing library (mathlib4) for algebraic structures and algorithms
- Supports structured proofs with `by` blocks and `sorry` placeholders

**Leanstral:** We will also explore Mistral's Lean agent as an auxiliary tool for proof construction, code generation, and invariant discovery.

Example specification:
```lean
structure RunnerState where
  attestation : Attestation
  policy : Policy
  secrets : Set Secret
  trust : Bool

/-- Invariant: secrets are valid only when trust is established -/
theorem secrets_valid_iff_trust (s : RunnerState) :
    s.secrets ≠ ∅ → s.trust = true := by
  sorry
```

---

## 3. Methodology

### 3.1 System Model

We will model a hardened ephemeral CI/CD runner architecture with the following entities:

- **CI/CD orchestration service**: Runner provisioning and lifecycle management
- **Attestation authority**: Validates execution integrity
- **Policy authority**: Defines execution constraints
- **Secret management service**: Issues ephemeral secret leases
- **Runner instances**: Execute CI/CD workloads (ephemeral, isolated)

### 3.2 Execution Lifecycle

The execution lifecycle consists of controlled phases:

1. **Provisioning & Bootstrap**: Runner created, remains untrusted
2. **Attestation & Authorization**: Runner proves integrity, receives execution capability
3. **Execution**: Runner performs workload with ephemeral secrets and single-use capability
4. **Revocation & Teardown**: Execution capability and secrets are destroyed

### 3.3 Security Objectives

We will formalize and prove the following security objectives:

| Objective | Description | Formalization |
|-----------|-------------|---------------|
| O1 — Attestation-Bound Trust | Trust requires valid attestation and explicit policy authorization | Invariant: `trust → valid_attestation ∧ authorized` |
| O2 — Ephemeral Authorization | Execution authorization is single-use and bounded | Invariant: `execution_capability.consumed ↔ executed` |
| O3 — Ephemeral Secret Leasing | Secrets bound to execution context, revoked on expiration/violation | Invariant: `secret.valid → trust ∧ !expired` |
| O4 — Immutable Policy Enforcement | Authorized policies remain immutable | Invariant: `∀ t, policy(t) = policy(0)` |
| O5 — Fail-Closed Revocation | Replay detection, attestation failure, expiration → revoke all privileges | Theorem: `violates_lifecycle → revoked` |
| O6 — Replay Resistance | Replay-derived authorization never establishes trusted execution | Theorem: `¬ (replay → trust)` |

### 3.4 Formalization in Why3 or Lean 4

**State representation:**
- Define types for attestations, policies, secrets, execution capabilities
- Model lifecycle states as a state machine
- Represent adversarial behaviors as transitions

**Safety properties:**
- Formalize O1-O6 as invariants or theorems
- Prove preservation under state transitions
- Establish correctness of the lifecycle model

**Adversarial scenarios:**
- Replay attacks
- Stale session reuse
- Unauthorized policy injection
- Cross-runner secret reuse
- Use-after-expiry execution attempts

### 3.5 Verification Approach

For each security objective, we will:

1. **Specify** the property in Why3/Lean 4
2. **Decompose** into manageable lemmas
3. **Prove** using tactics and automated provers (Why3) or direct proofs (Lean 4)
4. **Validate** with Leanstral assistance where applicable

### 3.6 Comparison: Why3 vs Lean 4

| Aspect | Why3 | Lean 4 |
|--------|------|-------|
| Type system | First-order logic | Dependent types |
| Automation | SMT solvers (Z3, Alt-Ergo) | Tactics + automation |
| Libraries | Why3 standard library | Mathlib4 |
| Learning curve | Moderate (FOL-based) | Steeper (dependent types) |
| LLM assistance | Limited context | Leanstral integration |

We will evaluate both and select the most suitable for the formalization.

---

## 4. Expected Contributions

### 4.1 Technical Contributions

1. **Deductive formalization** of ephemeral CI/CD runner execution using Why3 or Lean 4
2. **Lifecycle-oriented verification framework** with formally proved security objectives (O1-O6)
3. **Adversarial model** formalizing replay attacks, stale session reuse, unauthorized policy injection, cross-runner secret reuse, and use-after-expiry execution
4. **Proof library** for trusted execution properties reusable by other projects
5. **Evaluation of Leanstral** for proof construction in security-critical formal verification

### 4.2 Methodological Contributions

1. **Comparison of deductive vs. model checking** for CI/CD security verification
2. **Patterns for formalizing lifecycle-oriented systems** in proof assistants
3. **Best practices for combining** traditional proof techniques with LLM assistance
4. **Guidelines for selecting** Why3 vs. Lean 4 for security verification tasks

### 4.3 Deliverables

| Deliverable | Format | Audience |
|-------------|--------|----------|
| Thesis document | PDF | Academic |
| Formal specifications | Why3/Lean 4 files | Researchers, practitioners |
| Proof library | Reusable modules | Formal verification community |
| Leanstral evaluation | Markdown report | LLM-assisted verification community |
| Comparison analysis | Markdown | Both Why3 and Lean 4 communities |

---

## 5. Timeline

| Phase | Duration | Activities | Deliverables |
|-------|----------|------------|--------------|
| Literature Review | 1 month | Study CI/CD security, formal verification, Why3, Lean 4 | Annotated bibliography |
| Tool Setup | 1 month | Install Why3/Lean 4, configure provers, set up Leanstral | Working environment |
| System Modeling | 1.5 months | Define types, state machine, lifecycle phases | Formal system model |
| Security Objectives | 1.5 months | Formalize O1-O6, basic lemmas | Property specifications |
| Adversarial Model | 1 month | Model replay, stale reuse, policy injection | Adversarial transition system |
| Proof Construction | 2 months | Prove safety properties, use Leanstral | Complete proofs |
| Validation & Testing | 1 month | Test with examples, evaluate Leanstral | Validated model, evaluation |
| Why3/Lean 4 Comparison | 1 month | Compare both approaches, analyze trade-offs | Comparison report |
| Writing | 2 months | Thesis composition, revisions | Final thesis |
| **Total** | **11 months** | | |

---

## 6. References

### CI/CD Security
- SLSA Framework: https://slsa.dev/
- in-toto: https://in-toto.io/
- Zero Trust Architecture principles

### Trusted Execution
- Intel SGX documentation
- AMD SEV-SNP documentation
- Intel TDX documentation

### Deductive Verification
- Why3 documentation: https://why3.lri.fr/
- Why3 standard library
- Lean 4 documentation: https://lean-lang.org/
- Mathlib4 documentation: https://leanprover-community.github.io/

### LLM for Code
- Mistral AI documentation: https://docs.mistral.ai/
- Vibe CLI documentation: https://mistral.ai/news/vibe-code/
- Leanstral announcement: https://mistral.ai/news/leanstral-1-5/
