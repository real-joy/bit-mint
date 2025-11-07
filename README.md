# BitMint Protocol

## Turn Idle Bitcoin into Productive Capital

**BitMint** is a Bitcoin-collateralized lending protocol built on the **Stacks** blockchain. It enables Bitcoin holders to mint synthetic stablecoins by locking BTC-backed assets as collateral — unlocking liquidity without giving up Bitcoin exposure or custody.

---

## 🧠 System Overview

**BitMint** transforms Bitcoin holders into active DeFi participants by leveraging **Stacks’ native Bitcoin finality** and Clarity’s transparent smart contract logic.

Users can:

* **Lock BTC** as collateral (trustlessly tracked via Stacks)
* **Mint synthetic stablecoins** or borrow against the BTC value
* **Repay loans** to reclaim collateral
* Remain fully exposed to **BTC’s upside** throughout

The protocol maintains **overcollateralization guarantees**, automated liquidation mechanisms, and adjustable risk parameters controlled by the contract owner.

---

## ⚙️ Contract Architecture

| Component            | Description                                                                            |
| -------------------- | -------------------------------------------------------------------------------------- |
| **`CONTRACT-OWNER`** | Primary admin authorized to configure protocol parameters.                             |
| **`VALID-ASSETS`**   | Supported collateral assets (`"BTC"`, `"STX"`).                                        |
| **Error Codes**      | Structured `err uXXX` codes for precise on-chain debugging.                            |
| **State Variables**  | Tracks global protocol parameters (collateral ratio, liquidation threshold, fee rate). |
| **Data Maps**        | Persistent storage for loans, user positions, and price oracles.                       |

### Core Modules

1. **Administrative Controls**

   * `initialize-platform`: One-time setup to activate the protocol.
   * `update-collateral-ratio`, `update-liquidation-threshold`, `update-price-feed`: Owner-only functions for managing risk parameters and oracle data.

2. **Core Lending Operations**

   * `deposit-collateral`: Lock BTC or supported assets as collateral.
   * `request-loan`: Create a new overcollateralized loan position.
   * `repay-loan`: Fully settle a loan to unlock collateral.

3. **Risk & Liquidation**

   * Collateral ratios dynamically evaluated via the on-chain price feed.
   * Automatic liquidation if ratio falls below `liquidation-threshold`.

4. **Read-Only Accessors**

   * `get-loan-details`, `get-user-loans`, `get-platform-stats`: Publicly query protocol state and metrics.

---

## 🧩 Data Model

### Loan Structure

Each loan is stored in the `loans` map:

| Field                | Type           | Description                                               |
| -------------------- | -------------- | --------------------------------------------------------- |
| `borrower`           | `principal`    | Loan initiator.                                           |
| `collateral-amount`  | `uint`         | Amount of BTC or asset locked.                            |
| `loan-amount`        | `uint`         | Amount of synthetic stablecoin minted.                    |
| `interest-rate`      | `uint`         | Fixed rate applied per block.                             |
| `start-height`       | `uint`         | Block height when loan was issued.                        |
| `last-interest-calc` | `uint`         | Used for incremental interest computation.                |
| `status`             | `string-ascii` | Current state: `"active"`, `"repaid"`, or `"liquidated"`. |

---

## 🔐 Core Protocol Constants

| Constant                   | Default | Purpose                             |
| -------------------------- | ------- | ----------------------------------- |
| `minimum-collateral-ratio` | `u150`  | 150% collateralization requirement. |
| `liquidation-threshold`    | `u120`  | 120% ratio triggers liquidation.    |
| `platform-fee-rate`        | `u1`    | 1% fee baseline (configurable).     |

---

## 🧮 Key Internal Logic

* **Collateral Valuation**:
  `collateral-value = collateral * btc-price`

* **Required Collateral**:
  `required-collateral = loan-amount * minimum-collateral-ratio`

* **Interest Accrual**:
  Computed per block using:

  ```clarity
  (calculate-interest principal rate blocks)
  ```

* **Liquidation Trigger**:
  If `current-ratio <= liquidation-threshold`, the position is forcibly liquidated and collateral reclaimed.

---

## 📊 Protocol Metrics

* `total-btc-locked` — aggregate BTC collateral.
* `total-loans-issued` — cumulative loan count.
* `platform-initialized` — activation flag (single-run).

All protocol metrics are queryable through `get-platform-stats`.

---

## 🔄 Data Flow (Simplified)

```text
User deposits BTC
      ↓
Collateral value checked via price oracle
      ↓
Loan minted if above min ratio
      ↓
Interest accrues per block
      ↓
User repays loan or gets liquidated if ratio < threshold
```

---

## 🧱 Development Notes

* Built for **Stacks 2.x** using **Clarity**.
* Compatible with **Stacks testnet/mainnet** deployment pipelines.
* Designed for extension — can integrate future modules like dynamic fee rates or multi-collateral assets.

---

## 🧰 Deployment Checklist

1. Deploy contract via `clarinet deploy` or `stx-cli`.
2. Call `initialize-platform`.
3. Set initial price feeds for `"BTC"` (and `"STX"` if supported).
4. Adjust risk parameters as needed.
5. Begin collateral deposit and loan operations.

---

## 📜 License

MIT License © BitMint Contributors
Open for community-driven enhancements and protocol integrations.
