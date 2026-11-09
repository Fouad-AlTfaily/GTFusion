# GTFusion: Graph-Temporal Learning for IoT Intrusion Detection

This repository contains the code of the paper:

> **Graph-Temporal Learning for IoT Intrusion Detection in Centralized and Federated Settings via Adaptive Fusion Gating**
> Fouad Al Tfaily, Zakariya Ghalmane, Mohamed el Amine Brahmia, Hussein Hazimeh, Ali Jaber, Mourad Zghal.
> Submitted to *Cluster Computing*.

GTFusion is a dual-branch intrusion detection model for IoT networks. A graph neural network (GAT or GraphSAGE) learns from the graph of communicating devices, and a bidirectional LSTM learns from the features of each flow. The two branches run in parallel, and an adaptive gate assigns each flow its own weights for the structural and temporal embeddings before classification. In federated training, the two branches and the classifier are shared, while the gate and the batch normalization layers stay local on each client.

## Overview of the method

| Component | Description |
|---|---|
| Graph | Built from training flows only. Nodes are IP addresses, edges are observed communications, and each address is also linked to the next four addresses of its /24 subnet. Unseen test addresses map to a padding node (inductive setting). |
| Node features | Eight traffic statistics per device plus degree, betweenness and PageRank centrality, rescaled as log(1 + C(v)·\|V\|) and standardized. |
| GNN branch | Three GAT layers (8, 8 and 4 heads) or three GraphSAGE layers, 128 hidden units. A flow is represented by the embeddings of its source and destination devices (256 dimensions). |
| Temporal branch | Two-layer bidirectional LSTM, 128 units per direction (256 dimensions). |
| Adaptive gate | MLP 512 → 256 → 128 → 2 with softmax, giving the weights w_gat and w_lstm of the two embeddings for each flow. |
| Classifier | MLP 512 → 512 → 256 → 128 → C, trained with a class-weighted cross-entropy loss. |
| Federated training | Five non-IID clients (Dirichlet α = 0.5), 30 rounds of one local epoch, FedAvg with FedProx (μ = 0.01), local batch normalization (FedBN) and a local gate per client. |

## Repository structure

```
GTFusion/
├── README.md
├── LICENSE
├── requirements.txt
├── data/
│   └── README.md                       # dataset sources, expected files and preparation
└── notebooks/
    ├── GTFusion_NF-ToN-IoT.ipynb       # all NF-ToN-IoT experiments, including the fusion variants (Table 7)
    ├── GTFusion_CIC-IDS-2017.ipynb     # all CIC-IDS-2017 experiments
    └── GTFusion_CIC-ToN-IoT.ipynb      # all CIC-ToN-IoT experiments
```

Each notebook is self-contained. It defines the data pipeline, every compared model, centralized and federated training, the statistical tests and the report, and it runs every experiment reported for its dataset.

## Installation

Python 3.10 or later is recommended. Install PyTorch for your platform first (see [pytorch.org](https://pytorch.org/get-started/locally/)), then the remaining packages:

```bash
git clone https://github.com/Fouad-AlTfaily/GTFusion.git
cd GTFusion
pip install -r requirements.txt
```

A CUDA GPU is strongly recommended. The full runs on CIC-ToN-IoT and CIC-IDS-2017 take several hours per model and seed on a single GPU.

## Datasets

The experiments use three public datasets: NF-ToN-IoT, CIC-IDS-2017 and CIC-ToN-IoT. The datasets are not redistributed here. See [`data/README.md`](data/README.md) for the download links, the expected file names and the preparation steps.

## Running the experiments

1. Place the dataset files in `data/` as described in `data/README.md`.
2. Open the notebook of the dataset and run all cells from top to bottom.
3. Results are written to `results/<dataset>_FULL/` at the root of the repository. Every finished run is saved as a JSON file with its metrics, per-class scores, confusion matrix and training history, together with its test predictions, and is skipped when the notebook is run again.

Two modes are available at the top of each notebook:

| Mode | Data | Seeds | Epochs | Federated rounds | Purpose |
|---|---|---|---|---|---|
| `FULL` | full dataset | 42, 43, 44 | 20 | 30 × 1 local epoch | reproduces the paper |
| `FAST` | 20% stratified sample | 7 | 10 | 5 × 1 local epoch | checks that every part runs |

## Experimental protocol

- Stratified 80/20 train/test split for every seed, without a validation set.
- Every model trains for a fixed number of epochs (or rounds), and the final model is evaluated. Test scores are logged per epoch only to measure the time to accuracy, never to select a model.
- AdamW, batch size 2,048, learning rate 2 × 10⁻³, weight decay 10⁻⁴ and dropout 0.3 for every model.
- Metrics: accuracy, balanced accuracy, weighted and macro F1-score, weighted precision, FPR and FNR.
- Significance: McNemar test on the predictions of two models for the same test flows of each seed, and paired t-test on the accuracies of the three seeds.

## Compared models

| Model | Description |
|---|---|
| LSTM | Flow features only. |
| GAT, GraphSAGE | Four graph layers, graph only. |
| GAT-3L, GraphSAGE-3L | Same depth and heads as the GNN branch of GTFusion. |
| FN-GNN | Local feature path combined with a graph neighbourhood path. |
| Sequential GAT-LSTM | GAT embeddings passed to an LSTM (spatial then temporal). |
| N-STGAT | LSTM encoding of the recent flows of each node followed by graph attention (temporal then spatial). |
| FedGATSage | GAT on the device graph combined with GraphSAGE on a graph of Louvain communities. |
| GTFusion-GAT, GTFusion-SAGE | The proposed model with a GAT or GraphSAGE branch. |

On CIC-IDS-2017, the classifiers of GAT, GraphSAGE, FN-GNN and Sequential GAT-LSTM also receive the features of the flow (models with the suffix `+F` in the notebook), and the batch normalization statistics of the GAT classifier are recomputed on the training flows before evaluation. N-STGAT is evaluated on NF-ToN-IoT and CIC-IDS-2017.

## Where each result of the paper comes from

| Paper | Notebook section |
|---|---|
| Tables 2 and 3 (centralized and federated results) | Report, sections 1 and 8 |
| Table 4 (per-class F1-scores) | Report, section 7 |
| Table 5 and Figure 5 (effect of the gate per class, graph weights) | Report, section 6 |
| Table 6 (component ablation) | Report, section 3 |
| Table 7 (fusion module variants) | NF-ToN-IoT notebook, last section |
| Table 8 (cost and effect of centrality) | Report, section 4 |
| Tables 9 and 10 (training cost, time to accuracy) | Report, sections 5, 11 and 12 |
| Figure 6 (personalized gates per client) | Report, section 9 |
| Tables A1 and A2 (significance tests) | Report, sections 2 and 8 |

## Citation

If you use this code, please cite the paper:

```bibtex
@article{altfaily2026gtfusion,
  title   = {Graph-Temporal Learning for IoT Intrusion Detection in Centralized and Federated Settings via Adaptive Fusion Gating},
  author  = {Al Tfaily, Fouad and Ghalmane, Zakariya and Brahmia, Mohamed el Amine and Hazimeh, Hussein and Jaber, Ali and Zghal, Mourad},
  journal = {Cluster Computing},
  year    = {2026},
  note    = {Under review}
}
```

## License

This project is released under the MIT License. See [`LICENSE`](LICENSE).
