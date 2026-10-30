# Smart Auto-Scaling System: Predictive Cloud Resource Optimization

[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![AWS](https://img.shields.io/badge/AWS-Lambda%20%7C%20CloudWatch%20%7C%20EC2-FF9900?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![PySR](https://img.shields.io/badge/ML-Symbolic%20Regression%20(PySR)-brightgreen?style=flat)](https://github.com/MilesCranmer/PySR)
[![Dask](https://img.shields.io/badge/Data-Dask%20%7C%20Pandas-orange?style=flat&logo=dask&logoColor=white)](https://dask.org/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

An intelligent, data-driven auto-scaling engine deployed on AWS that replaces traditional reactive, threshold-based scaling with transparent, closed-form predictive models derived via **Symbolic Regression (PySR)**.

---

## 📌 Executive Summary

Traditional cloud auto-scalers react *after* resource utilization thresholds are crossed, resulting in SLA degradation during sudden traffic surges and excessive expenditure during idle periods. 

This project implements a proactive **MAPE-K control loop** in **AWS Lambda** that predicts demand patterns **5–15 minutes in advance**. By utilizing closed-form mathematical equations discovered from **1M+ historical CloudWatch records**, it achieves **sub-90ms inference** without the cold-start and black-box overhead of deep neural networks.

---

## 📊 Benchmark Results (30-Day Production Evaluation)

Tested against standard AWS CloudWatch Reactive Auto-Scaling across 3 Availability Zones:

| Metric | AWS Reactive Baseline | Smart Auto-Scaling | Impact |
| :--- | :---: | :---: | :---: |
| **Monthly Infrastructure Cost** | \$9,840 | **\$7,350** | **-25.3% (\$2,490/mo saved)** |
| **Peak Latency (P99)** | 450 ms | **180 ms** | **-60.0% faster** |
| **Thrashing Frequency** | 8.1 actions/hr | **2.3 actions/hr** | **-71.6% fewer oscillations** |
| **Average Instance Utilization** | 62.0% | **78.0%** | **+25.8% resource efficiency** |
| **SLA Compliance Rate** | 97.2% | **99.7%** | **+2.5% reliability gain** |
| **System Uptime** | 99.8% | **99.95%** | Exceeds 99.9% target |
| **Inference / Scaling Latency** | N/A (reactive) | **87 ms / 73 s** | Full scale-out under 90s SLA |

---

## 🏗️ Architecture & Control Flow

The architecture is structured around an autonomous **MAPE-K** (Monitor, Analyze, Plan, Execute, Knowledge) control loop:

```mermaid
flowchart TD
    subgraph Data Layer
        CW[AWS CloudWatch Telemetry] -->|1M+ Historical Metrics| S3[AWS S3 Parquet Lake]
        S3 --> Dask[Dask Distributed Preprocessing]
    end

    subgraph Intelligence Discovery
        Dask -->|50 Engineered Features| PySR[Symbolic Regression Discovery]
        PySR -->|Closed-Form Equation| ModelStore[Model Registry / JSON]
    end

    subgraph Production Runtime (MAPE-K)
        LiveCW[CloudWatch Live Metrics] -->|Every 5-10s| Lambda[AWS Lambda Engine]
        ModelStore -->|Cached Formula| Lambda
        Lambda -->|Calculate Capacity| Guardrails{Hysteresis & Cooldown}
        Guardrails -->|Suppressed Spikes| Skip[Hold Current State]
        Guardrails -->|Legitimate Delta| ASG[AWS Auto Scaling Group API]
        ASG -->|Scale In / Scale Out| EC2[EC2 Instance Fleet (3 AZs)]
        Lambda -->|Decision Audit Trail| RDS[(PostgreSQL Audit Log)]
        RDS --> Grafana[Grafana Observability Dashboard]
    end
```

---

## 🧮 Discovered Mathematical Model

Rather than deploying an opaque neural network with high serverless inference costs, the training pipeline ran across 150 generations using PySR to extract an interpretable mathematical formula:

$$\text{instances} = \max\left(2, \min\left(10, \left\lceil 1.8 + 0.42 \cdot \text{CPU}_{\text{P99}} + 0.31 \cdot \log(\text{Memory}) + 0.15 \cdot \frac{\text{Net}_{\text{P95}}}{1000} \right\rceil\right)\right)$$

- **Model Performance:** $R^2 = 0.89$ (Train) / $0.86$ (Validation)
- **Error Rates:** $\text{MAPE} = 9.1\%$, $\text{MAE} = 1.4$ instances
- **Model Complexity:** 5 operators, 7 terms (100% human-auditable)

---

## 🛡️ Anti-Thrashing Safeguards

To eliminate erratic scaling oscillations and unpredictable cloud billing, two safety layers govern all execution decisions:
1. **Hysteresis Smoothing:** Uses a rolling 30-second moving average window to filter out transient metric spikes.
2. **Cooldown Guardrails:** Enforces a strict 5-minute cooldown timer between consecutive scaling events to allow new instances to warm up and stabilize.

---

## 📁 Repository Structure

```text
├── src/
│   ├── lambda/
│   │   ├── lambda_function.py      # Core MAPE-K serverless decision handler
│   │   ├── safeguards.py           # Hysteresis and cooldown enforcement logic
│   │   └── audit_logger.py         # PostgreSQL transaction logging
│   ├── training/
│   │   ├── data_pipeline.py        # Dask data ingestion & cleaning
│   │   ├── feature_engineering.py  # 50+ rolling window & temporal features
│   │   └── train_pysr.py           # Symbolic regression search pipeline
│   └── config/
│       └── config.yaml             # Thresholds, ASG names, and parameters
├── tests/
│   ├── test_lambda.py              # Unit tests for formula & decision paths
│   └── test_safeguards.py          # Hysteresis & cooldown edge-case tests
├── terraform/                      # Infrastructure as Code (Lambda, ASG, IAM)
├── docs/                           # Architecture specifications & project report
├── requirements.txt
└── README.md
```

---

## 🚀 Quickstart & Local Setup

### 1. Prerequisites
- Python 3.9+
- AWS CLI configured with appropriate permissions (`autoscaling:*`, `cloudwatch:*`)
- Docker (optional, for local testing)

### 2. Clone & Install Dependencies
```bash
git clone https://github.com/yashavanthap55/Smart-Autoscaling-report.git
cd Smart-Autoscaling-report
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Run Unit & Integration Tests
```bash
pytest tests/ -v
```

### 4. Deploy Serverless Decision Engine
```bash
cd terraform
terraform init
terraform apply
```

---

## 👥 Contributors & Acknowledgments

- **Yashavantha P** ([GitHub](https://github.com/yashavanthap55) • [LinkedIn](https://linkedin.com/in/yashavantha-p-a66607296))
- **Arthik Jain**
- **Harshavardhan D Naik**

*Project completed under the guidance of **Prof. Narmada T**, School of Computer Science & Engineering, RV University, Bengaluru (2025–2026).*
