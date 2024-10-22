# Build HelloWorld " BLENDED App on Fluent Devnet "

## Step 1: Updates and Installation of Required Tools

### Update Packages
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential -y
```

```bash
# Node.js and npm Installation
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# pnpm Installation
npm install -g pnpm
```

### Rust and Cargo Installation
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

rustup target add wasm32-unknown-unknown
```
## Step 2: Initialize Rust Project

### Set Up Rust Project
```bash
cargo new --lib greeting
cd greeting
```

### Configure Rust Project

Enter the file with the `nano Cargo.toml` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
[package]
edition = "2021"
name = "greeting"
version = "0.1.0"

[dependencies]
alloy-sol-types = {version = "0.7.4", default-features = false}
fluentbase-sdk = {git = "https://github.com/fluentlabs-xyz/fluentbase", default-features = false}

[lib]
crate-type = ["cdylib", "staticlib"] # For accessing the C lib
path = "src/lib.rs"

[profile.release]
lto = true
opt-level = 'z'
panic = "abort"
strip = true

[features]
default = []
std = [
  "fluentbase-sdk/std",
]
```

### Write Rust Smart Contract
Firstly, remove the codes of the Rust Smart Contract with the `rm src/lib.rs` command. Then enter the file with the `nano src/lib.rs` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
#![cfg_attr(target_arch = "wasm32", no_std)]
extern crate alloc;
extern crate fluentbase_sdk;

use alloc::string::{String, ToString};
use fluentbase_sdk::{
    basic_entrypoint,
    derive::{router, signature},
    SharedAPI,
};

#[derive(Default)]
struct ROUTER;

pub trait RouterAPI {
    fn greeting<SDK: SharedAPI>(&self) -> String;
}

#[router(mode = "solidity")]
impl RouterAPI for ROUTER {
    #[signature("function greeting() external returns (string)")]
    fn greeting<SDK: SharedAPI>(&self) -> String {
        "Hello".to_string()
    }
}

impl ROUTER {
    fn deploy<SDK: SharedAPI>(&self) {
        // any custom deployment logic here
    }
}
basic_entrypoint!(ROUTER);
```

### Create a Makefile
Enter the file with the `nano Makefile` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
.DEFAULT_GOAL := all

# Compilation flags
RUSTFLAGS := '-C link-arg=-zstack-size=131072 -C target-feature=+bulk-memory -C opt-level=z -C strip=symbols'

# Paths to the target WASM file and output directory
WASM_TARGET := ./target/wasm32-unknown-unknown/release/greeting.wasm
WASM_OUTPUT_DIR := bin
WASM_OUTPUT_FILE := $(WASM_OUTPUT_DIR)/greeting.wasm

# Commands
CARGO_BUILD := cargo build --release --target=wasm32-unknown-unknown --no-default-features
RM := rm -rf
MKDIR := mkdir -p
CP := cp

# Targets
all: build

build: prepare_output_dir
	@echo "Building the project..."
	RUSTFLAGS=$(RUSTFLAGS) $(CARGO_BUILD)

	@echo "Copying the wasm file to the output directory..."
	$(CP) $(WASM_TARGET) $(WASM_OUTPUT_FILE)

prepare_output_dir:
	@echo "Preparing the output directory..."
	$(RM) $(WASM_OUTPUT_DIR)
	$(MKDIR) $(WASM_OUTPUT_DIR)

.PHONY: all build prepare_output_dir
```

### Build Wasm Project
```bash
make
```
## Step 3: Initialize Solidity Project

### Create Project Directory
```bash
cd
mkdir typescript-wasm-project

mkdir -p ~/typescript-wasm-project/greeting/bin
cp ~/greeting/target/wasm32-unknown-unknown/release/greeting.wasm ~/typescript-wasm-project/greeting/bin/

cd typescript-wasm-project
npm init -y
```
### Install Dependencies
```bash
npm install --save-dev typescript ts-node hardhat hardhat-deploy ethers dotenv @nomicfoundation/hardhat-toolbox @typechain/ethers-v6 @typechain/hardhat @types/node
pnpm add ethers@^5.7.2 @nomiclabs/hardhat-ethers@2.0.6
pnpm install
npx hardhat
```
After `npx hardhat` command, it will ask us for some information, Select `create a typescript project` then enter and y

## Configure TypeScript and Hardhat

### Update Hardhat Configuration
remove the codes of the Hardhat with the `rm hardhat.config.ts` command. Then enter the file with the `nano hardhat.config.ts` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "hardhat-deploy";
import * as dotenv from "dotenv";
import "./tasks/get-greeting"; 
import "@nomiclabs/hardhat-ethers"; 

dotenv.config();

const config: HardhatUserConfig = {
  defaultNetwork: "dev",
  networks: {
    dev: {
      url: process.env.RPC_URL || "https://rpc.dev.thefluent.xyz/",
      accounts: [process.env.DEPLOYER_PRIVATE_KEY || "your-private-key"],
      chainId: 20993,
    },
  },
  solidity: {
    compilers: [
      {
        version: "0.8.20",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
      {
        version: "0.8.24",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },
  namedAccounts: {
    deployer: {
      default: 0,
    },
  },
};

export default config;
```

### Update Package
Remove the codes of the package.json with the `rm package.json` command. Then enter the file with the `nano package.json` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
{
  "name": "blendedapp",
  "version": "1.0.0",
  "description": "Blended Hello, World",
  "main": "index.js",
  "scripts": {
    "compile": "npx hardhat compile",
    "deploy": "npx hardhat deploy"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-ethers": "^3.0.0",
    "@nomicfoundation/hardhat-toolbox": "^5.0.0",
    "@nomicfoundation/hardhat-verify": "^2.0.0",
    "@openzeppelin/contracts": "^5.0.2",
    "@typechain/ethers-v6": "^0.5.0",
    "@typechain/hardhat": "^9.0.0",
    "@types/node": "^20.12.12",
    "dotenv": "^16.4.5",
    "hardhat": "^2.22.4",
    "hardhat-deploy": "^0.12.4",
    "ts-node": "^10.9.2",
    "typescript": "^5.4.5"
  },
  "dependencies": {
    "ethers": "^6.12.2",
    "fs": "^0.0.1-security"
  }
}
```

### Set Up Environment Variables
Get ETH from [Fluent Devnet Faucet](https://faucet.dev.thefluent.xyz/) to the wallet will use.

Enter your private key where it says `your-private-key-here`.

```bash
nano .env
```

```bash
DEPLOYER_PRIVATE_KEY=your-private-key-here
```

## Write Solidity Contracts

### Define the Interface
Enter the file with the `nano contracts/IFluentGreeting.sol` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IFluentGreeting {
    function greeting() external view returns (string memory);
}
```

### Implement Greeting Contract
Enter the file with the `nano contracts/GreetingWithWorld.sol` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./IFluentGreeting.sol";

contract GreetingWithWorld {
    IFluentGreeting public fluentGreetingContract;

    constructor(address _fluentGreetingContractAddress) {
        fluentGreetingContract = IFluentGreeting(_fluentGreetingContractAddress);
    }

    function getGreeting() external view returns (string memory) {
        string memory greeting = fluentGreetingContract.greeting();
        return string(abi.encodePacked(greeting, ", World"));
    }
}
```

## Step 4: Deploy Both Contracts Using Hardhat

###  Deployment Script

This deployment script is responsible for deploying both the Rust smart contract (compiled to Wasm) and the Solidity smart contract.

Firstly, create a `deploy` folder with `mkdir deploy` command, then enter the file with the `nano deploy/01_deploy_contracts.ts` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";
import { ethers } from "hardhat";
import fs from "fs";
import crypto from "crypto";
import path from "path";
require("dotenv").config();

const DEPLOYER_PRIVATE_KEY = process.env.DEPLOYER_PRIVATE_KEY || "your-private-key";

const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts, network } = hre;
  const { deploy, save, getOrNull } = deployments;
  const { deployer: deployerAddress } = await getNamedAccounts();

  console.log("deployerAddress", deployerAddress);

  // Deploying WASM Contract
  console.log("Deploying WASM contract...");
  const wasmBinaryPath = "./greeting/bin/greeting.wasm";
  const provider = new ethers.providers.JsonRpcProvider(network.config.url);
  const deployer = new ethers.Wallet(DEPLOYER_PRIVATE_KEY, provider);

  const fluentGreetingContractAddress = await deployWasmContract(wasmBinaryPath, deployer, provider, getOrNull, save);

  // Deploying Solidity Contract
  console.log("Deploying GreetingWithWorld contract...");
  const greetingWithWorld = await deploy("GreetingWithWorld", {
    from: deployerAddress,
    args: [fluentGreetingContractAddress],
    log: true,
  });

  console.log(`GreetingWithWorld contract deployed at: ${greetingWithWorld.address}`);
};

async function deployWasmContract(
  wasmBinaryPath: string,
  deployer: ethers.Wallet,
  provider: ethers.providers.JsonRpcProvider,
  getOrNull: any,
  save: any
) {
  const wasmBinary = fs.readFileSync(wasmBinaryPath);
  const wasmBinaryHash = crypto.createHash("sha256").update(wasmBinary).digest("hex");
  const artifactName = path.basename(wasmBinaryPath, ".wasm");
  const existingDeployment = await getOrNull(artifactName);

  if (existingDeployment && existingDeployment.metadata === wasmBinaryHash) {
    console.log("WASM contract bytecode has not changed. Skipping deployment.");
    console.log(`Existing contract address: ${existingDeployment.address}`);
    return existingDeployment.address;
  }

  const gasPrice = (await provider.getFeeData()).gasPrice;

  const transaction = {
    data: "0x" + wasmBinary.toString("hex"),
    gasLimit: 3000000,
    gasPrice: gasPrice,
  };

  const tx = await deployer.sendTransaction(transaction);
  const receipt = await tx.wait();

  if (receipt && receipt.contractAddress) {
    console.log(`WASM contract deployed at: ${receipt.contractAddress}`);

    const artifact = {
      abi: [],
      bytecode: "0x" + wasmBinary.toString("hex"),
      deployedBytecode: "0x" + wasmBinary.toString("hex"),
      metadata: wasmBinaryHash,
    };

    const deploymentData = {
      address: receipt.contractAddress,
      ...artifact,
    };

    await save(artifactName, deploymentData);
  } else {
    throw new Error("Failed to deploy WASM contract");
  }

  return receipt.contractAddress;
}

export default func;
func.tags = ["all"];
```

### Create Hardhat Task
Firstly, create a `tasks` folder with `mkdir tasks` command, then enter the file with the `nano tasks/get-greeting.ts` command and edit it as in the code below. 
(Ctrl + X, Y and Enter will do to save)

```bash
import { task } from "hardhat/config";
import { ethers } from "hardhat";

task("get-greeting", "Fetches the greeting from the deployed GreetingWithWorld contract")
  .addParam("contract", "The address of the deployed GreetingWithWorld contract")
  .setAction(async ({ contract }, hre) => {
    const GreetingWithWorld = await hre.ethers.getContractAt("GreetingWithWorld", contract);
    const greeting = await GreetingWithWorld.getGreeting();
    console.log("Greeting:", greeting);
  });
```

## Step 5: Compile and Deploy the Contracts
Deploy the contract with the `pnpm hardhat deploy` command, then we get the tx output like "contract depolyed at: 0x.."

In below change the `pnpm hardhat get-greeting --contract "your deployed contact address" `



```bash
pnpm hardhat compile

pnpm hardhat deploy

pnpm hardhat get-greeting --contract 0x.....
```

Now you see `Greeting Hello, world` , it means it is done. You can search for tx through Explorer with your public address..

https://blockscout.dev.thefluent.xyz/



