# MyCut Audit

## About the Project

**MyCut** is a contest rewards distribution protocol designed to streamline the setup and management of multiple reward distributions. It provides authorized claimants with a **90-day window** to claim their rewards. If claimants fail to claim within this period, the manager takes a cut of the remaining pool, and the leftover funds are distributed equally among those who claimed on time.

### Actors

- **Owner/Admin (Trusted):**
  - Create new pots
  - Close pots when the claim period has elapsed
  - Fund pots

- **User/Player:**
  - Claim their share of a pot

## Competition Details

This audit was conducted as part of the [CodeHawks First Flight Challenge](https://codehawks.cyfrin.io/c/2024-08-MyCut) held in August 2024.

## Results

I am proud to have secured the **2nd rank** in this challenge. You can view the leaderboard and my result [here](https://codehawks.cyfrin.io/c/2024-08-MyCut/results?t=leaderboard&page=1).

## Audit Report

A comprehensive audit report is included in this repository as [`audit_report.md`](./audit_report.md). The report covers:

- **Security Analysis:** Identification and mitigation of potential vulnerabilities in the smart contracts.
- **Best Practices Compliance:** Ensuring adherence to industry-standard best practices for smart contract development.
- **Performance Optimization:** Recommendations to enhance the protocol's performance and scalability.
- **Comprehensive Findings:** Detailed observations and actionable recommendations to improve the protocol's robustness.

