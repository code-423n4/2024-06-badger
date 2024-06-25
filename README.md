

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

Route between different zaps and functionality.

## Key Functions
* One-click Leverage: use flash loans via leverage macro to lever loop in one action
* Flippening: a wrapper on leverage where you sell the final eBTC for more stETH (lever long ETH)
* Native ETH: Deposit from native ETH and auto-wrap into stETH
* ETH Variants deposits: come from WETH and wstETH as well for convenience

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
| [EbtcLeverageZapRouter.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)                | 254  | Main swap router contract that defines user facing external functions        | N/A               |
| [LeverageZapRouterBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)                | 216  | Base contract that implements functions for zap routers that use leverage        | N/A               |
| [ZapRouterBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)                                | 116  | Base contract that implements functions for all zap routers        | N/A                |
| [IEbtcLeverageZapRouter.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/interface/IEbtcLeverageZapRouter.sol)    | 37   | Leverage zap router interface contract and types        | N/A               |
| [LeverageMacroBase.sol](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol) | 383  | Base contract that performs flash loans, CDP operations and DEX swaps        | N/A                |
| TOTAL                                                                                                                                            | 1006 |         |                |

### Files out of scope
All files not in the scope table are Out Of Scope

## Scoping Q &amp; A

### General questions


| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       eBTC, stETH             |
| Test coverage                           | ebtc-protocol -  Lines: 72.80%, Functions: 57.22% & ebtc-zap-router - Lines: 47.06 %, Functions: 54.64%                         |
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


## Any novel or unique curve logic or mathematical models implemented in the contracts:

N/A


## Running tests
Clone the repo;
```bash
git clone --recurse https://github.com/code-423n4/2024-06-badger.git
```
For `ebtc-protocol` tests (Both for Hardhat and Foundry);

```bash
cd ebtc-protocol
yarn
cd packages/contracts
```
For `ebtc-protocol` Hardhat tests:
```bash
yarn test
```
For `ebtc-protocol` Foundry tests:
```bash
forge test
```
For `ebtc-protocol` test coverage:
```bash
forge coverage
```

| File                                                                        | % Lines            | % Statements       | % Branches        | % Funcs           |
|-----------------------------------------------------------------------------|--------------------|--------------------|-------------------|-------------------|
| Total                                                                       | 72.80% (3471/4768) | 72.15% (4385/6078) | 56.18% (850/1513) | 57.22% (622/1087) |

For `ebtc-zap-router` tests;
```bash
cd ebtc-zap-router
forge install
cd lib/ebtc
yarn
cd ../../
forge build
forge test
```
For `ebtc-zap-router` test coverage:
```bash
forge coverage
```




| File                                                | % Lines           | % Statements      | % Branches       | % Funcs          |
|-----------------------------------------------------|-------------------|-------------------|------------------|------------------|
| script/DeployEbtcZapRouter.sol                      | 0.00% (0/11)      | 0.00% (0/19)      | 100.00% (0/0)    | 0.00% (0/1)      |
| src/EbtcLeverageZapRouter.sol                       | 85.19% (69/81)    | 83.33% (90/108)   | 67.65% (23/34)   | 75.00% (12/16)   |
| src/EbtcZapRouter.sol                               | 88.16% (67/76)    | 90.00% (81/90)    | 46.43% (13/28)   | 88.89% (16/18)   |
| src/LeverageZapRouterBase.sol                       | 80.00% (56/70)    | 81.08% (60/74)    | 61.11% (11/18)   | 78.57% (11/14)   |
| src/ZapRouterBase.sol                               | 89.19% (33/37)    | 92.73% (51/55)    | 60.00% (6/10)    | 90.91% (10/11)   |
| src/invariants/ZapRouterActor.sol                   | 31.58% (6/19)     | 19.05% (4/21)     | 25.00% (2/8)     | 66.67% (2/3)     |
| src/invariants/ZapRouterProperties.sol              | 0.00% (0/5)       | 0.00% (0/14)      | 100.00% (0/0)    | 0.00% (0/5)      |
| src/invariants/ZapRouterStateSnapshots.sol          | 0.00% (0/107)     | 0.00% (0/109)     | 0.00% (0/48)     | 0.00% (0/3)      |
| src/testContracts/UniswapV3CollAdapter.sol          | 0.00% (0/62)      | 0.00% (0/80)      | 0.00% (0/20)     | 0.00% (0/3)      |
| src/testContracts/WstETH.sol                        | 35.56% (48/135)   | 32.35% (55/170)   | 18.33% (11/60)   | 30.91% (17/55)   |
| test/ZapRouterBaseInvariants.sol                    | 100.00% (52/52)   | 100.00% (63/63)   | 50.00% (4/8)     | 100.00% (13/13)  |
| test/invariants/TargetFunctionsBase.sol             | 95.08% (58/61)    | 96.05% (73/76)    | 50.00% (12/24)   | 80.00% (8/10)    |
| test/invariants/TargetFunctionsNoLeverage.sol       | 0.00% (0/175)     | 0.00% (0/213)     | 0.00% (0/48)     | 0.00% (0/12)     |
| test/invariants/TargetFunctionsWithLeverage.sol     | 67.08% (108/161)  | 65.44% (142/217)  | 36.67% (22/60)   | 60.71% (17/28)   |
| test/invariants/echidna/EchidnaLeverageTester.sol   | 0.00% (0/2)       | 0.00% (0/2)       | 100.00% (0/0)    | 0.00% (0/1)      |
| test/invariants/echidna/EchidnaNoLeverageTester.sol | 0.00% (0/2)       | 0.00% (0/2)       | 100.00% (0/0)    | 0.00% (0/1)      |
| Total                                               | 47.06% (497/1056) | 47.14% (619/1313) | 28.42% (104/366) | 54.64% (106/194) |

## Miscellaneous
Employees of Badger and employees' family members are ineligible to participate in this audit.


