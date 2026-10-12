# DL-CGM

Implementation notes for the manuscript

**Clinically Subgrouping T2DM Patients via Deep-learning based on CGM Data**

## 1. Code availability

The full source code will be released in this repository upon acceptance of the manuscript. The current version provides the framework outline of the proposed method and the implementation details of the benchmark methods.

---

## 2. Proposed method

| Setting | Value |
|---|---|
| Encoder | Temporal convolutional network (TCN) |
| Training objective | CGM reconstruction + Kullback–Leibler term + spectral auxiliary reconstruction |
| Group labels | A–D in ascending order of mean patient CGM |

Clinical variables were not used for model training or clustering; they were used only to characterize the resulting subgroups.

---

## 3. Benchmark methods

All methods used the same patients and daily CGM segments on the 15-minute grid and produced four groups. CH and DB were evaluated in a common standardized 96-point daily-curve space after assigning each patient's label to all of that patient's days.

| Method | Input / representation | Clustering | Patient label |
|---|---|---|---|
| DTW | Daily sequences | DTW-distance clustering of days | Majority vote over days* |
| K-shape | Daily sequences | Shape-based distance clustering of days | Majority vote over days* |
| PCA+K-means | Column-standardized daily curves; 16 PCs, each standardized | Size-constrained K-means (≥ 10 patients per cluster)† | Patient-level |
| Multilevel Clustering | Patient means of daily indices, standardized and averaged within 3 published factor groups | K-means++, 100 initializations | Patient-level |
| GSP | 9 glycemic-pattern frequencies, column-wise min–max normalization | K-means++, 100 initializations | Patient-level |
| GRI | GRI-related features, column-standardized then patient-wise L2-normalized | 10 K-means fits (3 initializations each) → co-assignment similarity → average-linkage clustering | Patient-level |
| DTCR | Learned daily representations | Clustering in the learned space | Majority vote over days* |
| GNN | GNN daily representations | Clustering in the learned space | Majority vote over days* |
| Chronos | Foundation-model embeddings of days | Clustering in the embedding space | Majority vote over days* |
| CGMformer | Publicly released pretrained encoder (no fine-tuning); mean-pooled daily embeddings, averaged per patient | Feature-wise standardization; K-means++, 100 initializations | Patient-level |
| GluFormer | Official architecture trained from scratch (pretrained weights not released); max-pooled daily embeddings, averaged per patient | Feature-wise standardization; K-means++, 100 initializations | Patient-level |

\* **Majority vote and ties.** Patients with tied votes were first excluded; four class prototypes were built from the non-tied patients; each tied patient was then assigned to the prototype with the smallest mean distance over that patient's days (DTW distance for DTW, shape-based distance for K-shape, and mean squared Euclidean distance in the representation space otherwise).

† **PCA+K-means size constraint.** Unconstrained K-means produced degenerate partitions with clusters of fewer than ten patients. Daily K-means with voting was used for initialization, followed by patient-level distance optimization with a minimum of ten patients per cluster. The constraint uses only CGM-derived features and patient counts.

**Adaptations of the CGM-specific methods**

- **GSP**: the published rate threshold of 0.06 mmol/L/min was retained; 95 within-day transitions were computed per eligible day. Following the original publication, the pattern frequencies were normalized column-wise by min–max scaling.
- GSP and GRI were originally described for 5-minute data (288 readings per day); GSP, GRI and Multilevel Clustering were run on the common 15-minute grid.

**CGM representation models**

- Glucose was converted to mg/dL on the native 15-minute grid, and each day was encoded separately.
- **CGMformer**: the publicly released encoder was used without fine-tuning, with mean pooling over the daily tokens.
- **GluFormer**: pretrained weights have not been publicly released. The official architecture was therefore trained from scratch with its next-token prediction objective, with max pooling over the daily tokens.
- For both models, daily embeddings were averaged across each patient's days; no daily voting was used.

---

## License

MIT License. See [LICENSE](LICENSE).

Questions can be raised through GitHub Issues.
