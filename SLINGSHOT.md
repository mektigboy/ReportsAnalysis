# SLINGSHOT

## High Severity

### [H-01] Incorrectly encoded arguments to `executeTrades()` can allow tokens to be stolen

This inding combines a couple weaknesses into one attack. The first weakness is a lack of validation on arguments to `executeTrades`, the second is that a pre-existing `fromToken` balance can be used in a trade:

1. Alice wants to convert 1000 DAI to WETH. She calls `executeTrades(DAI, WETH, 1000, [], 0, alice)`.
2. Since trades is an empty array, and `finalAmountMin` is 0, the result is that 100 DAI are transferred to the Slingshot contract.
3. Eve (a miner or other "front-runner") may observe this, and immediately call `executeTrades(DAI, WETH, 0, [{TradeData}], 0, eve)`.
4. With a correctly formatted array of `tradeData`, Eve will receive the proceeds of converting Alice's 1000 DAI to WETH.

This issue is essentially identical to the one described in Ethereum is a Dark Forest, where locked tokens are available to anyone, and thus recovery is susceptible to front-running. It also provides an unauthorized alternative to `rescueTokens()`, however it is still a useful function to have, as it provides a method to recover the tokens without allowing a front-runner to simulate and replay it.

## Medium Severity

### [M-01] Front-Running/Sandwich Attacks

If a `finalAmountMin` is chosen that does not closely reflect the received amount one would get at the market rate (even with just 1% slippage), this could lead to the trade being front-run and to less tokens than with a tighter slippage amount. Balancer and Curve modules don't have any slippage protection at all, which makes it easy for attackers to profit from such an attack. The min. amount returned is hardcoded to 1 for both protocols. The Sushiswap/Uniswap modules are vulnerable as well, depending on the `calldata` that is defined by the victim trader.
