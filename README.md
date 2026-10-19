# Explainable-Intrusion-Detection-using-Counterfactuals

Honours project, Rhodes University (2026) · **Munashe Madziwanyika** · Supervisor: Dr. Stones Dalitso Chindipha

This is the source code for my thesis on "Explainable Intrusion Detection using Counterfactuals". This project focused on using gradient-based counterfactual explanations to provide actionable insights when network attack alerts are raised by a deep learning classifier. A neural network (MLP) classifies SDN traffic from the **UNR-IDD** dataset into Normal plus five attack types. A **Wachter et al. (2017) counterfactual explainer** then shows, for each attack, the smallest change that would make the model call it Normal. Networking constraints are added to the search so the explanations describe traffic that could really exist, and the project measures what the constraints cost.

## Files
| File | Purpose |
|---|---|
| `read_UNR_data.ipynb` | Data exploration and preprocessing of `UNR-IDD.csv` |
| `project.ipynb` | Model training, baselines, counterfactual explainer and evaluation |

## Dataset
UNR-IDD (Das et al., 2023): 37,411 records of OpenFlow port statistics from a simulated network. Its six classes are Normal, TCP-SYN, Blackhole, Diversion, Overflow and PortScan, encoded 0–5. The dataset is not included in this repository.

## Method
- **Preprocessing:** constant and identifier columns are dropped, leaving 18 features, which are scaled with `StandardScaler`.
- **Grouped split:** records are grouped by switch–port pair with `StratifiedGroupKFold` so the model can't memorise connection points.
- **Classifier:** an MLP (18 → 128 → 64 → 32 → 16 → 6) trained with class-weighted cross-entropy and AdamW. Training stops early on validation macro F1.
- **Explainer:** minimises `CE(f(x'), Normal) + λ · MAD-L1(x, x')`, trying λ values from 100 down to 0.001.
- **Constraints (optional):** a penalty for breaking any of these rules:
  - port durations can't change
  - no negative values
  - counts are whole numbers
  - bytes stay within the 10 Mbps bandwidth
  - bytes per packet stays between 42 and 65,535
  - each delta counter is at most its cumulative counter
  - flow-table counts stay consistent

  A final sparsify pass then undoes any change the result doesn't need.

## Results
| Classifier (grouped split) | Macro F1 |
|---|---|
| MLP (this work) | **73.06%** |
| Random Forest | 83.29% |

| Explanations (300 attacks) | Unconstrained | Constrained |
|---|---|---|
| Validity | 1.00 | 0.80 |
| Sparsity (fraction of features changed) | 1.00 | 0.51 |
| Time per explanation | 2.33 s | 16.74 s (7.18× slower) |

With constraints, about half as many features change and the suggested traffic is physically plausible. The cost is lower validity and a much slower search.

## Running
```bash
pip install torch numpy pandas scikit-learn imbalanced-learn matplotlib seaborn jupyter
```
1. Put the UNR-IDD CSV in the project folder and run `read_UNR_data.ipynb`.
2. Run `project.ipynb`. Its input CSV must keep the `Switch ID`, `Port Number` and `Connection Point` columns, which the grouped split needs.

The full evaluation takes over an hour, and a GPU is optional.

## References
- Das, T. et al. (2023). *UNR-IDD: Intrusion Detection Dataset using Network Port Statistics.* IEEE CCNC.
- Wachter, S., Mittelstadt, B., & Russell, C. (2017). *Counterfactual Explanations without Opening the Black Box.* Harvard JOLT.
