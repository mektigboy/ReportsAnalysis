# SLINGSHOT

## High Severity

### [H-01] Incorrectly encoded arguments to `executeTrades()` can allow tokens to be stolen

This finding combines a couple weaknesses into one attack. The first weakness is a lack of validation on arguments to `executeTrades()`, the second is that a pre-existing `fromToken` balance can be used in a trade:

1. Alice wants to convert 1000 DAI to WETH. She calls `executeTrades(DAI, WETH, 1000, [], 0, alice)`.
2. Since trades is an empty array, and `finalAmountMin` is 0, the result is that 100 DAI are transferred to the Slingshot contract.
3. Eve (a miner or other "front-runner") may observe this, and immediately call `executeTrades(DAI, WETH, 0, [{TradeData}], 0, eve)`.
4. With a correctly formatted array of `tradeData`, Eve will receive the proceeds of converting Alice's 1000 DAI to WETH.

This issue is essentially identical to the one described in Ethereum is a Dark Forest, where locked tokens are available to anyone, and thus recovery is susceptible to front-running. It also provides an unauthorized alternative to `rescueTokens()`, however it is still a useful function to have, as it provides a method to recover the tokens without allowing a front-runner to simulate and replay it.

## Medium Severity

### [M-01] Front-running/sandwich attacks

If a `finalAmountMin` is chosen that does not closely reflect the received amount one would get at the market rate (even with just 1% slippage), this could lead to the trade being front-run and to less tokens than with a tighter slippage amount. Balancer and Curve modules don't have any slippage protection at all, which makes it easy for attackers to profit from such an attack. The min. amount returned is hardcoded to 1 for both protocols. The Sushiswap/Uniswap modules are vulnerable as well, depending on the `calldata` that is defined by the victim trader.

### [M-02] Front-running/sandwich attacka

If tokens are accidently sent to Slingshot, arbitrary trades can be executed and those funds can be stolen by anyone. This vulnerability impacts the `rescueTokens()` functionality and any funds trapped in Slingshot’s contract. Tokens and/or Eth have a higher likelihood of becoming trapped in Slingshot if `finalAmountMin` is not utilized properly.

**Recommendation:** Validating parameters in the `calldata` passed to modules and ensuring the `fromToken` and `amount` parameter from `executeTrades()` is equivalent to the token being swapped and amount passed to `swap()`. Additionally, approval values can be limited to value being traded and cleared after trades are executed.

### [M-03] Infinite approval abused by malicious admin

Current Slingshot contracts assume a rapid development environment so they use a proxy pattern with a trusted admin role. We do not expect any malicious behavior from admin,
however, we agree that in the current setup the admin potentially would be able to use unlimited approvals to steal user’s funds. We consider this medium severity.

## [M-04] Stuck tokens can be stolen

Any tokens in the Slingshot contract can be stolen by creating a fake token and a Uniswap pair for the stuck token and this fake token. Consider 10 WETH being stuck in the Slingshot contract. One can create a fake ERC20 token contract FAKE and a WETH <> FAKE Uniswap pair. The attacker provides a tiny amount of initial WETH liquidity (for example, 1 gwei) and some amount of FAKE tokens. The attacker then executes `executeTrades()` action such that the Slingshot contract uses its Uniswap module to trade the 10 WETH into this pair.

## [M-05] Admin role lockout

The `initializeAdmin(`) function in Adminable.sol sets/updates admin role address in one-step. If an incorrect address (zero address or other) is mistakenly used then future
administrative access or even recovering from this mistake is prevented because all `onlyAdmin` modifier functions (including `postUpgrade()` with `onlyAdminIfInitialized`, which ends up calling `initializeAdmin()`) require `msg.sender` to be the incorrectly used admin address (for which private keys may not be available to sign transactions). In such a case, contracts would have to be redeployed.

**Recommendation:** Using a two-step process where the new admin address first claims ownership in one transaction and a second transaction from the new admin address takes ownership.
