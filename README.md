# uzh-uniswap
minimal one-stop implementation for interacting with Uniswap v2/v3

## Step-by-step guide to build the project from scratch

This project provides a minimal one-stop solution for deploying Uniswap contracts to a custom Ethereum chain and a frontend for interacting with them. Follow these steps to build the project:

**Prerequisites:**

- Node.js and npm or yarn installed
- Basic understanding of Ethereum development and smart contracts
- Familiarity with Hardhat development environment (or similar tools)
- [degit](https://github.com/Rich-Harris/degit) installed globally (`npm install -g degit` or `yarn global add degit`)

**Setup:**

1.  Initialize a new Hardhat project:
    ```bash
    mkdir uzh-uniswap-project
    cd uzh-uniswap-project
    npm init -y
    npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
    npx hardhat
    ```
    or using yarn:
    ```bash
    mkdir uzh-uniswap-project
    cd uzh-uniswap-project
    yarn init -y
    yarn add --dev hardhat @nomicfoundation/hardhat-toolbox
    yarn create hardhat
    ```
    Choose "Create an empty hardhat.config.js" when prompted.

2.  Install necessary dependencies:
    ```bash
    npm install --save ethers dotenv @openzeppelin/contracts
    ```
    or using yarn:
    ```bash
    yarn add ethers dotenv @openzeppelin/contracts
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
     - **Uniswap V2:** Use `degit` to download Uniswap V2 repositories from Github. For stability, consider using a specific commit hash or tag.
       ```bash
       degit Uniswap/v2-core#<COMMIT_HASH_OR_TAG> uniswap-v2-core  # e.g., degit Uniswap/v2-core#v1.0.1 uniswap-v2-core
       degit Uniswap/v2-periphery#<COMMIT_HASH_OR_TAG> uniswap-v2-periphery # e.g., degit Uniswap/v2-periphery#v1.1.1 uniswap-v2-periphery
       ```
     - **Uniswap V3:** Use `degit` to download Uniswap V3 repositories from Github. For stability, consider using a specific commit hash or tag.
       ```bash
       degit Uniswap/v3-core#<COMMIT_HASH_OR_TAG> uniswap-v3-core # e.g., degit Uniswap/v3-core#v1.0.0 uniswap-v3-core
       degit Uniswap/v3-periphery#<COMMIT_HASH_OR_TAG> uniswap-v3-periphery # e.g., degit Uniswap/v3-periphery#v1.4.35 uniswap-v3-periphery
       ```
       **Replace `<COMMIT_HASH_OR_TAG>` with a specific commit hash or tag from the Uniswap repositories for more stable versions.** You can find these on the Uniswap GitHub repository releases or commit history.
   - **Install Dependencies and Compile:** Navigate into each downloaded directory (e.g., `uniswap-v2-core`) and install dependencies and compile the contracts. Refer to the Uniswap repository's README for specific instructions. For V2, it's usually:
     ```bash
     cd uniswap-v2-core
     npm install # or yarn install
     npm run build # or yarn build
     cd ../uniswap-v2-periphery
     npm install # or yarn install
     npm run build # or yarn build
     ```
     For V3, the commands might be slightly different, so always check the official documentation. **After compiling, ABIs for Uniswap contracts will be available in their respective `artifacts` directories, similar to your ERC20 token contract.**
   - **Deploy Contracts:**
     - **Factory and Router:** You need to deploy at least the Factory and Router contracts from the chosen Uniswap version to your custom Ethereum chain. **Deployment scripts for Uniswap V2 and V3 Factory and Router contracts are typically found within their respective periphery repositories (e.g., `v2-periphery` or `v3-periphery`) in the `deploy` directory or in the documentation. You will likely need to adapt these scripts to work within your Hardhat environment. This adaptation usually involves:**
        * **Adjusting Paths:** Ensure that any paths within the deployment scripts are correctly resolved within your Hardhat project structure. You might need to modify import paths or file references.
        * **Hardhat Environment:** Modify the scripts to use Hardhat's `hre` (Hardhat Runtime Environment) for deploying contracts and interacting with the network.
        * **Configuration:** Ensure the scripts read network configurations and deployer accounts from your `hardhat.config.js` and `.env` files. You might need to replace Uniswap-specific configuration methods with Hardhat's configuration access.

       **To deploy, use Hardhat's `run` command, pointing to the adapted deployment script within the Uniswap directory. For example, if you adapt a script located at `uniswap-v2-periphery/deploy/001_deploy_factory.js`, the command might look like:**
       ```bash
       npx hardhat run uniswap-v2-periphery/deploy/001_deploy_factory.js --network yourNetworkName # or yarn hardhat run uniswap-v2-periphery/deploy/001_deploy_factory.js --network yourNetworkName
       ```
       **Note the adjusted path `uniswap-v2-periphery/deploy/001_deploy_factory.js` which is now relative to your project root.**  Review the Uniswap deployment scripts carefully to understand their dependencies and configuration needs and adapt them accordingly for your Hardhat setup. Example deployment scripts for ERC20 tokens are provided later in this guide, which you can use as a template for further adaptation.**
     - **WETH (Wrapped ETH):**  For Uniswap V2, you will also need to deploy a WETH contract if you don't have one already on your network. Uniswap V2 periphery repository usually includes a WETH deployment script. For V3, WETH is also typically required.
   - **Note Contract Addresses:** After deploying the Factory, Router, and WETH (if applicable) contracts, note down their contract addresses. You will need these addresses in later steps and in your scripts.

**1. Create ERC20 Token:**

   - Write a smart contract for your ERC20 token (e.g., `contracts/MyToken.sol`). You can use standard ERC20 implementations like those from OpenZeppelin.  Create a file `contracts/MyToken.sol` and write your ERC20 contract code in it.  For example, you can create a simple ERC20 token inheriting from OpenZeppelin's ERC20 contract and minting initial tokens to the deployer.
   - Compile your contracts:
     ```bash
     npx hardhat compile # or yarn hardhat compile
     ```
   - Deploy your ERC20 contract to your custom Ethereum chain using a deployment script (e.g., `scripts/deploy_erc20.js`). Create a file `scripts/deploy_erc20.js` and write a deployment script using `ethers.js` to deploy your `MyToken` contract.  This script will use your deployer private key from the `.env` file and your network configuration from `hardhat.config.js`.
     ```bash
     npx hardhat run scripts/deploy_erc20.js --network yourNetworkName # or yarn hardhat run scripts/deploy_erc20.js --network yourNetworkName
     ```
   - Note down the contract address of your deployed ERC20 token from the deployment output.

**2. Create Liquidity Pool:**

   - Ensure you have deployed Uniswap V2 or V3 factory and router contracts and have their addresses.
   - Use a script (e.g., `scripts/create_pair.js`) to interact with the Uniswap Factory contract to create a new pair (liquidity pool) for your ERC20 token and WETH (Wrapped Ether) or another suitable token.  This script will use the Factory contract address, WETH address, and your ERC20 token address.  Remember to replace placeholders with your actual contract addresses in the script.
     ```bash
     npx hardhat run scripts/create_pair.js --network yourNetworkName # or yarn hardhat run scripts/create_pair.js --network yourNetworkName
     ```
   - Note down the contract address of the newly created pair (liquidity pool) from the script output.

**3. Provide Liquidity:**

   - Use a script (e.g., `scripts/add_liquidity.js`) to interact with the Uniswap Router contract to provide liquidity to the pair. This script will use the Router contract address, WETH address, pair contract address, and amounts of tokens to provide. You will need to approve the Router contract to spend your ERC20 tokens and WETH before adding liquidity.
     ```bash
     npx hardhat run scripts/add_liquidity.js --network yourNetworkName # or yarn hardhat run scripts/add_liquidity.js --network yourNetworkName
     ```

**4. Do Exchange (Swap):**

   - Use a script (e.g., `scripts/swap.js`) to interact with the Uniswap Router contract to perform a swap. This script will use the Router contract address, WETH address, token addresses, amount to swap, and recipient address.
     ```bash
     npx hardhat run scripts/swap.js --network yourNetworkName # or yarn hardhat run scripts/swap.js --network yourNetworkName
     ```

**5. Reap Fee (Collect Fees):**

   - In Uniswap V2, fees are distributed to liquidity providers proportionally to their liquidity. Fees are automatically accumulated in the pair contract. To collect fees, liquidity providers need to remove liquidity.
   - Use a script (e.g., `scripts/remove_liquidity.js`) to remove liquidity from the pool. When removing liquidity, accumulated fees are automatically distributed to your address based on your provided liquidity share.
     ```bash
     npx hardhat run scripts/remove_liquidity.js --network yourNetworkName # or yarn hardhat run scripts/remove_liquidity.js --network yourNetworkName
     ```
   - In Uniswap V3, fee collection is more granular and might require different functions. Refer to Uniswap V3 documentation for details and consider using the Uniswap V3 SDK for easier interaction.

**Frontend Implementation:**

This section provides instructions for setting up a minimal React frontend to interact with your deployed Uniswap contracts.

1.  **Create React App:**
    If you don't have a React frontend already, create one in the project root directory:
    ```bash
    npx create-react-app frontend
    cd frontend
    ```
    or using yarn:
    ```bash
    yarn create react-app frontend
    cd frontend
    ```

2.  **Install Frontend Dependencies:**
    Install necessary packages for interacting with Ethereum and building the UI:
    ```bash
    npm install ethers react-scripts # or yarn add ethers react-scripts
    ```
    We recommend using `ethers` for interacting with Ethereum contracts in your frontend as it is consistent with the Hardhat development environment.

3.  **Environment Variables for Frontend:**
    In your `frontend` directory, create a `.env.local` file to store contract addresses and network information.  This is specific to React projects.
    ```env
    REACT_APP_NETWORK_RPC_URL="your_network_rpc_url_here"
    REACT_APP_ERC20_TOKEN_ADDRESS="YOUR_ERC20_TOKEN_ADDRESS"
    REACT_APP_FACTORY_ADDRESS="YOUR_UNISWAP_FACTORY_ADDRESS"
    REACT_APP_ROUTER_ADDRESS="YOUR_UNISWAP_ROUTER_ADDRESS"
    REACT_APP_WETH_ADDRESS="YOUR_WETH_ADDRESS"
    ```
    Replace the placeholder values with your actual deployed contract addresses and network RPC URL.  Remember to use `REACT_APP_` prefix for React environment variables.

4.  **Basic Frontend Code Example (`frontend/src/App.js`):**
    Replace the content of `frontend/src/App.js` with the following example code. This is a very basic example to demonstrate connecting to a wallet, and basic interaction.  You will need to expand upon this for a full-fledged UI.

    ```javascript
    // frontend/src/App.js
    import React, { useState, useEffect } from 'react';
    import { ethers } from 'ethers';

    const ERC20_TOKEN_ADDRESS = process.env.REACT_APP_ERC20_TOKEN_ADDRESS;
    const FACTORY_ADDRESS = process.env.REACT_APP_FACTORY_ADDRESS;
    const ROUTER_ADDRESS = process.env.REACT_APP_ROUTER_ADDRESS;
    const WETH_ADDRESS = process.env.REACT_APP_WETH_ADDRESS;
    const NETWORK_RPC_URL = process.env.REACT_APP_NETWORK_RPC_URL;

    function App() {
        const [account, setAccount] = useState(null);
        const [tokenBalance, setTokenBalance] = useState(null);
        const [tokenSymbol, setTokenSymbol] = useState(null); // State for token symbol

        useEffect(() => {
            connectWallet();
        }, []);

        async function connectWallet() {
            if (window.ethereum) {
                try {
                    const accounts = await window.ethereum.request({ method: "eth_requestAccounts" });
                    setAccount(accounts[0]);
                    console.log("Connected account:", accounts[0]);
                    if (accounts[0]) {
                        await fetchTokenBalance(accounts[0]); // Fetch token balance after connecting
                    }
                } catch (error) {
                    console.error("Could not connect wallet:", error);
                }
            } else {
                console.log("Please install MetaMask!");
            }
        }

        async function fetchTokenBalance(account) {
            if (!ERC20_TOKEN_ADDRESS) return; // Return if token address is not set
            const provider = new ethers.providers.JsonRpcProvider(NETWORK_RPC_URL);
            const tokenContract = new ethers.Contract(ERC20_TOKEN_ADDRESS, ["function balanceOf(address) view returns (uint256)", "function symbol() view returns (string)", "function decimals() view returns (uint8)"], provider); // Minimal ABI for balance, symbol, and decimals
            try {
                const balance = await tokenContract.balanceOf(account);
                const symbol = await tokenContract.symbol();
                const decimals = await tokenContract.decimals() || 18; // Default to 18 decimals if not available
                setTokenBalance(`${ethers.utils.formatUnits(balance, decimals)}`);
                setTokenSymbol(symbol);
            } catch (error) {
                console.error("Error fetching token balance:", error);
                setTokenBalance("Error fetching balance");
                setTokenSymbol("N/A"); // Set symbol to N/A in case of error
            }
        }


        return (
            <div className="App">
                <h1>Uniswap Interaction</h1>
                {account ? (
                    <div>
                        <p>Connected Account: {account}</p>
                        {tokenBalance && <p>Token Balance: {tokenBalance} {tokenSymbol}</p>}
                    </div>
                ) : (
                    <button onClick={connectWallet}>Connect Wallet</button>
                )}

                {/*
                    For more complex interactions (e.g., swapping, providing liquidity),
                    you will need to create contract instances using ABIs.
                    Example (for demonstration - you'll need to import your ABI):

                    const tokenContractWithAbi = new ethers.Contract(
                        ERC20_TOKEN_ADDRESS,
                        ERC20_TOKEN_ABI, // Replace ERC20_TOKEN_ABI with your actual ABI
                        signer // You'll need to set up a signer (e.g., from MetaMask)
                    );

                    Then you can call contract functions like:
                    tokenContractWithAbi.transfer( ... );
                */}
            </div>
        );
    }

    export default App;
    ```

5.  **Run Frontend:**
    Start your React frontend development server from the `frontend` directory:
    ```bash
    cd frontend
    npm start # or yarn start
    ```
    This will usually open your frontend in a browser at `http://localhost:3000`.

6.  **Expand Frontend Functionality:**
    - **Contract Interaction:** Use `ethers.js` within your React components to interact with your deployed contracts. You will need to use Contract ABIs (Application Binary Interfaces) for your ERC20 token, Uniswap Factory, Router, and Pair contracts.

        **Example of creating a contract instance in your frontend using `ethers.js`:**

        ```javascript
        import { ethers } from 'ethers';

        const ERC20_TOKEN_ADDRESS = process.env.REACT_APP_ERC20_TOKEN_ADDRESS;
        const ERC20_TOKEN_ABI = [...] // Your ERC20 token ABI (from artifacts/contracts/MyToken.sol/MyToken.json)
        const NETWORK_RPC_URL = process.env.REACT_APP_NETWORK_RPC_URL;

        const provider = new ethers.providers.JsonRpcProvider(NETWORK_RPC_URL);
        const tokenContract = new ethers.Contract(ERC20_TOKEN_ADDRESS, ERC20_TOKEN_ABI, provider);

        // Now you can use tokenContract to call functions on your deployed ERC20 contract
        ```

        - **ERC20 Token ABI:** You can find the ABI for your `MyToken` contract in the compiled output in your Hardhat project under `artifacts/contracts/MyToken.sol/MyToken.json`. Look for the `abi` field in this JSON file.
        - **Uniswap V2 ABIs:** For Uniswap V2 contracts, after installing `@uniswap/v2-core` and `@uniswap/v2-periphery` in your Hardhat project, you can find the ABIs in the `node_modules` directory, for example, under `@uniswap/v2-core/build/abi` and `@uniswap/v2-periphery/build/abi`.
        - **Uniswap V3 ABIs:** If you are using Uniswap V3, you will similarly find ABIs in the `node_modules` directory after installing Uniswap V3 SDKs and libraries. **Consider using the official Uniswap V3 SDK for easier frontend integration with V3 contracts.**

    - **Testing on Local Network:** **It is highly recommended to test all steps of your project, including contract deployment, backend scripts, and frontend interactions, on a local Hardhat network *first*.** Hardhat Network provides a fast and easy-to-use local Ethereum environment for testing and development. You can start a local Hardhat network using `npx hardhat node` or `yarn hardhat node` and deploy your contracts to it. This local testing phase allows for faster iteration and debugging, and helps ensure everything works correctly before deploying to your custom Ethereum chain.

    - **UI Elements:** Create React components and UI elements (buttons, input fields, etc.) to allow users to perform actions like:
        - Connect/disconnect wallet
        - View token balances
        - Input amounts for swaps and liquidity provision
        - Select tokens for swaps and liquidity pools
        - Display transaction statuses and results
    - **Error Handling:** Implement error handling in your frontend to gracefully manage transaction failures and network issues.
    - **State Management:** For more complex frontends, consider using state management libraries like React Context or Redux to manage application state effectively.

**Further Steps:**

- **Configuration:**  Ensure your frontend is correctly configured with the right contract addresses and network RPC URL in the `.env.local` file. Double-check these values to match your deployed contracts.
- **Testing:** Thoroughly test your frontend by interacting with your deployed contracts, **starting with a local Hardhat network and then on your custom Ethereum chain.** Test all functionalities: connecting wallet, viewing balances, creating pools, providing liquidity, swapping tokens, and any other features you implement.

**Uniswap V2 vs V3:**

- This guide primarily focuses on Uniswap V2-like interactions.
- Uniswap V3 introduces significant changes, including:
    - **Concentrated Liquidity:**  Liquidity providers can choose price ranges within which they want to provide liquidity, leading to potentially higher capital efficiency.
    - **Multiple Fee Tiers:** V3 supports different fee tiers (e.g., 0.05%, 0.3%, 1%), allowing for more flexibility.
    - **Non-Fungible Liquidity Positions (NFTs):**  Liquidity positions in V3 are represented as NFTs, making them more complex to manage but also enabling more advanced strategies.
    - **Improved Oracle:** V3 has a more sophisticated oracle mechanism.
- If you are using Uniswap V3 contracts, you will need to adapt the scripts and interactions accordingly. **For frontend integration with Uniswap V3, it is highly recommended to use the official Uniswap V3 SDK.** Refer to the official Uniswap V3 documentation and SDK for specific details on contract addresses, ABIs, and function calls.

**Note:**

- Replace placeholders with your actual values and contract addresses.
- You will need to install Uniswap V2 or V3 contract ABIs (e.g., `@uniswap/v2-core`, `@uniswap/v2-periphery` or similar for V3) and import them in your scripts and frontend to interact with Uniswap contracts.
- This guide provides basic examples. You may need to add error handling, more robust input validation, and more advanced features for a production-ready application.
- **Security Best Practices: Security is paramount in blockchain development. Be extremely careful with handling private keys. Ensure your `.env` file is properly secured and never committed to version control. In a production environment, never expose contract addresses or private keys directly in frontend code. Use secure methods for managing environment variables and sensitive information. Regularly audit your code for security vulnerabilities.**
- Always review and understand the code and scripts before running them, especially when dealing with blockchain and smart contracts.

This updated `README.md` now more clearly explains the path adjustments and script adaptations needed for deploying Uniswap contracts within a Hardhat project. It also emphasizes reviewing the Uniswap deployment scripts and provides a more concrete example of the `hardhat run` command with an adjusted path.
