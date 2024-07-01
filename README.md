# Assessment Smart Contract and Frontend Integration

This project demonstrates a simple Ethereum smart contract for managing an account balance with deposit, withdraw, and burn functionalities. It also includes a frontend application to interact with the smart contract using MetaMask.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Smart Contract](#smart-contract)
- [Frontend](#frontend)
- [Setup](#setup)
- [Usage](#usage)
- [License](#license)

## Prerequisites
- Node.js
- MetaMask extension installed in your web browser
- An Ethereum wallet with test ETH (you can get some from a [faucet](https://faucet.rinkeby.io/))

## Smart Contract

### Contract: Assessment

The `Assessment` contract allows the owner to deposit, withdraw, and burn the balance.

#### Functions:
- **constructor**: Initializes the contract with an initial balance.
- **getBalance**: Returns the current balance.
- **deposit**: Allows the owner to deposit a specified amount.
- **withdraw**: Allows the owner to withdraw a specified amount.
- **burn**: Sets the balance to zero.

### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.9;

contract Assessment {
    address payable public owner;
    uint256 public balance;

    event Deposit(uint256 amount);
    event Withdraw(uint256 amount);

    constructor(uint initBalance) payable {
        owner = payable(msg.sender);
        balance = initBalance;
    }

    function getBalance() public view returns(uint256){
        return balance;
    }

    function deposit(uint256 _amount) public payable {
        uint _previousBalance = balance;
        require(msg.sender == owner, "You are not the owner of this account");
        balance += _amount;
        assert(balance == _previousBalance + _amount);
        emit Deposit(_amount);
    }

    error InsufficientBalance(uint256 balance, uint256 withdrawAmount);

    function withdraw(uint256 _withdrawAmount) public {
        require(msg.sender == owner, "You are not the owner of this account");
        uint _previousBalance = balance;
        if (balance < _withdrawAmount) {
            revert InsufficientBalance({
                balance: balance,
                withdrawAmount: _withdrawAmount
            });
        }
        balance -= _withdrawAmount;
        assert(balance == (_previousBalance - _withdrawAmount));
        emit Withdraw(_withdrawAmount);
    }
    
    function burn() public {
        balance = 0;
        emit Withdraw(balance);
    }
}
## Frontend

The frontend is built with React and interacts with the smart contract using ethers.js.

### Key Features:
- Connect to MetaMask
- Display account address and balance
- Deposit ETH
- Withdraw ETH
- Burn all ETH
```
import {useState, useEffect} from "react";
import {ethers} from "ethers";
import atm_abi from "../artifacts/contracts/Assessment.sol/Assessment.json";

export default function HomePage() {
  const [ethWallet, setEthWallet] = useState(undefined);
  const [account, setAccount] = useState(undefined);
  const [atm, setATM] = useState(undefined);
  const [balance, setBalance] = useState(undefined);
  const [amount, setAmount] = useState('');

  const contractAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3";
  const atmABI = atm_abi.abi;

  const getWallet = async() => {
    if (window.ethereum) {
      setEthWallet(window.ethereum);
    }
    if (ethWallet) {
      const account = await ethWallet.request({method: "eth_accounts"});
      handleAccount(account);
    }
  }

  const handleAccount = (account) => {
    if (account) {
      console.log ("Account connected: ", account);
      setAccount(account);
    } else {
      console.log("No account found");
    }
  }

  const connectAccount = async() => {
    if (!ethWallet) {
      alert('MetaMask wallet is required to connect');
      return;
    }
    const accounts = await ethWallet.request({ method: 'eth_requestAccounts' });
    handleAccount(accounts);
    getATMContract();
  };

  const getATMContract = () => {
    const provider = new ethers.providers.Web3Provider(ethWallet);
    const signer = provider.getSigner();
    const atmContract = new ethers.Contract(contractAddress, atmABI, signer);
    setATM(atmContract);
  }

  const getBalance = async() => {
    if (atm) {
      setBalance((await atm.getBalance()).toNumber());
    }
  }

  const deposit = async() => {
    if (atm) {
      let tx = await atm.deposit(Number(amount));
      await tx.wait();
      getBalance();
    }
  }

  const withdraw = async() => {
    if (atm) {
      let tx = await atm.withdraw(Number(amount));
      await tx.wait();
      getBalance();
    }
  }

  const burn = async() => {
    let tx = await atm.burn();
    await tx.wait();
    getBalance();
  }

  const initUser = () => {
    if (!ethWallet) {
      return <p>Please install MetaMask to use this ATM.</p>;
    }
    if (!account) {
      return <button onClick={connectAccount}>Connect MetaMask</button>;
    }
    if (balance === undefined) {
      getBalance();
    }
    return (
      <div>
        <p>Account: {account}</p>
        <p>Balance: {balance}</p>
        <input
          type="text"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
          placeholder="Enter amount"
        />
        <button onClick={deposit}>Deposit ETH</button>
        <button onClick={withdraw}>Withdraw ETH</button>
        <button onClick={burn}>Burn ETH</button>
      </div>
    );
  }

  useEffect(() => { getWallet(); }, []);

  return (
    <main className="container">
      <header><h1>My sweet ATM!</h1></header>
      {initUser()}
      <style jsx>{`
        .container {
          text-align: center;
        }
      `}
      </style>
    </main>
  );
}
```
## Setup

### Smart Contract
1. Install dependencies: `npm install`
2. Compile the contract: `npx hardhat compile`
3. Deploy the contract: `npx hardhat run scripts/deploy.js`

### Frontend
1. Navigate to the `frontend` directory: `cd frontend`
2. Install dependencies: `npm install`
3. Start the development server: `npm run dev`

## Usage
1. Ensure MetaMask is installed and connected to your preferred Ethereum test network.
2. Deploy the smart contract and note the contract address.
3. Update the `contractAddress` in `index.js` with your deployed contract address.
4. Start the frontend application.
5. Interact with the contract via the frontend.

## License
This project is licensed under the MIT License.
