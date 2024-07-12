# FundMe Solidity Smart Contract

This repo contains the `FundMe` smart contract along with the `PriceConverter` library which were both written in Solidity. The `FundMe` contract allows users to send ETH to the contract, which then uses Chainlink oracles to get real-world ETH/USD prices for conversion and setting funding requirements.

## Contracts and Libraries

### FundMe.sol

The main contract that manages funding and withdrawals.

### PriceConverter.sol

A Solidity library that provides functions to convert ETH amounts to USD using Chainlink price feeds.

## Features and Interactions

### Sending ETH through a Function

The `fund` function in the `FundMe` contract allows users to send ETH to the contract. This function uses the `msg.value` property to determine the amount of ETH sent and checks if it meets the minimum USD equivalent using the `PriceConverter` library.

```solidity
function fund() public payable {
    require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
    addressToAmountFunded[msg.sender] += msg.value;
    funders.push(msg.sender);
}
```
- **msg.value:** The amount of ETH sent with the transaction.
- **require:** Ensures that the sent ETH is worth more than the minimum USD amount.

### Solidity Reverts

Reverts stop the execution and revert state changes if certain conditions are not met. They are used in `require` statements and custom error handling to ensure conditions are met before proceeding.

```solidity
require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
```
- If the condition is not met, the transaction reverts with an error message.

### Chainlink Oracles - Getting Real-World Prices

The `PriceConverter` library interacts with Chainlink oracles to fetch the current ETH/USD price. This data is used to convert ETH amounts to USD for setting funding requirements.

```solidity
function getPrice() internal view returns (uint256) {
    AggregatorV3Interface priceFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
    (, int256 answer, , , ) = priceFeed.latestRoundData();
    return uint256(answer * 10000000000);
}
```
- **AggregatorV3Interface:** An interface provided by Chainlink for interacting with price feeds.
- **latestRoundData:** Fetches the latest price data.

### Solidity Interfaces

Interfaces define the functions that a contract must implement without providing the function bodies. The `AggregatorV3Interface` is used to interact with the Chainlink price feed.

```solidity
import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";
```
- **AggregatorV3Interface:** An interface for Chainlink's price feeds.

### Solidity Math

Solidity performs arithmetic operations to convert ETH amounts to USD. This is done in the `getConversionRate` function of the `PriceConverter` library.

```solidity
uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
```
- **ethPrice:** The current price of ETH in USD.
- **ethAmount:** The amount of ETH to convert.
- **ethAmountInUsd:** The equivalent amount in USD.

### Message Sender Explained

The `msg.sender` property is used to get the address of the account that called the contract. This is used to track contributions and manage access control.

```solidity
addressToAmountFunded[msg.sender] += msg.value;
```
- **msg.sender:** The address of the sender of the transaction.

### Solidity Libraries

Libraries contain reusable functions that can be included in other contracts. The `PriceConverter` library is used to provide conversion functions.

```solidity
using PriceConverter for uint256;
```
- **using PriceConverter for uint256:** Adds the library functions to the `uint256` type.

### SafeMath Library

Although not explicitly used in this contract, the SafeMath library prevents integer overflow and underflow, ensuring safer arithmetic operations.

### For Loop

A `for` loop iterates over the `funders` array to reset the funding amounts when withdrawing funds.

```solidity
for (uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++) {
    address funder = funders[funderIndex];
    addressToAmountFunded[funder] = 0;
}
funders = new address ;
```
- **for loop:** Iterates over the `funders` array to reset their funding amounts.

### Resetting an Array

The `funders` array is reset by creating a new empty array, effectively clearing the previous entries.

```solidity
funders = new address ;
```
- **new address :** Creates a new, empty array of addresses.

### Sending ETH from a Contract

The contract demonstrates different methods of sending ETH: `transfer`, `send`, and `call`. In this contract, `call` is used for sending ETH, which is the recommended way due to its flexibility and error handling capabilities.

```solidity
(bool callSuccess,) = payable(msg.sender).call{value: address(this).balance}("");
require(callSuccess, "Call failed");
```
- **call:** Sends ETH and provides a way to handle the success or failure of the transfer.

### Smart Contract Constructor

The constructor is a special function that is executed once when the contract is deployed. It sets the owner of the contract.

```solidity
constructor() {
    i_owner = msg.sender;
}
```
- **constructor:** Initializes the contract and sets the `i_owner` to the address that deployed the contract.

### Function Modifiers

Modifiers are used to modify the behavior of functions. The `onlyOwner` modifier restricts access to certain functions to the contract owner only.

```solidity
modifier onlyOwner() {
    if (msg.sender != i_owner) revert NotOwner();
    _;
}
```
- **onlyOwner:** Ensures that only the owner can call the modified function.

### Gas Optimizations

1. **Constants and Immutability:**
   - Constants and immutable variables save gas by storing values directly in the bytecode.
   ```solidity
   uint256 public constant MINIMUM_USD = 5 * 10 ** 18;
   address public immutable i_owner;
   ```

2. **Custom Errors:**
   - Custom errors are cheaper than `require` statements.
   ```solidity
   error NotOwner();
   ```

3. **Mappings and Arrays:**
   - Efficient use of mappings and arrays to store data.
   ```solidity
   mapping(address => uint256) public addressToAmountFunded;
   address[] public funders;
   ```

4. **Library Functions:**
   - Reusable library functions reduce code duplication and save gas.
   ```solidity
   using PriceConverter for uint256;
   ```

5. **Efficient Arithmetic:**
   - Using `uint256` for arithmetic operations ensures gas efficiency.

### Implementing the Receive Fallback Function

Fallback and receive functions handle incoming ETH payments. These functions ensure the contract can accept ETH sent directly to it, improving usability and functionality.

```solidity
fallback() external payable {
    fund();
}

receive() external payable {
    fund();
}
```
- **fallback:** Called when the contract receives ETH without data.
- **receive:** Called when the contract receives ETH with empty data.

---

This repo provides an overview of how the `FundMe` and `PriceConverter` contracts work, which highlight the key Solidity features and best practices used in these contracts. 
