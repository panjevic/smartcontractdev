1. Open VSCode and open a terminal window (`CTRL + ~`) <br/>__NOTE: GitBash works best for this ;)__
2. Initialize a new project for truffle
```
truffle init
```
3. This will create folder and base templates files for a new project that will serve as a base.  For the initial development, build and test we will use a built in Ethereum blockchain called Ganache to allow rapid development (instant transactions).  To get started, create a new smart contract to show storing state and retrieving state.  This is a simple example but demonstrates the most core operations of the blockchain.
4. In the contract folder, create a new file named `StoredState.sol`
5. Now we need to add some code to this file.  Add the following code snippet
```
pragma solidity ^0.4.23;

contract StoredState {
    uint storedData;

    function StoredState() public {
        storedData = 7;
    }

    function get() public view returns (uint) {
        return storedData;
    }

    function set(uint newVal) public {
        storedData = newVal;
    }
}
```
6. Next, we can compile this contract by running the following command.  Compilation will simply validate the syntax of the contract and create 2 output (binary and abi).  These can be found in the build folder after succesful compilation.
```
truffle compile
```
7. Next, we will want to deploy these contracts to a blockchain to start testing them.  In order to do this, we will write a deployment artifact to tell truffle how to deploy the contract.  To start, create a new file in the migrations folder named `2_deploy_contracts.js`
8. Open this file and add the following code snippet which will describe how to deploy the contract.
```
var StoredState = artifacts.require('./StoredState.sol');

module.exports = function(deployer){
    deployer.deploy(StoredState);
};
```
9. To deploy the contract, simple run the following commands:
```
truffle develop
migrate
```
10. Now you have Ganache running (a blockchain in memory), and you have deployed a smart contract, StoredState, to this chain.  Next we will do some light testing to ensure the contract actually works.  To start we will retrieve the current state, which should be a single unsigned integer set to 7.  To do this run the following command:
```
StoredState.deployed().then((i) => { return i.get.call() })
```
This should return this
```
BigNumber { s: 1, e: 0, c: [ 7 ] }
```
This validates that 7 is actually stored in the contract.
11.  Next we will change this value (update the state), with a simple transaction.  To do this run the following command:
```
StoredState.deployed().then((i) => { return i.set(42) })
```
When this is executed a transaction has been created to update the state.  The output will be similar to this.
```
{ tx: '0xa12ec414ec02382dc8eefd144335660c621d64dcd8d6d0b89c36d613c583a47e',
  receipt:
   { transactionHash: '0xa12ec414ec02382dc8eefd144335660c621d64dcd8d6d0b89c36d613c583a47e',
     transactionIndex: 0,
     blockHash: '0xad04012a2f940ce36fba4cc6b72e354815315bc58d77f3ce5a71d69032d6d26d',
     blockNumber: 5,
     gasUsed: 26669,
     cumulativeGasUsed: 26669,
     contractAddress: null,
     logs: [],
     status: '0x01',
     logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000' },
  logs: [] }
  ```
  12.  Now we can validate the state change by calling the getter function again.
  ```
  StoredState.deployed().then((i) => { return i.get.call() })
  ```
  This will output the updated state.
  ```
  BigNumber { s: 1, e: 1, c: [ 42 ] }
  ```
  13.  Finally we will want to now share this with others and publish to a blockchain beyond our local development machine.  To do this we will need to modify the truffle configuration to target our external private blockchain.  First open the truffle.js, remove the existing contents and replace with the snippet below.
  ```
  module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*" // Match any network id
    },
    azure_node0: {
      host: "<your VM external ip>",
      port: 8501,
      network_id: "1010",
      gasPrice: 0,
      gas: 500000,
      from: "<your address for node 1>"
    },
    azure_node1: {
      host: "<your VM external ip>",
      port: 8502,
      network_id: "1010",
      gasPrice: 0,
      gas: 500000,
      from: "<your address for node 2>"
    }
  }
};
```
14. Next we will run the migrate using the following command:
```
truffle migrate --network azure_node0
```
After a few seconds you should see the deployment of these resources to the Ethereum node running in Azure.