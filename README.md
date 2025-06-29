# Start - Cross-Chain Bridge Event Listener Simulation

This repository contains a Python-based simulation of an event listener component for a cross-chain bridge. It is designed as a robust, asynchronous application that demonstrates the architectural patterns required to monitor and process events from multiple blockchains concurrently.

This script is for educational and demonstrative purposes and simulates blockchain interactions rather than connecting to live networks.

## Concept

A cross-chain bridge allows users to transfer assets or data from one blockchain (e.g., Ethereum) to another (e.g., Polygon). A critical component of such a bridge is the **off-chain listener** (also known as a relayer or oracle).

This listener's job is to:
1.  **Monitor** the bridge's smart contract on the source chain for specific events, such as a user depositing tokens (`Deposit` event).
2.  **Verify** and **process** this event securely.
3.  **Relay** a corresponding message to the destination chain. This typically involves a group of validators signing a message that authorizes the bridge contract on the destination chain to release or mint the equivalent assets to the user (`Withdrawal` action).

This script simulates the first two steps of this process: listening for deposits on multiple chains and processing them in a structured way.

## Code Architecture

The application is built with a clear separation of concerns, using several key classes to manage different parts of the process:

```
+----------------------------+
|     CrossChainBridgeListener (Orchestrator)    |
| - Manages all components   |
| - Runs main async loop     |
+-------------+--------------+
              |
              | Manages
              v
+----------------------------+      +----------------------------+
|   BlockchainConnector (1)  |      |   BlockchainConnector (N)  |
| - Connects to Chain A      |      | - Connects to Chain B      |
| - Simulates event fetching |      | - Simulates event fetching |
+-------------+--------------+      +-------------+--------------+
              |                              |
              +------------------------------+
              | Emits DepositEvent
              v
+----------------------------+
|       EventProcessor       |
| - Validates event          |
| - Enriches data (API calls)|
| - Simulates next steps     |
| - Reports to monitoring    |
+----------------------------+
```

-   **`CrossChainBridgeListener`**: The main class that orchestrates the entire system. It initializes and manages one `BlockchainConnector` for each chain being monitored and passes any detected events to the `EventProcessor`.

-   **`BlockchainConnector`**: Responsible for the logic of connecting to a single blockchain. It contains a (simulated) method `listen_for_deposits` which continuously polls for new events. In a real-world scenario, this class would use `web3.py` to interact with a node via RPC.

-   **`EventProcessor`**: This class contains the business logic for what to do after an event is detected. Its responsibilities include:
    -   Validating the event data.
    -   Enriching the event with external data (e.g., fetching a token's price from an API like CoinGecko).
    -   Simulating the next step, such as requesting signatures from a validator network.
    -   Reporting the processed event to an external monitoring service.

-   **`DepositEvent` (Dataclass)**: A simple data structure that provides type safety and clarity for passing event data between components.

-   **`Config`**: A centralized class to hold configuration variables like RPC endpoints and contract addresses, making the application easy to configure.

The entire application is built on Python's `asyncio` library to handle concurrent network operations (listening to multiple chains, making API calls) efficiently.

## How it Works

1.  **Initialization**: The `main` function creates an instance of `CrossChainBridgeListener`, configured to listen to a list of chain IDs (e.g., `[1, 137]` for Ethereum and Polygon).

2.  **Connecting**: The `CrossChainBridgeListener` creates a `BlockchainConnector` instance for each configured chain.

3.  **Concurrent Listening**: The listener starts an independent, asynchronous task for each `BlockchainConnector`. Each task enters an infinite loop where it periodically simulates a check for new `Deposit` events.

4.  **Event Detection**: The `BlockchainConnector`'s simulation logic randomly generates a `DepositEvent` to mimic a user depositing funds into the bridge contract. The event contains details like the sender, recipient, amount, and destination chain.

5.  **Event Processing**: When an event is detected, it is passed to the single `EventProcessor` instance. The processor then executes the following steps asynchronously:
    a.  **Validation**: Checks if the event data is plausible (e.g., amount is not zero).
    b.  **Enrichment**: Makes an asynchronous HTTP GET request using `aiohttp` to an external API (CoinGecko) to fetch the current price of the token.
    c.  **Action Simulation**: Simulates the crucial step of creating a message hash and broadcasting it to validators for signing. This is where the off-chain consensus for the withdrawal would begin.
    d.  **Reporting**: Makes an asynchronous HTTP POST request to a mock monitoring endpoint, sending a summary of the processed event.

6.  **Looping**: The process repeats, allowing the listener to handle events from multiple chains in near real-time.

## Usage Example

### 1. Setup

First, clone the repository and navigate into the directory:

```bash
# This project is named 'Start'
git clone <your-repo-url>/Start.git
cd Start
```

Create a virtual environment and activate it:

```bash
python -m venv venv
# On Windows
source venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate
```

Install the required dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 2. Running the Script

Execute the Python script from your terminal:

```bash
python script.py
```

The script will start and you will see log output in your console as it initializes the connectors and begins polling for simulated events.

### 3. Sample Output

The output will look similar to this, showing the concurrent operation of listeners on both chains and the processing of detected events:

```
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - Connector initialized for chain ID 1 at https://mainnet.infura.io/v3/YOUR_INFURA_KEY
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - Connector initialized for chain ID 137 at https://polygon-rpc.com/
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - Starting bridge listener for chains: [1, 137]
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - EventProcessor HTTP session started.
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - Starting to listen for 'Deposit' events on chain 1...
2023-10-27 10:30:00 - CrossChainBridgeListener - INFO - Starting to listen for 'Deposit' events on chain 137...
2023-10-27 10:30:06 - CrossChainBridgeListener - INFO - [Chain 1] New event detected: Tx 0xc8d7a1...
2023-10-27 10:30:06 - CrossChainBridgeListener - INFO - Processing event nonce 1 from chain 1 to 137
2023-10-27 10:30:07 - CrossChainBridgeListener - INFO - Enriched event data: Deposit value ~$1523.45 USD
2023-10-27 10:30:07 - CrossChainBridgeListener - INFO - Requesting signatures for message hash: 0x2abf...
2023-10-27 10:30:07 - CrossChainBridgeListener - INFO - Successfully reported event nonce 1 to monitoring service.
2023-10-27 10:30:07 - CrossChainBridgeListener - INFO - Successfully processed event nonce 1.
2023-10-27 10:30:08 - CrossChainBridgeListener - INFO - [Chain 137] New event detected: Tx 0xfa34b2...
2023-10-27 10:30:08 - CrossChainBridgeListener - INFO - Processing event nonce 1 from chain 137 to 1
...
```

To stop the listener, press `Ctrl+C` for a graceful shutdown.
