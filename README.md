# Spatial Graph-Enhanced Accommodation Demand Forecasting Using KD-Tree: Algorithm-Agnostic Validation with Explainable AI

> **Published / Submitted to:** ICoICT 2026 Challenge Track — Travlr Challenge Dataset  
> **Affiliation:** Department of Information Technology, Telkom University Surabaya Campus, Surabaya, Indonesia  

---

## Authors

| Name | Affiliation | Contact |
|---|---|---|
| Muhammad Adib Kamali | Telkom University, Surabaya | adibmkamali@telkomuniversity.ac.id |
| Nur Alifia Rustan | Telkom University, Surabaya | nralifialrs@student.telkomuniversity.ac.id |
| Michael Angello Qadosy Riyadi | Telkom University, Surabaya | michaelangello@student.telkomuniversity.ac.id |
| Sulthonika Mahfudz Al Mujahidin | Telkom University, Surabaya | sulthonikamahfudzam@student.telkomuniversity.ac.id |

---

## Abstract

Accurate accommodation demand forecasting is critical for revenue optimization and resource allocation in the hospitality industry. This study proposes a spatial graph-based forecasting framework that constructs a KD-Tree proximity network between **1,455 hotel properties** and **452,044 tourism activities** sourced from the Travlr Challenge dataset. PageRank and Degree Centrality are extracted from the resulting bipartite graph (8,979 nodes, 43,650 edges) as supplementary predictive features for ensemble models. Validation employs a **2×3 algorithm-agnostic experimental matrix** encompassing LightGBM, XGBoost, and Random Forest under Baseline and Proposed architectures, with 5-Fold Cross-Validation enforcing zero label leakage. An ablation study further isolates the contribution of graph-derived topological features against four simpler spatial proximity baselines to confirm genuine structural predictive signal. **LightGBM Proposed achieves the lowest RMSE (1.1684 ± 0.1850)** with a statistically significant improvement over Baseline (Paired T-Test, p = 0.0103), representing a **3.22% reduction**. SHAP Out-of-Fold analysis identifies PageRank as the dominant predictor (SHAP = 0.2273), surpassing all intrinsic and spatial proximity features, while Degree Centrality contributes zero predictive signal. Graph feature extraction imposes negligible computational overhead of **< 0.15%** of total pipeline time, confirming production-scale viability.

**Keywords:** demand forecasting · spatial graph · KD-Tree · PageRank · ablation study · SHAP · cross-validation · algorithm-agnostic

---

## Research Problem and Motivation

The hospitality sector contributes approximately 10% of global GDP and employs over 300 million workers worldwide. Despite the proliferation of ensemble learning methods for tabular prediction, three critical gaps persist in the hotel demand forecasting literature:

1. **Spatial ignorance:** Prevailing models disregard geographic proximity to tourist attractions as a structural demand signal.
2. **Graph underexploration:** Graph-based spatial dependency modeling, well-established in transportation forecasting, remains severely unexplored in hospitality.
3. **Single-algorithm bias:** Most published studies validate on a single algorithm, precluding algorithm-agnostic conclusions.

This study addresses all three gaps within a unified, reproducible experimental framework.

---

## Proposed Framework

The methodology comprises eight sequential phases:

```
Data Acquisition
      ↓
Spatial Preprocessing (Coordinate Parsing)
      ↓
KD-Tree Spatial Graph Construction  [k=30 nearest activities per hotel]
      ↓
Topological Feature Extraction      [PageRank α=0.85 · Degree Centrality]
      ↓
Feature Engineering                 [Label Encoding · Spatial Proximity Features]
      ↓
5-Fold Cross-Validation             [KFold, random_state=42, zero label leakage]
      ↓
Statistical Significance Analysis   [Paired T-Test · Nemenyi CD · Pairwise P-Value Matrix]
      ↓
Interpretability Analysis           [Out-of-Fold SHAP · LightGBM Gain · Split Count]
```

### Graph Construction

The bipartite spatial graph **G(V, E)** is constructed using KD-Tree with O(n log n) complexity. For each of the 1,455 hotel nodes, the k = 30 nearest tourism activity nodes are identified. Edge weights are defined as:

$$w_{ij} = \frac{1}{d(i,j) + \varepsilon}, \quad \varepsilon = 10^{-4}$$

where d(i, j) denotes the Euclidean distance between geographic coordinates.

### Topological Features

**PageRank** (damping factor α = 0.85):

$$PR(v) = \frac{1 - \alpha}{N} + \alpha \sum_{u \in B(v)} \frac{PR(u)}{L(u)}$$

**Degree Centrality:**

$$C_D(v) = \frac{\deg(v)}{N - 1}$$

> **Transductive Setting Disclosure:** The graph, PageRank, and Degree Centrality are computed globally before fold assignment. Test-fold property nodes participate in PageRank propagation. This does not constitute label leakage: the graph is built solely from spatial coordinates with no demand labels embedded. Zero label leakage applies strictly to the supervised target (total_demand).

---

## Experimental Design

### Dataset

| Metric | Value |
|---|---|
| Raw Transactions | 3,734 |
| Unique Properties | 2,799 |
| Properties with Demand Labels (after inner join) | 1,455 |
| Total Accommodations | 3,694,025 |
| Total Activities | 452,044 |
| Graph Nodes | 8,979 |
| Graph Edges | 43,650 |

> Dataset source: **Travlr Challenge Dataset**, provided as part of the ICoICT 2026 Challenge Track.

### Algorithm-Agnostic 2×3 Experimental Matrix

| Architecture | LightGBM | XGBoost | Random Forest |
|---|:---:|:---:|:---:|
| Baseline (6 intrinsic features) | ✓ | ✓ | ✓ |
| Proposed (+PageRank, +Degree Centrality) | ✓ | ✓ | ✓ |

### Hyperparameter Configuration

| Parameter | LightGBM | XGBoost | Random Forest |
|---|---|---|---|
| boosting_type | GBDT | gbtree | Bagging |
| num_leaves / max_depth | 31 | 5 | 10 |
| learning_rate | 0.03 | 0.03 | — |
| feature_fraction | 0.7 | 0.7 | 0.7 |
| subsample | 0.8 | 0.8 | — |
| n_estimators | 400 | 400 | 200 |
| random_state | — | 42 | 42 |

### Ablation Study Design (LightGBM)

| Feature Set | Features (n) |
|---|---|
| Baseline (No Spatial/Graph) | 6 |
| Spatial Proximity Only | 10 |
| Graph Features Only (PageRank + DC) | 8 |
| Full (Graph + Spatial) | 12 |

---

## Results

### 5-Fold Cross-Validation Performance

| Model | RMSE | MAE | R² |
|---|---|---|---|
| **LightGBM Proposed Graph** | **1.1684 ± 0.1850** | 0.5735 ± 0.0295 | **0.0414 ± 0.2514** |
| LightGBM Baseline | 1.2074 ± 0.1929 | 0.6013 ± 0.0289 | -0.0209 ± 0.2561 |
| XGBoost Proposed Graph | 1.2167 ± 0.2055 | 0.5548 ± 0.0289 | -0.0284 ± 0.2295 |
| XGBoost Baseline | 1.2303 ± 0.2166 | 0.5722 ± 0.0339 | -0.0510 ± 0.2416 |
| **RandomForest Proposed Graph** | 1.1992 ± 0.1776 | **0.5487 ± 0.0281** | -0.0062 ± 0.2329 |
| RandomForest Baseline | 1.2223 ± 0.1941 | 0.5762 ± 0.0268 | -0.0371 ± 0.2099 |

### Statistical Significance (Paired T-Test, α = 0.05)

| Algorithm | P-Value | Significant |
|---|---|---|
| LightGBM | 0.0103 | ✅ Yes (p < 0.05) |
| XGBoost | 0.3690 | ❌ No |
| RandomForest | 0.1246 | ❌ No |

### Average Model Ranking (Critical Difference Analysis)

| Rank | Model | Avg. Rank |
|---|---|---|
| 1 | LightGBM Proposed Graph | 1.20 |
| 2 | LightGBM Baseline | 3.00 |
| 3 | RandomForest Proposed Graph | 3.80 |
| 4 | XGBoost Baseline | 4.00 |
| 5 | RandomForest Baseline | 4.40 |
| 6 | XGBoost Proposed Graph | 4.60 |

> Nemenyi post-hoc test (CD = 3.372, α = 0.05): all models fall within a single non-significantly-different clique. LightGBM Proposed consistently occupies the top rank.

### Ablation Study Results (LightGBM, 5-Fold CV)

| Feature Set | Features | RMSE | MAE | R² |
|---|---|---|---|---|
| Baseline | 6 | 1.2074 ± 0.1929 | 0.6013 ± 0.0289 | -0.0209 ± 0.2561 |
| Spatial Proximity Only | 10 | 1.1655 ± 0.1836 | 0.5714 ± 0.0276 | 0.0488 ± 0.2365 |
| Graph Features Only (PageRank + DC) | 8 | 1.1684 ± 0.1850 | 0.5735 ± 0.0295 | 0.0414 ± 0.2514 |
| Full (Graph + Spatial) | 12 | **1.1681 ± 0.1829** | **0.5715 ± 0.0282** | 0.0443 ± 0.2379 |

> Graph Features Only achieves comparable gains to Spatial Proximity Only without raw distance features, confirming that improvement derives from genuine structural topological signal rather than proximity effects alone.

### SHAP Out-of-Fold Feature Importance (LightGBM Proposed)

| Rank | Feature | Mean SHAP | Type |
|---|---|---|---|
| 1 | **pagerank** | **0.2273** | Graph Topological |
| 2 | availability_score | 0.1312 | Intrinsic |
| 3 | popularity_score | 0.1156 | Intrinsic |
| 4 | price_in_aud | — | Intrinsic |
| 5 | guest_rating | — | Intrinsic |
| 6 | star_rating | — | Intrinsic |
| 7 | provider_encoded | — | Intrinsic |
| — | degree_centrality | 0.0000 | Graph Topological |

> PageRank dominates all features. Degree Centrality yields zero SHAP contribution and zero LightGBM split count — once PageRank quality-weighted connectivity is available, raw degree count provides no additional discriminative signal.

### Computational Profiling

| Process Phase | Time (seconds) |
|---|---|
| PageRank Extraction | 0.1067 |
| Degree Centrality | < 0.001 |
| Train LightGBM Proposed Graph (5×) | 1.2312 |
| Train LightGBM Baseline (5×) | 1.0612 |
| Train XGBoost Proposed Graph (5×) | 1.0555 |
| Train XGBoost Baseline (5×) | 1.0416 |
| Train RandomForest Proposed Graph (5×) | 1.7227 |
| Train RandomForest Baseline (5×) | 1.7221 |

> Combined graph feature extraction overhead (PageRank + Degree Centrality): **< 0.15% of total pipeline time**.

---

## Key Findings

1. **PageRank as dominant predictor:** PageRank achieves the highest mean SHAP value (0.2273), surpassing all six intrinsic property features. This establishes spatial topological positioning as a first-order demand signal in hospitality forecasting.

2. **Degree Centrality is redundant:** Once PageRank is present, Degree Centrality contributes zero predictive signal (SHAP = 0, split count = 0), validating the superiority of quality-weighted topological representation over naive degree counts.

3. **Algorithm-agnostic consistency:** The Proposed architecture outperforms Baseline across all three algorithms, with LightGBM achieving statistical significance (p = 0.0103). XGBoost and RandomForest show consistent positive trends without reaching α = 0.05, attributable to LightGBM's GOSS/EFB mechanism more efficiently exploiting asymmetric PageRank score distributions.

4. **Structural signal, not location effect:** The ablation study confirms that Graph Features Only (RMSE = 1.1684) achieves gains comparable to Spatial Proximity Only (RMSE = 1.1655) without raw distance features, isolating graph topology as the source of improvement.

5. **Production viability:** < 0.15% computational overhead confirms direct integrability into existing revenue management systems without architectural modification.

---

## Limitations and Future Work

**Current limitations:**

- The spatial graph is static; it does not capture temporal evolution of activity distributions.
- The adaptive radius for spatial proximity yielded unrealistically large geographic coverage (~8,698 km), limiting density-based feature precision. Domain-calibrated thresholds are recommended.
- Low R² values (max. 0.0414) indicate the presence of unobserved demand confounders beyond the current feature coverage.
- Single-platform data limits cross-platform generalizability.
- Only two topological metrics (PageRank, Degree Centrality) are evaluated.

**Recommended future directions:**

- Dynamic temporal graph integration capturing seasonal activity distribution shifts.
- Domain-calibrated spatial radius thresholds (e.g., 5 km urban, 25 km regional).
- Additional topological metrics: Betweenness Centrality, Closeness Centrality, clustering coefficient.
- Multi-platform dataset validation.
- GNN-based feature extraction (e.g., GraphSAGE, GAT) for end-to-end spatial representation learning.

---

## Data Availability

This study uses the **Travlr Challenge Dataset** provided as part of the ICoICT 2026 Challenge Track. Data access is subject to the terms and conditions of the challenge organizers. Researchers seeking access should refer to the official ICoICT 2026 challenge portal.
