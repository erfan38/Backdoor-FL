# TASAD: Temporal Adversarial Suppression via Anomaly Detection against Stealthy Backdoor Attacks in Federated Learning

This repository contains the code for our paper accepted at **FPS 2026** (Foundations and Practice of Security).

**Paper:** arXiv / proceedings link: comming soon.

> **Abstract.** Federated learning (FL) enables a central server and several clients to collaboratively train a global model, but is vulnerable to backdoor attacks where malicious clients poison their local updates. Stealthy variants of these attacks operate more covertly across training rounds, evading defenses that inspect only single-round behavior. We propose TASAD (Temporal Adversarial Suppression via Anomaly Detection), a robust aggregation defense that exploits the behavioral oscillation inherent to periodic stealthy attackers. TASAD computes the coefficient of variation (CV) of each client's detection-layer update norms over time and uses a Median+MAD threshold to detect anomalous temporal behavior, requiring no validation data, attacker-specific signatures, or assumptions about the underlying data distribution. We evaluate TASAD on MNIST, CIFAR-10, and UNSW-NB15 against FedAvg, Krum, FoolsGold, and FLAME under varying heterogeneity, poison rate, and stealth activation probability.

## Results

Backdoor accuracy (BA, lower is better) and main-task accuracy (MA) at the primary evaluation setting (α=0.3, 20% attackers, 25% poison rate, stealth activation p=0.85):

| Dataset | Metric | FedAvg | Krum | FoolsGold | FLAME | **TASAD (Ours)** |
|---|---|---|---|---|---|---|
| MNIST | MA / BA | 98.7 / 74.8 | 98.5 / 0.2 | 98.8 / 88.6 | 98.1 / 0.2 | **98.0 / 0.2** |
| CIFAR-10 | MA / BA | 88.2 / 80.4 | 87.9 / 10.6 | 86.8 / 79.0 | 81.1 / 0.5 | **84.5 / 0.4** |
| UNSW-NB15 | MA / BA | 49.6 / 75.9 | 47.2 / 44.4 | 44.7 / 55.8 | 44.2 / 42.5 | **50.5 / 21.0** |

TASAD achieves near-zero backdoor accuracy on image classification datasets (MNIST, CIFAR-10) while maintaining main-task accuracy comparable to undefended FedAvg. On UNSW-NB15, TASAD substantially reduces backdoor accuracy relative to FedAvg, though — like all evaluated defenses — it does not fully suppress the attack; see Limitations below and Section 5 of the paper for the full operating envelope. Full results across all heterogeneity, poison-rate, and stealth-activation sweeps are in Tables 4–7 of the paper.

## Limitations

(Summarized from Section 5.2 of the paper — see the paper for full discussion.)

- **Cold-start inversion under extreme heterogeneity.** On CIFAR-10 with ResNet-18 at α < 0.3, benign clients' update norms can swing widely enough to cross the cold-start threshold, so the detector may flag them early — and because suspicion is persistent (the Temporal Ratchet), this early mistake can continue to shape aggregation decisions later in training.
- **Near-IID convergence on deep architectures.** As the global model converges (CIFAR-10, α ≥ 0.5), update norms shrink across all clients, and the CV signal becomes harder to estimate reliably — benign and malicious CV distributions increasingly overlap.
- **Scalability.** As the number of participating clients increases, the contrast the CV signal provides can be diluted across a larger population.
- **UNSW-NB15 / tabular FL-IDS remains an open challenge.** All evaluated defenses (not just TASAD) degrade substantially on UNSW-NB15 relative to the image datasets — see Table 6 and Section 4.3.

## What's in this repo

A single Jupyter notebook (`TASAD.ipynb`) containing the full pipeline:

- Federated learning simulation (client partitioning via Dirichlet distribution, local training, FedAvg-style rounds)
- Stealthy periodic backdoor attack implementation (Section 3.1 of the paper)
- TASAD detector (coefficient-of-variation + Median+MAD + Temporal Ratchet — Section 3.2, Algorithm 1)
- Baseline defenses: FedAvg (no defense), Krum, FoolsGold, FLAME
- Evaluation (main-task accuracy, backdoor accuracy, detection precision/recall/F1, computational overhead)
- A detection-layer selection diagnostic (referenced in Section 3.2's footnote)

Reproduces Tables 4–8 of the paper across three datasets: **MNIST**, **CIFAR-10**, and **UNSW-NB15**.

## Setup

### Requirements

```
torch
torchvision
numpy
pandas
scikit-learn
hdbscan
matplotlib
psutil
```

Install with:
```bash
pip install torch torchvision numpy pandas scikit-learn hdbscan matplotlib psutil
```

MNIST and CIFAR-10 download automatically via `torchvision` on first run.

### UNSW-NB15

UNSW-NB15 is **not included** in this repository (large file size, licensing). To run the UNSW-NB15 experiments:

1. Download `UNSW_NB15_training-set.csv` and `UNSW_NB15_testing-set.csv` — e.g. from [Kaggle](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15) or the [official UNSW research page](https://research.unsw.edu.au/projects/unsw-nb15-dataset).
2. Place both files in `data/`:
   ```
   data/UNSW_NB15_training-set.csv
   data/UNSW_NB15_testing-set.csv
   ```

## Running the experiments

Open `TASAD_camera_ready.ipynb` in Jupyter or Google Colab and run all cells top to bottom.

- The `DATASET` variable near the top of the configuration cell selects which dataset runs (`"mnist"`, `"cifar10"`, or `"unsw"`). The notebook is run once per dataset — set `DATASET` and re-run the full notebook for each.
- `METHODS` controls which defenses are evaluated in a given run (FedAvg, Krum, FoolsGold, FLAME, and TASAD/"Ours").
- Results, checkpoints, and figures are written to `results/`.

**Runtime:** MNIST: ~7 min per run. CIFAR-10: ~45 min per run. UNSW-NB15: ~4 min per run. Hardware: NVIDIA GeForce RTX 5080 (17.1 GB VRAM).

## Repository structure

```
.
├── TASAD.ipynb   # full pipeline: data, attack, defense, evaluation
├── data/                      # place UNSW-NB15 CSVs here (not tracked in git)
├── results/                   # experiment outputs (not tracked in git)
├── requirements.txt
├── .gitignore
└── README.md
```

## Citation

If you use this code, please cite:

```bibtex
@inproceedings{tasad2026,
  title     = {TASAD: Temporal Adversarial Suppression via Anomaly Detection against Stealthy Backdoor Attacks in Federated Learning},
  author    = {TODO: add author list},
  booktitle = {Foundations and Practice of Security (FPS)},
  year      = {2026}
}
```
*(Citation details will be finalized once the proceedings version is published — check back or open an issue if you need it sooner.)*

## License

MIT (see `LICENSE`) — can be revisited later if needed.
