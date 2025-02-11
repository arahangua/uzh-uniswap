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

**1. Create ERC20 Token:**

   - Write a smart contract for your ERC20 token (e.g., `contracts/MyToken.sol`). You can use standard ERC20 implementations like those from OpenZeppelin.
     ```solidity
     // contracts/MyToken.sol
     // SPDX-License-Identifier: MIT
     pragma solidity ^0.8.0;

     import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

     contract MyToken is ERC20 {
         constructor() ERC20("MyToken", "MTK") {
             _mint(msg.sender, 1000000 * 10**18); // Mint 1,000,000 tokens to deployer
         }
     }
     ```
   - Compile your contracts:
     ```bash
     npx hardhat compile
     ```
   - Deploy your ERC20 contract to your custom Ethereum chain using a deployment script (e.g., `scripts/deploy_erc20.js`).
     ```javascript
     // scripts/deploy_erc20.js
     const hre = require("hardhat");
     require("dotenv").config();

     async function main() {
       const MyToken = await hre.ethers.getContractFactory("MyToken");
       const myToken = await MyToken.deploy();

       await myToken.deployed();

       console.log("MyToken deployed to:", myToken.address);
     }

     main()
       .then(() => process.exit(0))
       .catch((error) => {
         console.error(error);
         process.exit(1);
       });
     ```
     ```bash
     npx hardhat run scripts/deploy_erc20.js --network yourNetworkName
     ```
   - Note down the contract address of your deployed ERC20 token from the deployment output.

**2. Create Liquidity Pool:**

   - Ensure you have deployed Uniswap V2 or V3 factory and router contracts to your custom Ethereum chain. If not, you'll need to deploy them first, following Uniswap's official documentation and using similar deployment scripts. Assume you have Factory and Router addresses.
   - Use a script (e.g., `scripts/create_pair.js`) to interact with the Uniswap Factory contract to create a new pair (liquidity pool) for your ERC20 token and WETH (Wrapped Ether) or another suitable token.  This script will use the Factory contract address and your ERC20 token address.
     ```javascript
     // scripts/create_pair.js
     const hre = require("hardhat");
     require("dotenv").config();

     const FACTORY_ADDRESS = "YOUR_UNISWAP_FACTORY_ADDRESS"; // Replace with your Factory address
     const WETH_ADDRESS = "YOUR_WETH_ADDRESS"; // Replace with your WETH address

     async function main() {
       const [deployer] = await hre.ethers.getSigners();
       const factory = await hre.ethers.getContractAt("IUniswapV2Factory", FACTORY_ADDRESS, deployer); // Assuming Uniswap V2 Factory ABI is available as IUniswapV2Factory

       const tokenAddress = "YOUR_ERC20_TOKEN_ADDRESS"; // Replace with your deployed ERC20 token address

       const tx = await factory.createPair(tokenAddress, WETH_ADDRESS);
       const receipt = await tx.wait();

       const pairCreatedEvent = receipt.events.find(event => event.event === 'PairCreated');
       const pairAddress = pairCreatedEvent.args.pair;

       console.log("Pair created at:", pairAddress);
     }

     main()
       .then(() => process.exit(0))
       .catch((error) => {
         console.error(error);
         process.exit(1);
       });
     ```
     ```bash
     npx hardhat run scripts/create_pair.js --network yourNetworkName
     ```
   - Note down the contract address of the newly created pair (liquidity pool) from the script output.

**3. Provide Liquidity:**

   - Use a script (e.g., `scripts/add_liquidity.js`) to interact with the Uniswap Router contract to provide liquidity to the pair. This script will use the Router contract address, pair contract address, and amounts of tokens to provide. You will need to approve the Router contract to spend your ERC20 tokens and WETH before adding liquidity.
     ```javascript
     // scripts/add_liquidity.js
     const hre = require("hardhat");
     require("dotenv").config();

     const ROUTER_ADDRESS = "YOUR_UNISWAP_ROUTER_ADDRESS"; // Replace with your Router address
     const WETH_ADDRESS = "YOUR_WETH_ADDRESS"; // Replace with your WETH address

     async function main() {
       const [deployer] = await hre.ethers.getSigners();
       const router = await hre.ethers.getContractAt("IUniswapV2Router02", ROUTER_ADDRESS, deployer); // Assuming Uniswap V2 Router ABI is available as IUniswapV2Router02

       const tokenAddress = "YOUR_ERC20_TOKEN_ADDRESS"; // Replace with your deployed ERC20 token address
       const pairAddress = "YOUR_PAIR_ADDRESS"; // Replace with your created Pair address

       const tokenAmount = hre.ethers.utils.parseUnits("1000", 18); // Example: 1000 tokens
       const wethAmount = hre.ethers.utils.parseEther("1"); // Example: 1 WETH

       // Approve router to spend tokens
       const tokenContract = await hre.ethers.getContractAt("ERC20", tokenAddress, deployer); // Assuming standard ERC20 ABI
       await tokenContract.approve(ROUTER_ADDRESS, tokenAmount);

       const wethContract = await hre.ethers.getContractAt("IWETH", WETH_ADDRESS, deployer); // Assuming WETH ABI is available as IWETH
       await wethContract.deposit({ value: wethAmount });
       await wethContract.approve(ROUTER_ADDRESS, wethAmount);


       const tx = await router.addLiquidity(
         tokenAddress,
         WETH_ADDRESS,
         tokenAmount,
         wethAmount,
         tokenAmount, // min amount A
         wethAmount, // min amount B
         deployer.address,
         Math.floor(Date.now() / 1000) + 60 * 10 // 10 minutes from now
       );
       await tx.wait();

       console.log("Liquidity added");
     }

     main()
       .then(() => process.exit(0))
       .catch((error) => {
         console.error(error);
         process.exit(1);
       });
     ```
     ```bash
     npx hardhat run scripts/add_liquidity.js --network yourNetworkName
     ```

**4. Do Exchange (Swap):**

   - Use a script (e.g., `scripts/swap.js`) to interact with the Uniswap Router contract to perform a swap. This script will use the Router contract address, token addresses, amount to swap, and recipient address.
     ```javascript
     // scripts/swap.js
     const hre = require("hardhat");
     require("dotenv").config();

     const ROUTER_ADDRESS = "YOUR_UNISWAP_ROUTER_ADDRESS"; // Replace with your Router address
     const WETH_ADDRESS = "YOUR_WETH_ADDRESS"; // Replace with your WETH address

     async function main() {
       const [deployer] = await hre.ethers.getSigners();
       const router = await hre.ethers.getContractAt("IUniswapV2Router02", ROUTER_ADDRESS, deployer); // Assuming Uniswap V2 Router ABI is available as IUniswapV2Router02

       const tokenAddress = "YOUR_ERC20_TOKEN_ADDRESS"; // Replace with your deployed ERC20 token address

       const amountIn = hre.ethers.utils.parseUnits("10", 18); // Example: 10 tokens to swap
       const amountOutMin = 0; // Minimum output amount
       const path = [tokenAddress, WETH_ADDRESS]; // Token -> WETH path
       const to = deployer.address; // Recipient address
       const deadline = Math.floor(Date.now() / 1000) + 60 * 10; // 10 minutes from now

       // Approve router to spend tokens (if not already approved)
       const tokenContract = await hre.ethers.getContractAt("ERC20", tokenAddress, deployer); // Assuming standard ERC20 ABI
       await tokenContract.approve(ROUTER_ADDRESS, amountIn);


       const tx = await router.swapExactTokensForETH(
         amountIn,
         amountOutMin,
         path,
         to,
         deadline
       );
       await tx.wait();

       console.log("Swap executed");
     }

     main()
       .then(() => process.exit(0))
       .catch((error) => {
         console.error(error);
         process.exit(1);
       });
     ```
     ```bash
     npx hardhat run scripts/swap.js --network yourNetworkName
     ```

**5. Reap Fee (Collect Fees):**

   - In Uniswap V2, fees are distributed to liquidity providers proportionally to their liquidity. Fees are automatically accumulated in the pair contract. To collect fees, liquidity providers need to remove liquidity.
   - Use a script (e.g., `scripts/remove_liquidity.js`) to remove liquidity from the pool. When removing liquidity, accumulated fees are automatically distributed to your address based on your provided liquidity share.
     ```javascript
     // scripts/remove_liquidity.js
     const hre = require("hardhat");
     require("dotenv").config();

     const ROUTER_ADDRESS = "YOUR_UNISWAP_ROUTER_ADDRESS"; // Replace with your Router address
     const WETH_ADDRESS = "YOUR_WETH_ADDRESS"; // Replace with your WETH address
     const PAIR_ADDRESS = "YOUR_PAIR_ADDRESS"; // Replace with your Pair address

     async function main() {
       const [deployer] = await hre.ethers.getSigners();
       const router = await hre.ethers.getContractAt("IUniswapV2Router02", ROUTER_ADDRESS, deployer); // Assuming Uniswap V2 Router ABI is available as IUniswapV2Router02

       const tokenAddress = "YOUR_ERC20_TOKEN_ADDRESS"; // Replace with your deployed ERC20 token address

       const liquidityToRemove = await getLiquidityBalance(PAIR_ADDRESS, deployer.address); // Implement getLiquidityBalance to fetch LP token balance

       const amountAMin = 0;
       const amountBMin = 0;
       const to = deployer.address;
       const deadline = Math.floor(Date.now() / 1000) + 60 * 10; // 10 minutes from now

       // Approve router to spend LP tokens
       const pairContract = await hre.ethers.getContractAt("IUniswapV2Pair", PAIR_ADDRESS, deployer); // Assuming Uniswap V2 Pair ABI is available as IUniswapV2Pair
       await pairContract.approve(ROUTER_ADDRESS, liquidityToRemove);


       const tx = await router.removeLiquidity(
         tokenAddress,
         WETH_ADDRESS,
         liquidityToRemove,
         amountAMin,
         amountBMin,
         to,
         deadline
       );
       await tx.wait();

       console.log("Liquidity removed and fees collected (if any)");
     }

     // Helper function to get LP token balance (example - you might need to adjust based on your setup)
     async function getLiquidityBalance(pairAddress, accountAddress) {
         const pairContract = await hre.ethers.getContractAt("IUniswapV2Pair", pairAddress); // Assuming Uniswap V2 Pair ABI
         const balance = await pairContract.balanceOf(accountAddress);
         return balance;
     }


     main()
       .then(() => process.exit(0))
       .catch((error) => {
         console.error(error);
         process.exit(1);
       });
     ```
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

This guide provides a more detailed step-by-step process with example code snippets and configuration instructions to help you build the project from scratch. Remember to adapt the scripts and configurations to your specific setup and refer to the official Uniswap documentation for more advanced features and details. Good luck!
