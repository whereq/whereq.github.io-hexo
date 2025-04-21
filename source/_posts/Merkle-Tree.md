---
title: Merkle Tree
date: 2025-04-21 18:43:23
categories:
- Cypto Currency
- Ethereum
tags:
- Cypto Currency
- Ethereum
---

## 🔍 What is a Merkle Tree?

A **Merkle Tree** (also called a binary hash tree) is a tree structure where:
- Every **leaf node** contains the cryptographic hash of a data block.
- Every **non-leaf node** contains the hash of the concatenation of its child hashes.

It’s used in blockchain systems (e.g., Bitcoin, Ethereum) to efficiently and securely verify the integrity of large sets of data, such as transactions.

---

## 🌳 Merkle Tree Structure

Example with 4 transactions (T1, T2, T3, T4):

```
                Root
               /    \
            H12      H34
           /  \     /   \
        H1   H2   H3   H4
```

- `H1 = hash(T1)`, `H2 = hash(T2)`, etc.
- `H12 = hash(H1 + H2)`, `H34 = hash(H3 + H4)`
- `Root = hash(H12 + H34)`

This root hash is called the **Merkle Root**.

---

## 🧠 Benefits of Merkle Tree

- Efficient and secure verification of data integrity.
- Only need a small part of the tree (a Merkle proof) to verify a data block.
- Used in **SPV (Simplified Payment Verification)** in Bitcoin.

---

## 💻 JavaScript Sample Code Using `crypto`

```javascript
const crypto = require('crypto');

// Hashing function
function sha256(data) {
  return crypto.createHash('sha256').update(data).digest('hex');
}

// Build Merkle Tree from an array of transactions
function buildMerkleTree(transactions) {
  if (transactions.length === 0) return [''];

  // Step 1: Hash each transaction
  let level = transactions.map(tx => sha256(tx));

  // Step 2: Build tree up to the root
  while (level.length > 1) {
    const nextLevel = [];

    for (let i = 0; i < level.length; i += 2) {
      const left = level[i];
      const right = i + 1 < level.length ? level[i + 1] : left; // duplicate last if odd
      const combined = left + right;
      nextLevel.push(sha256(combined));
    }

    level = nextLevel;
  }

  return level[0]; // Merkle root
}

// Example usage
const txs = ['tx1-data', 'tx2-data', 'tx3-data', 'tx4-data'];
const merkleRoot = buildMerkleTree(txs);

console.log('Merkle Root:', merkleRoot);
```

---

## 📎 Notes:
- This sample assumes a binary Merkle tree.
- If there's an odd number of leaves, the last hash is duplicated to make a pair.
- You can also implement **Merkle Proof** logic to verify a leaf's inclusion using hashes up to the root.

## Another drawn
```
               Merkle Root
                   │
           ┌───────┴────────┐
           │                │
        Hash01           Hash23
       ┌────┴────┐      ┌────┴────┐
    Hash0      Hash1  Hash2     Hash3
    │            │     │          │
  Tx0          Tx1   Tx2        Tx3
```