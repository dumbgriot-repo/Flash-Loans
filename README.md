# Flash-Loans
LearnWeb3 Tutorial for creating flash loans

Lesson Type: Quiz
Estimated Time: 2-4 hours
Current Score: 0%
Borrow a few million for free from Aave using Flash Loans
Image

Have you ever wanted to become a billionaire without having to collateralize anything? Well, that's what flash loans are for. 🤑🤑

In this level we will learn how to take a Flash Loan from Aave and utilize this new concept in DeFi which doesn't exist in the web2 world. There is no good analogy for this from the traditional finance world, since this is simply impossible outside blockchains.

Are you excited? Well, I surely am 🥳🥳

🤔 Which one of the following is the company behind flash loans?


Uniswap

Sushiswap

Aave
Traditional Banking Systems?
How do traditional banking systems work? If you want a loan you have to put forward a collateral against which you can take the loan. This is typically how lending/borrowing in DeFi also works.

However, you may need just a shit ton of money at times to execute some sort of attack that you cannot possibly provide collateral for, perhaps to execute a huge arbitrage trade or attack some contracts.

What are Flash Loans?
You might be thinking: Is it some kind of loan? Well, yes, it is. It's a special type of loan where a borrower can borrow an asset as long as they return the borrowed amount and some interest before the end of the transaction. Since the borrowed amount is returned back, with interest, in the same transaction, there is no possibility for anyone to run away with the borrowed money. If the loan is not repaid in the same transaction, the transaction fails overall and is reverted.

🤔 What happens if the borrower is not able to repay the borrowed money at the end in a flash loan transaction?


The borrower is penalized

The borrower is banned from taking a flash loan again

The transaction fails and is reverted as if the borrower never took a loan in the first place
This simple, but fascinating, detail is what allows you to borrow billions with no upfront capital or collateral, because you need to pay it back in the same transaction itself. However, you can go wild with that money in between borrowing it and paying it back.

🤔 What are flash loans?


It's a loan where a borrower can borrow assets without requiring a collateral

It's a loan where a borrower can borrow assets by providing a collateral

It is not a loan
Remember that all of this happens in one transaction 👀

🤔 How many transactions does it take to execute a flash loan?


2-10

1

10+
Applications of a Flash Loan
They help in arbitrage between assets, causing liquidations in DeFi lending protocols, often play part in DeFi hacks, and other use cases. You can essentially use your own creativity to create something new 😇

In this tutorial we will only focus on how a Simple Flash Loan works which includes being able to borrow one asset. There are alternatives where you can borrow multiple assets as well. To read about other kinds of flash loans, read the documentation from Aave

🤔 Which one of the following is not an application of a flash loan?


Arbitrage

Liquidations

All are valid examples
Let us try to go a little deep on one use case, which is arbitrage. What is arbitrage? Imagine there are two crypto exchanges - A and B. Now A is selling a token LW3 for a lower price than B. You can make profits if you buy LW3 from A in exchange for DAI and then sell it on B gaining more DAI than the amount you initially started with.

Trading off price differences across exchanges is called arbitrage. Arbitrageurs are a necessary evil that help keep prices consistent across exchanges.

How do Flash Loans work?
There are 4 basic steps to any flash loan. To execute a flash loan, you first need to write a smart contract that has a transaction that uses a flash loan. Assume the function is called createFlashLoan(). The following 4 steps happen when you call that function, in order:

Your contract calls a function on a Flash Loan provider, like Aave, indicating which asset you want and how much of it
The Flash Loan provider sends the assets to your contract, and calls back into your contract for a different function, executeOperation
executeOperation is all custom code you must have written - you go wild with the money here. At the end, you approve the Flash Loan provider to withdraw back the borrowed assets, along with a premium
The Flash Loan provider takes back the assets it gave you, along with the premium.
Image

If you look at this diagram, you can see how a flash loan helped the user make a profit in an arbitrage trade. Initially the user started a transaction by calling lets say a method createFlashLoan in your contract which is named as FlashLoan Contract. When the user calls this function, your contract calls the Pool Contract which exposes the liquidity management methods for a given pool of assets and has already been deployed by Aave. When the Pool Contract receives a request to create a flash loan, it calls the executeOperation method on your contract with the DAI in the amount user has requested. Note that the user didn't have to provide any collateral to get the DAI, he just had to call the transaction and that Pool Contract requires you to have the executeOperation method for it to send you the DAI

🤔 Which contract sends your contract the asset/assets you requested to borrow using the flash loan?


Pool Contract

Liquidity Contract

Asset Contract
Now in the executeOperation method after receiving the DAI, you can call the contract for Exchange A and buy some LW3 tokens from all the DAI that the Pool Contract sent you. After receiving the LW3 Tokens you can again swap them for DAI by calling the Exchange B contract.

🤔 Which method is required in your contract for the contract to be able to receive a flash loan?


`executeTask`

`execute`

`executeOperation`
By this time now your contract has made a profit, so it can allow the Pool Contract to withdraw the amount which it sent our contract along with some interest and return from the executeOperation method.

Once our contract returns from the executeOperation method, the Pool Contract has allowance to withdraw the DAI it originally sent along with the interest from our FlashLoan Contract, so it withdraws it.

All this happens in one transaction, if anything is not satisfied during the transaction like for example our contract fails in doing the arbitrage, remember everything will get reverted and it will be as if our contract never got the DAI in the first place. All you would have lost is the gas fees for executing all this.

🤔 What do you need to payback for borrowing the asset/assets using the flash loans ?


the assets/asset which you borrowed

premiums

the assets/asset which you borrowed and their respective premiums
User can now withdraw profits from the contract after the transaction is completed.

It has been suggested by Aave to withdraw the funds after a successful arbitrage and not keep them long in your contract because it can cause a griefing attack. An example has been provided here.

🤔 Why is it suggested to take out funds from your contract after a successful arbitrage as early as you can?


because of re entrancy

because of griefing attack

because of signature replay
Build
Lets build an example where you can experience how we can start a flash loan. Note we won't be actually doing an arbitrage here, because finding profitable arbitrage opportunities is the hardest part and not related to the code, but will essentially just learn how to execute a flash loan.

Lets get started 🚀

To setup a Hardhat project, Open up a terminal and execute these commands

npm init --yes
npm install --save-dev hardhat
If you are on Windows, please do this extra step and install these libraries as well :)

npm install --save-dev @nomicfoundation/hardhat-toolbox
In the same directory where you installed Hardhat run:

npx hardhat
Select Create a Javascript project
Press enter for the already specified Hardhat Project root
Press enter for the question on if you want to add a .gitignore
Press enter for `Do you want to install this sample project's dependencies with npm (@nomicfoundation/hardhat-toolbox)?
Now you have a hardhat project ready to go!

Install OpenZeppelin contracts, Aave contracts and dotenv in the same terminal

npm install @openzeppelin/contracts @aave/core-v3 dotenv
Now let's gets started with writing our smart contract, create your first smart contract inside the contracts directory and name it FlashLoanExample.sol

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "@aave/core-v3/contracts/flashloan/base/FlashLoanSimpleReceiverBase.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";


contract FlashLoanExample is FlashLoanSimpleReceiverBase {
  using SafeMath for uint;
  event Log(address asset, uint val);

  constructor(IPoolAddressesProvider provider)
    FlashLoanSimpleReceiverBase(provider)
  {}
  
  function createFlashLoan(address asset, uint amount) external {
      address receiver = address(this);
      bytes memory params = ""; // use this to pass arbitrary data to executeOperation
      uint16 referralCode = 0;

      POOL.flashLoanSimple(
       receiver,
       asset,
       amount,
       params,
       referralCode
      );
  }

   function executeOperation(
    address asset,
    uint256 amount,
    uint256 premium,
    address initiator,
    bytes calldata params
  ) external returns (bool){
    // do things like arbitrage here
    // abi.decode(params) to decode params
    
    uint amountOwing = amount.add(premium);
    IERC20(asset).approve(address(POOL), amountOwing);
    emit Log(asset, amountOwing);
    return true;
  }
}
Now let's try to decompose this contract and understand it a little better. When we declared the contract we did it like this contract FlashLoanExample is FlashLoanSimpleReceiverBase {, our contract is named FlashLoanExample and it is inheriting a contract named FlashLoanSimpleReceiverBase which is a contract from Aave which you use to setup your contract as the receiver for the flash loan.

Now after declaring the contract, if we look at the constructor, it takes in a provider of type IPoolAddressesProvider which is essentially the address of the Pool Contract we talked about in the example above wrapped around an interface of type IPoolAddressesProvider. This interface is also provided to us by Aave and can be found here. FlashLoanSimpleReceiverBase requires this provider in its constructor.

constructor(IPoolAddressesProvider provider)
    FlashLoanSimpleReceiverBase(provider)
  {}
The first function we implemented was createFlashLoan which takes in the asset and amount from the user for which he wants to start the flash loan. Now for the receiver address, you can specify the address of the FlashLoanExample Contract and we have no params, so let's just keep it as empty. For referralCode we kept it as 0 because this transaction was executed by user directly without any middle man. To read more about these parameters you can go here. After declaring these variables, you can call the flashLoanSimple method inside the instance of the Pool Contract which is initialized within the FlashLoanSimpleReceiverBase which our contract had inherited, you can look at the code here.

🤔 Which function do you use in your contract to create a flash loan to borrow only one asset?


flashLoanSimple

flashLoan

flashLoans
function createFlashLoan(address asset, uint amount) external {
      address receiver = address(this);
      bytes memory params = "";
      uint16 referralCode = 0;

      POOL.flashLoanSimple(
       receiver,
       asset,
       amount,
       params,
       referralCode
      );
  }
After making a flashLoanSimple call, Pool Contract will perform some checks and will send the asset in the amount that was requested to the FlashLoanExample Contract and will call the executeOperation method. Now inside this method you can do anything with this asset but in this contract we just give approval to the Pool Contract to withdraw the amount that we owe along with some premium. Then we emit a log and return from the function.

function executeOperation(
    address asset,
    uint256 amount,
    uint256 premium,
    address initiator,
    bytes calldata params
  ) external returns (bool){
    // do things like arbitrage, liquidation, etc
    // abi.decode(params) to decode params
    uint amountOwing = amount.add(premium);
    IERC20(asset).approve(address(POOL), amountOwing);
    emit Log(asset, amountOwing);
    return true;
  }
}
Now we will try to create a test to actually see this flash loan in action.

Now because Pool Contract is deployed on Polygon Mainnet, we need some way to interact with it in our tests.

We use a feature of Hardhat known as Mainnet Forking which can simulate having the same state as mainnet, but it will work as a local development network. That way you can interact with deployed protocols and test complex interactions locally.

Note this has been referenced from the official documentation of Hardhat

To configure this, open up your hardhat.config.js and replace its already existing content with the following lines of code

require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config({ path: ".env" });

const QUICKNODE_RPC_URL = process.env.QUICKNODE_RPC_URL;

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: "0.8.10",
  networks: {
    hardhat: {
      forking: {
        url: QUICKNODE_RPC_URL,
      },
    },
  },
};
You will see that we configured hardhat forking here.

Now lets add the env variable for QUICKNODE_RPC_URL.

Create a new file called .env and add the following lines of code to it.

QUICKNODE_RPC_URL="QUICKNODE-RPC-URL-FOR-POLYGON-MAINNET"
Replace QUICKNODE-RPC-URL-FOR-POLYGON-MAINNET with the URL of the node for Polygon Mainnet. To get this URL go to Quicknode and login. After that click on Create an Endpoint and select chain as Polygon and network as Mainnet. Click Continue and create the app in Discover mode to remain on the free tier. Copy the HTTP Provider link from your dashboard, and add it to the environment file.

After creating the .env file, you will need one more file before we can actually write the test.

Create a new file named config.js and add the following lines of code to it

// Mainnet DAI Address
const DAI = "0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063";
// Random user's address that happens to have a lot of DAI on Polygon Mainnet
const DAI_WHALE = "0xdfD74E3752c187c4BA899756238C76cbEEfa954B";

// Mainnet Pool contract address
const POOL_ADDRESS_PROVIDER = "0xa97684ead0e402dc232d5a977953df7ecbab3cdb";
module.exports = {
  DAI,
  DAI_WHALE,
  POOL_ADDRESS_PROVIDER,
};
If you look at this file, we have three variables - DAI, DAI_WHALE and POOL_ADDRESS_PROVIDER. DAI is the address of the DAI contract on polygon mainnet. DAI_WHALE is an address on polygon mainnet with lots of DAI and POOL_ADDRESS_PROVIDER is the address of the PoolAddressesProvider on polygon mainnet that our contract is expecting in the constructor. The address can be found here.

Since we are not actually executing any arbitrage, and therefore will not be able to pay the premium if we run the contract as-is, we use another Hardhat feature called impersonation that lets us send transactions on behalf of any address, even without their private key. However, of course, this only works on the local development network and not on real networks. Using impersonation, we will steal some DAI from the DAI_WHALE so we have enough DAI to pay back the loan with premium.

Awesome 🚀, we have everything setup now lets go ahead and write the test.

Inside your test folder create a new file deploy.js and add the following lines of code to it.

const { expect, assert } = require("chai");
const { BigNumber } = require("ethers");
const { ethers, waffle, artifacts } = require("hardhat");
const hre = require("hardhat");

const { DAI, DAI_WHALE, POOL_ADDRESS_PROVIDER } = require("../config");

describe("Deploy a Flash Loan", function () {
  it("Should take a flash loan and be able to return it", async function () {
    const flashLoanExample = await ethers.getContractFactory(
      "FlashLoanExample"
    );

    const _flashLoanExample = await flashLoanExample.deploy(
      // Address of the PoolAddressProvider: you can find it here: https://docs.aave.com/developers/deployed-contracts/v3-mainnet/polygon
      POOL_ADDRESS_PROVIDER
    );
    await _flashLoanExample.deployed();

    const token = await ethers.getContractAt("IERC20", DAI);
    const BALANCE_AMOUNT_DAI = ethers.utils.parseEther("2000");
    
    // Impersonate the DAI_WHALE account to be able to send transactions from that account
    await hre.network.provider.request({
      method: "hardhat_impersonateAccount",
      params: [DAI_WHALE],
    });
    const signer = await ethers.getSigner(DAI_WHALE);
    await token
      .connect(signer)
      .transfer(_flashLoanExample.address, BALANCE_AMOUNT_DAI); // Sends our contract 2000 DAI from the DAI_WHALE

    const tx = await _flashLoanExample.createFlashLoan(DAI, 1000); // Borrow 1000 DAI in a Flash Loan with no upfront collateral
    await tx.wait();
    const remainingBalance = await token.balanceOf(_flashLoanExample.address); // Check the balance of DAI in the Flash Loan contract afterwards
    expect(remainingBalance.lt(BALANCE_AMOUNT_DAI)).to.be.true; // We must have less than 2000 DAI now, since the premium was paid from our contract's balance
  });
});
Now let's try to understand whats happening in these lines of code.

First, using Hardhat's extended ethers version, we call the function getContractAt to get the instance of DAI deployed on Polygon Mainnet. Remember Hardhat will simulate Polygon Mainnet, so when you get the contract at the address of DAI which you had specified in the config.js, Hardhat will actually create an instance of the DAI contract which matches that of Polygon Mainnet.

After that, the lines given below will again try to impersonate/simulate the account on Polygon Mainnet with the address from DAI_WHALE. Now the fascinating point is that even though Hardhat doesn't have the private key of DAI_WHALE in the local testing environment, it will act as if we already know its private key and can sign transactions on the behalf of DAI_WHALE. It will also have the amount of DAI it has on the polygon mainnet.

await hre.network.provider.request({
      method: "hardhat_impersonateAccount",
      params: [DAI_WHALE],
    });
Now we create a signer for DAI_WHALE so that we can call the simlulated DAI contract with the address of DAI_WHALE and transfer some DAI to FlashLoanExample Contract. We need to do this so we can pay off the loan with premium, as we will otherwise not be able to pay the premium. In real world applications, the premium would be paid off the profits made from arbitrage or attacking a smart contract.

const signer = await ethers.getSigner(DAI_WHALE);
    await token
      .connect(signer)
      .transfer(_flashLoanExample.address, BALANCE_AMOUNT_DAI);
After this we start a flash loan and checking that the remaining balance of FlashLoanExampleContract is less than the amount it initially started with, the amount will be less because the contract had to pay a premium on the loaned amount.

const tx = await _flashLoanExample.createFlashLoan(DAI, 1000);
    await tx.wait();
    const remainingBalance = await token.balanceOf(_flashLoanExample.address);
    expect(remainingBalance.lt(BALANCE_AMOUNT_DAI)).to.be.true;
To run the test you can simply execute on your terminal:

npx hardhat test
If the tests pass, you have successfully executed a flash loan.

Hurray 🥂🥂🥂🥂🥂

This is huge 🚀🚀🚀
