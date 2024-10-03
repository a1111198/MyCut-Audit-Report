# MyCut - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Incorrect Remaining Rewards Distribution Calculation](#H-01)
    - ### [H-02. Unbounded Loop in closePot Function Can Lead to Denial of Service](#H-02)
    - ### [H-03. Overwriting Rewards Instead of Adding Due to Lack of Duplicate Check in Players Array](#H-03)

- ## Low Risk Findings
    - ### [L-01. Precision Loss in Reward Calculation Causes Dead Funds in Contract ](#L-01)
    - ### [L-02. Centralization Risk for trusted owners](#L-02)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #23

### Dates: Aug 29th, 2024 - Sep 5th, 2024

[See more contest details here](https://codehawks.cyfrin.io/c/2024-08-MyCut)

# <a id='results-summary'></a>Results Summary

### Number of findings:
- High: 3
- Medium: 0
- Low: 2


# High Risk Findings

## <a id='H-01'></a>H-01. Incorrect Remaining Rewards Distribution Calculation            



# Summary:

Incorrect distribution of remaining rewards among claimants in the `closePot` function. The function incorrectly divides the remaining rewards (after the manager's cut) by the total number of players (`i_players.length`) instead of by the number of actual claimants (`claimants.length`), leading to potential misallocation of funds.

## Vulnerability Details:

The `closePot` function is responsible for distributing the remaining rewards after 90 days to the claimants and the manager. The function currently calculates each claimant's share of the remaining rewards by dividing the remaining rewards (after deducting the manager's cut) by the total number of players (`i_players.length`). This approach is flawed because it assumes that all players have claimed their rewards, which is not necessarily the case. As a result, the rewards are incorrectly allocated among all players rather than just the claimants.

**Vulnerability Location:**\
[Pot.sol: Line 57](https://github.com/Cyfrin/2024-08-MyCut/blob/946231db0fe717039429a11706717be568d03b54/src/Pot.sol#L57)

## Impact:

This vulnerability could lead to an incorrect distribution of rewards:

* **Overpayment or Underpayment:** Claimants may receive more or less than their rightful share of the remaining rewards, causing financial discrepancies and potential disputes.
* **Loss of Trust:** Users may lose trust in the protocol due to perceived unfairness or mismanagement of funds.
* **Potential Exploit:** Players might strategically delay their claims or avoid claiming altogether if they believe the distribution mechanism is flawed, impacting the protocol's integrity.

## Tools Used:

* Manual Review

## Recommendations:

To mitigate this issue, modify the calculation in the `closePot` function to divide the remaining rewards by the number of actual claimants (`claimants.length`) instead of the total number of players (`i_players.length`). This ensures that only players who have claimed their rewards receive a portion of the remaining rewards.

**Updated Function:**

```solidity
function closePot() external onlyOwner {
    if (block.timestamp - i_deployedAt < 90 days) {
        revert Pot__StillOpenForClaim();
    }
    if (remainingRewards > 0) {
        uint256 managerCut = remainingRewards / managerCutPercent;
        i_token.transfer(msg.sender, managerCut);

        // Correct calculation:
        uint256 claimantCut = (remainingRewards - managerCut) / claimants.length;
        for (uint256 i = 0; i < claimants.length; i++) {
            _transferReward(claimants[i], claimantCut);
        }
    }
}
```

## <a id='H-02'></a>H-02. Unbounded Loop in closePot Function Can Lead to Denial of Service            



## Summary:

The `closePot` function contains an unbounded `for` loop that iterates over the `claimants` array to distribute rewards. If the number of claimants is large, the loop could exceed the block gas limit, resulting in a denial-of-service (DoS) condition where the function fails to execute.

## Vulnerability Details:

In the `closePot` function, there is a `for` loop that iterates through the `claimants` array to transfer rewards:

```solidity
for (uint256 i = 0; i < claimants.length; i++) {
    _transferReward(claimants[i], claimantCut);
}
```

If the `claimants` array becomes too large, the gas required to execute all the iterations and subsequent reward transfers may exceed the block gas limit. This would prevent the transaction from being successfully mined, effectively causing a denial-of-service condition where the rewards cannot be distributed, and the pot cannot be closed.

## Impact:

The impact of this vulnerability includes:

* **Denial of Service:** If the loop consumes more gas than the block gas limit allows, the `closePot` function will revert. This prevents the contract from executing critical functions such as reward distribution and pot closure.
* **Locked Funds:** Funds meant to be distributed to claimants could remain locked in the contract if the loop cannot be executed due to gas limits.
* **User Frustration:** Users expecting to receive rewards may be frustrated by the inability to claim them due to the transaction failures caused by the gas limitations.

## Tools Used:

* Manual Review

## Recommendations:

To mitigate the risk of a denial-of-service attack due to an unbounded loop, consider the following approaches:

1. **Batch Processing:** Implement batch processing to distribute rewards in smaller batches over multiple transactions. This would prevent the function from exceeding the block gas limit.

2. **Gas Limit Checks:** Introduce a check for the gas limit before the loop execution and halt the operation if the number of claimants is too large to process in a single transaction.

3. **Off-Chain Calculations:** Use off-chain mechanisms to calculate and submit reward distributions to reduce on-chain computation.

## <a id='H-03'></a>H-03. Overwriting Rewards Instead of Adding Due to Lack of Duplicate Check in Players Array            



## Summary:

**High Severity:** The contract does not check for duplicate entries in the `players` array during initialization. If a player appears multiple times with different rewards, the current implementation overwrites the previous reward value in the `playersToRewards` mapping instead of adding the rewards together. This results in incorrect reward allocation, where the player could receive either less or more than their rightful total rewards, leading to financial discrepancies.

## Vulnerability Details:

In the constructor of the `Pot` contract, player rewards are initialized using a loop that assigns rewards from the `i_rewards` array to the players in the `i_players` array:

```solidity
for (uint256 i = 0; i < i_players.length; i++) {
    playersToRewards[i_players[i]] = i_rewards[i];
}
```

Since there is no check for duplicate entries in the `i_players` array, a player listed multiple times with different rewards will have their reward overwritten in the `playersToRewards` mapping. Instead of adding the rewards for duplicate entries, the mapping only retains the last specified reward. This leads to an incorrect total reward for that player, as earlier rewards are overridden.

## Impact:

The impact of this vulnerability includes:

* **Incorrect Reward Allocation:** A player appearing multiple times in the `i_players` array may end up receiving either more or less than their total calculated reward, depending on the order of entries.
* **Financial Discrepancies:** This could lead to significant financial discrepancies and dissatisfaction among players who expect to receive their total allocated rewards.
* **Loss of Trust:** The integrity of the reward distribution process is compromised, potentially leading to a loss of trust in the contract's fairness.

## Tools Used:

* Manual Review

## Recommendations:

To efficiently resolve this issue, modify the reward assignment in the constructor to accumulate rewards for each player instead of overwriting them. This ensures that players receive the total sum of all their rewards, even if they appear multiple times in the `i_players` array.

**Updated Constructor with Accumulation Logic:**

```solidity
constructor(address[] memory players, uint256[] memory rewards, IERC20 token, uint256 totalRewards) {
    require(players.length > 0, "Players array must not be empty");
    require(players.length == rewards.length, "Players and rewards arrays must be of equal length");

    i_players = players;
    i_rewards = rewards;
    i_token = token;
    i_totalRewards = totalRewards;
    remainingRewards = totalRewards;
    i_deployedAt = block.timestamp;

    // Accumulate rewards for players to handle duplicates efficiently
    for (uint256 i = 0; i < i_players.length; i++) {
        playersToRewards[i_players[i]] += i_rewards[i];
    }
}
```

By changing `playersToRewards[i_players[i]] = i_rewards[i];` to `playersToRewards[i_players[i]] += i_rewards[i];`, the contract correctly accumulates the total rewards for each player, ensuring accurate reward distribution without the need for additional gas-consuming checks for duplicates.

This approach is more gas-efficient than adding extra checks or requiring unique entries and still resolves the issue effectively.

    


# Low Risk Findings

## <a id='L-01'></a>L-01. Precision Loss in Reward Calculation Causes Dead Funds in Contract             



## Summary:

**High Severity:** The reward distribution calculation in the `closePot` function is susceptible to precision loss due to integer division, potentially leaving a small amount of tokens undisbursed and permanently locked in the contract as dead funds.

## Vulnerability Details:

In the `closePot` function, the calculation of each claimant's share of the remaining rewards uses integer division:

```solidity
uint256 claimantCut = (remainingRewards - managerCut) / i_players.length;
```

Since Solidity performs integer division, any remainder resulting from the division is discarded. This truncation leads to precision loss, where the total amount distributed does not exactly match the `remainingRewards`. As a result, a small number of tokens may remain in the contract after the distribution, which cannot be claimed or utilized and are effectively locked as dead funds.

**Vulnerability Location:**\
[Pot.sol: Line 57](https://github.com/Cyfrin/2024-08-MyCut/blob/946231db0fe717039429a11706717be568d03b54/src/Pot.sol#L57)

## Impact:

The impact of this vulnerability includes:

* **Dead Funds:** Small amounts of tokens (remainders) can become permanently locked in the contract, reducing the efficiency of fund distribution.
* **Potential Financial Loss:** Even small amounts of dead funds can accumulate over multiple distributions, leading to a significant total loss over time.
* **Inefficient Fund Management:** The presence of dead funds could be perceived as poor management and a lack of precision in the contract's financial operations, potentially reducing user trust.

## Tools Used:

* Manual Review

## Recommendations:

To mitigate this issue, the contract should include logic to handle any remainders left after dividing the rewards among claimants. These remainders could be redistributed in a way that prevents them from becoming dead funds or handled as part of the manager's cut or a future distribution. Although it still depends on the team how they choose to manage these funds, the following approach is one possible way to address the issue:

**Updated Function Example to Handle Remainders:**

```solidity
function closePot() external onlyOwner {
    if (block.timestamp - i_deployedAt < 90 days) {
        revert Pot__StillOpenForClaim();
    }
    if (remainingRewards > 0) {
        uint256 managerCut = remainingRewards / managerCutPercent;
        i_token.transfer(msg.sender, managerCut);

        uint256 totalClaimantCut = remainingRewards - managerCut;
        uint256 baseCutPerClaimant = totalClaimantCut / claimants.length;
        uint256 remainder = totalClaimantCut % claimants.length;

        for (uint256 i = 0; i < claimants.length; i++) {
            _transferReward(claimants[i], baseCutPerClaimant);
        }

        // Optionally, transfer the remainder to the manager to prevent dead funds
        if (remainder > 0) {
            i_token.transfer(msg.sender, remainder);
        }
    }
}
```

By implementing this logic, the contract ensures that any leftover tokens (remainders) are not permanently locked as dead funds, maintaining efficient fund management and full utilization of the pot's resources.

## <a id='L-02'></a>L-02. Centralization Risk for trusted owners            



## Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates.



