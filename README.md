

# Multi-Signature Wallet Contract – README

## Overview

This **Clarity smart contract** implements a **multi-signature (M-of-N) wallet**. It ensures that critical transactions (such as STX transfers) can only be executed after approval from a required number of owners. The wallet provides **transaction submission, signing, and execution logic** with built-in **expiration, memo support, and ownership validation**.

This design is particularly useful for:

* DAO treasuries
* Joint custody wallets
* Corporate accounts
* Trust funds requiring multiple approvals

---

## 🔑 Key Features

* **M-of-N Threshold Security**: A transaction requires `M` out of `N` owners’ signatures before execution.
* **Owner Management**: Owners are initialized at deployment.
* **Transaction Lifecycle**: `pending → executed` with expiration handling.
* **Expiration Support**: Transactions expire after a configurable block height.
* **Memo Support**: Transactions may include a short memo for context.
* **Event Logging**: Important events (e.g., submitted transactions) are emitted for tracking.
* **Safety Checks**: Prevents duplicate signing, invalid thresholds, expired transactions, or unauthorized actions.

---

## ⚙️ Data Structures

### Data Variables

* `required-signatures (uint)` → Minimum number of signatures required.
* `total-owners (uint)` → Total number of owners.
* `tx-nonce (uint)` → Incrementing ID for transactions.
* `tx-expiration (uint)` → Default expiration in blocks (\~24 hours).

### Data Maps

* `owners { principal → bool }` → Registry of valid owners.
* `transactions { uint → { recipient, amount, status, signatures, signers, expiration-height, memo } }` → Transaction details.
* `transaction-signers { tx-id, signer → bool }` → Tracks who signed a given transaction.

---

## 🚀 Public Functions

### Initialization

```clarity
(initialize (owners-list (list 20 principal)) (threshold uint))
```

* Sets up owners and signature threshold.
* Validates threshold against number of owners.

### Transaction Submission

```clarity
(submit-transaction (recipient principal) (amount uint))
```

* Creates a new transaction with default expiration.

```clarity
(submit-transaction-with-memo (recipient principal) (amount uint) (memo (string-ascii 100)) (expiration-blocks uint))
```

* Same as above, with **memo + custom expiration**.

### Execution

```clarity
(execute-transaction (tx-id uint))
```

* Executes a transaction if:

  * Required signatures are met
  * Transaction is not expired
  * Funds are available

### Read-Only Queries

* `get-transaction (tx-id)` → Get details of a transaction.
* `get-required-signatures` → Returns M.
* `get-total-owners` → Returns N.
* `is-valid-owner (user)` → Check if an address is an owner.

---

## 🔒 Security Considerations

* **Replay Protection**: Nonce ensures unique transaction IDs.
* **Owner Validation**: Only registered owners may submit or sign.
* **Signature Tracking**: Prevents duplicate signatures.
* **Expiration**: Prevents stale or forgotten transactions.
* **Threshold Enforcement**: Ensures M-of-N approvals before execution.

---

## 📝 Example Workflow

1. **Initialization**

   ```clarity
   (initialize (list 'SP123... 'SP456... 'SP789...) u2)
   ```

   → 3 owners, 2 signatures required.

2. **Submit Transaction**

   ```clarity
   (submit-transaction 'SP999... u1000)
   ```

   → Creates a pending transaction with ID = 0.

3. **Owners Sign**
   Owners add signatures through signing logic (tracked via `transaction-signers`).

4. **Execute Transaction**

   ```clarity
   (execute-transaction u0)
   ```

   → Executes transfer if 2/3 signatures are present.

---