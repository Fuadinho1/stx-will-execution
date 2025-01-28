# Stacks Automated Will Execution

## Overview
The contract ensures trustless and transparent distribution of assets among beneficiaries while providing flexibility for tax deductions, oracle confirmations, and emergency actions.

---

## Features

### 1. **Oracle-Based Death Confirmation**

- Uses a multi-signature oracle system to confirm the death of the contract owner before activating inheritance distribution.
- Configurable number of confirmations required (`required-confirmations`).

### 2. **Time-Locked Inheritance**

- Each beneficiary can have a specific lock period for their inheritance, ensuring assets are distributed only after the stipulated block height.

### 3. **NFT Inheritance**

- Beneficiaries can inherit NFTs by specifying token IDs. NFT ownership is updated on-chain upon claim.

### 4. **Inheritance Tax**

- A configurable tax percentage (`inheritance-tax`) is applied to all inherited funds for contract maintenance.

### 5. **Dispute Resolution**

- Beneficiaries can raise disputes by submitting evidence hashes.
- Disputes can be resolved via voting or manually by the contract owner.

### 6. **Phased Inheritance Release**

- Inheritance can be split into multiple phases to distribute assets incrementally.

### 7. **Emergency Controls**

- The contract owner can deactivate the contract or update the number of required confirmations in emergencies.

### 8. **Enhanced Readability**

- Multiple read-only functions provide insights into the contract’s state and specific details about beneficiaries, NFT ownership, and disputes.

---

## Contract Variables

### Global Variables

| Variable                 | Description                                                     |
| ------------------------ | --------------------------------------------------------------- |
| `contract-owner`         | The principal that owns and controls the contract.              |
| `is-active`              | Indicates whether the contract is active.                       |
| `death-confirmed`        | True if the death of the owner has been confirmed by oracles.   |
| `last-will-hash`         | Hash of the last will document for reference.                   |
| `inheritance-tax`        | Percentage tax applied to all inheritance distributions.        |
| `required-confirmations` | Number of oracle confirmations required for death confirmation. |
| `confirmation-count`     | Current count of oracle confirmations.                          |

### Maps

| Map Name             | Key                        | Value                                                                | Description                                      |
| -------------------- | -------------------------- | -------------------------------------------------------------------- | ------------------------------------------------ |
| `oracles`            | `principal`                | `bool`                                                               | Stores authorized oracle accounts.               |
| `beneficiaries`      | `{beneficiary: principal}` | `{share, claimed, time-lock, nft-tokens}`                            | Tracks inheritance details for each beneficiary. |
| `nft-ownership`      | `uint`                     | `principal`                                                          | Maps NFT token IDs to their owners.              |
| `disputes`           | `{disputer: principal}`    | `{evidence-hash, resolved}`                                          | Tracks disputes raised by beneficiaries.         |
| `inheritance-phases` | `{beneficiary: principal}` | `{phase-1-claimed, phase-2-claimed, phase-1-amount, phase-2-amount}` | Tracks phased inheritance claims.                |

---

## Public Functions

### Initialization

- **`initialize-contract (oracle-list (list 5 principal))`**
  - Adds multiple oracles to the contract.
  - Can only be called by the `contract-owner`.

### Adding Beneficiaries

- **`add-beneficiary (beneficiary principal) (share uint) (lock-period uint) (nft-list (list 10 uint))`**
  - Adds a beneficiary with a specified share percentage, lock period, and NFT allocation.
  - Ensures the total share does not exceed 100%.

### Death Confirmation

- **`confirm-death`**
  - Allows oracles to confirm the death of the contract owner.
  - Once the required number of confirmations is reached, `death-confirmed` is set to `true`.

### Claiming Inheritance

- **`claim-inheritance`**
  - Allows a beneficiary to claim their inheritance if:
    - Death is confirmed.
    - The time lock has passed.
    - The inheritance has not already been claimed.
  - Automatically transfers NFTs and STX (after deducting the inheritance tax).

### Dispute Resolution

- **`raise-dispute (evidence-hash (buff 32))`**
  - Allows beneficiaries to raise disputes by submitting an evidence hash.
- **`resolve-dispute (disputer principal)`** (private)
  - Marks a dispute as resolved.

### Updating the Will

- **`update-will-hash (new-hash (buff 32))`**
  - Updates the hash of the last will document.

### Emergency Functions

- **`deactivate-contract`**
  - Deactivates the contract, preventing further actions.
- **`update-required-confirmations (new-count uint)`**
  - Updates the number of confirmations required for death confirmation.

### Phased Inheritance

- **`claim-phase-1 (phase-data {phase-1-claimed, phase-2-claimed, phase-1-amount, phase-2-amount})`** (private)
  - Allows beneficiaries to claim the first phase of their inheritance.

---

## Read-Only Functions

### Beneficiary Information

- **`get-beneficiary-info (beneficiary principal)`**
  - Retrieves inheritance details for a specific beneficiary.

### Contract Status

- **`get-contract-status`**
  - Returns the current state of the contract, including active status, death confirmation, and required confirmations.

### NFT Ownership

- **`get-nft-owner (token-id uint)`**
  - Returns the current owner of a specific NFT.

---

## Error Codes

| Error Code                       | Description                                             |
| -------------------------------- | ------------------------------------------------------- |
| `ERR-NOT-AUTHORIZED`             | Caller is not authorized to perform the action.         |
| `ERR-ALREADY-CLAIMED`            | Beneficiary has already claimed their inheritance.      |
| `ERR-INVALID-SHARE`              | Specified share exceeds 100%.                           |
| `ERR-NOT-ACTIVE`                 | Contract is not active.                                 |
| `ERR-DEATH-NOT-CONFIRMED`        | Death confirmation has not been completed.              |
| `ERR-TIME-LOCK`                  | The time lock period has not yet passed.                |
| `ERR-INVALID-NFT`                | NFT token ID is invalid or unassigned.                  |
| `ERR-INSUFFICIENT-CONFIRMATIONS` | Not enough oracle confirmations for death confirmation. |

---

## Example Usage

### Initialization

```clarity
(initialize-contract [(principal 'SP123) (principal 'SP456)])
```

### Adding a Beneficiary

```clarity
(add-beneficiary (principal 'SP789) u50 u100 [(uint 1) (uint 2)])
```

### Confirming Death

```clarity
(confirm-death)
```

### Claiming Inheritance

```clarity
(claim-inheritance)
```

### Raising a Dispute

```clarity
(raise-dispute 0x123456789abcdef)
```

---
