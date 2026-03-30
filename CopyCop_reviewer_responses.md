# Review Responses



## Reviewer M7hT

### W1

- Replace $|G|$ with $n$ (or $|V|$) for the number of nodes to avoid confusion since $G$ denotes the graph structure.
- Rename Assumption 3.4 to *Non-degenerate transformation*. The intended condition is local invertibility (non-zero directional derivatives), which ensures that $\phi$ does not collapse distinct embeddings. If this fails, one could perturb the input embedding $h$ in some direction without changing $\phi(h)$, meaning that $\phi$ discards information present in $h$.
- $\mathcal{D}$ is a distribution over graph-feature pairs, where each sample $(G,X)$ is one graph together with its node-feature matrix.
- Add an overview figure in Section 1 showing:
  - The threat model, where a victim GNN $M$ produces embeddings, an adversary trains a surrogate $M'$ and transforms its outputs via $\phi$.
  - Why prior watermarking and embedding-similarity methods fail under extraction and transformations.
  - CopyCop's key idea that stationary points of $M$ are invariant under $\phi$, so testing stationarity on a suspect model reveals surrogates.

### W2

- Assumption 3.4 is not needed for the invariance result in Theorem 3.11 used by Algorithm 2 under rotations, scaling, and up-projections. More general transformations are handled via Assumption 3.13.
- Mapping from $\mathbb{R}^d$ to $\mathbb{R}^{d'}$ with $d' < d$ removes some information and, in general, makes imitation of the victim's embedding behavior harder. An attacker seeking a high-fidelity surrogate would therefore avoid projections that discard information relevant to the victim's downstream utility.
- The subtraction $\|\hat{h}_i - h_i\|$ in Eq. 1 is always interpreted in $\mathbb{R}^d$ (the victim's embedding space), not in the surrogate's space.
- The definition does not require the surrogate to explicitly compute $\hat{h}_i$. Rather, it states that if $M'$ is a close surrogate, then there must exist some mapping from $M'$'s outputs back to the victim space that approximately recovers $h_i$; otherwise $M'$ would not preserve the victim's embedding behavior to the tolerance required by Definition 3.2.
- Since $h'_i = \phi(\hat{h}_i)$, we may define $\hat{h}_i := \phi^{-1}(h'_i)$ when $\phi$ is invertible, or more generally as the vector in $\mathbb{R}^d$ minimizing $\|\hat{h}_i - h_i\|$.



- $S(M)$ can be infinite (even uncountable), and there are infinitely many directions $w$ with $\|w\| = 1$.
- The countability requirement in Assumption 3.8 is unnecessarily restrictive. In practice, Eq. 5 defines a sampling procedure over stationary points: we draw $(G, X_0, i, w) \sim \mathcal{D}_{\mathrm{exp}}$ and optimize to a nearby stationary point. This induces a probability measure $\mu_M$ over $S(M)$ regardless of whether $S(M)$ is finite, countable, or uncountable.
- We can revise Assumption 3.8 to remove countability and instead state that $S(M)$ is equipped with a probability measure $\mu_M$ induced by the sampling procedure, such that $P_{\mu_M}(S(M) \setminus S(I)) \ge \gamma_M$ for any independent model $I$.

### W3 - Experimental Setting

- Our threat model targets settings where the victim GNN's embeddings are directly accessible - for example, through an embeddings API or because the victim releases model weights up to an intermediate layer. In this setting, the adversary genuinely has access to embeddings, not just downstream predictions. This is distinct from the prediction-only threat model, and both are valid settings studied in the literature.
- Given this threat model, the attack procedure is appropriate: when embeddings are accessible, a competent adversary would query the victim's embeddings, train surrogates across architectures, and select the best match. The comparison with PreGIP is fair in this context because PreGIP also operates at the embedding level. Thus, the gap in Table 1 is not due to setup bias favoring CopyCop - it reflects that extraction and post-hoc embedding transformations remove watermark information, whereas CopyCop is designed to remain robust to such transformations.



## Reviewer iNur

### W1 - Convergence of the Optimization and Local Minima

- CopyCop does not require Eq. 5 to find exact stationary points. Algorithm 2 works with approximate stationary points: it computes the ratio $\hat{\beta}_Z$ (Eq. 6), which measures how close to stationary a point is relative to random points. If the optimizer returns a point where the directional derivative is small but not exactly zero, this point still yields a small $q_Z(t)$ for the victim and its surrogates, while independent models show no such pattern. Thus, exact global optimality is unnecessary: what matters is the *gap* between $q_Z(t)$ for surrogates versus independent models, not whether $q_Z(t)$ is exactly zero.
- Empirically, this is confirmed: our solver finds only approximate stationary points, yet CopyCop achieves near-perfect AUCs (Table 1).
- Figure 4 shows that the sampled points are nearly orthogonal to each other, indicating that the optimizer is not collapsing to a single basin.
- Figure 3 shows that 20–40 such approximate points suffice for reliable detection.
- Since we use a derivative-free solver on a non-convex objective, we cannot provide global convergence guarantees, but the empirical evidence strongly suggests that approximate solutions are sufficient.

### W2 - How Small Is the Reconstruction Error $\epsilon$ in Practice?

We make two observations.

1. Lemma 3.14 shows that CopyCop's detection metric $\beta_{M'}$ scales as $O(\max(\epsilon/\delta, \delta))$, so detection degrades with $\epsilon$. The method does not require $\epsilon = 0$; it requires $\epsilon = o(\delta)$.
2. The key insight is that $\epsilon$ is constrained by the adversary's own goals: a surrogate with large $\epsilon$ is less faithful to the victim's embedding behavior, and in general would be expected to preserve downstream behavior less reliably. In our model extraction experiments (Table 1), the adversary explicitly minimizes embedding distance, yielding small $\epsilon$ by construction, and CopyCop achieves near-perfect detection.

- Even under fine-tuning and pruning (Figures 1a, 1b), which would increase $\epsilon$, CopyCop remains robust.
- We acknowledge that an attacker who approximates the victim's function without closely matching embeddings (e.g., through knowledge distillation on predictions alone) may achieve larger $\epsilon$. In such cases, CopyCop's guarantees weaken, and prediction-level fingerprinting methods may be more appropriate. We view this as a complementary regime rather than a limitation: CopyCop is designed for the embedding-accessible threat model.

### W3 - Scalability to Higher Embedding Dimensions

- Finding stationary points is a one-time offline cost for the victim model owner. Once computed, testing any suspect model $Z$ requires only forward passes through $Z$, which is fast regardless of embedding dimension. Figure 3 shows that only 20–40 stationary points are needed. Even at our largest embedding dimension (64), Figure 1d shows roughly 40–80 seconds per point, so the entire fingerprinting process completes in under an hour.
- Our current implementation uses Nevergrad (Bennet et al., 2021), a derivative-free optimizer. Since the victim model owner has full access to $M$'s architecture and weights, they can compute gradients of Eq. 5 directly. Switching to a gradient-based optimizer would significantly reduce computation time, especially in higher dimensions where derivative-free methods scale poorly.
- We used Nevergrad to demonstrate that CopyCop works even without gradient access, but in practice the model owner would not face this limitation.

### W4 - Non-linear Transformations

- Lemma 3.7 holds for any differentiable invertible transformation $\phi$ satisfying Assumption 3.4; the proof uses the chain rule, which applies to any differentiable $\phi$, not just linear maps.
- Table 2 empirically tests several non-linear transformations: $\mathrm{sigmoid}$, $\tanh$, $\arctan$, $\exp$, $\sinh$, and polynomial powers ($h^3$, $h^5$). CopyCop performs well under most of these, with exponentiation being the most challenging due to its unbounded distortion of distances (violating Assumption 3.13).

### W5 - What If Embeddings Are Not Accessible?

- CopyCop is designed for settings where the victim provides embeddings through an API or public model release, and the suspect model $Z$'s embeddings can be queried.
- In settings where only downstream predictions are available, CopyCop in its current form is not directly applicable.
- We view CopyCop as complementary to existing methods: CopyCop is stronger when embeddings are accessible, while boundary-based fingerprinting methods (Lukas et al., 2021b; Peng et al., 2022) apply when only predictions are available.

### W6 - Scaling to Extremely Large Graphs

- The optimization in Eq. 5 operates on node features $X$, not on the graph structure $G$. The graph $G$ is sampled once and held fixed during optimization, so the per-iteration cost is dominated by a single forward pass through the GNN on $G$.
- In our experiments, graph sizes range from 10 to 600 average nodes (Table 4). The wall-clock times in Figure 1d show that variation across datasets of different graph sizes is modest compared to variation across embedding dimensions, suggesting that, within the range we study, embedding dimension has a larger effect on runtime than graph size.

### W7 - Number of Stationary Points Needed

- Figure 3 directly answers this: CopyCop needs only 20–40 stationary points to achieve near-maximum detection AUC across all GNN architectures.
- Theoretically, Theorem 3.9 shows that the failure probability decays exponentially as $e^{-2|T|\gamma_M^2}$, so even moderate $|T|$ suffices when $\gamma_M$ is bounded away from zero.
- We will move Figure 3 from the Appendix to the main paper given its importance.



## Reviewer 3yYA

### W1 - Testing Against Other Published Model Extraction Attacks

- The attack of Shen et al. (2022) consists of two stages: first, training a surrogate to mimic the victim's embeddings, and second, fine-tuning on downstream labels. This closely matches the pipeline we already evaluate. Table 1 reports CopyCop's AUC under model extraction, and Figure 1b shows robustness under subsequent fine-tuning. We will make this connection explicit in the revision.

### W2 - Do Published Attacks Satisfy the Assumptions?

- In the embedding-response extraction setting we study, the surrogate is trained to minimize embedding discrepancy directly, so the effective transformation $\phi$ is the identity. Thus, Assumptions 3.4 and 3.13 are trivially satisfied in that setting.
- The transformations in Table 2 (rotations, scaling, projections, and non-linear maps) are not taken from those attacks directly; rather, they are adversarial countermeasures that we introduce as stress tests to evaluate whether a copied model could evade detection after extraction.
- Assumptions 3.4 and 3.13 describe properties such countermeasures should satisfy if the transformed surrogate is to remain useful. Our experiments also include transformations that violate these assumptions (e.g., translation violates $\phi(0) = 0$ in Assumption 3.13), yet CopyCop still performs well, suggesting that the method may remain effective beyond the exact scope of the formal guarantees.

### W3 - No Standard Deviations in Results

- Each AUC in Table 1 is computed by aggregating over all surrogates (across random seeds and all 5 GNN architectures) against all independent models (again across seeds and architectures).

### W4 - Does Training on a Subgraph Affect Definition 3.2?

- Practical extraction attacks are trained only on a subset of queried graphs, so the reconstruction error $\epsilon$ may be smaller on queried graphs than on unseen ones. This does not invalidate our setup for two reasons.
  1. As noted in Remark 3.3, the uniform condition in Definition 3.2 can be relaxed to hold with high probability under the data distribution $\mathcal{D}$, rather than uniformly over all inputs. CopyCop queries the suspect model using graphs sampled from $\mathcal{D}$ (through $\mathcal{D}_{\text{exp}}$) and nearby perturbed graphs induced by the optimization in Eq. 5. As long as $\epsilon$ remains small in this region - which is expected when the adversary's queried subgraphs are sampled from the same distribution - detection signal remains strong.
  2. Practical extraction attacks are designed to generalize beyond the queried subset; indeed, Shen et al. (2022) and Podhajski et al. (2024) both report strong fidelity on held-out data. Empirically, in our experiments the victim–surrogate embedding distance over the full dataset is far smaller than the victim–independent distance, confirming that $\epsilon$ remains small where it matters for detection.



## Reviewer b7h2

### W1 - Missing Related Work [1, 2]

**Chen et al. (2021).** The white-box schemes (V1/V2) are not applicable in our setting because a surrogate GNN trained via model extraction is a separately trained model with different weights, architecture, and sparsity patterns; there is no shared mask from which to extract a signature. The black-box scheme (V3) is a trigger-based watermark / backdoor method. As discussed in Section 2 and shown by prior work on model extraction, such watermarking is vulnerable because the surrogate is trained on ordinary victim outputs and need not inherit the trigger behavior. CopyCop, by contrast, is a fingerprinting method rather than a watermarking method, and Table 1 shows robustness under model extraction.

**Wu et al. (2024).** This work addresses *integrity verification*, i.e., whether a deployed model has been tampered with, whereas CopyCop addresses *ownership verification*, i.e., whether a separately trained suspect model is a surrogate of the victim. Their method assumes the attacker modifies the same deployed model and checks whether predictions on fingerprint nodes still match pre-recorded labels. That threat model is fundamentally different from ours, where the suspect is a new model trained to mimic the victim.

### W2 - Definition 3.2 Is Too Loose / No Upper Bound on $\epsilon$

- Definition 3.2 is not intended as a binary classifier of whether $M'$ is or is not a surrogate. Rather, it parameterizes the closeness of $M'$ to $M$ through the reconstruction error $\epsilon$. Our theory makes this dependence explicit: Lemma 3.14 shows $\beta_{M'} = O\left((C/c)^2 \max(\epsilon/\delta, \delta)\right)$, so detection works when $\epsilon$ is sufficiently small relative to the perturbation scale $\delta$.
- Lemma 3.16 shows that independent models satisfy $\beta_I \ge \gamma_M$, bounded away from zero. Thus, models with small $\epsilon$ behave like surrogates, while models with large $\epsilon$ behave like independent models. We agree that this should be stated more clearly immediately after Definition 3.2, and we will revise the text accordingly.

### W3 - How Can $h'_i$ and $\hat{h}_i$ Differ, and How Is $\hat{h}_i - h_i$ Well-Defined?

- Model $M'$ outputs only one embedding, namely $h'_i \in \mathbb{R}^{d'}$. The vector $\hat{h}_i \in \mathbb{R}^d$ is a latent vector used only in the analysis, defined through the factorization $h'_i = \phi(\hat{h}_i)$, where $\phi : \mathbb{R}^d \to \mathbb{R}^{d'}$ is the adversarial transformation.
- Thus, $\hat{h}_i$ and $h_i$ both live in the victim space $\mathbb{R}^d$, while $h'_i$ lives in the surrogate output space $\mathbb{R}^{d'}$. Therefore $\|\hat{h}_i - h_i\|$ is always well-defined, regardless of whether $d' = d$ or not. Only in the trivial case $\phi = \mathrm{id}$ and $d = d'$ do $\hat{h}_i$ and $h'_i$ coincide.

### W4 - Closure of Assumption 3.4 Under Composition

- We agree that the current wording is too informal. The intended statement is a closure property and should indeed be stated as a lemma. If $f$ and $g$ each satisfy Assumption 3.4, then $f \circ g$ also satisfies it by the chain rule: $\nabla_w (f \circ g)|_h = J_f|_{g(h)} \cdot \nabla_w g|_h$.
- Since $g$ satisfies Assumption 3.4, $\nabla_w g|_h \neq 0$, and since $f$ satisfies Assumption 3.4, $J_f|_{g(h)}$ maps nonzero directional derivatives to nonzero vectors. Hence $\nabla_w (f \circ g)|_h \neq 0$.

### W5 - Meaning of $P(S(M) \setminus S(I)) \ge \gamma_M$

- The intended meaning is the $\mu_M$-measure of the subset $S(M) \setminus S(I)$, where $\mu_M$ is the probability measure over stationary points of $M$ induced by our sampling procedure. Equivalently, $P_{t \sim \mu_M}(t \notin S(I)) \ge \gamma_M$.
- That is, if we sample a stationary point of the victim according to $\mu_M$, then with probability at least $\gamma_M$ it is not also stationary for an independent model. We will rewrite Assumption 3.8 in this explicit form.

### W6 - Hyperparameters, Seeds, and Reproducibility

- We used standard GNN configurations from the literature / PyTorch Geometric defaults (2–3 convolution layers with ReLU and dropout, followed by linear layers where needed).
- Our goal was not hyperparameter optimization, but to evaluate whether CopyCop works across a diverse range of trained models. The results span 5 architectures and 14 datasets, which provides evidence that CopyCop is reasonably robust to variation in architecture and training configuration. Each AUC in Table 1 is an aggregate over all surrogate and independent models across architectures, splits, and random initializations, rather than a single-run value.

### W7 - Feasible Inputs, Shorthand, and Notation

- By "feasible" we mean inputs $(G, X)$ in the support of the data distribution $\mathcal{D}$.
- We will also reduce notation overload: add a clearer reminder after Eq. 3 that $t = (G, X, i, w)$, avoid reusing $t$ with different meanings, and fix typographical issues such as writing $\mu_M$ consistently instead of $\mu(M)$.

### W8 - Algorithm 1 and Sample Complexity

- Theorem 3.9 shows that the error probability is at most $\exp(-2|T|\gamma_M^2)$, so detection confidence improves exponentially with the number of sampled stationary points. This matches Figure 3, where roughly 20–40 points suffice empirically to reach near-maximal AUC.
- **Does $S(I)$ need to be countable?** No. What matters is only the $\mu_M$-measure of the subset $S(M) \setminus S(I)$; the algorithm never samples from $S(I)$.
- **What is Haar?** Here, Haar measure refers to the uniform probability measure on the unit sphere.