# Getting started with Ethereum Clients

Ethereum is a decentralized platform that allows developers to build and deploy smart contracts and decentralized applications (DApps) on top of its blockchain network. In order to interact with the Ethereum network, you need an Ethereum client. Ethereum clients come in two main types: execution layer clients and consensus layer clients. Execution layer clients are used to deploy and interact with smart contracts, while consensus layer clients participate in the consensus process of validating transactions and maintaining the blockchain network. In this lesson, we will cover the different types of Ethereum clients, how to set them up on your Linux system, and how to interact with the Ethereum network.

## Execution and Consensus Layer Node Clients

#### What is the Ethereum Consensus Layer?

The Ethereum consensus layer refers to the underlying mechanism that enables the network to reach agreement on the state of the blockchain. Consensus is essential in a decentralized system like Ethereum to ensure that all nodes in the network have the same version of the blockchain.

In Ethereum, the consensus layer is implemented using a consensus algorithm called Proof of Work (PoW), which is used to validate transactions and add new blocks to the blockchain. This process involves miners competing to solve complex mathematical puzzles, with the first miner to solve the puzzle being rewarded with newly minted Ether.

However, Ethereum is in the process of transitioning to a new consensus algorithm called Proof of Stake (PoS) through a protocol upgrade known as Ethereum 2.0. Under PoS, validators put up a stake of Ether to participate in block validation, and the probability of being selected to validate a block is proportional to the amount of Ether staked. This mechanism is expected to be more energy-efficient than PoW, which requires a lot of computational power to solve the puzzles.

In summary, the Ethereum consensus layer is the foundational mechanism that enables the network to maintain a consistent and accurate version of the blockchain, and it is implemented through a consensus algorithm such as PoW or PoS.

***

#### What is the Ethereum Execution Layer?

The Ethereum execution layer refers to the component of the Ethereum network responsible for executing and processing smart contracts and decentralized applications (DApps).

Smart contracts are self-executing contracts with the terms of the agreement between buyer and seller being directly written into lines of code. They are a key feature of the Ethereum network and enable developers to create decentralized applications that can run on the blockchain.

The execution layer of Ethereum is implemented using the Ethereum Virtual Machine (EVM), which is a virtual machine that runs on every node in the network. The EVM is responsible for executing the bytecode of smart contracts and DApps, ensuring that the execution is consistent and predictable across all nodes in the network.

The execution layer also includes a range of tools and infrastructure for developers to build and deploy smart contracts and DApps, such as the Solidity programming language and the Truffle development framework.

Overall, the Ethereum execution layer is the component of the network that enables the processing and execution of smart contracts and DApps, and it is implemented using the Ethereum Virtual Machine and associated development tools.

#### What is the relation between the Ethereum consensus layer and execution layer?

The consensus layer is responsible for maintaining the integrity and security of the network by ensuring that all nodes in the network agree on the current state of the blockchain. This is achieved through a consensus algorithm such as Proof of Work (PoW) or Proof of Stake (PoS).

The execution layer, on the other hand, is responsible for executing and processing smart contracts and DApps on the blockchain. This is achieved through the Ethereum Virtual Machine (EVM), which is a decentralized virtual machine that runs on every node in the network.

In summary, while the consensus layer ensures the consistency and security of the network, the execution layer enables developers to create and deploy decentralized applications on the blockchain.

***

### Node Clients

#### What is an Execution Layer (EL) Client?

An Ethereum execution layer client, also known as an Ethereum client, is a piece of software that allows users to interact with the Ethereum network and execute smart contracts and decentralized applications (DApps).

Ethereum clients can be divided into three main categories: full, light and archive node clients. Full nodes store a copy of the entire Ethereum blockchain and validate all transactions and smart contracts on the network, while light clients only download a portion of the blockchain and rely on full nodes to provide them with transaction data and block headers. Full nodes typically store only the most recent state of the blockchain, whereas Archive nodes store a complete history of all states since the beginning of the blockchain. This makes Archive nodes useful for querying historical blockchain data that is not accessible on Full nodes.

There are several Ethereum client implementations available, each with their own features and characteristics. Some of the most popular Ethereum clients include Geth, Nethermind, Erigon, Hyperledger Besu and OpenEthereum.

Ethereum clients typically provide a range of functionalities for interacting with the network, including sending and receiving Ether and other tokens, creating and deploying smart contracts, and interacting with DApps. They also allow developers to build and test their own DApps and smart contracts.

In summary, an Ethereum execution layer client is a software application that allows users to interact with the Ethereum network and execute smart contracts and DApps, and there are several different client implementations available.

#### Popular EL Clients

1. **Geth**: Geth is an Ethereum client implemented in the Go programming language. It is a full node client that allows users to download and store a copy of the entire Ethereum blockchain. Geth is widely used by developers and is known for its reliability and performance.
2. **OpenEthereum**: OpenEthereum, formerly known as Parity-Ethereum, is an Ethereum client implemented in Rust. It is a full node client that is designed to be modular and flexible, allowing users to customize their node's functionality. OpenEthereum is known for its ease of use and comprehensive documentation.
3. **Nethermind**: Nethermind is an Ethereum client implemented in .NET. It is a full node client that is optimized for performance and scalability. Nethermind is known for its user-friendly interface and support for mining.
4. **Besu**: Besu is an Ethereum client implemented in Java. It is a full node client that is designed to be enterprise-friendly, with features such as privacy and permissioning. Besu is known for its compatibility with enterprise blockchain use cases and its focus on interoperability.

#### What is a Consensus Layer (CL) Client?

A consensus layer client is a software application that implements the consensus rules of the Ethereum network. Consensus layer clients play a critical role in ensuring that all nodes on the network agree on the state of the blockchain. These clients are responsible for validating transactions and adding new blocks to the chain based on the rules of the Ethereum consensus protocol.

#### Polular CL Clients

Some popular Ethereum consensus layer clients include Lighthouse, Prysm, Nimbus, Teku, and Lodestar. These clients are designed to implement the Ethereum consensus protocol and are responsible for validating transactions, adding new blocks to the blockchain, and maintaining network consensus.

1. Lighthouse - Lighthouse is an open-source Ethereum 2.0 client developed by Sigma Prime. It is written in Rust and is designed to be fast, efficient, and reliable. Lighthouse supports both mainnet and testnet environments, and includes features such as validator management, peer-to-peer networking, and Eth1 integration.
2. Prysm - Prysm is an open-source Ethereum 2.0 client developed by Prysmatic Labs. It is written in Go and is designed to be easy to use and accessible to developers of all skill levels. Prysm includes features such as validator management, peer-to-peer networking, and Eth1 integration.
3. Nimbus - Nimbus is an open-source Ethereum 2.0 client developed by Status. It is written in Nim and is designed to be lightweight and resource-efficient. Nimbus supports both mainnet and testnet environments, and includes features such as validator management, peer-to-peer networking, and Eth1 integration.
4. Teku - Teku is an open-source Ethereum 2.0 client developed by ConsenSys. It is written in Java and is designed to be scalable, reliable, and secure. Teku supports both mainnet and testnet environments, and includes features such as validator management, peer-to-peer networking, and Eth1 integration.
5. Lodestar - Lodestar is an open-source Ethereum 2.0 client developed by Chainsafe. It is written in TypeScript and is designed to be modular and extensible. Lodestar supports both mainnet and testnet environments, and includes features such as validator management, peer-to-peer networking, and Eth1 integration.

Each of these clients is designed to implement the Ethereum consensus protocol, which is responsible for maintaining network consensus and ensuring the integrity of the blockchain. They provide different features and benefits, depending on the needs of the developer or organization using them.

***

### Setting Up The Client

#### EL Client

*   How to Set Up the Geth Client (Homebrew)

    ```
    brew tap ethereum/ethereum
    brew install ethereum
    brew install ethereum --devel
    ```
*   How to Set Up the Erigon Client

    ```
    git clone --recurse-submodules -j8 https://github.com/ledgerwatch/erigon.git
    cd erigon
    make erigon
    ./build/bin/erigon
    ```
*   How to Set Up the Besu Client (Homebrew)

    ```
    brew tap hyperledger/besu
    brew install hyperledger/besu/besu
    ```
*   How to Set Up the Nethermind Client (Homebrew)

    ```
    brew tap nethermindeth/nethermind
    brew install nethermind
    ```

#### CL Client

*   How to Set Up the Lighthouse Client (Homebrew)

    ```
    brew install lighthouse
    ```
*   How to Set Up the Prysm Client (Homebrew)

    ```
    mkdir prysm && cd prysm
    curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
    ```
*   How to Set Up the Nimbus Client

    ```
    git clone https://github.com/status-im/nimbus-eth1
    cd nimbus-eth1
    make update
    make nimbus
    ./build/nimbus
    ```
*   How to Set Up the Teku Client (Homebrew)

    ```
    brew tap ConsenSys/teku
    brew install ConsenSys/teku/teku
    ```
*   How to Set Up the Lodestar Client

    ```
    npm install -g @chainsafe/lodestar-cli
    ```
