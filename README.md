# Content Recommendation System
Real-time content recommendation engine built on top of a partitioned PostgreSQL database. 
User preference profiles are updated incrementally from a stream of interaction events, and 
top-3 category recommendations are generated using SQL-native batch queries with temporal decay.

The system simulates a social media platform where 4,000 users interact with 15 content 
categories over 30 days (60,000 events across 15 ingestion rounds). The core idea is that 
a user's future interest can be approximated from their recent interaction history, with 
older interactions down-weighted by an exponential decay.

All computation happens **inside the database**, no Python loops, no data movement.
The entire pipeline (insert → score update → top-K retrieval) runs as a single batch SQL query.

## Key Features
- **Temporal decay scoring**: exponential weighting with 7-day half-life
- **SQL-native batch pipeline**: window functions for per-user normalization
- **Time-partitioned schema**: weekly partitions aligned with the decay half-life
- **Cold-start fallback**: demographic similarity or global popularity
- **Streaming simulation**: 15 ingestion rounds, 4,000 events per round

## Results

| Metric | Value |
|--------|-------|
| Hit@3 (strict) — round 1 | 66.4% |
| Hit@3 (strict) — round 7 | 100% |
| Throughput (round 1) | 940 interactions/s |
| Throughput (round 15) | 202 interactions/s |
| Slowdown (15× more data) | 4.64 (sub-linear) |
| Partition pruning speedup | 7.8 (single partition vs. full scan) |

## Setup
Tested on Google Colab with PostgreSQL 15.

```bash
# Install PostgreSQL (Colab)
apt-get install -y postgresql postgresql-contrib
service postgresql start

# Python dependencies
pip install psycopg2-binary sqlalchemy pandas numpy matplotlib
```

Then run the notebook in order.

## Dataset
Based on the [Social Media Ad Engagement Dataset](https://www.kaggle.com/datasets/ziya07/social-media-ad-engagement-dataset) (Kaggle, 2025), 
synthetically extended from 5 to 15 content categories with temporal stratification.

Available here: https://www.kaggle.com/datasets/aaidoudi/user-social-network-interaction-temporal

## Author
Aaron Aidoudi - M1 Distributed Artificial Intelligence, Université Paris Cité

Supervised by Mr. Themis Palpanas
