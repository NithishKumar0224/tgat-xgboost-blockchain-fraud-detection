# Blockchain Fraud Detector — ATGAT Hybrid System
**Author:** Ragupathi Nithish Kumar  
**Version:** 2.0.0

## Overview
Production-grade hybrid fraud detection system combining:
- **ATGAT** (Augmented Temporal-aware Graph Attention Network) as feature extractor
- **XGBoost** as the final classifier on combined ATGAT embeddings + manual features
- Dual dataset support: Bitcoin (Elliptic) + Ethereum (Phishing)

## Paper Structure (as per supervisor Norbert Oláh)
1. **Baseline** — XGBoost on manual features only
2. **Innovation** — ATGAT with self-supervised pretraining + temporal modeling
3. **Comparison** — Hybrid (ATGAT+XGBoost) vs all baselines

## Architecture
```
Self-supervised Pretraining:
  Masked Node Reconstruction (BERT-style, 15% mask ratio)
  + Edge Existence Prediction (link prediction)
  → ATGAT learns rich representations from ALL nodes (incl. unlabeled)

Supervised Fine-tuning:
  ATGAT → temporal + structural embeddings per node

Hybrid Classification:
  [ATGAT embeddings | manual features] → XGBoost → fraud/licit
```

## Quick Start
```bash
pip install -r requirements.txt

# Full pipeline (both datasets)
python train.py

# Single dataset
python train.py --dataset bitcoin
python train.py --dataset ethereum

# Skip pretraining (faster, for testing)
python train.py --skip_pretrain

# Custom epochs
python train.py --epochs 200

# Launch dashboard + API
python serve.py
# Open: http://localhost:5000
```

## Dataset Setup
Place Elliptic files in `data/bitcoin/`:
- `elliptic_txs_classes.csv`
- `elliptic_txs_edgelist.csv`
- `elliptic_txs_features.csv` (optional — uses 4 structural features if absent)

Ethereum dataset: auto-generated synthetic data if not present.  
To use real data, place in `data/ethereum/ethereum_phishing.csv`.

## API Endpoints
| Endpoint | Description |
|---|---|
| `GET /api/results/{dataset}` | All model metrics |
| `GET /api/figures/{dataset}` | Available figures |
| `GET /api/compare` | Cross-dataset comparison |
| `POST /api/predict` | Risk score for a transaction |
| `GET /api/summary` | Dashboard summary stats |

## Key Files
```
blockchain_fraud_detector/
├── src/
│   ├── models/
│   │   ├── atgat.py            ← ATGAT architecture
│   │   ├── hybrid_pipeline.py  ← ATGAT+XGBoost hybrid
│   │   └── baselines.py        ← GCN, GAT, SAGE, GIN, LR, RF, XGB, MLP
│   ├── training/
│   │   ├── pretrain.py         ← Self-supervised pretraining
│   │   └── trainer.py          ← Fine-tuning + GNN training loops
│   ├── preprocessing/
│   │   ├── bitcoin_loader.py   ← Elliptic + 49 timesteps
│   │   └── ethereum_loader.py  ← Phishing + synthetic timestamps
│   └── evaluation/
│       └── metrics.py          ← All figures + tables
├── train.py                    ← Main pipeline
├── serve.py                    ← Dashboard server
└── configs/config.yaml         ← All hyperparameters
```
