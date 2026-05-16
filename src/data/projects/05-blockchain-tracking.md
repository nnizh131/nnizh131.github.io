---
title: Cross-Chain Tracking on Ethereum & Polygon
description: Investigating whether users can be tracked and profiled across EVM-compatible blockchains by linking addresses and transaction behavior between Ethereum mainnet and Polygon.
github: https://github.com/nnizh131/DEDS
pdf: /blockchain-tracking.pdf
draft: false
tags: [Python, Blockchain, Data Analysis, Privacy]
---

## Overview

Ethereum's congestion and high gas fees drove many users to EVM-compatible sidechains like Polygon. Because these chains share the same address format and account model, a user's identity on one chain is immediately linkable on the other — a privacy implication most users aren't aware of. This project investigates how much can be learned about a user by combining on-chain data from both networks, and whether gas price increases on Ethereum actually push users to Polygon.

The full blockchain history of Ethereum is around 10 TB. To query it at scale, we used the Google BigQuery API against the Ethereum ETL dataset — a relational mirror of the chain partitioned across blocks, transactions, token transfers, contracts, traces, and logs. Polygon was queried through the same interface. Each query processed roughly 300 GB and completed in about two minutes.

## Bridging Activity

The first question: does rising transaction cost on Ethereum push users to Polygon?

We tracked average gas price on Ethereum over time and compared it against total transaction volume on Polygon, converting Matic to ETH for a common scale. The expected positive correlation — more expensive Ethereum → more activity on Polygon — was not observed. Gas price spikes on Ethereum did not produce a measurable uptick in Polygon volume. The relationship is not trivial, and a complete picture would require including more sidechains (Avalanche, Fantom, etc.) in the analysis.

## Aave Protocol Analysis

Aave is a decentralized lending protocol that exists on both Ethereum and Polygon. We analyzed user transaction types on Aave for December 2020 — the protocol's early period — across 12 distinct transaction categories. Borrowing was the dominant activity, followed by deposits. Total borrow volume for the month was around 30 ETH, far below Aave's current scale, consistent with the protocol still finding its user base at the time.

## Case Study: Profiling a Single Address

To demonstrate what cross-chain tracking enables in practice, we selected a high-activity Ethereum address at random and reconstructed her financial behavior from on-chain data alone.

From public transactions we identified: which tokens she traded most (Tether USD, ChainLink, Molecular Future), which she held long-term versus traded short-term, and her transaction timing patterns. The network graph of her ten most-traded tokens made her portfolio structure immediately visible.

Her Polygon activity told a different story: almost no token sales, and only a handful of purchases of CryptoMETH — effectively no presence. This held for the nine other most-active Ethereum addresses we examined: despite high gas prices, the heaviest Ethereum users were not moving to Polygon.

One behavioral observation: she sold a large position in Universal Euro at one of its lowest values, and her largest sale landed the same day the token gained 60% — suggesting short-term trading behavior driven by factors the on-chain data alone can't explain (personal liquidity needs, risk tolerance).

## Findings

Cross-chain tracking on EVM-compatible networks is straightforward: knowing a user's Ethereum address immediately exposes her full history on any compatible chain. No de-anonymization step is required — the address is the identity.

Despite this, the assumption that high Ethereum fees push users to Polygon did not hold for the user cohort we analyzed. The most active Ethereum users stayed on Ethereum. Gas price is apparently not the primary driver of chain selection for established users.

The case study confirmed that on-chain data alone is sufficient to build a usable behavioral profile — trading frequency, portfolio composition, holding periods — without any personal information. What it cannot explain is intent: the same transaction looks different depending on whether the user needed liquidity or simply mistimed a trade.

## Stack

- Python (Pandas, NumPy, Matplotlib, Seaborn, NetworkX)
- Google BigQuery API (Ethereum ETL dataset)
- Jupyter Notebooks
