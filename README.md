

# Badger audit details
- Total Prize Pool: $40,000 in USDC
  - HM awards: $31,875 in USDC
  - QA awards: $1,325 in USDC 
  - Judge awards: $3,300 in USDC
  - Validator awards: $3,000 USDC 
  - Scout awards: $500 USDC 
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2024-06-badger/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts June 24, 2024 20:00 UTC
- Ends July 8, 2024 20:00 UTC

## This is a Private audit

This audit repo and its Discord channel are accessible to **certified wardens only.** Participation in private audits is bound by:

1. Code4rena's [Certified Contributor Terms and Conditions](https://github.com/code-423n4/code423n4.com/blob/main/_data/pages/certified-contributor-terms-and-conditions.md)
2. C4's [Certified Contributor Code of Professional Conduct](https://code4rena.notion.site/Code-of-Professional-Conduct-657c7d80d34045f19eee510ae06fef55)

*All discussions regarding private audits should be considered private and confidential, unless otherwise indicated.*

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-06-badger/blob/main/4naly3er-report.md).



_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

- Rounding error from (stETH/wstETH) conversion
- Owner is TechOps multisig
- We did not add ERC20 Permits, we could add them later
- We assume the user will chose the correct external function based on their collateral type (ETH/WETH/stETH/wstETH)
- DEX is hardcoded and will either be 0x or 1Inch
- A user is expected to rationally chose between normal zap router and leverage zap router



# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  https://gist.github.com/GalloDaSballo/4e345cec4e3aef9682f320149b3027aa
- **Documentation:** https://github.com/ebtc-protocol/ebtc-zap-router/blob/main/README.md
- **Website:** https://badger.com/
- **X/Twitter:** https://twitter.com/badgerdao
- **Discord:** https://discord.gg/badgerdao

---

# Scope



### Files in scope

| Contract                                                                                                                                         | SLOC | Purpose | Libraries used |
| ------------------------------------------------------------------------------------------------------------------------------------------------ |:---- | ------- | -------------- |
| [EbtcLeverageZapRouter.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)                | 254  |         |                |
| [LeverageZapRouterBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)                | 216  |         |                |
| [ZapRouterBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)                                | 116  |         |                |
| [IEbtcLeverageZapRouter.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/interface/IEbtcLeverageZapRouter.sol)    | 37   |         |                |
| [LeverageMacroBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol) | 383  |         |                |
| TOTAL                                                                                                                                            | 1006 |         |                |

### Files out of scope
All files not in the scope table are Out Of Scope

## Scoping Q &amp; A

### General questions


| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       eBTC, stETH             |
| Test coverage                           | ✅ SCOUTS: Please populate this after running the test coverage command                          |
| ERC721 used  by the protocol            |            None              |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Ethereum |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |   No  |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |  No  |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) | No    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |   No  |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              | No    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      | No    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              | No    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            | No    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    | No    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    | No    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    | No    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  | No    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |  No   |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          | No    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |   No  |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              | No    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                | No    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
N/A



# Additional context

## Main invariants

- Position manager permit needs to be revoked after each operation
- ZapRouter contract should not hold any CDP positions
- ZapRouter contract should not have any tokens after each operation (aside from rounding)

## Attack ideas (where to focus for bugs)
- Fee should be assessed only when debt increases
- Owner cannot execute arbitrary calls


## All trusted roles in the protocol


| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| TechOps multisig                          | Can rescue tokens             |


## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

N/A


## Running tests

forge build
forge test --gas-report

✅ SCOUTS: Please format the response above 👆 using the template below👇

```bash
git clone https://github.com/code-423n4/2023-08-arbitrum
git submodule update --init --recursive
cd governance
foundryup
make install
make build
make sc-election-test
```
To run code coverage
```bash
make coverage
```
To run gas benchmarks
```bash
make gas
```

✅ SCOUTS: Add a screenshot of your terminal showing the gas report
✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Badger and employees' family members are ineligible to participate in this audit.


