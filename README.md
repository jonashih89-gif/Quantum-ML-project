# QML Project: Quantum Kernel SVMs on Iris

Final project for a Quantum Science and Technology course. It compares two kernel-based quantum support vector machines (QSVMs) built in Qiskit on the Iris dataset: one using the **ZZ feature map** and one using a **hardware-efficient (HE) feature map**.

## What It Does

- Encodes each 4-feature sample into a 4-qubit state and computes a fidelity kernel, K(x, x') = |⟨φ(x)|φ(x')⟩|², which is then used to train an SVM.
- Compares both QSVMs against a classical RBF-kernel SVM using repeated 5-fold cross-validation (50 folds).
- Studies how the feature-scaling range affects each kernel.
- Tests robustness under simulated depolarizing noise with shot sampling. The noise simulator is validated against Qiskit Aer.

## Key Results

| Model | CV accuracy (mean ± SD) |
| --- | --- |
| QSVM, ZZ map | 68.7% ± 6.7% |
| QSVM, HE map | 92.2% ± 4.1% |
| Classical RBF SVM | 96.0% ± 3.4% |

- The ZZ map's low accuracy comes from feature scaling. At [0, 2π], its kernel values concentrate near 1/16, the value expected for unrelated 4-qubit states. Scaling to [0, π/2] or less raises ZZ to 95–96%.
- The HE map uses half as many CX gates (6 vs. 12). At 10% CX error, it keeps 95.9% accuracy while ZZ drops to 91.3%.
- Neither quantum kernel beats the classical baseline on this dataset.

## How to Run

Open `QML project.ipynb` in Google Colab or Jupyter and run all cells. The first cell installs the required packages (`qiskit`, `qiskit-aer`, `qiskit-machine-learning`, `scikit-learn`). A full run takes about 6 minutes.
