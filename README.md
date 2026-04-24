<p align="center">
  <a href="">
    <img alt="Hero" src= "https://github.com/user-attachments/assets/a4f1aaa1-ea8b-4f53-bc71-61e220a887d1" />
  </a>
</p>

## First Smart Contract Deployment Guide to Arc Testnet (Foundry + Ubuntu 22.04)
- This guide is compiled from Arc Docs, primarily the official "Deploy on Arc" documentation. Our goal is to deploy a simple smart contract to Arc Testnet using Foundry on Ubuntu 22.04 from scratch and interact with it.

### Table of Contents

- Arc Testnet Connection Information
- Prerequisites
- System Dependencies and Foundry Installation (Ubuntu 22.04)
- Foundry Project Creation and Arc RPC Configuration
- Writing the HelloArchitect Contract
- Testing and Compilation
- Wallet Creation, Getting Testnet USDC, and Deployment
- Interaction: Explorer Check and Reading with Cast

1️⃣ Arc Testnet Connection Information

- These values will be used throughout the setup (from the official Connect to Arc page).
- Network name: Arc Testnet
- RPC endpoint: https://rpc.testnet.arc.network
- Chain ID: 5042002
- Currency (gas): USDC
- Block explorer: https://testnet.arcscan.app
- Faucet: https://faucet.circle.com (to get USDC, network selection: Arc Testnet)

2️⃣ Prerequisites

- Machine: Ubuntu 22.04 (server/desktop does not matter)
- User: A user with sudo privileges or root
- Network: A connection that can go out over HTTPS (for foundryup and RPC)
- Wallet: You can optionally install MetaMask in your browser; however, this guide uses Foundry's cast wallet new CLI wallet
- Account: A wallet address to get testnet USDC from the Circle Faucet (we will create it via CLI)

3️⃣ System Dependencies and Foundry Installation (Ubuntu 22.04)

- Although Foundry's own installation consists of just two commands, it is good practice to install basic build tools first on Ubuntu.

3.1) Update system packages and install basic dependencies.
```
sudo apt update
sudo apt install -y build-essential curl git ca-certificates
```

3.2) Download and install the foundryup script.

- Official installation (same flow as Foundry README and Arc documentation): 
````
curl -L https://foundry.paradigm.xyz | bash
````
- In the output, it will say that foundryup has been added to the PATH (usually ~/.bashrc or similar). To activate it:
````
source ~/.bashrc   # or depending on your shell: source ~/.zshrc
````

3.3) Install Foundry tools (forge, cast, anvil, chisel).

- All at once:
````
foundryup
````

3.4) Verify the installation.
````
forge --version
cast --version
anvil --version
chisel --version
````
- At least the forge and cast versions should be visible.

4️⃣ Foundry Project Creation and Arc RPC Configuration

This section is taken verbatim from the Arc documentation.

4.1) Initialize a new Foundry project.
````
forge init hello-arc && cd hello-arc
````
- This creates src/, script/, test/ etc. directories along with the classic Counter.sol template.

4.2) Create a .env file for Arc RPC.

- Arc recommends storing the RPC URL in a .env file.
- In the project root (hello-arc/):
````
nano .env
````
- Add the following to the .env file ( RPC, from the Arc Connect page):
````
# Arc Testnet RPC
ARC_TESTNET_RPC_URL="https://rpc.testnet.arc.network"

# We will use this for deployment shortly:
# PRIVATE_KEY="0x..."           # NEVER write in plain text in .env in real projects; this guide is only for testing
# HELLOARCHITECT_ADDRESS="0x..." # To be filled after deployment
````

5️⃣ Writing the HelloArchitect Contract

- The following steps are verbatim from the Arc documentation; only file paths are shown suitable for Ubuntu.

5.1) Delete the default Counter.sol.
````
ls -R test/ | grep -i counter
rm src/Counter.sol
forge clean
````
5.2) Create src/HelloArchitect.sol and paste the following code.
````
nano src/HelloArchitect.sol
````
- Content (as in the documentation):
````
solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

contract HelloArchitect {
    string private greeting;

    // Event emitted when the greeting is changed
    event GreetingChanged(string newGreeting);

    // Constructor that sets the initial greeting to "Hello Architect!"
    constructor() {
        greeting = "Hello Architect!";
    }

    // Setter function to update the greeting
    function setGreeting(string memory newGreeting) public {
        greeting = newGreeting;
        emit GreetingChanged(newGreeting);
    }

    // Getter function to return the current greeting
    function getGreeting() public view returns (string memory) {
        return greeting;
    }
}
````

6️⃣ Testing and Compilation

6.1) Clean up old script and test files.

- The Arc documentation recommends removing script/test files that reference Counter.sol.
```
rm -rf script
````
6.2) Create a new test file.
````
nano test/HelloArchitect.t.sol
````
- Paste the following code into the test/HelloArchitect.t.sol file (as in the documentation):
````
solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

import "forge-std/Test.sol";
import "../src/HelloArchitect.sol";

contract HelloArchitectTest is Test {
    HelloArchitect helloArchitect;

    function setUp() public {
        helloArchitect = new HelloArchitect();
    }

    function testInitialGreeting() public view {
        string memory expected = "Hello Architect!";
        string memory actual = helloArchitect.getGreeting();
        assertEq(actual, expected);
    }

    function testSetGreeting() public {
        string memory newGreeting = "Welcome to Arc Chain!";
        helloArchitect.setGreeting(newGreeting);
        string memory actual = helloArchitect.getGreeting();
        assertEq(actual, newGreeting);
    }

    function testGreetingChangedEvent() public {
        string memory newGreeting = "Building on Arc!";
        // Expect the GreetingChanged event to be emitted
        vm.expectEmit(true, true, true, true);
        emit HelloArchitect.GreetingChanged(newGreeting);
        helloArchitect.setGreeting(newGreeting);
    }
}
````

6.3) Run the tests.
````
forge test
````
- All tests should pass.

6.4) Compile the contract.
````
forge build
````
- This generates bytecode and ABI in the out/ directory; Foundry will use these during deployment.

7️⃣ Wallet Creation, Getting Testnet USDC, and Deployment

-This section is again exactly aligned with the "Deploy your contract to Arc testnet" section of the Arc documentation.

7.1) Create a new wallet.
````
cast wallet new
````
- The output will look like this (example output from the documentation):
````
Successfully created new keypair.
Address:     0xB815A0c4bC23930119324d4359dB65e27A846A2d
Private key: 0xcc1b30a6af68ea9a9917f1dd.......................................97c5
````
- Address → we will send testnet USDC to this address
- Private key → we will use this for deployment (since it is testnet, we will put it in .env in this guide; never do this in production)

7.2) Update the .env file.

- Add the PRIVATE_KEY value from earlier to the .env file:

```
# Arc Testnet RPC
ARC_TESTNET_RPC_URL="https://rpc.testnet.arc.network"

# For deployment (TESTNET ONLY)
PRIVATE_KEY="0xcc1b30a6af68ea9a9917f1dd....97c5"  # change this with your own private key

# To be filled after deployment
# HELLOARCHITECT_ADDRESS="0x...."
````
- Save:
````
source .env
````

7.3) Send testnet USDC to the wallet.

- Copy the Address: value from the cast wallet new output.
- Go to the Faucet from your browser: https://faucet.circle.com
- Select "Arc Testnet" as the network (or it will automatically recognize when you enter the address).
- Paste the address and request testnet USDC.
- Since USDC is the native gas token on Arc, this balance is sufficient to pay for gas during deployment.

7.4) Deploy the contract to Arc Testnet.

The deploy command in the Arc documentation (with a very minor formatting adaptation):
````
forge create src/HelloArchitect.sol:HelloArchitect \
  --rpc-url $ARC_TESTNET_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast
````
- If successful, the output will look like this:
````
Compiler run successful!
Deployer:      0xB815A0c4bC23930119324d4359dB65e27A846A2d
Deployed to:   0x32368037b14819C9e5Dbe96b3d67C59b8c65c4BF
Transaction hash: 0xeba0fcb5e528d586db0aeb2465a8fad0299330a9773ca62818a1827560a67346
````
- Save the address from the Deployed to: line.

7.5) Save the contract address to .env and reload.
````
HELLOARCHITECT_ADDRESS="0x32368037b14819C9e5Dbe96b3d67C59b8c65c4BF"  # change with your own address
````
- If successful, the output will look like this:
````
source .env
````

8️⃣ Interaction: Explorer Check and Reading with Cast

8.1) Check on the Explorer.

- Go to the Arc Testnet Explorer: https://testnet.arcscan.app
- Paste the Transaction hash value from the deployment output into the search box.
- View the details: status, from/to (contract creation), etc.

8.2) Call getGreeting() with cast.

- We can apply the example from the Arc documentation exactly:
```
cast call $HELLOARCHITECT_ADDRESS "getGreeting()(string)" \
  --rpc-url $ARC_TESTNET_RPC_URL
````
- Returned value (decoded string):
```
Hello Architect!
````
- This shows that the contract is working live.

8.3) (Optional) Call setGreeting() with cast send.

Example (gas will be paid when you have a USDC balance):
````
cast send $HELLOARCHITECT_ADDRESS \
  "setGreeting(string)" \
  "Hello Arc!" \
  --rpc-url $ARC_TESTNET_RPC_URL \
  --private-key $PRIVATE_KEY
````
- Then again:
````
cast call $HELLOARCHITECT_ADDRESS "getGreeting()(string)" \
  --rpc-url $ARC_TESTNET_RPC_URL
````
- Now it should return ````Hello Arc!````.

Note: The official sources I used are below. You can find more detailed steps in the links.

- Deploy on Arc (with Foundry): https://docs.arc.network/arc/tutorials/deploy-on-arc
- Connect to Arc (RPC, Chain ID, Explorer, Faucet): https://docs.arc.network/arc/references/connect-to-arc
- Foundry official installation: https://github.com/foundry-rs/foundry
