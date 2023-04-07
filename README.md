# Constant Product AMM Liquidity Pool

Here I've built a Constant Product AMM Liquidity Pool that is able to host 2 tokens and records liquidity providers share of liquidity in pool. Users are able to:

- Add liquidity of the 2 tokens into the pool and receive swap fees generated by traders
- Perform token swaps between token0 and token1, paying a 0.3% fee
- Remove their added liquidity including earned fees generated by traders

The reserves of token0 & token1 are controlled internally by the addLiquidity, removeLiquidity & swap functions to prevent users from sending token0/token1 directly into pool and manipulating balances. If token0/token1 balances are manipulated it can mess up the calculations for user's swaps and shares from adding or removing liquidity.

[See Contract File](contracts/CPAMM.sol)

<!-- TABLE OF CONTENTS -->

  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#what-functionality-can-be-added-to-contract">What Functionality Can Be Added To Contract</a></li>
    <li><a href="#adding-liquidity-to-pool">Adding Liquidity To Pool</a></li>
    <li><a href="#removing-liquidity-from-pool">Removing Liquidity From Pool</a></li>
    <li><a href="#token-swaps">Token Swaps</a></li>
    <li><a href="#test-the-cp-amm-contract">Test The CP AMM Contract</a></li>
  </ol>

## What Functionality Can Be Added To Contract

- Contract owner
- Minimum liquidity amount for reserves ratio of each token to set prices
- Record cumulative prices of token0 & token1
- Re-entrancy modifier
- Gas efficient reserves call like UniswapV2 LP's
- Mint fee
- Events for Approval, Transfer, Liquidity Add, Liquidity Removal, Swap, Reserves Update

## Adding Liquidity To Pool

```shell
function addLiquidity(uint _amount0, uint _amount1) external returns(uint shares)
```

In order to add liquidity to the pool, users will first approve the LP to spend the amount of token0 and token1 that the user has specified. If approval is complete, the pool first checks whether the amounts of token0 & token1 being added by the user does not affect the prices of the tokens in the pool. If the prices of the tokens are affect then the pool will throw an error. Once both steps are passed, the user's shares are then calculated using the constant product AMM shares formula which is:

```shell
### If liquidity in pool is equal to 0
- Shares to mint = sqrt of (amount0_added * amount1_added)
```

#### or

```shell
### If liquidity in pool is greater than 0
#   x = amount0_added | y = amount1_added | dx = reserve0 | dy = reserve1 | T = total supply in pool
- Shares to mint based on token0 = dx / x * T
- Shares to mint based on token1 = dy / y * T
```

If shares are greater than 0, then the LP mints the shares to the user and reserve0 and reserve1 are updated at the end of the function.

## Removing Liquidity From Pool

```shell
function removeLiquidity(uint _shares) external returns(uint amount0, uint amount1)
```

First the LP checks the balance of token0 and token1 inside the pool. The amount of liquidity to be withdrawn is then calculated based on the shares that the user has passed in. The calculations of user's shares for token0 and token1 is done using the following formula:

```shell
### Token0 shares calculation
#   dx = token0 amount out | s = shares | x = balance of token0 in LP | T = total supply in pool
- dx = (s * x) / T

### Token1 shares calculation
#   dy = token1 amount out | s = shares | y = balance of token1 in LP | T = total supply in pool
- dy = (s * y) / T
```

A require statement is then put in place to ensure that liquidity being removed is greater than 0. Shares are then burned/removed from the user and total supply in LP. Reserves of token0 and token1 are updated and the liquidity share of token0 & token1 including trading fees are sent to the user.

## Token Swaps

```shell
function swap(address _tokenIn, uint _amountIn) external returns(uint amountOut)
```

For swaps, only the token and amount being put into the pool is passed in. There are 2 requirement checks that are done, namely, ensure that the token being passed in is actually in the pool and that the swap amount is greater than 0. The contract then determines whether the token passed in is token0 or token1. Once this is done, the contract transfers the token from the user, takes a 0.3% swap fee and then transfers the desired token out to the user. The swap fee calculation is as follows:

```shell
### Reduce 0.3% fee from amount put into the pool
- amountInWithFee = (_amountIn * 997) / 1000

### Calculate the conversion from tokenIn to tokenOut
#   dy = tokenOut amount | y = reserveOut | x = reserveIn | dx = amountInWithFee
- dy = (y * dx) / (x + dx)

```

Finally, the reserves are updated with the user's swap effect on tokens.

## Test The CP AMM Contract

To test this, you can download or clone this repository and do the following:

```shell
# Install Hardhat:
npm i -D hardhat

# Run this and click enter 4 times:
npx hardhat

# Install dependencies:
npm install --save-dev @nomicfoundation/hardhat-chai-matchers
npm i -D @nomiclabs/hardhat-waffle

# Run the test file using:
npx hardhat test test/cpamm_test.js
```
