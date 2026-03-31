# Review Responses

## Reviewer JGsU

### Sensitivity to the robustness hyperparameter $\sigma$

- The hyperparameter $\sigma$ acts like a regularization parameter. It is not surprising that the robustness parameter matters in small-data problems.
- We show that one default value of $\sigma=1$ works well for all 100 proteins. Indeed, it works well for both ECFP and MACCS embeddings.

### Failure to reach extremely rare PIC=9

- This is expected; GP-EI and GP-PI fail at the same rate as SPADE, and GP-UCB, GP-M, and TabM fail at about 45% higher rate.

### SPADE relies heavily on the richness of the feature space.

- The fact that SPADE does much better with ECFP (2048-dim) rather than MACCS (167-dim) embeddings shows that SPADE is able to learn from high-dimensions in spite of the small training data.
- We have also run experiments with the (normalized) ChemBERTa embedding (600-dim), which shows a similar pattern as MACCS and ECFP.

| Target PIC $\Rightarrow$ | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| **SPADE is better** | **11%** | **16%** | **17%** | **20%** | 14% |
| GP-PI is better | 2% | 5% | 8% | 15% | **29%** |

- Our results confirm recent benchmarking results (“Benchmarking Pretrained Molecular Embedding Models For Molecular Representation Learning” by Praski et al, August 2025) noting that "nearly all neural models show negligible or no improvement over the baseline ECFP molecular fingerprint."
- Hence, we believe ECFP is a reasonable primary representation for our experiments, while the ChemBERTa results demonstrate that SPADE is not tied to handcrafted fingerprints alone. We will clarify this more explicitly in the revision.


### === EXTRA

- https://arxiv.org/html/2508.06199v2


## Reviewer o72q

### Comparison of TabPFN and SPADE on MACCS embedding

- The comparison of SPADE versus MACCS (Table 2) showed a fair comparison using MACCS for both (all other results in Table 2 use ECFP). To clarify:

| Target PIC $\Rightarrow$ | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| **SPADE (MACCS) is better** | **8%** | **9%** | **12%** | **14%** | **24%** |
| TabPFN (MACCS) is better | 5% | 5% | 8% | 9% | 4% |

- If we use ECFP for SPADE and MACCS for TabPFN, then the difference is much greater:

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| **SPADE (ECFP) is better** | **42%** | **46%** | **49%** | **45%** | **43%** |
| TabPFN (MACCS) is better | 2% | 1% | 1% | 3% | 1% |


### Theoretical optimum or bound for hyperparameter $\sigma$

- $\sigma$ denotes regularization strength. Theoretical results regarding the optimal regularization usually consider the large-$n$ regime, but we have small $n$ ($n=40$, for target PIC=8) and the number of positive samples stays fixed (only the top-k ligands seen so far are in our positive set).
- Instead, the practical problem is to select a good $\sigma$ for any new protein, since we will not have enough samples to cross-validate. We show that **a single default value works well** for all proteins (specifically, $\sigma=1$ for ECFP).

### The meaning of "Δ PIC" in Table 6

- This denotes the difference in PICs between the settings with and without the 10-ligand limit (for any DMTA cycle). We will clarify this in the paper.

### A kernelized or shallow nonlinear version of SPADE

- We agree that nonlinear models are more expressive. But our model allows for a **closed-form expression** for the expected loss, which is key to our scalability to high-dimensional feature spaces.
- We have (as yet) not found closed-form solutions for popular kernels. Without that, we would have to approximate the expected loss by sampling. However, the number of necessary samples grows quickly with the dimensionality of the feature space, and the approximation error may affect the algorithm's quality.


### Comparison of all methods using MACCS embeddings

- The Table below shows the median (over 100 proteins) of the mean-ligands-to-target metric for PIC=8 (lower is better):

|  | SPADE | GP-M | GP-UCB | GP-PI | GP-EI | TabPFN | TabM |
|---|---|---|---|---|---|---|---|
| MACCS | 48 | 54 | 48 | 46 | 51 | 48 | 55 |
| ECFP | **40** | 53 | 51 | 41 | 44 | x | 59 |

- ECFP gives the best results (mean-ligands-to-target PIC=8 is only 40 for SPADE and 41 for GP-PI). GP-PI is the closest competitor across both embeddings, but SPADE is 3× faster (Table 4 of paper). GP-UCB is good for MACCS but much worse for ECFP.
- The following table compares SPADE against its closest competitor (GP-PI). It shows that SPADE is better until PIC=8, and both are roughly comparable for PIC=8.5.

| Target PIC $\Rightarrow$ | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE (MACCS) is better | **10%** | **10%** | **16%** | 13% | 14% |
| GP-PI (MACCS) is better | 3% | 6% | 8% | **16%** | **26%** |

-    We also tested with ChemBERTa (600 dimensions). For all methods, the median ligands-to-target is around 15% worse than ECFP. Between methods, we see the same patterns (SPADE and GP-PI are close, though SPADE is better in the race-to-8).

| Target PIC $\Rightarrow$ | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| **SPADE (ChemBERTa) is better** | **11%** | **16%** | **17%** | **20%** | 14% |
| GP-PI (ChemBERTa) is better | 2% | 5% | 8% | 15% | **29%** |


### Choice of default value of $\sigma$

- We tested various $\sigma$ over around 10 proteins to pick one value ($\sigma=1$). Then, we ran experiments on the other 90 proteins.
- Figure 2 shows that results with $\sigma=0.5$ are within 5% of those for $\sigma=1$.

### Data-driven choice of $\sigma$

- A data-driven strategy would be better when more training data is available. But in our experiments, we usually reach our target PIC of 8 within 40 samples. With so few samples, any data-driven strategy risks additional error.

### Experimental results for different $k\in 5/10/20$ ($k$=number of ligands tested in each DMTA cycle)

- For the same protein and same number of training points, SPADE's average PIC with k=5 is within 0.3% ($\pm$ 1.05%) of SPADE with k=10 (metric=average top-10 PIC).
- For k=20, the difference is 0.4% ($\pm$ 1.8%).


## Reviewer va8h

### Gaussian perturbations for discrete/binary ECFP representations

- We never generate samples from the Gaussian. The Gaussian term is integrated away in the expected-loss calculation (Theorem 1).
- While a discrete distribution could be more interpretable than a Gaussian for ECFP, we may not have a closed form expression. Instead, we must estimate the expected loss by sampling. But the number of samples grows quickly with the dimensionality of the feature space, and the approximation error due to sampling may hurt results.

### Other benchmark datasets and methods

- **Datasets:** We combined several popular datasets with our own data, and ran experiments on the union.
- **Methods:** Our primary domain-relevant baseline is the Gaussian Process framework of Griffiths et al. (NeurIPS 2024), which is, to our knowledge, the most recent method specifically designed for sparse-data iterative molecular optimization of the type we study. SPADE outperforms all four GP variants in Table 1.
- **Other problem settings**: Many other popular methods in drug discovery address a different regime from ours.
    * In particular, docking, free-energy perturbation, and other structure-based pipelines require protein-ligand structural information. They are far slower than the setting we target here, where the goal is to screen large candidate sets over repeated DMTA cycles with minimal prior data.
    * Likewise, widely used affinity datasets such as Davis, BindingDB, PDBbind, and related benchmarks are valuable resources, but many are either too small or designed for different prediction tasks rather than the race-to-8 setting studied here. This is precisely why we constructed a larger benchmark by combining established public data with our new PubChem-derived data.


### Integrating with deep learning molecular representations

- SPADE is orthogonal to the choice of molecular representation or embedding. The main hyperparameter that may depend on the embedding is the robustness scale $\sigma$, although in our experiments $\sigma=1$ worked well across all representations we tested.

- We added new experiments to test SPADE with ChemBERTa, a learned transformer-based embedding (600 dimensions). Both SPADE and GP-PI (the strongest competitor under ECFP) perform somewhat worse with ChemBERTa than with ECFP on the race-to-8 task, but the relative pattern remains the same: SPADE continues to outperform GP-PI at our main target PICs of 8 and 8.5.

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| **SPADE** is better | **11%** | **16%** | **17%** | **20%** | 14% |
| GP-PI is better | 2% | 5% | 8% | 15% | **29%** |

- Our results confirm recent benchmarking results (“Benchmarking Pretrained Molecular Embedding Models For Molecular Representation Learning” by Praski et al, August 2025) noting that "nearly all neural models show negligible or no improvement over the baseline ECFP molecular fingerprint."
- Hence, we believe ECFP is a reasonable primary representation for our experiments, while the ChemBERTa results demonstrate that SPADE is not tied to handcrafted fingerprints alone. We will clarify this more explicitly in the revision.


## Reviewer wkP5

### Dataset has unrealistic affinity distribution

- We agree that PICs are much lower in real-world screening settings. However, we are limited by the availability of public data.
- We combined data from the Davis and BindingDB datasets along with our own 1.5M samples from PubChem. For every protein, we used the union of all ligands for that protein in all these datasets.
- If we draw stratified samples to mimic real-world PIC distributions, we would be left with too few high-PIC ligands.
- Even if the numbers reported in Table 1 would be larger in real-world settings, the results provide a fair comparison between various methods. 

### Even random sampling would quickly identify high-affinity ligands in such a binder-rich dataset

- We compare SPADE against the Random baseline in Table 1, showing that SPADE is far better than Random. For example, to reach PIC=8, SPADE needs around 40 samples while Random needs 76.

### Comparison against standard molecular docking or DL-based methods like Boltz2

- SPADE is **3–4 orders of magnitude faster** at scoring ligands than molecular docking methods and Boltz2 (1.7s/1000 ligands for SPADE versus seconds per ligand for other methods even with GPU acceleration).
- In early-stage drug discovery, we need to quickly screen up to millions of ligands. SPADE is much better adapted for such problems.
- Once SPADE yields a small set of candidates, MD and Boltz2-like methods can be used to winnow this smaller set further.

### Comparison to LIT-PCBA from PubChem BioAssay.

- Our data was curated from PubChem BioAssay. But LIT-PCBA is a smaller dataset (15 proteins with 7,761 actives and 382,674 inactives) than our 1.5M ligand-protein affinity dataset.

### Effect of the activity cliff

- SPADE handles activity cliffs by treating them as local discrimination problems: each classifier separates a top ligand from growing negatives, making low‑affinity near‑neighbors highly informative.
- This avoids assumptions of local smoothness, allows rapid learning of the feature combinations defining high‑PIC ligands from few examples, and is better suited to sharp local affinity changes.
- However, we agree that extreme cliffs still challenge embedding‑based methods when key chemical drivers are not represented in the embeddings.


### "Despite gaining more data, our task becomes harder", is ambiguous.

- We will remove this imprecise statement.
- We meant to say that we are judged on the top-k ligands we find. So, at any point, our goal is to find a ligand better than the current k-th best ligand. In classification terms, the positive set always has k points, no matter how much training data we see.
- To solve this problem, SPADE builds a robust classifier to distinguish the top-k ligands (k stays fixed) from the negative class (which grows in size). The **difference from standard classification** is that the need for robustness does not diminish as we get more data.

## Extra Results

### Mean Ligands to PIC=8

| | SPADE | GP-M | GP-UCB | GP-PI | GP-EI | TabPFN | TabM |
|---|---|---|---|---|---|---|---|
| MACCS | 48 | 54 | 48 | 46 | 51 | 48 | 55 |
| ECFP | 40 | 53 | 51 | 41 | 44 | x | 59 |
| ChemBERTa | 45 | 43 | 42 | 42 | 47 | | |
| ChemBERTa (norm) | 45 | 43 | 42 | 42 | 47 | | |

### Median Ligands to PIC=8

| | SPADE | GP-M | GP-UCB | GP-PI | GP-EI | TabPFN | TabM |
|---|---|---|---|---|---|---|---|
| ChemBERTa | 34 | 30 | 32 | 36 | 43 | | |
| MACCS | 34–35 | 34 | 34 | 38 | 45 | 33 | |
| ECFP | 30 | 29 | 29 | 29 | 37 | x | |

### ChemBERTa (normalized) — Pairwise Comparisons by Target PIC

**SPADE vs GP-PI:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 11% | 16% | 17% | 20% | 14% |
| GP-PI is better | 2% | 5% | 8% | 15% | 29% |

**SPADE vs GP-EI:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 82% | 81% | 63% | 31% | 18% |
| GP-EI is better | 1% | 1% | 3% | 9% | 17% |

**SPADE vs GP-M:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 2% | 7% | 7% | 9% | 7% |
| GP-M is better | 35% | 36% | 26% | 18% | 26% |

**SPADE vs GP-UCB:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 5% | 6% | 8% | 6% | 3% |
| GP-UCB is better | 16% | 21% | 18% | 26% | 35% |

### MACCS — Pairwise Comparisons by Target PIC

**SPADE vs GP-PI:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 10% | 10% | 16% | 13% | 14% |
| GP-PI is better | 3% | 6% | 8% | 16% | 26% |

**SPADE vs GP-EI:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 69% | 75% | 56% | 27% | 22% |
| GP-EI is better | 0% | 2% | 2% | 9% | 17% |

**SPADE vs GP-M:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 3% | 6% | 9% | 15% | 21% |
| GP-M is better | 10% | 19% | 17% | 13% | 11% |

**SPADE vs GP-UCB:**

| | 7.0 | 7.5 | 8.0 | 8.5 | 9.0 |
|---|---|---|---|---|---|
| SPADE is better | 4% | 8% | 6% | 7% | 14% |
| GP-UCB is better | 7% | 13% | 15% | 11% | 26% |
