# Aave Pool Codebase Review

Aave is a decentralized non-custodial liquidity market protocol where users can participate as suppliers or borrowers.

## Suppliers & Borrowers

Suppliers provide liquidity to the market and earn a passive income, while borrowers are able to borrow unexpected expenses, leveraging their holding.

Liquidity is at the heart of the Aave Protocol as it enables the protocol's operation and user experience.

The liquidity of the protocol is measured by the availability of assets for basic protocol operations such as borrowing of assets backed by collateral and claiming of supplied assets along with accrued yield. Thus, a lack of liquidity will block operations.

## How Aave Works

Aave works through lending pools

- A lending pool is a smart contract that allows users to deposit and borrow money( crypto currencies)

Every lending pool has a specific crypto currency. Lenders deposits tokens into a lending pool containing the same type of token or crypto currency.

For example: ETH owners will deposit into an ETH lending pool or USDC owners will deposit into a USDC lending pool.

Just like banks, lenders or depositors will be able to earn interest and borrowers can borrow various assets at the cost of interest (which means they pay interest for borrowing).

The rate of lending or borrowing is reflected in annualized figure called Annual Percentage Yield (APY) or Annual Percentage Rate (APR).

## LENDING

When lenders deposits funds into a lending pool, the interest they earn, accumulates in form of *aTokens*. The letter "a" stands for Aave.

aTokens are interest bearing tokens, native to the Aave protocol, that are minted and burned upon deposit and withdrawal of assets from the lending pools.

Each aToken it pegged to the value of the corresponding cryptocurrency in the lending pool. Which means if you deposit funds into the ETH lending pool, you will receive aETH which will increase in amount as it passively accrues interest in your wallet.

## BORROWING

To borrow cryptocurrency, you must deposit tokens that are worth more than the amount you will like to borrow.

This is known as overcollaterization. Once this is done, borrowers can borrow a certain percentage of their collateral's value in dollars.

The percentage is known as he collateral ratio, which is predetermined by the protocol or DAO.

## Liquidation in Aave

If you want to borrow $80 worth of USDT from AAVE and you are bringing ETH as collateral, you will need to deposit $100 worth of ETH.

Now if ETH increases in value over the period while you still held the USDT, when you bring back the USDT, and get back you ETH, you will return $80USDT but to get back your ETH which the value must has increased to $200 because of the increased value over time. On this example, this is profitable

Looking at this from this other example: let's say you want to borrowed $80 USDT and want to deposit ETH, the Maximum Loan Value of ETH is 80% which means to borrow $80 USDT you must deposit $100 worth of ETH. So you deposited collateral in ETH worth $100.
Then while you are still holding the USDT, if the price of ETH drops to more than 82.5% of its value which is the liquidation percentage, AAVE will automatically take back your ETH and pay the lender and you will have to keep the $80 USDT that you borrowed.

## Pool contract review

The pool contract provides the main point of interaction with an Aave protocol's market. With it, users can perform the actions listed below;

- Supply: Users can supply assets to the pool.
- Withdraw: Users can withdraw assets from the pool.
- Borrow: Users can borrow assets from the pool.
- Repay: Users can repay borrowed assets.
- Swap: Users can swap their loans between variable and stable interest rates.
- Enable/Disable Collateral: Users can enable or disable supplied assets as collateral.
- Rebalance Stable Rate Borrow Positions: Users can rebalance their stable rate borrow positions.
- Liquidate Positions: Users can liquidate positions.
- Execute Flash Loans: Users can execute flash loans.

The contract is covered by a proxy contract, and is also owned by the `PoolAddressesProvider` of the specific market.

All admin functions are callable by the `PoolConfigurator` contract defined also in the `PoolAddressesProvider`.

The Pool.sol contract is the main user facing contract of the protocol. It exposes the liquidity management methods.

The `Pool.sol` is owned by the `PoolAddressedProvider` of the specific market.

All admin functions are callable by the `PoolConfigurator` contract, which is defined in the `PoolAddressesProvider`.

The contract is inheriting `VersionedInitializable, PoolStorage, IPool` contracts.

The `VersionedInitializable` is inspired by the OpenZeppelin Initializable contract. It is a helper contract used to implement initializer functions for the purpose  of upgrade-ability. It is used to ensure that initialization is explicit rather than automatic, providing more control and flexibility during contract deployment and upgrades.

The `PoolStorage` contract is used as storage of the `Pool` contract. It defines the storage layout of the `Pool` contract.

The `PoolConfigurator` contract implements the configuration methods for the Aave protocol.

Below is a diagram representing the flow of the Pool `contract`

![Pool_Flow.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/c34bb45a-b5f5-457b-b330-6bfb67cf8a48/9dc034e0-7556-4a16-a802-97a92e22fb55/Pool_Flow.webp)

`using ReserveLogic for DataTypes.ReserveData;`:

This line code block above means that the `ReserveLogic` library is being used for the `ReserveData` struct, which is defined in the `DataTypes` library. It allows functions from `ReserveLogic` to be used with the `ReserveData` struct.

`uint256 public constant POOL_REVISION = 0x1;`:

This line declares a public constant variable `POOL_REVISION` with the value `0x1` (hexadecimal 1). This variable is used to track the revision of the contract.

`IPoolAddressesProvider public immutable ADDRESSES_PROVIDER;`:

here, we have a public immutable state variable `ADDRESSES_PROVIDER` of type `IPoolAddressesProvider`. It's initialized during contract creation and cannot be changed after that.

`modifier onlyPoolConfigurator()`:

This is a custom modifier named `onlyPoolConfigurator`. It is used to restrict access to certain functions to only the pool configurator.

`modifier onlyPoolAdmin()`:

This is a custom modifier named `onlyPoolAdmin`. It is used to restrict access to certain functions to only the pool admin.

`modifier onlyBridge()`:

This is a custom modifier named `onlyBridge`. It is used to restrict access to certain functions to only the bridge.

`function _onlyPoolConfigurator() internal view virtual`:

This internal function is used by the `onlyPoolConfigurator` modifier. It checks whether the sender of the transaction is the pool configurator, based on the address provided by `ADDRESSES_PROVIDER`.

`function _onlyPoolAdmin() internal view virtual`:

This internal function is used by the `onlyPoolAdmin` 
modifier. It checks whether the sender of the transaction is a pool 
admin by consulting the ACL (Access Control List) manager contract 
provided by `ADDRESSES_PROVIDER`.

`function _onlyBridge() internal view virtual`:

This internal function is used by the `onlyBridge` modifier. It checks whether the sender of the transaction is a bridge by consulting the ACL manager contract provided by `ADDRESSES_PROVIDER`.

`function getRevision() internal pure virtual override returns (uint256)`:

This internal function is a requirement for contracts that inherit from `VersionedInitializable`. It returns the revision of the contract, which is defined as `POOL_REVISION`.

The constructor initializes the `ADDRESSES_PROVIDER` variable when the contract is deployed. It takes an argument of type `IPoolAddressesProvider` and sets it as an immutable variable. This contract is designed to work with the provided `IPoolAddressesProvider`.

```jsx
function initialize(IPoolAddressesProvider provider) external virtual initializer {
    require(provider == ADDRESSES_PROVIDER, Errors.INVALID_ADDRESSES_PROVIDER);
    _maxStableRateBorrowSizePercent = 0.25e4;
  }
```

This function is called by the proxy contract when the Pool contract is added to the `PoolAddressesProvider` of the market. It caches the address of the `PoolAddressesProvider`. Caching the address is meant to reduce gas consumption because the address only needs to be determined once during initialization and then used throughout the contract's lifetime.

`require(provider == ADDRESSES_PROVIDER, Errors.INVALID_ADDRESSES_PROVIDER)`:

This  `require` statement above is an assertion that must be true for the function to execute. It checks if the provided `provider` argument matches the contract's `ADDRESSES_PROVIDER`. If they do not match, it raises an exception with the error message "INVALID_ADDRESSES_PROVIDER.

Then it sets the value of the maximum size of stable rate borrowings.

```solidity
function mintUnbacked(
address asset, 
uint256 amount, 
address onBehalfOf, 
uint16 referralCode
) external virtual override onlyBridge:
```

- This is a function definition with the following parameters:
    - `address asset`: The address of the asset (token) to be minted.
    - `uint256 amount`: The amount (quantity) of the asset to be minted.
    - `address onBehalfOf`: The address on behalf of which the minting operation is performed.
    - `uint16 referralCode`: A referral code, which might be used for tracking or rewarding referrals in the context of the operation.
- The function is marked as `external`, indicating it can be called from outside the contract. It's also marked as `virtual` and uses the `override` keyword, meaning it can be overridden by derived contracts that inherit from the same interface.
- The `onlyBridge` modifier restricts access to this function. Only transactions initiated by an address that is recognized as a "bridge" can call this function.

1. `BridgeLogic.executeMintUnbacked(...)`:
    - This line calls function `executeMintUnbacked` from a library named `BridgeLogic`. It passes the following parameters :
        - `_reserves`:  state variable representing reserves.
        - `_reservesList`: A list of reserves.
        - `_usersConfig[onBehalfOf]`: This is a user-specific configuration or data associated with the `onBehalfOf` address.
        - `asset`: The address of the asset to be minted.
        - `amount`: The quantity of the asset to be minted.
        - `onBehalfOf`: The address on behalf of which the minting operation is performed.
        - `referralCode`: The referral code associated with the operation.

The `mintUnbacked` function is used to mint a specific asset on behalf of a user. The minting operation is carried out by invoking an external function from the `BridgeLogic` contract or library. Access to this function is restricted to addresses recognized as "bridge" addresses, as enforced by the `onlyBridge` modifier.

```solidity
function backUnbacked(
address asset, 
uint256 amount, 
uint256 fee
) external virtual override onlyBridge returns (uint256):
```

This is a function definition with the following parameters:

- `address asset`: The address of the asset (token) that is being backed (returned).
- `uint256 amount`: The amount (quantity) of the asset to be backed.
- `uint256 fee`: The fee associated with the backing operation.

The function is marked as `external`, indicating it can be called from outside the contract. It's also marked as `virtual` and uses the `override` keyword, meaning it can be overridden by derived contracts that inherit from the same interface.

- The `onlyBridge` modifier restricts access to this function. Only transactions initiated by an address recognized as a "bridge" can call this function.
- The function returns a `uint256` value, which seems to be the result of the backing operation.

Inside the function, it calls the `executeBackUnbacked` function from the `BridgeLogic` contract.

It passes several arguments to this function:

- `_reserves[asset]`: It appears to be a specific reserve related to the asset.
- `asset`: The address of the asset that is being backed.
- `amount`: The quantity of the asset to be backed.
- `fee`: The fee associated with the backing operation.
- `_bridgeProtocolFee`: This is a protocol fee associated with the bridge operation.

**`funtion supply()`**

The `supply()` function supplies an amount of underlying asset into the reserve, receiving in return overlying aTokens. These aTokens serve as a receipt that a user supplied assets to the pool.

Eg: User supplies 100 USDC and gets in return 100 aUSDC

```solidity
function supply(
address asset,
uint256 amount,
address onBehalfOf,
uint16 referralCode
) external;
```

- `asset`: address of the asset being supplied to the pool
- `amount`: amount of asset being supplied to the pool
- `onBehalfOf`: address that will receive the corresponding aTokens. Note: Only the `onBehalfOf` address will be able to withdraw asset from the pool
- `referralCode`: unique code for 3rd party referral program integration. Use 0 (zero) for no referral.

Inside the function we are going to find that it is only making a call to an internal function `executeSupply()` which is inside `SupplyLogic.sol`:

- `asset`: address of the asset being supplied to the pool
- `amount`: amount of asset being supplied to the pool
- `onBehalfOf`: address that will receive the corresponding aTokens. Note: Only the `onBehalfOf` address will be able to withdraw asset from the pool
- `referralCode`: unique code for 3rd party referral program integration. Use 0 (zero) for no referral.

You will notice `DataTypes.ExecuteSupplyParams()` call inside the above function's parameter. It is simply passing a list of parameters wrapped in a struct.

Now let's look into the `executeSupply()` function found in the `SupplyLogic.sol` contract.

We can split it in three parts:

- The updates and validation:

```
reserve.updateState(reserveCache);

ValidationLogic.validateSupply(reserveCache, reserve, params.amount);

reserve.updateInterestRates(reserveCache, params.asset, params.amount, 0);
```

- From the code above, the first function `resereve.updateState(reserveCache)` will update the reserves with the provided `reserveData` from the specific asset passed as argument.
- Right after it, it will pass this data to validation where the main checks will be done, this asset fulfills the following:

```
require(isActive, Errors.RESERVE_INACTIVE);
require(!isPaused, Errors.RESERVE_PAUSED);
require(!isFrozen, Errors.RESERVE_FROZEN);
```

The above checks is inside the `validateSupply()` function found in `the ValidationLogic.sol`

- The last thing is to update the interest rates with the `updateInterestRates()` function, which receives as parameters, the reserves cached, the specific asset and the amount of the asset.
- The main action of the function is where the supply of the ERC20 token is done, by using the `safeTransferFrom()` function and the `mint()` of aTokens. See the code below:

```
IERC20(params.asset).safeTransferFrom(msg.sender, reserveCache.aTokenAddress, params.amount);
```

```
IAToken(reserveCache.aTokenAddress).mint(msg.sender, params.onBehalfOf, params.amount, reserveCache.nextLiquidityIndex);
```

- Setting the Collateral Value This will only happen if this is the first time the sender is making a supply. That is why in the code you will see a condition `isFirstSupply` The validation before modifying anything and finally the setter.

```solidity
function supplyWithPermit(
    address asset,
    uint256 amount,
    address onBehalfOf,
    uint16 referralCode,
    uint256 deadline,
    uint8 permitV,
    bytes32 permitR,
    bytes32 permitS
  ) public virtual override {
    IERC20WithPermit(asset).permit(
      msg.sender,
      address(this),
      amount,
      deadline,
      permitV,
      permitR,
      permitS
    );
    SupplyLogic.executeSupply(
      _reserves,
      _reservesList,
      _usersConfig[onBehalfOf],
      DataTypes.ExecuteSupplyParams({
        asset: asset,
        amount: amount,
        onBehalfOf: onBehalfOf,
        referralCode: referralCode
      })
    );
  }
```

The function is used for supplying assets to the protocol, and it leverages the ERC-2612 and ERC-713 standards for permit-based token transfers.

- `address asset`: This parameter specifies the address of the underlying asset (token) that the user wants to supply to the protocol.
- `uint256 amount`: This parameter specifies the amount of the asset to be supplied.
- `address onBehalfOf`: The address that will receive the aTokens, same as `msg.sender` if the user wants to receive them on his own wallet, or a different address if the beneficiary of aTokens is a different wallet
- `uint256 deadline`: This parameter specifies the deadline timestamp until which the permit signature is valid. After this timestamp, the permit is no longer considered valid.
- `uint16 referralCode`: This parameter is a code used to register the integrator or originating entity of the operation. It's often used for tracking and potentially rewarding integrators or middle-men who facilitate the supply.
- `uint8 permitV`, `bytes32 permitR`, `bytes32 permitS`: These parameters collectively represent the components of a permit signature generated according to the ERC-2612 standard. A permit signature allows a user to give permission for the protocol to spend their tokens without executing a direct transaction. This is commonly used to save gas costs and simplify interactions.

Inside the function, it calls the `permit` function from the `IERC20WithPermit`
 contract, passing in several parameters. These parameters include the 
sender, the contract address, the amount, the deadline, and the permit 
parameters.

The parameters provided to the `permit` function includes:

- `msg.sender`: The address of the user calling the function (the user's wallet).
- `address(this)`: The address of the contract itself, indicating the address that is authorized to spend the user's tokens.
- `amount`: The quantity of tokens being authorized for spending.
- `deadline`: The deadline timestamp until which the permit signature is valid.
- `permitV`, `permitR`, `permitS`: Components of the permit signature generated by the user.

After the permit is executed, it calls the `executeSupply` function from the `SupplyLogic` contract, passing in several parameters. These parameters include the reserves, reserves list, user configuration, and an `ExecuteSupplyParams` struct that includes the asset, amount, user, and referral code.

[`**function withdraw()**`](https://github.com/casweeney/aave-v3-protocol-breakdown/blob/main/breakdown.md#function-withdraw)

This function allows a user to withdraw an `amount` of underlying asset from the reserve, burning the equivalent aTokens owned

- E.g. User has 100 aUSDC, calls `withdraw()` and receives 100 USDC, burning the 100 aUSDC

This is the description provided in the `IPool` interface codebase and it is an excellent summary of what happens inside this function.

```solidity
function withdraw(
  address asset,
  uint256 amount,
  address to
) external returns (uint256);
```

Just like the `supply()` function, `withdraw()` function is also directly calling to the internal function `executeWithdraw()` which is inside the `SupplyLogic.sol`

```solidity
SupplyLogic.executeWithdraw(
  _reserves,
  _reservesList,
  _eModeCategories,
  _usersConfig[msg.sender],
  DataTypes.ExecuteWithdrawParams({
    asset: asset,
    amount: amount,
    to: to,
    reservesCount: _reservesCount,
    oracle: ADDRESSES_PROVIDER.getPriceOracle(),
    userEModeCategory: _usersEModeCategory[msg.sender]
  })
);
```

Let's divide what is happening inside the `executeWithdraw()` function into two parts:

- Update and validate

```solidity
ValidationLogic.validateWithdraw(reserveCache, amountToWithdraw, userBalance);

reserve.updateInterestRates(reserveCache, params.asset, 0, amountToWithdraw);
```

Just like in the `executeSupply()` it validates the provided parameters in order to update interest rates.

Then it will check one of the parameters provided which is `isUsingAsCollateral()`. If this returns true and the amount desired to withdraw is as high as the `userBalance`, then it will cancel the existing collateral.

```solidity
if (isCollateral && amountToWithdraw == userBalance) {
  userConfig.setUsingAsCollateral(reserve.id, false);
}
```

- Burn the aToken: Burns aTokens from the user and sends the equivalent amount of underlying token to the address specified in the `params.to` parameter.

```solidity
IAToken(reserveCache.aTokenAddress).burn(msg.sender, params.to, amountToWithdraw, reserveCache.nextLiquidityIndex);
```

**[`function borrow()`](https://github.com/casweeney/aave-v3-protocol-breakdown/blob/main/breakdown.md#function-borrow)**

The `borrow()` function allows users to borrow a specific amount of the reserve underlying asset, provide that the borrower already supplied enough collateral, or he was given enough allowance by a credit delegator on the corresponding debt token (StableDebtToken or VariableDebtToken)

Eg: User borrows 100USDC passing as `onBehalfOf` his own address, receiving the 100 USDC in his wallet and 100 stable/variable debt tokens, depending on the `interestRateMode`

```solidity
function borrow(
    address asset,
    uint256 amount,
    uint256 interestRateMode,
    uint16 referralCode,
    address onBehalfOf
) external;
```

Inside the `borrow()` function and like in the previous functions, it will directly call an internal function `executeBorrow()` which is inside the `BorrowLogic.sol` see code below:

```solidity
BorrowLogic.executeBorrow(
    _reserves,
    _reservesList,
    _eModeCategories,
    _usersConfig[onBehalfOf],
    DataTypes.ExecuteBorrowParams({
        asset: asset,
        user: msg.sender,
        onBehalfOf: onBehalfOf,
        amount: amount,
        interestRateMode: DataTypes.InterestRateMode(interestRateMode),
        referralCode: referralCode,
        releaseUnderlying: true,
        maxStableRateBorrowSizePercent: _maxStableRateBorrowSizePercent,
        reservesCount: _reservesCount,
        oracle: ADDRESSES_PROVIDER.getPriceOracle(),
        userEModeCategory: _usersEModeCategory[onBehalfOf],
        priceOracleSentinel: ADDRESSES_PROVIDER.getPriceOracleSentinel()
    })
);
```

One thing to highlight from the list of parameters passed in the code above is the `interestRateMode`. It is basically an ENUM. It determines which debt token will be minted.

```solidity
enum InterestRateMode {
  NONE,
  STABLE,
  VARIABLE
}
```

If the `interestRateMode` is `STABLE`, it will execute:

```solidity
IStableDebtToken(reserveCache.stableDebtTokenAddress).mint(params.user, params.onBehalfOf, params.amount, currentStableRate);

```

And if the `interestRateMode` is `VARIABLE` it will execute:

```solidity
IVariableDebtToken(reserveCache.variableDebtTokenAddress).mint(params.user, params.onBehalfOf, params.amount, reserveCache.nextVariableBorrowIndex);

```

Once the tokens are minted, what's left is the actual transfer of the asset/underlying to the user:

`IAToken(reserveCache.aTokenAddress).transferUnderlyingTo(params.user, params.amount);`

**[`function repay()`](https://github.com/casweeney/aave-v3-protocol-breakdown/blob/main/breakdown.md#function-repay)**

The `repay()` function is called to repay a borrowed amount on a specific reserve, burning the equivalent debt tokens owned.

Eg: User repays 100USDC, burning 100 variable/stable debt tokens of the `onBehalfOf` address

```solidity
function repay(
  address asset,
  uint256 amount,
  uint256 interestRateMode,
  address onBehalfOf
) external returns (uint256);
```

Just like in other functions, `repay()` function is also calling an internal function `executeRepay()` inside the `BorrowLogic.sol`:

```solidity
BorrowLogic.executeRepay(
  _reserves,
  _reservesList,
  _usersConfig[onBehalfOf],
  DataTypes.ExecuteRepayParams({
    asset: asset,
    amount: amount,
    interestRateMode: DataTypes.InterestRateMode(interestRateMode),
    onBehalfOf: onBehalfOf,
    useATokens: false
  })
);
```

Notice from the parameters. `useATokens` which in this case is directly set to `false`. This is because there is another method which allows user to repay with aTokens without leaving dust from interest and it's called `repayWithAtokens()`.

This means that when users want to repay their debt, it will be by either:

- Burning their stable/variable debt token.

```solidity
IStableDebtToken(reserveCache.stableDebtTokenAddress).burn(params.onBehalfOf, paybackAmount);
```

```solidity
IVariableDebtToken(reserveCache.variableDebtTokenAddress).burn(params.onBehalfOf, paybackAmount, reserveCache.nextVariableBorrowIndex);
```

Or by:

- Burning the aTokens received when providing liquidity through `Supply()` function

```solidity
IAToken(reserveCache.aTokenAddress).burn(msg.sender, reserveCache.aTokenAddress, paybackAmount, reserveCache.nextLiquidityIndex);
```

The difference between above two methods of repaying is that:

In the case of using aTokens to repay, the transaction is finished (there is actually lack of any transfer) because as you may remember, we mentioned that if the user wants to withdraw the asset supplied, the equivalent aTokens will be returned. Which means if a user repays with aTokens, they won't be able to withdraw the asset supplied since they don't have anymore equivalent aTokens to return.

Otherwise, there will still be the need to execute the `IERC20().safeTransferFrom()` to the specified amount from `msg.sender`

`IERC20(params.asset).safeTransferFrom(msg.sender, reserveCache.aTokenAddress, paybackAmount);

IAToken(reserveCache.aTokenAddress).handleRepayment(msg.sender, params.onBehalfOf, paybackAmount);`

`fucnction repayWithPermit()`

The `repayWithPermit` function is used to repay a borrowed debt, with the specific amount, interest rate mode, and on behalf of a designated user, using a permit-based transfer. This function leverages the ERC-2612 and ERC-713 standards for permit functionality.

```solidity
function repayWithPermit(
    address asset,
    uint256 amount,
    uint256 interestRateMode,
    address onBehalfOf,
    uint256 deadline,
    uint8 permitV,
    bytes32 permitR,
    bytes32 permitS
  ) public virtual override returns (uint256) {
    {
      IERC20WithPermit(asset).permit(
        msg.sender,
        address(this),
        amount,
        deadline,
        permitV,
        permitR,
        permitS
      );
    }
    {
      DataTypes.ExecuteRepayParams memory params = DataTypes.ExecuteRepayParams({
        asset: asset,
        amount: amount,
        interestRateMode: DataTypes.InterestRateMode(interestRateMode),
        onBehalfOf: onBehalfOf,
        useATokens: false
      });
      return BorrowLogic.executeRepay(_reserves, _reservesList, _usersConfig[onBehalfOf], params);
    }

```

Find below the parameters for the function;

- `address` `asset`: The address of the borrowed underlying asset previously borrowed
- `uint256` `amount`: The amount to repay. You can Send the value type(uint256).max in order to repay the whole debt for `asset` on the specific `debtMode`
- `uint256` `interestRateMode:` The interest rate mode at of the debt the user wants to repay: 1 for Stable, 2 for Variable
- `address` `onBehalOf:` Address of the user who will get his debt reduced/removed. Should be the address of then user calling the function if he wants to reduce/remove his own debt, or the address of any other borrower whose debt should be removed.
- `uint256` `deadline`: The deadline timestamp that the permit is valid
- `uint8` `permitV:` The V parameter of ERC712 permit sig
- `bytes32` `permitR:` The R parameter of ERC712 permit sig
- `bytes32` `permitS` : The S parameter of ERC712 permit sig

- `ERC20WithPermit(asset).permit(...)`:
    - This block of code calls the `permit` function on an ERC-20 token contract that supports the permit functionality (typically implemented using the ERC-2612 standard). It allows the user to provide a permit signature to authorize the contract to spend a certain amount of their tokens.
    - The parameters provided to the `permit` function include the user's address (`msg.sender`), the contract's address (`address(this)`), the amount to be spent, the permit's deadline, and the permit signature components (`permitV`, `permitR`, `permitS`).
- `DataTypes.ExecuteRepayParams memory params = DataTypes.ExecuteRepayParams({ ... })`:
    - This part creates a `DataTypes.ExecuteRepayParams` struct that encapsulates the parameters required for the repay operation. It specifies the asset to be repaid, the amount, the interest rate mode, the user on whose behalf the debt will be reduced/removed, and whether aTokens are used.
- `return BorrowLogic.executeRepay(...)`:
    - This line returns the result of calling the `executeRepay` function from the library named `BorrowLogic`. It passes several arguments to this function, including the user's repayment details, as defined in the `params` struct.
    - The `executeRepay` function carries out the actual repayment operation and returns the final amount repaid.

`**function repayWithATokens()**`

This function is used to repay a borrowed `amount` on a specific reserve using the reserve aTokens, burning the equivalent debt tokens.

E.g. User repays 100 USDC using 100 aUSDC, burning 100 variable/stable debt tokens

 Note that passing uint256.max as amount will clean up any residual aToken dust balance, if the user aToken balance is not enough to cover the whole debt.

```solidity
function repayWithATokens(
    address asset,
    uint256 amount,
    uint256 interestRateMode
  ) public virtual override returns (uint256) {
    return
      BorrowLogic.executeRepay(
        _reserves,
        _reservesList,
        _usersConfig[msg.sender],
        DataTypes.ExecuteRepayParams({
          asset: asset,
          amount: amount,
          interestRateMode: DataTypes.InterestRateMode(interestRateMode),
          onBehalfOf: msg.sender,
          useATokens: true
        })
      );
  }
```

The function takes in the parameters of ;

`address` `asset` : The address of the borrowed underlying asset previously borrowed

`uint256` `amount` : The amount to repay

`uint256` `interestRateMode`: The interest rate mode at of the debt the user wants to repay: 1 for Stable, 2 for Variable

Then the final amount repaid is returned 

- `return BorrowLogic.executeRepay(...)`:
- This line immediately returns the result of calling the `executeRepay` function from the library named `BorrowLogic`. It passes several arguments to this function, including the user's repayment details, as defined in a `DataTypes.ExecuteRepayParams` struct.
- The `executeRepay` function carries out the actual repayment operation, which involves burning aTokens and debt tokens.
- The provided comment above the function explains the purpose and usage of the `repayWithATokens` function:
    - "Repays a borrowed `amount` on a specific reserve using the reserve aTokens, burning the equivalent debt tokens."
    - It elaborates on specific use cases:
        - Users can repay a certain amount of a specific reserve's borrowed underlying asset using a corresponding amount of aTokens. This action also burns a corresponding amount of variable or stable debt tokens.
        - Users can choose to repay their entire debt by sending `type(uint256).max` as the amount.
        - Users can specify the interest rate mode (1 for Stable or 2 for Variable) at which they want to repay the debt.

**`function swapBorrowRateMode()`**

This function allows a borrower to swap his debt between stable and variable mode, or vice versa.

The function takes two parameters:

- `asset`: This is the address of the underlying asset that the borrower wants to
swap.
- `interestRateMode`: This is an integer that represents the current interest rate mode of
the position being swapped. The value 1 represents a Stable interest
rate mode, and the value 2 represents a Variable interest rate mode.

The function first calls `BorrowLogic.executeSwapBorrowRateMode` with the following parameters:

- `_reserves[asset]`: This is the reserves of the asset that the borrower wants to swap. The
reserves represent the total amount of the asset that is currently
available for borrowing.
- `_usersConfig[msg.sender]`: This is the configuration for the borrower who is calling the function. This configuration includes details about the borrower's borrowing
behavior.
- `asset`: This is the address of the asset that the borrower wants to swap.
- `DataTypes.InterestRateMode(interestRateMode)`: This is the mode of the interest rate for the borrowing. It is converted to the appropriate data type using `DataTypes.InterestRateMode`.

`**function**` **`rebalanceStableBorrowRate()`**

The `rebalanceStableBorrowRate` function rebalances the stable interest rate of a user to the current stable rate defined on the reserve.

- It outlines the conditions under which users can be rebalanced:
    - The usage ratio, which represents the proportion of collateral used for borrowing, is above 95%.
    - The current supply Annual Percentage Yield (APY) is below a certain threshold (`REBALANCE_UP_THRESHOLD` times the maximum variable borrow rate). This condition indicates that too much has been borrowed at a stable rate, and suppliers are not earning enough.
- The function parameters are described as:
    - `asset`: The address of the underlying asset that was borrowed.
    - `user`: The address of the user to be rebalanced.

`**function setUserUseReserveAsCollateral()**`

The function allows suppliers to enable/disable a specific supplied asset as collateral.

```solidity
function setUserUseReserveAsCollateral(
    address asset,
    bool useAsCollateral
  ) public virtual override {
    SupplyLogic.executeUseReserveAsCollateral(
      _reserves,
      _reservesList,
      _eModeCategories,
      _usersConfig[msg.sender],
      asset,
      useAsCollateral,
      _reservesCount,
      ADDRESSES_PROVIDER.getPriceOracle(),
      _usersEModeCategory[msg.sender]
    );
  }
```

The function takes two parameters:

1. `asset`: This is the address of the underlying asset supplied.
2. `useAsCollateral`: It is set to true if the user wants to use the supply as collateral,  or false if otherwise.

The function then calls the the `executeUseReserveAsCollateral` function from the `SupplyLogic` library taking in the following parameters:

- `_reserves` : The state of all the reserves
- `_reservesList` : The addresses of all the active reserves
- `_eModeCategories` : The configuration of all the efficiency mode categories
- `_usersConfig[msg.sender]` : The users configuration mapping that track the supplied/borrowed assets, in the case `msg.sender`
- `asset` : The address of the asset being configured as collateral
- `useAsCollateral` : It is set to true if the user wants to use the supply as collateral,  or false if otherwise.
- `_reservesCount` : The number of initialized reserves
- `ADDRESSES_PROVIDER.getPriceOracle()` : The address of the price oracle
- `_usersEModeCategory[msg.sender]` : The eMode category chosen by the user

With this function, users can choose whether they want to use the supplied asset as collateral within the protocol by setting the `useAsCollateral` flag to either `true`or `false`. This provides users with flexibility in managing their positions and collateral requirements within the lending protocol.

**`function liquidationCall()`**

This a  function to liquidate a non-healthy position collateral-wise, with Health Factor below 1

- The caller (liquidator) covers `debtToCover` amount of debt of the user getting liquidated, and receives a proportional amount of the `collateralAsset` plus a bonus to cover market risk.

```solidity
function liquidationCall(
    address collateralAsset,
    address debtAsset,
    address user,
    uint256 debtToCover,
    bool receiveAToken
  ) public virtual override {
    LiquidationLogic.executeLiquidationCall(
      _reserves,
      _reservesList,
      _usersConfig,
      _eModeCategories,
      DataTypes.ExecuteLiquidationCallParams({
        reservesCount: _reservesCount,
        debtToCover: debtToCover,
        collateralAsset: collateralAsset,
        debtAsset: debtAsset,
        user: user,
        receiveAToken: receiveAToken,
        priceOracle: ADDRESSES_PROVIDER.getPriceOracle(),
        userEModeCategory: _usersEModeCategory[user],
        priceOracleSentinel: ADDRESSES_PROVIDER.getPriceOracleSentinel()
      })
    );

```

Upon calling the function, the parameters below are used:

- `address collateralAsset` : The address of the underlying asset used as collateral, to receive as result of the liquidation
- `address debtAsset` : The address of the underlying borrowed asset to be repaid with the liquidation
- `address user` :  The address of the borrower getting liquidated
- `uint256 debtToCover` : The debt amount of borrowed `asset` the liquidator wants to cover
- `bool receiveAToken`:  Set to true if the liquidators wants to receive the collateral in aTokens, `false` if he wants to receive the underlying collateral asset directly

Then the `executeLiquidationCall()` from `LiquidationLogic`  library get  triggered taking the parameters of;

- `_reserves` : The state of all the reserves
- `_reservesList` : The addresses of all the active reserves
- `_eModeCategories` : The configuration of all the efficiency mode categories

the `ExecuteLiquidationCallParams()` from the `DataTypes` library is also called as a parameter