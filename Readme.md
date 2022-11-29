## Bounty Challenge
To send the token from one chain to another on testnet and also store the information associated with the transaction on recievers contract.

#### I made changes in `examples/call-contract-with-token`

### Here is what i did

#### Changes in Solidity file

1. Created struct to store the transaction data. Each transaction would look like this
    ```Solidity
       struct paymentInfo {
        string message;
        bytes payload;
        uint256 timeStamp;
        address sender;
    }
    ```

2. To store each transactions I created the mapping along with the transaction id variable that will be incremented before each transaction to give it unique id.
   ```Solidity
    mapping(uint256 => paymentInfo) private s_Transactions;
    uint256 private s_txId = 0;
    ```

3. Created another parameter to receive the message
   ```Solidity
    function sendToMany(
        string memory destinationChain,
        string memory destinationAddress,
        address[] calldata destinationAddresses,
        string memory symbol,
        uint256 amount,
        string calldata message // New parameter for message
    ) 
    ```

4. Encoded the message along with the payload and now making call to the gasReciever and then gateway
   ```Solidity
        bytes memory payload = abi.encode(destinationAddresses, msg.sender, message);
        if (msg.value > 0) {
            gasReceiver.payNativeGasForContractCallWithToken{ value: msg.value }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                symbol,
                amount,
                msg.sender
            );
        }

        gateway.callContractWithToken(destinationChain, destinationAddress, payload, symbol, amount);
   ```

5. Changed `_executeWithToken` to store the transaction data like this after decoding payload
   ```Solidity
        s_txId++;
        s_Transactions[s_txId] = paymentInfo(message, payload, block.timestamp, sender);
   ```

#### Changes in `index.js`

1. Added one more variable to accept message from command line
   ```Javascript
    const accounts = args.slice(3,4);
    const userMessage = args[4];
    ```
2. Changed functon call to accept message argument and printing transaction hash once it is confirmed
   ```Javascript
    const sendTx = await source.contract.sendToMany(destination.name, destination.distributionExecutable, accounts, 'aUSDC', amount, userMessage, {
        value: BigInt(Math.floor(gasLimit * gasPrice)),
    });
    await sendTx.wait();
    console.log("--transaction hash --")
    console.log(sendTx.hash);
    ```
3. At the end printing transaction that is stored on the reciever's contract
   ```Javascript
    console.log("--transaction data---")
    const res = await destination.contract.getTransactionWithID(Number(txID)+1);
    console.log(res.toString());
   ```

### changes in `testnet.json`
Only kept 3 networks
   ```Json
   [
  {
    "name": "Avalanche",
    "chainId": 43113,
    "rpc": "https://api.avax-test.network/ext/bc/C/rpc",
    "gateway": "0xC249632c2D40b9001FE907806902f63038B737Ab",
    "gasReceiver": "0xbE406F0189A0B4cf3A05C286473D23791Dd44Cc6",
    "tokenName": "Avax",
    "tokenSymbol": "AVAX",
    "executableSample": "0x9372350Abb3B5c10DB1BB858e0bCf91eFa74d946",
    "constAddressDeployer": "0xd5bf7311032fe5dde956de6f2916f135a140e7dd",
    "crossChainToken": "0xb5ADB662B1DebDCc96c4d7c406aDF20F310aEe0B",
    "distributionExecutable": "0xAc4af61e20AA7303d059d44141e4570A5e41561e"
  },
  {
    "name": "Polygon",
    "chainId": 80001,
    "gateway": "0xBF62ef1486468a6bd26Dd669C06db43dEd5B849B",
    "rpc": "https://polygon-mumbai.g.alchemy.com/v2/Ksd4J1QVWaOJAJJNbr_nzTcJBJU-6uP3",
    "gasReceiver": "0xbE406F0189A0B4cf3A05C286473D23791Dd44Cc6",
    "tokenName": "Matic",
    "tokenSymbol": "MATIC",
    "executableSample": "0x49Bd72FD6f59B57418DA42521628EfFD7df9DB1B",
    "constAddressDeployer": "0xd5bf7311032fe5dde956de6f2916f135a140e7dd",
    "crossChainToken": "0xb5ADB662B1DebDCc96c4d7c406aDF20F310aEe0B",
    "distributionExecutable": "0x2B432658A86A89246273a91E1AbE091a3307FCC4"
  },
  {
    "name": "Moonbeam",
    "chainId": 1287,
    "gateway": "0x5769D84DD62a6fD969856c75c7D321b84d455929",
    "rpc": "https://moonbeam-alpha.api.onfinality.io/public",
    "gasReceiver": "0xbE406F0189A0B4cf3A05C286473D23791Dd44Cc6",
    "tokenName": "DEV",
    "tokenSymbol": "DEV",
    "executableSample": "0x2120A291a99746726CFa3bCcd3C388da75f7e718",
    "constAddressDeployer": "0xd5bf7311032fe5dde956de6f2916f135a140e7dd",
    "crossChainToken": "0xb5ADB662B1DebDCc96c4d7c406aDF20F310aEe0B",
    "distributionExecutable": "0xaA65057a65FA0e47d5Aa54bE2aD1D4c7A3a8b8f0"
  }
]
```
### Making transaction on testnet
After deploying with this command `node scripts/deploy examples/call-contract-with-token testnet`

Ran this command to make transaction `node scripts/test examples/call-contract-with-token testnet "M testnet "Moonbeam" "Polygon"  1 0xe66d5D2158CbF77bFdCDB2131D5eB8FF50046D77 "hello from me"`

***The transaction data***

```bash
--- Initially ---
0xe66d5D2158CbF77bFdCDB2131D5eB8FF50046D77 has 4 aUSDC
--transaction hash --
0x5bfce8dca3cc5e2ba089f4c3a85ce33fe88498af8451198a665c9c6f69180289
--- After ---
0xe66d5D2158CbF77bFdCDB2131D5eB8FF50046D77 has 5 aUSDC
--transaction data---
hello from me,0x0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000e66d5d2158cbf77bfdcdb2131d5eb8ff50046d7700000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001000000000000000000000000e66d5d2158cbf77bfdcdb2131d5eb8ff50046d77000000000000000000000000000000000000000000000000000000000000000d68656c6c6f2066726f6d206d6500000000000000000000000000000000000000,1669698971,0xe66d5D2158CbF77bFdCDB
```