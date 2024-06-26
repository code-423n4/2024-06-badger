# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 2 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 1 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables rather than re-reading them from storage | 2 |
| [GAS-4](#GAS-4) | For Operations that will not overflow, you could use unchecked | 35 |
| [GAS-5](#GAS-5) | Use Custom Errors instead of Revert Strings to save Gas | 10 |
| [GAS-6](#GAS-6) | Avoid contract existence checks by using low level calls | 18 |
| [GAS-7](#GAS-7) | State variables only set in the constructor should be declared `immutable` | 15 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 3 |
| [GAS-9](#GAS-9) | Increments/decrements can be unchecked in for-loops | 1 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 25 |
| [GAS-11](#GAS-11) | `internal` functions not called by the contract should be removed | 12 |
### <a name="GAS-1"></a>[GAS-1] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (2)*:
```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

437:             marginDecrease += _params.stEthMarginBalance;

442:             marginIncrease += _params.stEthMarginBalance;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

35:     bool internal immutable willSweep;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (2)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

234:         swaps[0].addressForSwap = DEX;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

85:                 address(wstEth),

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-4"></a>[GAS-4] For Operations that will not overflow, you could use unchecked

*Instances (35)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

73:         noFlashloan // Use this to not perform a FL and just `doOperation`

101:         ICdpManagerData.Status expectedStatus; // NOTE: THIS IS SUPERFLUOUS

295:         SwapOperation[] swapsBefore; // Empty to skip

296:         SwapOperation[] swapsAfter; // Empty to skip

297:         OperationType operationType; // Open, Close, etc..

298:         bytes OperationData; // Generic Operation Data, which we'll decode to use

308:         SwapCheck[] swapChecks; // Empty to skip

318:         None, // Swaps only

438:                 ++i;

488:             for (uint256 i; i < length; ++i) {

506:         require(addy != address(this)); // If it could call this it could fake the forwarded caller

585:                 _gas, // gas

586:                 _target, // recipient

587:                 _value, // ether value

588:                 add(_calldata, 0x20), // inloc

589:                 mload(_calldata), // inlen

590:                 0, // outloc

591:                 0 // outlen

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

216:         _requireAtLeastMinNetStEthBalance(_stEthDepositAmount - LIQUIDATOR_REWARD);

305:         uint256 _stETHDiff = _zapStEthBalanceAfter - _zapStEthBalanceBefore;

437:             marginDecrease += _params.stEthMarginBalance;

442:             marginIncrease += _params.stEthMarginBalance;

465:         uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

47:             false // Do not sweep

138:         uint256 newDebt = _cdp._isDebtIncrease ? debt + _cdp._EBTCChange : debt - _cdp._EBTCChange;

140:             ? coll + stEth.getSharesByPooledEth(_cdp._stEthBalanceIncrease)

141:             : coll - stEth.getSharesByPooledEth(_cdp._stEthBalanceDecrease);

262:                     value: (_totalCollateral * _collValidationBuffer) / BPS,

284:                 (flData.eBTCToMint * zapFeeBPS) / BPS

308:                     (flData._EBTCChange * zapFeeBPS) / BPS

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

39:         uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;

46:         uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;

50:         uint256 _rawETHConverted = address(this).balance - _rawETHBalBefore;

67:         uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;

111:         uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-5"></a>[GAS-5] Use Custom Errors instead of Revert Strings to save Gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

Additionally, custom errors can be used inside and outside of contracts (including interfaces and libraries).

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:

> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Consider replacing **all revert strings** with custom errors in the solution, and particularly those that have multiple occurrences:

*Instances (10)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

40:         revert("Must be overridden");

45:         require(owner() == msg.sender, "Must be owner");

282:             require(check.value >= valueToCheck, "!LeverageMacroReference: gte post check");

284:             require(check.value <= valueToCheck, "!LeverageMacroReference: let post check");

286:             require(check.value == valueToCheck, "!LeverageMacroReference: equal post check");

288:             revert("Operator not found");

402:         require(initiator == address(this), "LeverageMacroReference: wrong initiator for flashloan");

471:         require(success, "Call has failed");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

78:         revert("disabled");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

55:         require(msg.value == _initialETH, "EbtcZapRouter: Incorrect ETH amount");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-6"></a>[GAS-6] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (18)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

253:         uint256 ebtcBal = ebtcToken.balanceOf(address(this));

491:                     IERC20(swapChecks[i].tokenToCheck).balanceOf(address(this)) >

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

302:         uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));

304:         uint256 _zapStEthBalanceAfter = stEth.balanceOf(address(this));

346:         uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));

364:         uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));

382:         uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));

400:         uint256 _zapStEthBalanceBefore = stEth.balanceOf(address(this));

465:         uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

86:         uint256 bal = ebtcToken.balanceOf(address(this));

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

36:         uint256 _balBefore = stEth.balanceOf(address(this));

39:         uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;

44:         uint256 _wETHBalBefore = wrappedEth.balanceOf(address(this));

46:         uint256 _wETHReiceived = wrappedEth.balanceOf(address(this)) - _wETHBalBefore;

65:         uint256 _stETHBalBefore = stEth.balanceOf(address(this));

67:         uint256 _stETHReiceived = stEth.balanceOf(address(this)) - _stETHBalBefore;

109:         uint256 _balBefore = stEth.balanceOf(address(this));

111:         uint256 _deposit = stEth.balanceOf(address(this)) - _balBefore;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-7"></a>[GAS-7] State variables only set in the constructor should be declared `immutable`
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around **20 000 gas** per variable) and replace the expensive storage-reading operations (around **2100 gas** per reading) to a less expensive value reading (**3 gas**)

*Instances (15)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

60:         borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);

61:         activePool = IActivePool(_activePool);

62:         cdpManager = ICdpCdps(_cdpManager);

63:         ebtcToken = IEBTCToken(_ebtc);

64:         stETH = ICollateralToken(_coll);

65:         sortedCdps = ISortedCdps(_sortedCdps);

67:         willSweep = _sweepToCaller;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

54:         theOwner = params.owner;

55:         DEX = params.dex;

56:         zapFeeBPS = params.zapFeeBPS;

57:         zapFeeReceiver = params.zapFeeReceiver;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

28:         MIN_CHANGE = IMinChangeGetter(_borrowerOperations).MIN_CHANGE();

29:         wstEth = _wstEth;

30:         wrappedEth = _wEth;

31:         stEth = _stEth;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-8"></a>[GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (3)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

18:     address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

19:     uint256 public constant LIQUIDATOR_REWARD = 2e17;

20:     uint256 public constant MIN_NET_STETH_BALANCE = 2e18;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="GAS-9"></a>[GAS-9] Increments/decrements can be unchecked in for-loops
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

The change would be:

```diff
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```

These save around **25 gas saved** per instance.

The same can be applied with decrements (which should use `break` when `i == 0`).

The risk of overflow is non-existent for `uint256`.

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

488:             for (uint256 i; i < length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (25)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

176:         if (operation.amountToTransferIn > 0) {

256:         if (ebtcBal > 0) {

260:         if (collateralBal > 0) {

329:         if (beforeSwapsLength > 0) {

347:         if (afterSwapsLength > 0) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

221:         if (_positionManagerPermit.length > 0) {

247:         if (_positionManagerPermit.length > 0) {

294:         if (_positionManagerPermit.length > 0) {

307:         if (_positionManagerPermit.length > 0) {

320:             _stEthBalanceIncrease > 0 || _stEthBalanceDecrease > 0 || _debtChange > 0,

347:         if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

365:         if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

383:         if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

401:         if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

427:         if (_positionManagerPermit.length > 0) {

436:         if (!_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

441:         if (_params.isStEthMarginIncrease && _params.stEthMarginBalance > 0) {

467:         if (_positionManagerPermit.length > 0) {

471:         if (_zapStEthBalanceDiff > 0) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

50:         if (params.zapFeeBPS > 0) {

88:         if (bal > 0) {

100:         if (bal > 0) {

139:         uint256 newColl = _cdp._stEthBalanceIncrease > 0

271:         if (zapFeeBPS > 0) {

292:         if (zapFeeBPS > 0) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="GAS-11"></a>[GAS-11] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (12)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

129:     function _adjustCdpOperation(

162:     function _openCdpOperation(

196:     function _closeCdpOperation(

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

43:     function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {

54:     function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {

59:     function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {

72:     function _transferStEthToCaller(

107:     function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {

115:     function _permitPositionManagerApproval(

135:     function _requireZeroOrMinAdjustment(uint256 _change) internal view {

142:     function _requireAtLeastMinNetStEthBalance(uint256 _stEthBalance) internal pure {

149:     function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Array indices should be referenced via `enum`s rather than via numeric literals | 8 |
| [NC-2](#NC-2) | `constant`s should be defined rather than using magic numbers | 1 |
| [NC-3](#NC-3) | Control structures do not follow the Solidity Style Guide | 7 |
| [NC-4](#NC-4) | Default Visibility for constants | 1 |
| [NC-5](#NC-5) | Function ordering does not follow the Solidity style guide | 2 |
| [NC-6](#NC-6) | Functions should not be longer than 50 lines | 27 |
| [NC-7](#NC-7) | Change int to int256 | 4 |
| [NC-8](#NC-8) | Interfaces should be defined in separate files from their usage | 2 |
| [NC-9](#NC-9) | NatSpec is completely non-existent on functions that should have them | 2 |
| [NC-10](#NC-10) | Incomplete NatSpec: `@param` is missing on actually documented functions | 2 |
| [NC-11](#NC-11) | Incomplete NatSpec: `@return` is missing on actually documented functions | 1 |
| [NC-12](#NC-12) | Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor | 1 |
| [NC-13](#NC-13) | `address`s shouldn't be hard-coded | 1 |
| [NC-14](#NC-14) | `require()` / `revert()` statements should have descriptive reason strings | 6 |
| [NC-15](#NC-15) | Contract does not follow the Solidity style guide's suggested layout ordering | 2 |
| [NC-16](#NC-16) | Use Underscores for Number Literals (add an underscore every 3 digits) | 1 |
| [NC-17](#NC-17) | Internal and private variables and functions names should begin with an underscore | 1 |
| [NC-18](#NC-18) | `public` functions not called by the contract should be declared `external` instead | 1 |
### <a name="NC-1"></a>[NC-1] Array indices should be referenced via `enum`s rather than via numeric literals

*Instances (8)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

231:         swaps[0].tokenForSwap = _tokenIn;

232:         swaps[0].addressForApprove = DEX;

233:         swaps[0].exactApproveAmount = _tradeData.approvalAmount;

234:         swaps[0].addressForSwap = DEX;

235:         swaps[0].calldataForSwap = _tradeData.exchangeData;

237:             swaps[0].swapChecks = _getSwapChecks(_tokenOut, _tradeData.expectedMinOut);

247:         checks[0].tokenToCheck = tokenToCheck;

248:         checks[0].expectedMinOut = expectedMinOut;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-2"></a>[NC-2] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (1)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

150:         uint256 _tmp = uint256(cdpId) >> 96;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="NC-3"></a>[NC-3] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (7)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

130:         if (

139:         } else if (

506:         require(addy != address(this)); // If it could call this it could fake the forwarded caller

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

305:         uint256 _stETHDiff = _zapStEthBalanceAfter - _zapStEthBalanceBefore;

311:         _transferStEthToCaller(_cdpId, EthVariantZapOperationType.CloseCdp, _useWstETH, _stETHDiff);

465:         uint256 _zapStEthBalanceDiff = stEth.balanceOf(address(this)) - _zapStEthBalanceBefore;

476:                 _zapStEthBalanceDiff

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

### <a name="NC-4"></a>[NC-4] Default Visibility for constants
Some constants are using the default visibility. For readability, consider explicitly declaring them as `internal`.

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

37:     bytes32 constant FLASH_LOAN_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="NC-5"></a>[NC-5] Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

1: 
   Current order:
   external Cdps
   public owner
   internal _assertOwner
   external doOperation
   internal _doOperation
   public sweepToCaller
   public sweepToken
   internal _doCheckValueType
   internal _handleOperation
   public decodeFLData
   external onFlashLoan
   internal _doSwaps
   internal _doSwap
   internal _doSwapChecks
   internal _ensureNotSystem
   internal _openCdpCallback
   internal _openCdpForCallback
   internal _closeCdpCallback
   internal _adjustCdpCallback
   internal _claimSurplusCallback
   internal excessivelySafeCall
   
   Suggested order:
   external Cdps
   external doOperation
   external onFlashLoan
   public owner
   public sweepToCaller
   public sweepToken
   public decodeFLData
   internal _assertOwner
   internal _doOperation
   internal _doCheckValueType
   internal _handleOperation
   internal _doSwaps
   internal _doSwap
   internal _doSwapChecks
   internal _ensureNotSystem
   internal _openCdpCallback
   internal _openCdpForCallback
   internal _closeCdpCallback
   internal _adjustCdpCallback
   internal _claimSurplusCallback
   internal excessivelySafeCall

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

1: 
   Current order:
   public owner
   external doOperation
   private _sweepEbtc
   private _sweepStEth
   private _getAdjustCdpParams
   internal _adjustCdpOperation
   internal _openCdpOperation
   internal _closeCdpOperation
   internal _getSwapOperations
   internal _getSwapChecks
   internal _getPostCheckParams
   internal _openCdpForCallback
   internal _adjustCdpCallback
   
   Suggested order:
   external doOperation
   public owner
   internal _adjustCdpOperation
   internal _openCdpOperation
   internal _closeCdpOperation
   internal _getSwapOperations
   internal _getSwapChecks
   internal _getPostCheckParams
   internal _openCdpForCallback
   internal _adjustCdpCallback
   private _sweepEbtc
   private _sweepStEth
   private _getAdjustCdpParams

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-6"></a>[NC-6] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (27)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

15:     function Cdps(bytes32) external view returns (ICdpManagerData.Cdp memory);

39:     function owner() public virtual returns (address) {

267:     function sweepToken(address token, uint256 amount) public {

277:     function _doCheckValueType(CheckValueAndType memory check, uint256 valueToCheck) internal {

327:     function _handleOperation(LeverageMacroOperation memory operation) internal {

388:     function decodeFLData(bytes calldata data) public view returns (LeverageMacroOperation memory) {

432:     function _doSwaps(SwapOperation[] memory swapData) internal {

448:     function _doSwap(SwapOperation memory swapData) internal {

485:     function _doSwapChecks(SwapCheck[] memory swapChecks) internal {

500:     function _ensureNotSystem(address addy) internal {

510:     function _openCdpCallback(bytes memory data) internal virtual {

524:     function _openCdpForCallback(bytes memory data) internal virtual {

540:     function _closeCdpCallback(bytes memory data) internal virtual {

548:     function _adjustCdpCallback(bytes memory data) internal virtual {

562:     function _claimSurplusCallback() internal virtual {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

66:     function owner() public override returns (address) {

270:     function _openCdpForCallback(bytes memory data) internal override {

291:     function _adjustCdpCallback(bytes memory data) internal override {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

14:     function MIN_CHANGE() external view returns (uint256);

34:     function _depositRawEthIntoLido(uint256 _initialETH) internal returns (uint256) {

43:     function _convertWrappedEthToStETH(uint256 _initialWETH) internal returns (uint256) {

54:     function _convertRawEthToStETH(uint256 _initialETH) internal returns (uint256) {

59:     function _convertWstEthToStETH(uint256 _initialWstETH) internal returns (uint256) {

107:     function _transferInitialStETHFromCaller(uint256 _initialStETH) internal returns (uint256) {

135:     function _requireZeroOrMinAdjustment(uint256 _change) internal view {

142:     function _requireAtLeastMinNetStEthBalance(uint256 _stEthBalance) internal pure {

149:     function _getOwnerAddress(bytes32 cdpId) internal pure returns (address) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="NC-7"></a>[NC-7] Change int to int256
Throughout the code base, some variables are declared as `int`. To favor explicitness, consider changing all instances of `int` to `int256`

*Instances (4)*:
```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

234:         cdp.eBTCToMint = _debt;

235:         cdp._upperHint = _upperHint;

236:         cdp._lowerHint = _lowerHint;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

284:                 (flData.eBTCToMint * zapFeeBPS) / BPS

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-8"></a>[NC-8] Interfaces should be defined in separate files from their usage
The interfaces below should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

14: interface ICdpCdps {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

13: interface IMinChangeGetter {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="NC-9"></a>[NC-9] NatSpec is completely non-existent on functions that should have them
Public and external functions that aren't view or pure should have NatSpec comments

*Instances (2)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

66:     function owner() public override returns (address) {

70:     function doOperation(

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-10"></a>[NC-10] Incomplete NatSpec: `@param` is missing on actually documented functions
The following functions are missing `@param` NatSpec comments.

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

265:     /// @notice Transfer an arbitrary token back to you
         /// @dev If you delegatecall into this, this will transfer the tokens to the caller of the DiamondLike (and not the contract)
         function sweepToken(address token, uint256 amount) public {

393:     /// @notice Proper Flashloan Callback handler
         function onFlashLoan(
             address initiator,
             address token,
             uint256 amount,
             uint256 fee,
             bytes calldata data

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="NC-11"></a>[NC-11] Incomplete NatSpec: `@return` is missing on actually documented functions
The following functions are missing `@return` NatSpec comments.

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

393:     /// @notice Proper Flashloan Callback handler
         function onFlashLoan(
             address initiator,
             address token,
             uint256 amount,
             uint256 fee,
             bytes calldata data
         ) external returns (bytes32) {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="NC-12"></a>[NC-12] Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor
If a function is supposed to be access-controlled, a `modifier` should be used instead of a `require/if` statement for more readability.

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

45:         require(owner() == msg.sender, "Must be owner");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="NC-13"></a>[NC-13] `address`s shouldn't be hard-coded
It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

*Instances (1)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

18:     address public constant NATIVE_ETH_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="NC-14"></a>[NC-14] `require()` / `revert()` statements should have descriptive reason strings

*Instances (6)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

502:         require(addy != address(borrowerOperations));

503:         require(addy != address(sortedCdps));

504:         require(addy != address(activePool));

505:         require(addy != address(cdpManager));

506:         require(addy != address(this)); // If it could call this it could fake the forwarded caller

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

51:             require(params.zapFeeReceiver != address(0));

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-15"></a>[NC-15] Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be:

1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions

However, the contract(s) below do not follow this ordering

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

1: 
   Current order:
   FunctionDefinition.Cdps
   UsingForDirective.IERC20
   VariableDeclaration.borrowerOperations
   VariableDeclaration.activePool
   VariableDeclaration.cdpManager
   VariableDeclaration.ebtcToken
   VariableDeclaration.sortedCdps
   VariableDeclaration.stETH
   VariableDeclaration.willSweep
   VariableDeclaration.FLASH_LOAN_SUCCESS
   FunctionDefinition.owner
   FunctionDefinition._assertOwner
   FunctionDefinition.constructor
   EnumDefinition.FlashLoanType
   EnumDefinition.PostOperationCheck
   EnumDefinition.Operator
   StructDefinition.CheckValueAndType
   StructDefinition.PostCheckParams
   FunctionDefinition.doOperation
   FunctionDefinition._doOperation
   FunctionDefinition.sweepToCaller
   FunctionDefinition.sweepToken
   FunctionDefinition._doCheckValueType
   StructDefinition.LeverageMacroOperation
   StructDefinition.SwapOperation
   StructDefinition.SwapCheck
   EnumDefinition.OperationType
   FunctionDefinition._handleOperation
   StructDefinition.OpenCdpOperation
   StructDefinition.OpenCdpForOperation
   StructDefinition.AdjustCdpOperation
   StructDefinition.CloseCdpOperation
   FunctionDefinition.decodeFLData
   FunctionDefinition.onFlashLoan
   FunctionDefinition._doSwaps
   FunctionDefinition._doSwap
   FunctionDefinition._doSwapChecks
   FunctionDefinition._ensureNotSystem
   FunctionDefinition._openCdpCallback
   FunctionDefinition._openCdpForCallback
   FunctionDefinition._closeCdpCallback
   FunctionDefinition._adjustCdpCallback
   FunctionDefinition._claimSurplusCallback
   FunctionDefinition.excessivelySafeCall
   
   Suggested order:
   UsingForDirective.IERC20
   VariableDeclaration.borrowerOperations
   VariableDeclaration.activePool
   VariableDeclaration.cdpManager
   VariableDeclaration.ebtcToken
   VariableDeclaration.sortedCdps
   VariableDeclaration.stETH
   VariableDeclaration.willSweep
   VariableDeclaration.FLASH_LOAN_SUCCESS
   EnumDefinition.FlashLoanType
   EnumDefinition.PostOperationCheck
   EnumDefinition.Operator
   EnumDefinition.OperationType
   StructDefinition.CheckValueAndType
   StructDefinition.PostCheckParams
   StructDefinition.LeverageMacroOperation
   StructDefinition.SwapOperation
   StructDefinition.SwapCheck
   StructDefinition.OpenCdpOperation
   StructDefinition.OpenCdpForOperation
   StructDefinition.AdjustCdpOperation
   StructDefinition.CloseCdpOperation
   FunctionDefinition.Cdps
   FunctionDefinition.owner
   FunctionDefinition._assertOwner
   FunctionDefinition.constructor
   FunctionDefinition.doOperation
   FunctionDefinition._doOperation
   FunctionDefinition.sweepToCaller
   FunctionDefinition.sweepToken
   FunctionDefinition._doCheckValueType
   FunctionDefinition._handleOperation
   FunctionDefinition.decodeFLData
   FunctionDefinition.onFlashLoan
   FunctionDefinition._doSwaps
   FunctionDefinition._doSwap
   FunctionDefinition._doSwapChecks
   FunctionDefinition._ensureNotSystem
   FunctionDefinition._openCdpCallback
   FunctionDefinition._openCdpForCallback
   FunctionDefinition._closeCdpCallback
   FunctionDefinition._adjustCdpCallback
   FunctionDefinition._claimSurplusCallback
   FunctionDefinition.excessivelySafeCall

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

1: 
   Current order:
   FunctionDefinition.MIN_CHANGE
   VariableDeclaration.NATIVE_ETH_ADDRESS
   VariableDeclaration.LIQUIDATOR_REWARD
   VariableDeclaration.MIN_NET_STETH_BALANCE
   VariableDeclaration.MIN_CHANGE
   VariableDeclaration.wstEth
   VariableDeclaration.stEth
   VariableDeclaration.wrappedEth
   FunctionDefinition.constructor
   FunctionDefinition._depositRawEthIntoLido
   FunctionDefinition._convertWrappedEthToStETH
   FunctionDefinition._convertRawEthToStETH
   FunctionDefinition._convertWstEthToStETH
   FunctionDefinition._transferStEthToCaller
   FunctionDefinition._transferInitialStETHFromCaller
   FunctionDefinition._permitPositionManagerApproval
   FunctionDefinition._requireZeroOrMinAdjustment
   FunctionDefinition._requireAtLeastMinNetStEthBalance
   FunctionDefinition._getOwnerAddress
   
   Suggested order:
   VariableDeclaration.NATIVE_ETH_ADDRESS
   VariableDeclaration.LIQUIDATOR_REWARD
   VariableDeclaration.MIN_NET_STETH_BALANCE
   VariableDeclaration.MIN_CHANGE
   VariableDeclaration.wstEth
   VariableDeclaration.stEth
   VariableDeclaration.wrappedEth
   FunctionDefinition.MIN_CHANGE
   FunctionDefinition.constructor
   FunctionDefinition._depositRawEthIntoLido
   FunctionDefinition._convertWrappedEthToStETH
   FunctionDefinition._convertRawEthToStETH
   FunctionDefinition._convertWstEthToStETH
   FunctionDefinition._transferStEthToCaller
   FunctionDefinition._transferInitialStETHFromCaller
   FunctionDefinition._permitPositionManagerApproval
   FunctionDefinition._requireZeroOrMinAdjustment
   FunctionDefinition._requireAtLeastMinNetStEthBalance
   FunctionDefinition._getOwnerAddress

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="NC-16"></a>[NC-16] Use Underscores for Number Literals (add an underscore every 3 digits)

*Instances (1)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

24:     uint256 internal constant BPS = 10000;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="NC-17"></a>[NC-17] Internal and private variables and functions names should begin with an underscore
According to the Solidity Style Guide, Non-`external` variable and function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

568:     function excessivelySafeCall(

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="NC-18"></a>[NC-18] `public` functions not called by the contract should be declared `external` instead

*Instances (1)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

267:     function sweepToken(address token, uint256 amount) public {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | `approve()`/`safeApprove()` may revert if the current approval is not zero | 6 |
| [L-2](#L-2) | Some tokens may revert when zero value transfers are made | 7 |
| [L-3](#L-3) | Do not use deprecated library functions | 2 |
| [L-4](#L-4) | `safeApprove()` is deprecated | 2 |
| [L-5](#L-5) | External call recipient may consume all transaction gas | 3 |
| [L-6](#L-6) | Loss of precision | 3 |
| [L-7](#L-7) | Sweeping may break accounting if tokens with multiple addresses are used | 14 |
| [L-8](#L-8) | Unsafe ERC20 operation(s) | 11 |
### <a name="L-1"></a>[L-1] `approve()`/`safeApprove()` may revert if the current approval is not zero
- Some tokens (like the *very popular* USDT) do not work when changing the allowance from an existing non-zero allowance value (it will revert if the current approval is not zero to protect against front-running changes of approvals). These tokens must first be approved for zero and then the actual allowance can be approved.
- Furthermore, OZ's implementation of safeApprove would throw an error if an approve is attempted from a non-zero value (`"SafeERC20: approve from non-zero to non-zero allowance"`)

Set the allowance to zero immediately before each of the existing allowance calls

*Instances (6)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:         IERC20(swapData.tokenForSwap).safeApprove(

477:         IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

60:         ebtcToken.approve(address(borrowerOperations), type(uint256).max);

61:         stETH.approve(address(borrowerOperations), type(uint256).max);

62:         stETH.approve(address(activePool), type(uint256).max);

63:         stEth.approve(address(wstEth), type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="L-2"></a>[L-2] Some tokens may revert when zero value transfers are made
Example: https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers.

In spite of the fact that EIP-20 [states](https://github.com/ethereum/EIPs/blob/46b9b698815abbfa628cd1097311deee77dd45c5/EIPS/eip-20.md?plain=1#L116) that zero-valued transfers must be accepted, some tokens, such as LEND will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.

*Instances (7)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

177:             IERC20(operation.tokenToTransferIn).safeTransferFrom(

270:         IERC20(token).safeTransfer(msg.sender, amount);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

282:             IERC20(address(ebtcToken)).safeTransfer(

306:                 IERC20(address(ebtcToken)).safeTransfer(

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

45:         wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);

61:             wstEth.transferFrom(msg.sender, address(this), _initialWstETH),

91:             wstEth.transfer(msg.sender, _wstETHVal);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="L-3"></a>[L-3] Do not use deprecated library functions

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:         IERC20(swapData.tokenForSwap).safeApprove(

477:         IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="L-4"></a>[L-4] `safeApprove()` is deprecated
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead. The function may currently work, but if a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to no longer has this function, you'll encounter unnecessary delays in porting and testing replacement contracts.

*Instances (2)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

455:         IERC20(swapData.tokenForSwap).safeApprove(

477:         IERC20(swapData.tokenForSwap).safeApprove(swapData.addressForApprove, 0);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

### <a name="L-5"></a>[L-5] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances (3)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

469:             swapData.calldataForSwap

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

235:         swaps[0].calldataForSwap = _tradeData.exchangeData;

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

38:         payable(address(stEth)).call{value: _initialETH}("");

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="L-6"></a>[L-6] Loss of precision
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

*Instances (3)*:
```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

262:                     value: (_totalCollateral * _collValidationBuffer) / BPS,

284:                 (flData.eBTCToMint * zapFeeBPS) / BPS

308:                     (flData._EBTCChange * zapFeeBPS) / BPS

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="L-7"></a>[L-7] Sweeping may break accounting if tokens with multiple addresses are used
There have been [cases](https://blog.openzeppelin.com/compound-tusd-integration-issue-retrospective/) in the past where a token mistakenly had two addresses that could control its balance, and transfers using one address impacted the balance of the other. To protect against this potential scenario, sweep functions should ensure that the balance of the non-sweepable token does not change after the transfer of the swept tokens.

*Instances (14)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

35:     bool internal immutable willSweep;

58:         bool _sweepToCaller

67:         willSweep = _sweepToCaller;

241:         if (willSweep) {

242:             sweepToCaller();

247:     function sweepToCaller() public {

267:     function sweepToken(address token, uint256 amount) public {

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

47:             false // Do not sweep

81:     function _sweepEbtc() private {

93:     function _sweepStEth() private {

158:         _sweepEbtc();

192:         _sweepEbtc();

193:         _sweepStEth();

220:         _sweepEbtc();

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

### <a name="L-8"></a>[L-8] Unsafe ERC20 operation(s)

*Instances (11)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

257:             ebtcToken.transfer(msg.sender, ebtcBal);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/LeverageZapRouterBase.sol

60:         ebtcToken.approve(address(borrowerOperations), type(uint256).max);

61:         stETH.approve(address(borrowerOperations), type(uint256).max);

62:         stETH.approve(address(activePool), type(uint256).max);

63:         stEth.approve(address(wstEth), type(uint256).max);

89:             ebtcToken.transfer(msg.sender, bal);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/LeverageZapRouterBase.sol)

```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

45:         wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);

61:             wstEth.transferFrom(msg.sender, address(this), _initialWstETH),

91:             wstEth.transfer(msg.sender, _wstETHVal);

103:             stEth.transfer(msg.sender, _stEthVal);

110:         stEth.transferFrom(msg.sender, address(this), _initialStETH);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Contracts are vulnerable to fee-on-transfer accounting-related issues | 2 |
| [M-2](#M-2) | `block.number` means different things on different L2s | 3 |
| [M-3](#M-3) | Return values of `transfer()`/`transferFrom()` not checked | 3 |
| [M-4](#M-4) | Unsafe use of `transfer()`/`transferFrom()` with `IERC20` | 3 |
### <a name="M-1"></a>[M-1] Contracts are vulnerable to fee-on-transfer accounting-related issues
Consistently check account balance before and after transfers for Fee-On-Transfer discrepancies. As arbitrary ERC20 tokens can be used, the amount here should be calculated every time to take into consideration a possible fee-on-transfer or deflation.
Also, it's a good practice for the future of the solution.

Use the balance before and after the transfer to calculate the received amount instead of assuming that it would be equal to the amount passed as a parameter. Or explicitly document that such tokens shouldn't be used and won't be supported

*Instances (2)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

45:         wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);

61:             wstEth.transferFrom(msg.sender, address(this), _initialWstETH),

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="M-2"></a>[M-2] `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

*Instances (3)*:
```solidity
File: ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol

136:                 block.number,

150:                 block.number,

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-protocol/packages/contracts/contracts/LeverageMacroBase.sol)

```solidity
File: ebtc-zap-router/src/EbtcLeverageZapRouter.sol

230:         cdpId = sortedCdps.toCdpId(msg.sender, block.number, sortedCdps.nextCdpNonce());

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/EbtcLeverageZapRouter.sol)

### <a name="M-3"></a>[M-3] Return values of `transfer()`/`transferFrom()` not checked
Not all `IERC20` implementations `revert()` when there's a failure in `transfer()`/`transferFrom()`. The function signature has a `boolean` return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually making a payment

*Instances (3)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

45:         wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);

61:             wstEth.transferFrom(msg.sender, address(this), _initialWstETH),

91:             wstEth.transfer(msg.sender, _wstETHVal);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)

### <a name="M-4"></a>[M-4] Unsafe use of `transfer()`/`transferFrom()` with `IERC20`
Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens.  For example Tether (USDT)'s `transfer()` and `transferFrom()` functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to `IERC20`, their [function signatures](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) do not match and therefore the calls made, revert (see [this](https://gist.github.com/IllIllI000/2b00a32e8f0559e8f386ea4f1800abc5) link for a test case). Use OpenZeppelin's `SafeERC20`'s `safeTransfer()`/`safeTransferFrom()` instead

*Instances (3)*:
```solidity
File: ebtc-zap-router/src/ZapRouterBase.sol

45:         wrappedEth.transferFrom(msg.sender, address(this), _initialWETH);

61:             wstEth.transferFrom(msg.sender, address(this), _initialWstETH),

91:             wstEth.transfer(msg.sender, _wstETHVal);

```
[Link to code](https://github.com/code-423n4/2024-06-badger/blob/main/ebtc-zap-router/src/ZapRouterBase.sol)
