# Quantum Kernel SVMs on Iris

Final project for my Quantum Science and Technology course (Jan–Jun 2026).

I compared two quantum kernels for SVM classification on the Iris dataset, built in Qiskit. One uses the ZZ feature map and the other uses a hardware-efficient feature map (EfficientSU2 with the rotation angles tied to the input features). Each of the 4 features goes on one qubit, and the kernel is the fidelity between the encoded states.

## The question

My first version used a single 80/20 split, and the result looked one-sided: the hardware-efficient map scored 96.7% and ZZ only 76.7%.

That made me wonder whether ZZ is really a worse feature map, or whether it was just poorly configured in my setup. To answer that, I needed a fair comparison where every model gets the same chance.

## Building a fair comparison

I set up the comparison so that no model had an advantage:

- Every model is trained and tested on the same 50 splits (5-fold CV, repeated 10 times), instead of one lucky or unlucky split.
- Each model gets its own settings tuned with nested cross-validation. The feature-scaling range and C are picked using training data only, so the test data never influences the choice.
- A classical RBF-kernel SVM, tuned the same way, serves as a reference point.
- Differences are checked with a corrected resampled t-test, which accounts for the overlap between CV splits.

## What I found

| Model | Original settings ([0, 2π], C = 1) | Tuned (nested CV) |
| --- | --- | --- |
| QSVM, ZZ map | 68.7% ± 6.7% | 96.4% ± 3.3% |
| QSVM, hardware-efficient map | 92.2% ± 4.1% | 95.1% ± 3.8% |
| Classical RBF SVM | 96.0% ± 3.4% | 95.8% ± 3.2% |

ZZ was not a worse feature map. It was hurt by the feature-scaling range. With features scaled to [0, 2π], ZZ becomes too sensitive to small differences: two almost identical flowers that differ by 0.1 cm get a similarity of only 0.53. Most kernel values end up near 1/16, which is what you get for two unrelated 4-qubit states, so the SVM has almost nothing to learn from.

Once the range is tuned, ZZ goes from 68.7% to 96.4%, and the three models end up within about one point of each other. None of the differences are significant (p > 0.3).

## Do the tuned settings survive real noise?

Nested CV kept choosing very small ranges like [0, π/32]. At those ranges almost every kernel value is close to 1, and the model relies on tiny differences between them. That made me suspect the "best" settings might not hold up on a real device.

To check, I ran the kernel circuits under the noise model of IBM's 133-qubit Torino processor (`FakeTorino` in Qiskit), on the 4-qubit chain with the lowest CZ and readout errors.

- At [0, π/2], neither map lost any accuracy.
- At [0, π/32], the hardware-efficient map dropped from 95.0% to 86.0% (p = 0.004), and ZZ dropped 2.7 points.

So the settings that work best in a noiseless simulation are not necessarily the best on hardware. The feature range should be tuned under the noise you actually expect.

## Running it

Open `QML project.ipynb` in Colab or Jupyter and run all cells. The first cell installs `qiskit`, `qiskit-aer`, `qiskit-machine-learning`, `qiskit-ibm-runtime`, and `scikit-learn`. A full run takes around 15 minutes, mostly in the device-noise section. Seeds are fixed, so the numbers should match.

Everything here is simulated. Iris is also small and close to linearly separable, so this project says nothing about quantum advantage. It is about checking whether a result holds up under a fair comparison and realistic noise.
