# Wallet Risk Scoring for Compound Protocol

## Overview
This project calculates risk scores (0-1000) for Ethereum wallets based on their interaction history with Compound V2. Scores are derived from key risk indicators in borrowing activities, with higher scores indicating higher risk profiles.

## Methodology

### Data Collection
- Fetched transaction logs from Etherscan API
- Focused on 16 major Compound V2 cToken contracts
- Tracked three critical events:
  - `Borrow`: Wallet takes out a loan
  - `RepayBorrow`: Wallet repays loan
  - `LiquidateBorrow`: Wallet gets liquidated

### Feature Selection
1. **Number of Borrows (`num_borrows`)**  
   Measures leverage frequency → Higher = More risk

2. **Number of Repays (`num_repays`)**  
   Indicates debt management → Lower repay frequency = Higher risk

3. **Borrow/Repay Ratio (`borrow_repay_ratio`)**  
   `num_borrows/(num_repays + 1)` → Higher ratio = Riskier behavior

4. **Number of Liquidations (`num_liquidations`)**  
   Direct failure indicator → Any liquidation = High risk

### Scoring Logic
```python
L = min(1, num_liquidations / 5.0)          # 50% weight
B = min(1, num_borrows / 100.0)             # 30% weight
R = min(1, (borrows/(repays+1)) / 10.0)     # 20% weight

risk_index = 0.5*L + 0.3*B + 0.2*R
risk_score = int(risk_index * 1000)
```

### Risk Indicator Justification
| Indicator | Weight | Rationale |
|-----------|--------|-----------|
| Liquidations | 50% | Direct evidence of collateral failure - strongest risk signal |
| Borrow Frequency | 30% | Frequent borrowing indicates leverage dependency |
| Borrow/Repay Ratio | 20% | Imbalance suggests poor debt management |

## Setup Instructions

### Prerequisites
- Python 3.8+
- Etherscan API key (free tier sufficient)

### Installation
```bash
pip install pandas requests
```

### Configuration
1. Get Etherscan API key at: https://etherscan.io/register
2. Replace `YOUR_ETHERSCAN_API_KEY` in the script

### Execution
```bash
python risk_scoring.py
```

## Output
CSV file (`risk_scores.csv`) with columns:
- `wallet_id`: Ethereum address
- `score`: Risk score (0-1000)

Sample output:
```
wallet_id,score
0xfaa0768bde6298067,732
0x39c3a4620656c5d26f4,120
...
```

## Performance Notes
- Processes ~3 wallets/second (Etherscan rate limits)
- 100 wallets take approximately 5 minutes
- Invalid wallets are skipped automatically

## Limitations & Future Improvements
- Currently only analyzes Compound V2
- Does not consider transaction amounts or timestamps
- Future versions could incorporate:
  - Compound V3 support
  - Collateralization ratios
  - Time-based decay of risk events
  - Machine learning approach with more data

## Files
- `risk_scoring.py`: Main scoring script
- `risk_scores.csv`: Output file with scores
- `cToken_addresses.txt`: List of Compound V2 contracts analyzed

> **Note:** The Google Sheets wallet list is automatically fetched during execution
