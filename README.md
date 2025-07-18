# Aave V2 On-Chain Credit Scoring Model
 # Important - Run only first block of code if you want a csv file contaning credit scores of each wallet and if graphs and summary is needed run rest of the code blocks.
 # File named wallet_score.csv contains output of first code block.

## 1. Overview

This project provides a robust machine learning model that assigns a **credit score between 0 and 1000** to Aave V2 wallets. The score is derived exclusively from a wallet's historical on-chain transaction behavior.

-   **Higher Scores (e.g., 750-1000):** Indicate responsible, reliable wallets that provide liquidity and consistently repay debts.
-   **Lower Scores (e.g., 0-400):** Reflect risky behavior, such as being liquidated, high leverage, or potentially bot-like activity.

The model is designed to be transparent, auditable, and easily extensible.

## 2. Scoring Philosophy

The credit score is built on a simple, powerful premise: **reward responsible financial behavior and penalize risky actions.**  we use on-chain data as a definitive record of behavior.

The model analyzes transactions (`deposit`, `borrow`, `repay`, `redeem`, `liquidation`) to build a holistic profile of each wallet, focusing on risk management, reliability, and overall contribution to the protocol's health.

## 3. Feature Engineering

The raw transaction data is aggregated per wallet to engineer six key features that form the basis of the credit score.

#### Positive Factors (Rewarded)
* **Wallet Lifespan:** The time between a wallet's first and last transaction. Longer lifespans indicate commitment and are viewed favorably.
* **Repayment-to-Borrow Ratio:** The total USD value of repayments divided by borrows. A ratio is a strong signal of a user who responsibly manages their debt.
* **Net Liquidity Provided:** Total deposits minus total withdrawals (redeems). Wallets that are net liquidity providers are vital to the Aave ecosystem and receive a higher score.
* **Transaction Count:** The total number of interactions. Sustained engagement is a positive indicator.

#### Negative Factors (Penalized)
* **Liquidation Count:** The number of times a wallet's collateral was seized to cover its debt. **This is the single most significant penalty in the model**, as it represents a definitive failure in risk management.
* **Borrow Utilization:** The ratio of total borrows to total deposits. A higher ratio signifies greater leverage and increased risk of liquidation, thus lowering the score.

## 4. The Scoring Algorithm

The score is calculated in a transparent, four-step process:

1.  **Feature Calculation:** Raw feature values (e.g., total USD deposited, number of liquidations) are calculated for each unique wallet address.

2.  **Normalization:** To compare features on a level playing field, all feature values are scaled to a common range of 0 to 1 using Min-Max scaling. For negative factors (like liquidations), the score is inverted so that for all features, a higher normalized value is always better.

3.  **Weighted Sum:** A `Raw Score` is calculated by multiplying each normalized feature by its assigned weight and summing the results. The weights reflect the feature's importance in assessing creditworthiness.

    | Feature | Weight |
    | :--- | :---: |
    | Liquidation Count | 40% |
    | Repayment-to-Borrow Ratio | 25% |
    | Borrow Utilization | 15% |
    | Net Liquidity Provided | 10% |
    | Wallet Lifespan & Tx Count | 10% |

    _Formula: `Raw Score = 0.40 * F_liq + 0.25 * F_repay + ...`_

4.  **Final Scaling:** The `Raw Score` is scaled proportionally to produce the final **Credit Score** on a scale from 0 to 1000.

## 5. How to Run the Script

The model can be run from the command line with a single command. Ensure you have Python, Pandas, and NumPy installed (`pip install pandas numpy`).

```bash
python score_wallets.py --input /path/to/your/transactions.json --output /path/to/scores.csv
