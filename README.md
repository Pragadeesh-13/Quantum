## About The Project

BBB, or blood-brain barrier permeability, tells us whether a compound can cross into the brain or is likely to be blocked by the barrier. This matters in drug discovery because BBB behavior is a key factor when screening molecules for neurological treatments.

This project is useful because it explores a compact machine learning workflow for predicting the binary `BBB+/BBB-` class from molecular descriptors. In the notebook, the target label is `BBB+ = 1` and `BBB- = 0`.

Because quantum circuits are expensive to simulate as qubit count grows, the notebook first uses classical feature selection to keep only the most informative variables, then compresses them with PCA so they can be encoded on 4 qubits.

The goal is to show how classical preprocessing and quantum machine learning can be combined in a practical BBB classification workflow when simulator cost and hardware limits matter.

## What This Project Does

This notebook starts with a high-dimensional feature set, reduces it aggressively with classical methods, and then maps the final representation into a quantum kernel model.

The main idea is simple: quantum simulation gets expensive fast as qubit count grows, so instead of trying to send the full feature space into the circuit, we first:

1. Use L1 logistic regression to identify the most informative features.
2. Narrow that set down to the top 20 features by checking model accuracy.
3. Apply PCA again on those 20 features to compress them into 4 components.
4. Feed those 4 components into a 4-qubit `ZZFeatureMap` and train a QSVC.

## Why 4 Qubits

The quantum simulator and hardware both become much slower as the number of qubits increases. Since this project was built around a 4-qubit constraint, PCA was used as the final compression step to make the model practical while still preserving useful signal from the top features.

In the notebook, the 4 PCA components explain about **48.49%** of the variance.

## Pipeline

```mermaid
flowchart LR
   A[Original dataset] --> B[L1 logistic regression]
   B --> C[Top 20 features]
   C --> D[PCA to 4 components]
   D --> E[4-qubit ZZFeatureMap]
   E --> F[QSVC / quantum kernel]
   F --> G[Evaluation]
```

## Reported Results

The following metrics were read directly from the notebook outputs.

| Model | Setup | Accuracy | F1 Score | ROC AUC |
| --- | --- | ---: | ---: | ---: |
| Linear SVM | PCA 20 -> 4, 200/80 split | 76.67% | 83.72% | 0.818 |
| QSVC on simulator | 4-qubit quantum kernel | 73.33% | 81.82% | 0.751 |

## Feature Selection Summary

The notebook reports the following top 20 L1-selected features:

`MDEC-33`, `ATSC6se`, `VSA_EState7`, `GATS2p`, `CIC0`, `NssssN`, `nG12Ring`, `SaasC`, `C3SP2`, `PEOE_VSA6`, `AATS3p`, `Xch-5d`, `nAcid`, `Lipinski`, `PEOE_VSA9`, `GATS3p`, `JGI2`, `EState_VSA5`, `AATSC1i`, `NsOH`

These features were then compressed into 4 PCA components to match the qubit budget.

## Repository Contents

- `Quantum_Project.ipynb`: Main notebook with preprocessing, classical baseline, quantum kernel experiments, and hardware execution.
- `qsvc_feature_map_4_qubits.qasm`: Exported OpenQASM 2.0 circuit for the 4-qubit feature map.
- `README.md`: Project overview and results summary.

## Notes

- The IBM hardware result was produced on a very small batch because cloud quantum hardware access is constrained and queue time matters.
- The Qiskit notebook output shows a deprecation warning for `ZZFeatureMap`; the circuit still runs, but a future update should switch to the newer `zz_feature_map` API.
- Because this is a quantum workflow, results can vary depending on backend noise, sampling, and the exact train/test split.

## Reproducing The Workflow

To rerun the project, open `Quantum_Project.ipynb` and execute the cells in order:

1. Load and preprocess the dataset.
2. Run feature selection with L1 logistic regression.
3. Reduce the selected features to 4 dimensions using PCA.
4. Build the 4-qubit feature map.
5. Train and evaluate the classical and quantum models.
