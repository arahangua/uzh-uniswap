# uzh-uniswap
minimal one-stop implementation for interacting with Uniswap v2/v3

## Step-by-step guide to build the project from scratch

This project provides a minimal one-stop solution for deploying Uniswap contracts to a custom Ethereum chain and a frontend for interacting with them. Follow these steps to build the project:

**Prerequisites:**

- Node.js and npm installed
- Basic understanding of Ethereum development and smart contracts
- Familiarity with Hardhat development environment (or similar tools)
- Git

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

**Configuration:**

1.  **Hardhat Configuration (`hardhat.config.js`):**
    Configure your network settings in `hardhat.config.js`. Example:
    ```javascript
    require("@nomicfoundation/hardhat-toolbox");
    require("dotenv").config();

    /** @type import('hardhat/config').HardhatUserConfig */
    module.exports = {
      solidity: "0.8.17",
      networks: {
        localhost: {
          url: "http://127.0.0.1:8545" // or your local node URL
        },
        yourNetworkName: { // Replace with your network name
          url: process.env.YOUR_NETWORK_RPC_URL, // Your network RPC URL from .env
          accounts: [process.env.DEPLOYER_PRIVATE_KEY], // Deployer private key from .env
        },
      },
    };
    ```

2.  **.env file:**
    Create a `.env` file in your project root and add your environment variables:
    ```env
    DEPLOYER_PRIVATE_KEY="your_private_key_here"
    YOUR_NETWORK_RPC_URL="your_network_rpc_url_here"
    ```
    **Important:** Never commit your private key to a public repository.

**Steps:**

**Deploy Uniswap V2 or V3 Contracts:**

   - **Choose Uniswap Version:** Decide whether you want to use Uniswap V2 or V3. This guide will assume you are using V2 for simplicity, but instructions for V3 are similar.
   - **Fetch Uniswap Contracts:**
     - **Uniswap V2:** Clone the Uniswap V2 repositories from Github:
       ```bash
       git clone https://github.com/Uniswap/v2-core.git uniswap-v2-core
       git clone https://github.com/Uniswap/v2-periphery.git uniswap-v2-periphery
       ```
     - **Uniswap V3:** Clone the Uniswap V3 repositories from Github:
       ```bash
       git clone https://github.com/Uniswap/v3-core.git uniswap-v3-core
       git clone https://github.com/Uniswap/v3-periphery.git uniswap-v3-periphery
       ```
   - **Install Dependencies and Compile:** Navigate into each cloned repository (e.g., `uniswap-v2-core`) and install dependencies and compile the contracts. Refer to the Uniswap repository's README for specific instructions. For V2, it's usually:
     ```bash
     cd uniswap-v2-core
     npm install
     npm run build
     cd ../uniswap-v2-periphery
     npm install
     npm run build
     ```
     For V3, the commands might be slightly different, so always check the official documentation.
   - **Deploy Contracts:**
     - **Factory and Router:** You need to deploy at least the Factory and Router contracts from the chosen Uniswap version to your custom Ethereum chain. Use deployment scripts similar to the ERC20 deployment script (see below), adapting them for the Uniswap Factory and Router contracts. You will find deployment scripts or instructions within the Uniswap repositories themselves or their documentation.
     - **WETH (Wrapped ETH):**  For Uniswap V2, you will also need to deploy a WETH contract if you don't have one already on your network. Uniswap V2 periphery repository usually includes a WETH deployment script. For V3, WETH is also typically required.
   - **Note Contract Addresses:** After deploying the Factory, Router, and WETH (if applicable) contracts, note down their contract addresses. You will need these addresses in later steps and in your scripts.

**1. Create ERC20 Token:**

   - Write a smart contract for your ERC20 token (e.g., `contracts/MyToken.sol`). You can use standard ERC20 implementations like those from OpenZeppelin.  Create a file `contracts/MyToken.sol` and write your ERC20 contract code in it.  For example, you can create a simple ERC20 token inheriting from OpenZeppelin's ERC20 contract and minting initial tokens to the deployer.
   - Compile your contracts:
     ```bash
     npx hardhat compile
     ```
   - Deploy your ERC20 contract to your custom Ethereum chain using a deployment script (e.g., `scripts/deploy_erc20.js`). Create a file `scripts/deploy_erc20.js` and write a deployment script using `ethers.js` to deploy your `MyToken` contract.  This script will use your deployer private key from the `.env` file and your network configuration from `hardhat.config.js`.
     ```bash
     npx hardhat run scripts/deploy_erc20.js --network yourNetworkName
     ```
   - Note down the contract address of your deployed ERC20 token from the deployment output.

**2. Create Liquidity Pool:**

   - Ensure you have deployed Uniswap V2 or V3 factory and router contracts and have their addresses.
   - Use a script (e.g., `scripts/create_pair.js`) to interact with the Uniswap Factory contract to create a new pair (liquidity pool) for your ERC20 token and WETH (Wrapped Ether) or another suitable token.  This script will use the Factory contract address, WETH address, and your ERC20 token address.  Remember to replace placeholders with your actual contract addresses in the script.
     ```bash
     npx hardhat run scripts/create_pair.js --network yourNetworkName
     ```
   - Note down the contract address of the newly created pair (liquidity pool) from the script output.

**3. Provide Liquidity:**

   - Use a script (e.g., `scripts/add_liquidity.js`) to interact with the Uniswap Router contract to provide liquidity to the pair. This script will use the Router contract address, WETH address, pair contract address, and amounts of tokens to provide. You will need to approve the Router contract to spend your ERC20 tokens and WETH before adding liquidity.
     ```bash
     npx hardhat run scripts/add_liquidity.js --network yourNetworkName
     ```

**4. Do Exchange (Swap):**

   - Use a script (e.g., `scripts/swap.js`) to interact with the Uniswap Router contract to perform a swap. This script will use the Router contract address, WETH address, token addresses, amount to swap, and recipient address.
     ```bash
     npx hardhat run scripts/swap.js --network yourNetworkName
     ```

**5. Reap Fee (Collect Fees):**

   - In Uniswap V2, fees are distributed to liquidity providers proportionally to their liquidity. Fees are automatically accumulated in the pair contract. To collect fees, liquidity providers need to remove liquidity.
   - Use a script (e.g., `scripts/remove_liquidity.js`) to remove liquidity from the pool. When removing liquidity, accumulated fees are automatically distributed to your address based on your provided liquidity share.
     ```bash
     npx hardhat run scripts/remove_liquidity.js --network yourNetworkName
     ```
   - In Uniswap V3, fee collection is more granular and might require different functions. Refer to Uniswap V3 documentation for details.

**Further Steps:**

- **Frontend Implementation:** Develop a minimal frontend (e.g., using React, Vue, or plain HTML/JS) to interact with your deployed contracts. Use a library like ethers.js or web3.js to connect to your Ethereum chain and interact with the smart contracts.
    - You can use libraries like React or Vue.js for building the frontend.
    - Use ethers.js or web3.js to interact with your smart contracts from the frontend.
    - Connect your frontend to a wallet like MetaMask to sign transactions.
    - Create UI elements to:
        - Connect to wallet
        - Display token balances
        - Allow user to input amounts for adding/removing liquidity and swapping tokens
        - Display transaction status and results

- **Configuration:**  Make sure to configure your frontend to connect to your custom Ethereum chain and use the correct contract addresses for your ERC20 token, Uniswap Factory, Router, and Pair contracts.  This usually involves setting up environment variables in your frontend application to point to the correct contract addresses and network RPC URL.

- **Testing:** Thoroughly test each step on your custom Ethereum chain to ensure everything works as expected.  Test different scenarios, including swaps, adding/removing liquidity with varying amounts, and fee collection.

**Uniswap V2 vs V3:**

- This guide primarily focuses on Uniswap V2-like interactions.
- Uniswap V3 introduces significant changes, including:
    - **Concentrated Liquidity:**  Liquidity providers can choose price ranges within which they want to provide liquidity, leading to potentially higher capital efficiency.
    - **Multiple Fee Tiers:** V3 supports different fee tiers (e.g., 0.05%, 0.3%, 1%), allowing for more flexibility.
    - **Non-Fungible Liquidity Positions (NFTs):**  Liquidity positions in V3 are represented as NFTs, making them more complex to manage but also enabling more advanced strategies.
    - **Improved Oracle:** V3 has a more sophisticated oracle mechanism.
- If you are using Uniswap V3 contracts, you will need to adapt the scripts and interactions accordingly. Refer to the official Uniswap V3 documentation and SDK for specific details on contract addresses, ABIs, and function calls.

**Note:**

- Replace placeholders like `YOUR_NETWORK_RPC_URL`, `DEPLOYER_PRIVATE_KEY`, `YOUR_UNISWAP_FACTORY_ADDRESS`, `YOUR_UNISWAP_ROUTER_ADDRESS`, `YOUR_WETH_ADDRESS`, `YOUR_ERC20_TOKEN_ADDRESS`, and `YOUR_PAIR_ADDRESS` with your actual values.
- You will need to install Uniswap V2 or V3 contract ABIs (e.g., `@uniswap/v2-core`, `@uniswap/v2-periphery` or similar for V3) and import them in your scripts to interact with Uniswap contracts.
- This guide provides basic examples. You may need to add error handling, more robust input validation, and more advanced features for a production-ready application.
- Always review and understand the code and scripts before running them, especially when dealing with blockchain and smart contracts.

This guide provides a more detailed step-by-step process with instructions to fetch and deploy Uniswap contracts and create your own ERC20 token and interact with Uniswap for liquidity provision and token swapping. Remember to adapt the scripts and configurations to your specific setup and refer to the official Uniswap documentation for more advanced features and details. Good luck!
