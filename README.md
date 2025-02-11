# uzh-uniswap
minimal one-stop implementation for interacting with Uniswap v2/v3

## Step-by-step guide to build the project from scratch

This project provides a minimal one-stop solution for deploying Uniswap contracts to a custom Ethereum chain and a frontend for interacting with them. Follow these steps to build the project:

**1. Create ERC20 Token:**

   - You will need to write a smart contract for your ERC20 token. You can use standard ERC20 implementations like those from OpenZeppelin.
   - Deploy this ERC20 contract to your custom Ethereum chain.
   - Note down the contract address of your deployed ERC20 token.

**2. Create Liquidity Pool:**

   - This project will need to interact with Uniswap V2 or V3 contracts. Ensure you have deployed Uniswap V2 or V3 factory and router contracts to your custom Ethereum chain.
   - Use the Uniswap Factory contract to create a new pair (liquidity pool) for your ERC20 token and WETH (Wrapped Ether) or another suitable token.
   - You will need to interact with the Uniswap Factory contract using its `createPair` function.
   - Note down the contract address of the newly created pair (liquidity pool).

**3. Provide Liquidity:**

   - To provide liquidity, you will interact with the newly created pair contract.
   - You will need to approve the pair contract to spend your ERC20 tokens and WETH (or the other token you used).
   - Use the `addLiquidity` or `addLiquidityETH` function on the Uniswap Router contract to provide liquidity to the pair.
   - You will need to specify the amounts of ERC20 tokens and WETH you want to provide.

**4. Do Exchange (Swap):**

   - To perform a swap, you will interact with the Uniswap Router contract.
   - Use functions like `swapExactTokensForTokens`, `swapTokensForExactTokens`, `swapExactETHForTokens`, or `swapTokensForExactETH` on the Uniswap Router contract.
   - Specify the input token, output token, amount to swap, and the recipient address.

**5. Reap Fee (Collect Fees):**

   - In Uniswap V2, fees are distributed to liquidity providers proportionally to their liquidity. Fees are automatically accumulated in the pair contract.
   - To collect fees, liquidity providers need to remove liquidity. When removing liquidity, accumulated fees are automatically distributed.
   - In Uniswap V3, fee collection is more granular. You might need to use specific functions in the V3 contracts to collect accumulated fees without removing liquidity, depending on the implementation.

**Further Steps:**

- **Frontend Implementation:** Develop a minimal frontend (e.g., using React, Vue, or plain HTML/JS) to interact with your deployed contracts. Use a library like ethers.js or web3.js to connect to your Ethereum chain and interact with the smart contracts.
- **Configuration:**  Make sure to configure your frontend to connect to your custom Ethereum chain and use the correct contract addresses for your ERC20 token, Uniswap Factory, Router, and Pair contracts.
- **Testing:** Thoroughly test each step on your custom Ethereum chain to ensure everything works as expected.

This guide provides a high-level overview. You will need to refer to the Uniswap V2 or V3 documentation and your chosen development tools for detailed implementation instructions. Good luck!
