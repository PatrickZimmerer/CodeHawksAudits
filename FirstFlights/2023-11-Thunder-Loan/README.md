# First Flight by CodeHawks: Thunder Loan

## Questions, Ideas and Observations

- **Since we are interacting with arbitrary ERC20 tokens we shouldn't assume transfer and transferFrom are non-reentrant!**

- Typo in word `ThunderLoan__ExhangeRateCanOnlyIncrease` but this occurs in both contracts

- **Storage layout in the upgrade contract is not the same as in the currently implemented contract, also the storage varaible names are different!**

- How does this affect the transferred data and future function calls?

---

- Old layout:

1. s_tokenToAssetToken
2. s_feePrecision
3. s_flashLoanFee
4. s_currentlyFlashLoaning

- New layout:

1. s_tokenToAssetToken
2. s_flashLoanFee
3. FEE_PRECISION
4. s_currentlyFlashLoaning

---

- Second `initialize()` is missing the `s_feePrecision` initialization (new variable at storage slot 2), also the other storage layout now should set `s_flashLoanFee` which is now at storage slot 3 instead of storage slot 2 like before, does this mean the precision now get set to 3e15 insterad of 1e18?

- `deposit()` in the upgraded contract is nnot calculating a fee and updating the ExchangeRate of the assetToken anymore => check for vulnerability

- `getCalculatedFee()` which is used inside `flashloan()` makes use of the `s_feePrecision` variable which may be invalid after the update

- In `flashLoan()` we update the exchange rate of the assetToken which will be updated by the possibly invalid fee, also after the loan got paid and the external call was made, we check the endingBalance where the fee is also involved

- Lines 281 - 282 & 286 - 287 could go after the else block to remove redundant code

- **`getCalculatedFee()` & `updateFlashloanFee` needs to be checked after the storage slot fuck up!!**

### Prize Pool

- High - 100xp
- Medium - 20xp
- Low - 2xp

- Starts: Noon UTC Wednesday, Nov 01 2023
- Ends: Noon UTC Wednesday, Nov 08 2023

## Stats

- nSLOC: 387
- Complexity Score: 325

## A flash loan protocol based on [Aave](https://aave.com/) and [Compound](https://compound.finance/).

- [First Flight by CodeHawks: Thunder Loan](#first-flight-by-codehawks-thunder-loan)
  - [Questions, Ideas and Observations](#questions-ideas-and-observations)
    - [Prize Pool](#prize-pool)
  - [Stats](#stats)
  - [A flash loan protocol based on Aave and Compound.](#a-flash-loan-protocol-based-on-aave-and-compound)
- [About](#about)
- [Getting Started](#getting-started)
  - [Requirements](#requirements)
  - [Quickstart](#quickstart)
    - [Optional Gitpod](#optional-gitpod)
- [Usage](#usage)
  - [Testing](#testing)
    - [Test Coverage](#test-coverage)
  - [Audit Scope Details](#audit-scope-details)
  - [Compatibilities](#compatibilities)
  - [Roles](#roles)
  - [Known Issues](#known-issues)

# About

The ⚡️ThunderLoan⚡️ protocol is meant to do the following:

1. Give users a way to create flash loans
2. Give liquidity providers a way to earn money off their capital

Liquidity providers can `deposit` assets into `ThunderLoan` and be given `AssetTokens` in return. These `AssetTokens` gain interest over time depending on how often people take out flash loans!

What is a flash loan?

A flash loan is a loan that exists for exactly 1 transaction. A user can borrow any amount of assets from the protocol as long as they pay it back in the same transaction. If they don't pay it back, the transaction reverts and the loan is cancelled.

Users additionally have to pay a small fee to the protocol depending on how much money they borrow.

We are planning to upgrade from the current `ThunderLoan` contract to the `ThunderLoanUpgraded` contract. Please include this upgrade in scope of a security review.

# Getting Started

## Requirements

- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
  - You'll know you did it right if you can run `git --version` and you see a response like `git version x.x.x`
- [foundry](https://getfoundry.sh/)
  - You'll know you did it right if you can run `forge --version` and you see a response like `forge 0.2.0 (816e00b 2023-03-16T00:05:26.396218Z)`

## Quickstart

```
git clone https://github.com/Cyfrin/2023-11-Thunder-Loan
cd 2023-11-Thunder-Loan
make
```

### Optional Gitpod

If you can't or don't want to run and install locally, you can work with this repo in Gitpod. If you do this, you can skip the `clone this repo` part.

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#github.com/Cyfrin/6-thunder-loan-audit)

# Usage

## Testing

```
forge test
```

### Test Coverage

```
forge coverage
```

and for coverage based testing:

```
forge coverage --report debug
```

## Audit Scope Details

- Commit Hash: e8ce05f5530ca965165d41547b289604f873fdf6
- In Scope:

```
├── interfaces
│   ├── IFlashLoanReceiver.sol
│   ├── IPoolFactory.sol
│   ├── ITSwapPool.sol
│   #── IThunderLoan.sol
├── protocol
│   ├── AssetToken.sol
│   ├── OracleUpgradeable.sol
│   #── ThunderLoan.sol
#── upgradedProtocol
    #── ThunderLoanUpgraded.sol
```

## Compatibilities

- Solc Version: 0.8.20
- Chains:
  - ETH
- Tokens:
  - All tokens that follow the ERC20 Standard
  - Any ERC20 that does not is only included if it is specified below:
    - USDT: 0xdAC17F958D2ee523a2206206994597C13D831ec7
    - USDC: 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
    - STA: 0xa7DE087329BFcda5639247F96140f9DAbe3DeED1
    - PAXG: 0x45804880De22913dAFE09f4980848ECE6EcbAf78
    - BNB: 0xB8c77482e45F1F44dE1745F52C74426C631bDD52
    - ZIL: 0x05f4a42e251f2d52b8ed15E9FEdAacFcEF1FAD27
    - KNC: 0xdd974D5C2e2928deA5F71b9825b8b646686BD200

## Roles

- Owner: The owner of the protocol who has the power to upgrade the implementation.
- Liquidity Provider: A user who deposits assets into the protocol to earn interest.
- User: A user who takes out flash loans from the protocol.

## Known Issues

None
