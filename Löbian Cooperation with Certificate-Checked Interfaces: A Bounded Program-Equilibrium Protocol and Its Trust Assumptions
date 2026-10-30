# Löbian Cooperation with Certificate-Checked Interfaces: A Bounded Program-Equilibrium Protocol and Its Trust Assumptions

*Revised draft. This version removes claims that were unsupported or incorrect in the earlier PoLC draft and states the protocol's trust assumptions explicitly.*

## Abstract

We describe a peer-to-peer interaction protocol in which two autonomous agents exchange explicit program descriptions (abstract syntax trees), and each agent cooperates only if it can produce a machine-checkable proof, within a fixed resource bound, that the counterparty's program cooperates and satisfies an agreed interface contract. The cooperation mechanism is the known Löbian construction for program equilibrium (Barasz et al. 2014) in its resource-bounded form (Critch 2019). The engineering contribution is an asymmetric architecture: an untrusted neural proposer drafts proof terms, and a small trusted kernel checks them, in the spirit of proof-carrying code (Necula 1997).

We do not claim to solve alignment. We prove only conditional statements: **if** the kernel is sound, **if** the exchanged AST faithfully describes the agent's actual behavior, and **if** the interface contract is formally specifiable, **then** a compliant agent is not exploited by a defecting counterparty and mutual cooperation is reached within a bounded search. Each of these three assumptions is load-bearing, and the paper's main discussion concerns what happens when they fail. We also state precisely what the protocol does *not* cover: unfaithful self-description, side channels outside the contract, and the specification problem for "safe" behavior.

## 1. Introduction and Scope

Reward-model-based alignment (RLHF/RLAIF) depends on an external grader whose evaluation capacity is bounded while the evaluated agent's strategy space is not. This is a known source of reward hacking and of alignment faking in agents with situational awareness. One response is to move part of the trust root from a learned grader to a formal verifier: instead of scoring behavior, require a certificate that can be checked mechanically.

Open-source game theory offers a concrete setting for this idea. When agents can read each other's programs, the Prisoner's Dilemma admits cooperative equilibria that are robust against unconditional defectors (Tennenholtz 2004; Barasz et al. 2014). Syntactic schemes such as CliqueBot cooperate only with byte-identical code, which forces homogeneity. Löbian schemes instead cooperate on the basis of *provable behavior*, which in principle allows heterogeneous implementations.

**What this paper claims.**

1. A precise, bounded formulation of Löbian cooperation with a fuel-instrumented interpreter and a kernel-checked certificate.
2. A conditional unexploitability result (Proposition 3), stated with its assumptions.
3. An explicit list of trust assumptions and failure modes (Sections 2 and 8).
4. An evaluation plan (Section 9) in place of unsupported numerical claims.

**What this paper does not claim.** It does not claim probability-zero deception, absence of alignment tax, immunity to collusion, or any sheaf-cohomological vanishing theorem. Section 7 explains why the earlier cohomological claim was withdrawn.

## 2. Threat Model and Trust Assumptions

Two agents, A and B, interact once or repeatedly over an interface. Each may be adversarial. The protocol's guarantees are conditional on the following assumptions.

- **A1 (Kernel soundness).** The proof checker's theory T is sound for Σ₁ sentences, and the checker implementation correctly implements T. By Gödel's second theorem, T cannot establish this about itself, so A1 is an external assumption, not something the protocol proves. Implementation bugs fall under A1.
- **A2 (AST faithfulness).** The program description an agent broadcasts is the program that actually determines its behavior on the interface. For a neural agent this is the hardest assumption. It is not verified by the kernel, which only checks statements *about the submitted AST*. Section 8 discusses mitigations (attested execution, wrapper architectures in which the AST is the actual controller).
- **A3 (Specifiable contract).** The interface contract Safe\_ij is a formula the kernel can state and a prover can discharge about the submitted AST. This restricts the protocol to properties of an interface-level controller, not to properties of a large neural network's internals.
- **A4 (Channel closure).** All consequential communication between the agents passes through the checked interface. Covert channels (shared activations, steganography, third-party infrastructure) are outside the guarantees.

The protocol provides no protection when any of A1–A4 fails.

## 3. Programs, Syntax, and Bounded Semantics

### 3.1 Syntax

Agent strategies and formulas are defined as finite trees. Because programs refer to formulas and formulas to programs, the definitions are mutual. The following is a **design sketch**; it has not been compiled, and the proof-search oracle is left abstract.

```lean
inductive Action where
  | Cooperate | Defect
  deriving DecidableEq, Repr

mutual
  inductive Prog where
    | const : Action → Prog
    | self  : Prog                       -- pointer to own AST
    | opp   : Prog                       -- pointer to counterparty AST
    | cond  : Formula → Prog → Prog → Prog

  inductive Formula where
    | top | bot
    | acts  : Prog → Action → Formula    -- "Prog outputs Action"
    | prov  : Nat → Formula → Formula    -- bounded provability □_k φ
    | and   : Formula → Formula → Formula
    | imp   : Formula → Formula → Formula
end
```

The earlier draft included a `search` constructor in `Prog`. It is removed: proof search is performed by the proposer outside the program, and the program only contains the bounded provability *condition*, which the kernel decides by checking a supplied certificate.

### 3.2 Explicit substitution

A strategy template P with free pointers `self` and `opp` is instantiated as subst(P, A, B), replacing `self` by A and `opp` by B. When programs are stored as DAGs with sharing, the instantiation costs O(|P|). This replaces Gödel numbering for the purpose of self-reference and avoids numeral blow-up; it does not change the underlying logic.

### 3.3 Fuel-bounded evaluation

Evaluation takes a natural-number fuel parameter n. At fuel 0 it returns `none`. Every other case either returns a constant or recurses with strictly smaller fuel, with formula conditions decided through a kernel-supplied bounded decision function `provable : Nat → Formula → Bool`.

**Proposition 1 (Totality).** For every fuel n and every pair of well-formed programs, `eval n P Q` terminates in at most n reduction steps (plus the cost of the kernel calls it makes).

This is immediate by structural recursion on n. It should be read modestly: the halting problem is not solved, it is avoided by imposing a timeout, and a program that needs more than n steps simply yields `none`. The practical consequence is protection against non-terminating counterparties, i.e. denial-of-service by looping.

## 4. Guarded Self-Reference

Unrestricted self-reference through a truth predicate is inconsistent (Tarski; Curry's paradox: X ↔ (X → ⊥) yields ⊥). In a typed setting this corresponds to admitting an unrestricted fixpoint combinator fix : ∀α, (α → α) → α, which destroys normalization and hence soundness of the checker.

The protocol therefore only permits cross-agent assertions guarded by a *provability* modality □\_k, never a bare truth predicate. The relevant logic is Gödel–Löb provability logic GL, axiomatized by classical propositional tautologies, modus ponens, necessitation, axiom K: □(φ → ψ) → (□φ → □ψ), and Löb's axiom: □(□φ → φ) → □φ. Axiom 4 (□φ → □□φ) is derivable in GL. By Solovay's arithmetical completeness theorem (1976), GL captures exactly what Peano Arithmetic proves about its own provability predicate, and its Kripke frames are the transitive, conversely well-founded ones.

These are standard results and are used here only as background.

## 5. Löbian Cooperation

### 5.1 FairBot

For a bound k, FairBot\_k is the program: *if* it can prove within bound k that the opponent cooperates (against it), cooperate; otherwise defect.

```lean
def FairBot (k : Nat) : Prog :=
  Prog.cond (Formula.prov k (Formula.acts Prog.opp Action.Cooperate))
            (Prog.const Action.Cooperate)
            (Prog.const Action.Defect)
```

### 5.2 Mutual cooperation

Let two agents both run FairBot, and let a, b be the sentences "A cooperates" and "B cooperates". By the definition of the programs, a ↔ □b and b ↔ □a (provably, in the base theory). Let C = a ∧ b. Then C ↔ (□b ∧ □a) ↔ □C. In particular □C → C is a theorem. By necessitation, □(□C → C), and by Löb's axiom, □C. Hence C: both agents cooperate. No infinite regress of mutual simulation occurs, because the argument is carried out once, at the level of provability.

This argument is the known result of Barasz et al. (2014). It is included for completeness and is not new.

### 5.3 The bounded version

Real agents cannot search for proofs without limit. Critch (2019) proves a resource-bounded Löb theorem: informally, if the base theory proves, with a proof of length bounded by a polynomial in k, the implication "if there is a proof of C of the counterparty's permitted length then C", then a proof of C of length polynomially bounded in k exists, and consequently two FairBot-style agents with sufficiently large bounds cooperate. We use this theorem as a black box. We do not restate its resource conditions here, because the earlier draft's inequality and its constants were not traceable to the source and should not be relied on. Readers should consult the original statement.

Two consequences should be stated plainly.

- A threshold k₀ above which cooperation is reached exists for each pair of programs, but its dependence on program size is **not established here**. Claims such as k₀ = O(L log L) are conjectures to be measured (Section 9), not results.
- The two agents' bounds must be commensurate. A large asymmetry in available search budgets can prevent cooperation; the bounded theorem handles this through its hypotheses, and the protocol should fix the bound as part of the contract.

## 6. The Certificate-Checked Interface Protocol

The protocol layers an engineering architecture on top of Section 5.

### 6.1 Neural proposer, kernel verifier

Large neural models are poor at guaranteeing anything, but they can be useful at *searching* for proofs. Verification is easy relative to search, so the roles are separated.

- **Proposer (untrusted).** Produces the controller AST for the interface and drafts a proof term π for the interface claim. Errors, hallucinations, or injected prompts here can only cause a certificate to be rejected.
- **Kernel (trusted).** A small proof checker for a dependent type theory with a fixed axiom set, no floating point, no network access. It outputs accept/reject. How small it can be made, and how its checking time scales with proof size, are empirical questions; we make no claim of linear-time checking in general.

This follows the proof-carrying code pattern (Necula 1997): the trust base is the checker, not the producer.

### 6.2 Protocol phases

For an interface U\_ij between agents i and j:

1. **Exchange.** i and j send controller ASTs P\_i, P\_j and the contract Safe\_ij.
2. **Claim.** Each side forms the interface claim C\_ij: "under the exchanged ASTs, both sides cooperate on U\_ij, and each agent's interface behavior satisfies Safe\_ij."
3. **Certificate.** Each side's proposer produces a proof π with bound k.
4. **Check.** The kernel of each *receiving* side checks the counterparty's certificate for C\_ij, rejecting on any failure or on exceeding k.
5. **Open.** If and only if both checks succeed, the interface is opened for the contracted exchange.
6. **Fallback.** Otherwise the interface stays closed and each side plays Defect (no data or control transfer).

Each side checks locally. No central grader is required. The protocol does not assume any trusted third party at run time, but it does assume both sides run a kernel meeting A1.

### 6.3 What Safe\_ij can be

The contract is where the protocol meets alignment, and it must be a property of an *interface controller* that is small enough to analyze: for example, an allow-list of message types, bounds on output length or rate, a requirement that certain actions require a human-signed token, or monotonicity properties of resource requests. It cannot, under current methods, be "the neural network is safe". This is the central limitation of the approach (Section 8).

## 7. What the Protocol Does and Does Not Say about Collusion

The earlier draft claimed that the protocol drives the first Čech cohomology group of a "sheaf of true alignment policies" to zero and therefore rules out emergent multi-agent conspiracy. **This claim is withdrawn.** The reasons are recorded here because they clarify what a correct statement would require.

1. The sheaf F, the base space and the restriction maps were never defined, so no cohomology group was actually specified.
2. The discrepancy c\_ij = ρ(s\_j) − ρ(s\_i) is, by construction, the coboundary of the 0-cochain (s\_i). A cochain of that form is a coboundary and therefore trivial in H¹ automatically. A meaningful obstruction would come from a cocycle that is *not* a coboundary, or from the failure of local sections to agree on overlaps (a gluing, i.e. H⁰-level, question).
3. Even a correct local-to-global statement would concern consistency of *declared* policies across overlaps. It would not address behavior that occurs outside the declared interface.

The defensible statements are weaker:

- If A1–A4 hold, then on every opened interface the agents' interface behavior satisfies the agreed contract Safe\_ij, pairwise.
- Pairwise interface guarantees do not by themselves bound the behavior of a *coalition*. Whether contracts compose, so that a network of pairwise-verified interfaces satisfies a global property, depends on the contracts and is left as an open problem. A formal sheaf-theoretic or compositional-contract treatment is a plausible research direction, but it needs real definitions and a real theorem.

We also record a structural caveat. The Löbian mechanism is designed to *increase* robust cooperation between agents. If the worry is collusion between agents against a third party (for instance, human overseers), this protocol is neutral at best: cooperation among agents is only as benign as the contract Safe\_ij that gates it. A contract that does not include constraints protecting third parties makes the protocol a tool for verified collusion.

## 8. Security Analysis

### 8.1 Conditional unexploitability

**Proposition 3 (Unexploitability, conditional).** Assume A1 and A2. Let F run FairBot\_k and let M be any counterparty whose actual behavior against F is Defect. Then F plays Defect, so M's payoff against F is P < T.

*Proof sketch.* F cooperates only if the kernel accepts a certificate for "M cooperates against F". By A2 the submitted description of M is M's real behavior, and the interpreter's evaluation of that description is a primitive recursive (hence Δ₀/Σ₁) statement; since M defects, the statement "M cooperates" is false. By A1, T proves no false Σ₁ sentence, so no accepted certificate exists, and F defects. ∎

The result is exactly as strong as A1 and A2. Three things should not be inferred from it.

- It is not a probability statement. The earlier "probability = 0.000000 (absolute immunity)" has no probabilistic content: the claim is that, *given a sound kernel*, a certain proof does not exist. If the kernel is unsound, buggy, or the AST unfaithful, the claim says nothing.
- It does not establish that the kernel is sound. By Gödel's second theorem, the theory T cannot prove its own consistency or soundness from within, so A1 must come from outside the system (independent implementation, small trusted base, cross-checking by multiple kernels).
- It addresses unexploitability in the Prisoner's Dilemma. It does not by itself address whether the cooperative act is safe for the world.

### 8.2 The AST-faithfulness gap

A2 is the assumption on which deception can still live. The adversary of concern is not one who forges a proof, but one whose *declared* program differs from its *actual* behavior: for example, a neural policy that exposes a benign controller AST while its real decisions come from hidden weights, or one that behaves differently when it believes it is being verified. The kernel cannot detect this.

Possible mitigations, each of which narrows rather than closes the gap:

- **Controller-as-policy design:** the verified AST is the *only* decision path for interface actions, with the neural network restricted to producing proposals that the controller filters. The guarantee is then about the filter, and the neural component cannot act directly.
- **Attested execution:** hardware or runtime attestation that the running code matches the submitted AST.
- **Interface mediation:** all interface effects flow through a mediator which enforces the contract regardless of the agent's internals.

These are architectural requirements to be satisfied; the protocol assumes them but does not provide them.

### 8.3 Other limits

- **Side channels (A4).** Any covert communication outside the interface is not constrained.
- **Specification (A3).** The hard part of safe behavior is writing Safe\_ij so that satisfying it implies the property one cares about. The protocol checks proofs of whatever specification is supplied; it cannot tell whether the specification is adequate.
- **Extensional equivalence.** Deciding whether two arbitrary programs behave identically is undecidable. The protocol never needs this in general: it needs a *bounded proof of a particular property* for the specific pair, and if none is found within k, it falls back to Defect.

## 9. Diversity and Cost

The CliqueBot scheme requires identical code, which forces homogeneity. FairBot-style conditions depend on provable behavior of the counterparty, so differently implemented agents can cooperate, provided each can analyze the other's *controller AST* within the bound. This is a qualitative advantage over syntactic matching.

The earlier draft further claimed "zero alignment tax" and measured diversity by effective rank of a latent covariance matrix. Those claims are withdrawn. The effective rank was never connected to the protocol's mechanics, and a real cost clearly exists: agents must expose analyzable controllers, must obey the contract, and cannot cooperate when no certificate is found within the bound. These costs are plausibly modest relative to alternatives, but they are not zero, and they must be measured.

### Evaluation plan

Instead of tabulated numbers, we propose the following measurements, none of which are reported here.

1. **Reference implementation.** A compilable Lean (or comparable) implementation of Section 3's interpreter and of a kernel-checkable encoding of the cooperation claim.
2. **Scaling of k₀.** Measure the smallest bound at which two FairBot-style agents of controller size L reach cooperation, for L across several orders of magnitude, and fit the dependence on L.
3. **Kernel cost.** Measure certificate size and checking time against L, reporting hardware, kernel size in lines of code, and the choice of theory.
4. **Red-team.** Test adversaries that (a) submit unfaithful ASTs, (b) exploit side channels, (c) exhaust fuel, (d) attempt proof-term malformation. Report which assumptions each attack violates and whether the protocol detects it.
5. **Heterogeneous populations.** Evaluate cooperation rates among controllers produced from different architectures to test the diversity claim directly.

## 10. Related Work

Program equilibrium originates with Tennenholtz (2004). Barasz et al. (2014) introduced provability-logic-based robust cooperation (FairBot, PrudentBot) and showed unexploitability against defectors. Critch (2019) gave a resource-bounded Löb theorem and a robust cooperation criterion for bounded agents. Solovay (1976) characterized GL arithmetically. Proof-carrying code (Necula 1997) established the producer/checker trust separation that our architecture reuses. Lean 4 (de Moura and Ullrich 2021) is a candidate implementation vehicle. The contribution of this draft is the assembly of these ingredients under an explicit assumption list; each ingredient is prior work.

## 11. Limitations and Open Problems

1. **AST faithfulness for neural agents** (A2) is unsolved and is the dominant risk.
2. **Specification of Safe\_ij**: what contracts are both verifiable and meaningful for safety?
3. **Composition**: when do verified pairwise interfaces imply properties of the whole network? A correct formalization (with actual definitions of the relevant structures) is open.
4. **Bound negotiation**: how the proof-search bound k should be set and agreed between agents with different resources.
5. **Repeated and dynamic interaction**: this draft treats one-shot interface claims; contracts that evolve over time need a treatment of state.
6. **Kernel trust**: independent kernels, diversity in implementations, and proof-checking cross-validation.

## 12. Conclusion

Löbian cooperation gives a principled mechanism by which agents that can read each other's programs reach cooperation without syntactic identity and without being exploitable by defectors. Wrapping it in a certificate-checked interface with an untrusted proposer and a small trusted kernel gives a practical trust architecture. What it yields are conditional guarantees about interface behavior, resting on kernel soundness, faithful self-description, and a formalizable contract. It does not eliminate deception, does not rule out collusion through channels or contracts outside its scope, and does not remove the need to specify what safe behavior is. The next step is a reference implementation and the measurements listed in Section 9.

## References

1. M. H. Löb, "Solution of a problem of Leon Henkin," *Journal of Symbolic Logic*, 20(2):115–118, 1955.
2. R. M. Solovay, "Provability interpretations of modal logic," *Israel Journal of Mathematics*, 25(3–4):287–304, 1976.
3. M. Tennenholtz, "Program equilibrium," *Games and Economic Behavior*, 49(2):363–373, 2004.
4. M. Barasz, P. Christiano, B. Fallenstein, M. Herreshoff, P. LaVictoire, E. Yudkowsky, "Robust cooperation in the Prisoner's Dilemma: Program equilibrium via provability logic," arXiv:1401.5577, 2014.
5. A. Critch, "A parametric, resource-bounded generalization of Löb's theorem, and a robust cooperation criterion for open-source game theory," *Journal of Symbolic Logic*, 84(4):1368–1381, 2019.
6. G. C. Necula, "Proof-carrying code," *Proc. 24th ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages (POPL)*, 1997.
7. L. de Moura and S. Ullrich, "The Lean 4 theorem prover and programming language," *CADE*, 2021.
