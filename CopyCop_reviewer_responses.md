# Review Responses



## Reviewer M7hT

### Notation
* We will use n (or |V|) for the number of nodes to avoid confusion with |G|
* $\mathcal{D}$ is a distribution over graph-feature pairs $(G, X)$.

### Assumption 3.4 (No info loss)
* Our condition ensures that $\phi(v_1) \neq \phi(v_2)$ for distinct vectors $v_1$ and $v_2$ in the same local neighborhood. Hence, $\phi(\cdot)$ is locally invertible. We will rename it to **local invertibility** assumption.

### Problem setting
* In our problem setting, **all models output graph node embeddings**. For example, the victim provides embeddings-as-a-service (Peng et al/2023) or releases model weights up to an intermediate layer (Section 1).
* The adversary has access to the victim model's embeddings, **not downstream predictions.**
* Other recent papers (Waheed et al/2023) also use this model.

### Applicability of no-info-loss condition
* If the attacker's embedding is lower-dimensional than the victim, then it will carry less information and will be less useful for downstream tasks. So no one will use it.
* Hence, the attacker wants to extract the victim's embeddings accurately, and then output a lossless transformation of these embeddings.

### Dimension misalignment between victim and surrogate models
* The victim model outputs embeddings $h_i$, and the attacker can see its dimension.
* The attacker trains their surrogate model $M’$ that outputs $h’_i$, whose dimension can be different from $h_i$. 
* Def 3.2 only says **there exist** $\hat{h}_i$ and $\phi$ that link the two embeddings $h_i$ and $h'_i$. In particular, $\hat{h}_i$ has the same dimension as $h_i$.
  * *Analysis*: We use the fact of the existence of $\hat{h}_i$ and $\phi(\cdot)$. We use $\hat{h}_i - h_i$ only in our analysis.
  * *Algorithm*: We **never use $\hat{h}_i$**, only $h_i$ and $h’_i$. We never compute $h’_i - h_i$, because they can be of different dimensions.
  * *Justification*: We justify Def 3.2 in the para after Eq. 1.

### Assumption 3.8 (infinite stationary points)
* We only need to sample stationary points. Specifically, we draw $(G, X_0, i, w)\sim\mathcal{D}_{exp}$ and find a nearby stationary point (Eq. 5). This induces a probability measure $\mu_M$ over the set of stationary points $S(M)$, whether $S(M)$ is finite, countably infinite, or uncountable.
* The current Assumption 3.8 assumes $S(M)$ is countably infinite. We will remove this requirement.
* If $(G, X, i, w)$ is a stationary point with $\|w\|=1$, all multiples of $w$ will also be stationary. But these are not "independent" stationary points; they are all in the same equivalence class. Hence, we only define stationary points with $\|w\|=1$.

### Experimental setting
* In our setting, the adversary has access to embeddings, not downstream predictions. Given this setting, the attack procedure is appropriate: a competent adversary would query the victim's embeddings, train surrogates across architectures, and select the best match.
* The comparison with PreGIP is fair in this context because PreGIP also operates at the embedding level. Thus, the gap in Table 1 is not due to setup bias favoring CopyCop. It shows that model extraction and post-hoc embedding transformations remove watermark information, whereas CopyCop is designed to remain robust to such transformations.

## ==== Older for Reviewer M7hT ===
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

### Convergence of the Optimization and Local Minima in Eq 5

- We do not need exact the exact optimum of Eq. 5. Algorithm 2 works with approximate stationary points.
- Specifically, the ratio $\beta_Z$ (Eq. 4) measures how close to stationary a point is relative to random points. If the optimizer returns a local optimum with a small enough directional derivative, we still get a small ratio for the victim and its surrogates, but not for independent models. This is all we need.
- Since we use a derivative-free solver on a non-convex objective, we cannot provide global convergence guarantees, but the empirical evidence strongly suggests that approximate solutions are sufficient.
- Figure 4 shows that the sampled points are nearly orthogonal to each other, indicating that the optimizer is not collapsing to a single basin.

### How small is the reconstruction error $\epsilon$ in practice?

* Empirically, the reconstruction error $\|\hat{h}_i-h_i\|/\|h_i\|$, averaged over a dataset, is typically less than 2%.
* While Def 3.2 defines $\epsilon$ in terms of the sup-norm, we only need $\epsilon$ to be small over ``most'' of the dataset (Remark 3.3, line 142). The average reconstruction error of 2% suggests this is likely the case.
* We note that the adversary *wants* a small error: a surrogate with large $\epsilon$ is less faithful to the victim's embeddings, and would be more likely to underperform the victim on downstream tasks.
* CopyCop's performance is robust even under fine-tuning and pruning (Figures 1a, 1b), which increase the difference between the victim and surrogate models.
* If the adversary can achieve good downstream performance without matching the victim's embeddings (say, with a lot of extra training data), CopyCop's guarantees weaken. But then the adversary that uses a lot of extra information is closer to an independent model.


### Scalability to Higher Embedding Dimensions

- Finding stationary points is a one-time offline cost for the victim model owner. Once computed, testing any suspect model $Z$ requires only forward passes through $Z$, which is fast regardless of embedding dimension.
- Figure 3 shows that only 20–40 stationary points are needed.
- Our current implementation uses Nevergrad (Bennet et al., 2021), a derivative-free optimizer. If we assume full access to the victim model, we can use faster optimizers that need more fine-grained information.
- We used Nevergrad to demonstrate that CopyCop works even without fine-grained access to the victim model, but in practice the model owner would not face this limitation.

### Non-linear Transformations

- Lemma 3.7 holds for any differentiable invertible transformation $\phi$ satisfying Assumption 3.4; the proof uses the chain rule, which applies to any differentiable $\phi$, not just linear maps.
- Table 2 empirically tests several non-linear transformations: $\mathrm{sigmoid}$, $\tanh$, $\arctan$, $\exp$, $\sinh$, and polynomial powers ($h^3$, $h^5$). CopyCop performs well under most of these, with exponentiation being the most challenging due to its unbounded distortion of distances (violating Assumption 3.13).

### What If Embeddings Are Not Accessible?

- CopyCop is designed for settings where the victim provides embeddings through an API or public model release, and the surrogate model $Z$'s embeddings can be queried. Other papers (Waheed et al, 2023) have the same setting.
- In settings where only downstream predictions are available, CopyCop in its current form is not directly applicable.
- We view CopyCop as complementary to existing methods: CopyCop is stronger when embeddings are accessible, while boundary-based fingerprinting methods (Lukas et al., 2021b; Peng et al., 2022) apply when only predictions are available.

### Scaling to Extremely Large Graphs

- The optimization in Eq. 5 operates on node features $X$, not on the graph structure $G$. The graph $G$ is sampled once and held fixed during optimization, so the per-iteration cost is dominated by a single forward pass through the GNN on $G$.
- In our experiments, graph sizes range from 10 to 600 average nodes (Table 4). The wall-clock times in Figure 1d show that variation across datasets of different graph sizes is modest compared to variation across embedding dimensions, suggesting that, within the range we study, embedding dimension has a larger effect on runtime than graph size.

### Number of Stationary Points Needed

- Figure 3 shows that CopyCop needs only 20–40 stationary points to achieve near-maximum detection AUC across all GNN architectures.
- Theoretically, Theorem 3.9 shows that the failure probability decays exponentially as $e^{-2|T|\gamma_M^2}$, so even moderate $|T|$ suffices when $\gamma_M$ is bounded away from zero.
- We will move Figure 3 from the Appendix to the main paper given its importance.


### === EXTRA for Reviewer iNur ===
We make two observations.

1. Lemma 3.14 shows that CopyCop's detection metric $\beta_{M'}$ scales as $O(\max(\epsilon/\delta, \delta))$, so detection degrades with $\epsilon$. The method does not require $\epsilon = 0$; it requires $\epsilon = o(\delta)$.
2. The key insight is that $\epsilon$ is constrained by the adversary's own goals: a surrogate with large $\epsilon$ is less faithful to the victim's embedding behavior, and in general would be expected to preserve downstream behavior less reliably. In our model extraction experiments (Table 1), the adversary explicitly minimizes embedding distance, yielding small $\epsilon$ by construction, and CopyCop achieves near-perfect detection.

- Even under fine-tuning and pruning (Figures 1a, 1b), which would increase $\epsilon$, CopyCop remains robust.
- We acknowledge that an attacker who approximates the victim's function without closely matching embeddings (e.g., through knowledge distillation on predictions alone) may achieve larger $\epsilon$. In such cases, CopyCop's guarantees weaken, and prediction-level fingerprinting methods may be more appropriate. We view this as a complementary regime rather than a limitation: CopyCop is designed for the embedding-accessible threat model.


## Reviewer 3yYA

### Testing Against Other Published Model Extraction Attacks

- The attack of Shen et al. (2022) consists of two stages: first, training a surrogate to mimic the victim's embeddings, and second, fine-tuning on downstream labels. This closely matches the pipeline we already evaluate. Table 1 reports CopyCop's AUC under model extraction, and Figure 1b shows robustness under subsequent fine-tuning. We will make this connection explicit in the revision.

### Do Published Attacks Satisfy the Assumptions?

- The transformations in Table 2 (rotations, scaling, projections, and non-linear maps) are not taken from the published attacks. Rather, they are adversarial countermeasures that we introduce as stress tests to evaluate whether a copied model could evade detection after extraction.
- Assumptions 3.4 and 3.13 describe properties such countermeasures should satisfy if the transformed surrogate is to remain useful. Our experiments also include transformations that violate these assumptions (e.g., translation violates $\phi(0) = 0$ in Assumption 3.13), yet CopyCop still performs well, suggesting that the method may remain effective beyond the exact scope of the formal guarantees.

### No Standard Deviations in Results

- Each AUC in Table 1 is computed by classifying all surrogates (across all random seeds and all 5 GNN architectures) against all independent models (again across seeds and architectures).

### Does Training on a Subgraph Affect Definition 3.2?

- The reconstruction error may be smaller on the training graphs/subgraphs than on unseen ones. Still, our approach remains valid, for several reasons:
  1. Practical extraction attacks are designed to generalize beyond the queried subset; indeed, Shen et al. (2022) and Podhajski et al. (2024) both report strong fidelity on held-out data.
  2. We observe the same: the relative error between the victim and surrogate embeddings (without any transformation) **over unseen graphs** is typically less than 2% on average.
  3. As noted in Remark 3.3, we only need the reconstruction error to be small with high probability, rather than uniformly over all graphs. CopyCop queries the suspect model using graphs sampled from the data distribution and then perturbed slightly by the optimization of Eq. 5. We expect the reconstruction error to be small sice the adversary also samples graphs from the same data distribution during model extraction.

### === EXTRA for Reviewer 3yYA ===
- In the embedding-response extraction setting we study, the surrogate is trained to minimize embedding discrepancy directly, so the effective transformation $\phi$ is the identity. Thus, Assumptions 3.4 and 3.13 are trivially satisfied in that setting.


## Reviewer b7h2

### Missing Related Work

**Chen et al. (2021):** The white-box schemes (V1/V2) do not apply in our setting, since an extracted surrogate GNN is a separately trained model with different weights, architecture, and sparsity, so no shared mask exists. The black-box scheme (V3) uses watermarks, which are fragile under extraction (Section 2). CopyCop also outperforms PreGIP, a recent GNN watermark scheme (Table 1).

**Wu et al. (2024):** This studies integrity verification of a deployed model, whereas CopyCop addresses ownership verification of a separately trained surrogate. Their method assumes modification of the same deployed model, which differs fundamentally from our setting where the suspect is independently trained to mimic the victim.

### How can $h'_i$ and $\hat{h}_i$ differ, and how is $\hat{h}_i - h_i$ well-defined?

* The victim and surrogate models $M$ and $M'$ output embeddings $h_i$ and $h’_i$, whose dimensions can be different.
* Def 3.2 only says **there exist** $\hat{h}_i$ and $\phi$ that link the two embeddings $h_i$ and $h'_i$. In particular, $\hat{h}_i$ has the same dimension as $h_i$.
  * Our analysis uses the fact of the existence of $\hat{h}_i$ and $\phi(\cdot)$. We use $\hat{h}_i - h_i$ only in our analysis.
  * Our **algorithm never uses $\hat{h}_i$**, only $h_i$ and $h’_i$. We never compute $h’_i - h_i$, because they can be of different dimensions.
  * We justify Def 3.2 in the para after Eq. 1.

### Definition 3.2 has no upper bound on $\epsilon$

- Definition 3.2 parameterizes the closeness of models $M'$ and $M$ through $\epsilon$.
- We can detect surrogates when $\epsilon$ is small. Lemma 3.14 shows Algorithm 2 needs a perturbation scale $\delta$ such $\epsilon=o(\delta)$ for surrogate detection to succeed. We will state this up front, right after Defn 3.2
- If \epsilon is large, the surrogate embeddings deviate from the victim embeddings even for $\phi=id$, so the surrogate is unlikely to perform as well as the victim on downstream tasks. Hence, the adversary has an incentive to keep \epsilon small (line 133).

### Closure of Assumption 3.4 Under Composition

- The intended statement is a closure property and we agree that it should indeed be stated as a lemma.
- **Lemma:** If $f(\cdot)$ and $g(\cdot)$ each satisfy Assumption 3.4, then $(f \circ g)(\cdot)$ also satisfies it.
    - **Proof:** $\nabla_w f = J_f w$ for any $w$, where $J_f$ is the Jacobian of $f(\cdot)$. By assumption, $J_f w\neq 0$ and $J_g v\neq 0$ for any vector $w$ and $v$. Then, by the chain rule of Jacobians, $\nabla_w f\circ g = J_{f\circ g} w = J_f J_g w\neq 0$. Hence, $f\circ g$ also satisfies Assumption 3.4.

### Meaning of $P(S(M) \setminus S(I)) \ge \gamma_M$

- $P(S(M) \setminus S(I))$ is the $\mu_M$-measure of the subset $S(M) \setminus S(I)$, where $\mu_M$ is a probability measure over $S(M)$.
- For instance, if $S(M)$ was finite and $\mu_M$ was the uniform measure on $S(M)$, then $P(S(M) \setminus S(I)) = |S(M) \setminus S(I)|/|S(M)|$.
- Informally, if we sample a stationary point from $S(M)$ according to $\mu_M$, then "with probability at least $\gamma_M$", this point is not also a stationary point for the independent model $I$. We will rewrite Assumption 3.8 to clarify this.

### Hyperparameters and Reproducibility

- We used standard GNN configurations from the literature (2–3 convolution layers with ReLU, dropout, and linear layers where needed). We will update the appendix with this information.
- Our goal was to evaluate whether CopyCop works across a diverse range of trained models, and not tuning hyperparameters for the highest accuracy. The results span 5 architectures and 14 datasets, which shows that CopyCop is reasonably robust to such variations. We will release the code after incorporating comments from reviews.

### Explanation of Table 1 and 2

- Each AUC in the tables is computed by classifying all surrogates (across all random seeds and all 5 GNN architectures) against all independent models (again across seeds and architectures). The tables are not from single runs.

### Feasible Inputs, Shorthand, and Notation

- By "feasible" we mean inputs $(G, X)$ in the support of the data distribution $\mathcal{D}$. 
- We will reduce notation overload: add a clearer reminder after Eq. 3 that $t = (G, X, i, w)$, avoid reusing $t$ with different meanings, and fix typo issues such as writing $\mu_M$ consistently instead of $\mu(M)$.

### Algorithm 1 and Sample Complexity

- Detection confidence improves exponentially with the number of sampled stationary points (Theorem 3.9). Figure 3 in the Appendix shows that 20–40 points suffice empirically to reach near-maximal AUC.
- **Does $S(I)$ need to be countable?** No. What matters is only the $\mu_M$-measure of the subset $S(M) \setminus S(I)$; the algorithm never samples from $S(I)$.
- **What is Haar?** Haar measure refers to the uniform probability measure on the unit sphere.

### === EXTRA
- Lemma 3.16 shows that independent models satisfy $\beta_I \ge \gamma_M$, bounded away from zero. Thus, models with small $\epsilon$ behave like surrogates, while models with large $\epsilon$ behave like independent models. We agree that this should be stated more clearly immediately after Definition 3.2, and we will revise the text accordingly.

- Model $M'$ outputs only one embedding, namely $h'_i \in \mathbb{R}^{d'}$. The vector $\hat{h}_i \in \mathbb{R}^d$ is a latent vector used only in the analysis, defined through the factorization $h'_i = \phi(\hat{h}_i)$, where $\phi : \mathbb{R}^d \to \mathbb{R}^{d'}$ is the adversarial transformation.
- Thus, $\hat{h}_i$ and $h_i$ both live in the victim space $\mathbb{R}^d$, while $h'_i$ lives in the surrogate output space $\mathbb{R}^{d'}$. Therefore $\|\hat{h}_i - h_i\|$ is always well-defined, regardless of whether $d' = d$ or not. Only in the trivial case $\phi = \mathrm{id}$ and $d = d'$ do $\hat{h}_i$ and $h'_i$ coincide.

- The intended meaning is the $\mu_M$-measure of the subset $S(M) \setminus S(I)$, where $\mu_M$ is the probability measure over stationary points of $M$ induced by our sampling procedure. Equivalently, $P_{t \sim \mu_M}(t \notin S(I)) \ge \gamma_M$.
- That is, if we sample a stationary point of the victim according to $\mu_M$, then with probability at least $\gamma_M$ it is not also stationary for an independent model. We will rewrite Assumption 3.8 in this explicit form.

- For instance, if the node features are integers, then feasible graphs cannot have floating-point features.



# Rebuttal respose

## iNur

### Experimental results on convergence for larger graphs
* We show below the average norm(embedding gradient)/norm(embedding) ratio at randomly chosen points and at the stationary points chosen by our optimization (Eq. 5), for the top-2 datasets by node size and the top-2 by feature size. In every case, the optimization finds points with smaller gradient ratios (typically less than 0.05).

| | ARMA | GCN | GIN | GraphSAGE | MixHop |
|---|---|---|---|---|---|
|Citeseer (random) | 0.23  | 0.16  | 0.25  | 0.26 | 0.21  |
|Citeseer (stationary) | 0.05 | 0.04  | 0.01 | 0.05 | 0.05 |
|DBLP (random) | 0.51 | 0.21  | 0.46 | 0.35 | 0.46 |
|DBLP (stationary) | 0.01 | 0.02 | 0.00  | 0.01 | 0.02 |
|Fin (random) | 0.08 | 0.08  | 0.12  | 0.16 | 0.10 |
|Fin (stationary) | 0.05 | 0.06 | 0.04 | 0.06 | 0.07 |
|Coco (random) | 0.04 | 0.06  | 0.48 | 0.14 | 0.14 |
|Coco (stationary) | 0.02 | 0.05 | 0.07 | 0.05 | 0.09 |

* For very large graphs, we **do not need to optimize over the entire $X$**. Since the goal is to get a stationary point for the embedding of one node, we can optimize over the features of that node and the k-nearest-neighbor nodes in its neighborhood. This would give enough flexibility to find a good stationary point, and the complexity would depend on $k$, not the number of nodes.

## b7h2

### (c and d) How $h'_i(G,X;M')$ and $\hat{h}_i(G,X;M')$ can be different if they are both computed on the same G and X

Here's an example. Suppose the attacker decides to use $\phi(x)= Mx$, where $x \in\mathbb{R}^d$ and $M\in\mathbb{R}^{d'\times d}$ is a fixed matrix. The attacker collects input-output samples of the form $((G,X), {h_i})$ from the victim model $M$, and trains a GNN to match these pairs. That is, given (G,X), this GNN outputs $\hat{h}_i$ which are very close to $h_i$, and this behavior generalizes to unseen $(G,X)$ too. Then, the attacker adds an extra fully-connected layer to this GNN so that it output $M\hat{h}_i$ instead of $\hat{h_i}$ (one fully-connected layer suffices since it is a linear transformation). This GNN-with-extra-layer becomes the attacker's GNN $M'$.

In this example, $M'$ outputs $h'_i = M\hat{h}_i \in \mathbb{R}^{d'}$, where $\hat{h}_i$ is an "internal" embedding generated within $M'$ that is never output. This is why our algorithm only uses $h_i$ and $h'_i$, never $\hat{h}_i$. Only our analysis uses the fact that such an $\hat{h}_i$ exists.

We emphasize that this is only one example. The attacker can use any approach to train $M'$. We just assume that the outputs $h'_i$ from $M'$ are linked to the outputs $h_i$ of $M$ via some (unknown) $\phi$ and $\hat{h}_i$.

To answer (c): The adversary can choose any embedding dimension for $h'_i$, but not for $\hat{h}_i$. In the example above, $h'_i$ has dimension $d'$, while $h_i$ and $\hat{h}_i$ have dimension $d$.

### (g and h) Empirical settings

| Model | Setup |
|---|---|
| MixHop | (MixHopConv + ReLu) * 2 + Linear |
| ARMA | (ARMAConv + ReLu + Dropout) * 3 + Linear |
| GraphSAGE | (SAGEConv + ReLu + Dropout) * 2 + SAGEConv |
| GCN | (GCNConv + ReLu) * 3 + GCNConv |
| GIN | (GINConv + ReLu) * 2 + GINConv |

All models: If the final output is at the node level, there is a final linear layer. If the final output is at the graph level, there is a GlobalMeanPooling + Dropout + Linear layer at the end.

All seeds are fixed. However, we think (but are not sure) that Nevergrad's outputs might depend on the system load (we use nevergrad to solve Eq. 5).

### (i) Repo
The code is available at [here](https://anonymous.4open.science/r/CopyCop---Graph-Ownership-Verification--120D/README.md)

### Presentation

Thank you for your detailed suggestions. We will revise the paper accordingly.
