# uzh-uniswap
minimal one-stop implementation for interacting with Uniswap v2/v3

## Step-by-step guide to build the project from scratch

This project provides a minimal one-stop solution for deploying Uniswap contracts to a custom Ethereum chain and a frontend for interacting with them. Follow these steps to build the project:

**Prerequisites:**

- Node.js and npm installed
- Basic understanding of Ethereum development and smart contracts
- Familiarity with Hardhat development environment (or similar tools)

**Setup:**

1.  Initialize a new Hardhat project:
    ```bash
    mkdir uzh-uniswap-project
    cd uzh-uniswap-project
    npm init -y
    npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
    npx hardhat
    ```
    Choose "Create an empty hardhat.config.js" when prompted.

2.  Install necessary dependencies:
    ```bash
    npm install --save ethers dotenv @openzeppelin/contracts
    ```

**Steps:**

**1. Create ERC20 Token:**

   - Write a smart contract for your ERC20 token (e.g., `contracts/MyToken.sol`). You can use standard ERC20 implementations like those from OpenZeppelin.
   - Compile your contracts:
     ```bash
     npx hardhat compile
     ```
   - Deploy your ERC20 contract to your custom Ethereum chain using a deployment script (e.g., `scripts/deploy_erc20.js`).  You'll need to configure your network in `hardhat.config.js` and set up environment variables (e.g., `.env` file) for your deployer private key and network RPC URL.
     ```bash
     npx hardhat run scripts/deploy_erc20.js --network your-network-name
     ```
   - Note down the contract address of your deployed ERC20 token from the deployment output.

**2. Create Liquidity Pool:**

   - Ensure you have deployed Uniswap V2 or V3 factory and router contracts to your custom Ethereum chain. If not, you'll need to deploy them first, following Uniswap's official documentation and using similar deployment scripts as above.
   - Use a script (e.g., `scripts/create_pair.js`) to interact with the Uniswap Factory contract to create a new pair (liquidity pool) for your ERC20 token and WETH (Wrapped Ether) or another suitable token.  This script will use the Factory contract address and your ERC20 token address.
     ```bash
     npx hardhat run scripts/create_pair.js --network your-network-name
     ```
   - Note down the contract address of the newly created pair (liquidity pool) from the script output.

**3. Provide Liquidity:**

   - Use a script (e.g., `scripts/add_liquidity.js`) to interact with the Uniswap Router contract to provide liquidity to the pair. This script will use the Router contract address, pair contract address, and amounts of tokens to provide. You will need to approve the Router contract to spend your ERC20 tokens and WETH before adding liquidity.
     ```bash
     npx hardhat run scripts/add_liquidity.js --network your-network-name
     ```

**4. Do Exchange (Swap):**

   - Use a script (e.g., `scripts/swap.js`) to interact with the Uniswap Router contract to perform a swap. This script will use the Router contract address, token addresses, amount to swap, and recipient address.
     ```bash
     npx hardhat run scripts/swap.js --network your-network-name
     ```

**5. Reap Fee (Collect Fees):**

   - In Uniswap V2, fees are distributed to liquidity providers proportionally to their liquidity. Fees are automatically accumulated in the pair contract. To collect fees, liquidity providers need to remove liquidity.
   - Use a script (e.g., `scripts/remove_liquidity.js`) to remove liquidity from the pool. When removing liquidity, accumulated fees are automatically distributed to your address based on your provided liquidity share.
     ```bash
     npx hardhat run scripts/remove_liquidity.js --network your-network-name
     ```
   - In Uniswap V3, fee collection is more granular and might require different functions. Refer to Uniswap V3 documentation for details.

**Further Steps:**

- **Frontend Implementation:** Develop a minimal frontend (e.g., using React, Vue, or plain HTML/JS) to interact with your deployed contracts. Use a library like ethers.js or web3.js to connect to your Ethereum chain and interact with the smart contracts.
- **Configuration:**  Make sure to configure your frontend to connect to your custom Ethereum chain and use the correct contract addresses for your ERC20 token, Uniswap Factory, Router, and Pair contracts.
- **Testing:** Thoroughly test each step on your custom Ethereum chain to ensure everything works as expected.

**Note:**

- Replace `your-network-name` with the actual name of your network configured in `hardhat.config.js`.
- You will need to write the scripts mentioned (e.g., `scripts/deploy_erc20.js`, `scripts/create_pair.js`, etc.) to perform the contract interactions. These scripts will typically use libraries like `ethers.js` to interact with your deployed contracts.
- This guide assumes you are using Uniswap V2-like contracts. For Uniswap V3, the contract interactions and fee collection mechanisms might be different. Refer to the official Uniswap V3 documentation for specific details.

This guide provides a more concrete step-by-step process with command-line examples to help you build the project from scratch. Remember to adapt the scripts and network configurations to your specific setup. Good luck!
