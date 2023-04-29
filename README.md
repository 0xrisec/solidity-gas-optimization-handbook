# ⛽ Solidity Gas Optimization Handbook 📔

A collection of Solidity code snippets and techniques for **optimizing gas**. Feel free to improve with your optimization and techniques! I ❤️ pull requests :)

<p align="center">
  <img src="https://github.com/ROOTBABU/solidity-gas-optimization-handbook/blob/main/images/banner.png">
</p>

## Suggestions

### Do Accurate Gas Cost Comparison

`To accurately compare gas costs between functions, it is recommended that you do not include them in the same contract.`

Why? 👇

This is because the position of function selectors in the contract can affect gas costs. When a function is called, the EVM searches for the correct function selector using a binary search algorithm. If the correct function selector is located closer to the beginning of the list, the search process will require less gas compared to if the correct function selector is located closer to the end of the list.

For example, in the contract below, calling function `a()` may require slightly less gas than calling function `b()`.

```sol
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;


contract Example {
  // Execution cost: 337 gas 
  function b() public pure returns(uint){
     return 1;
  }

  // Execution cost: 315 gas 
  function a() public pure returns(uint){
     return 1;
  }
}
```
