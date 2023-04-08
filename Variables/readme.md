# Gas Optimizations in Variables

## Avoid Explicitly Initializing Variables with Default Values

One way to optimize gas usage is to avoid explicitly initializing variables with their default values. In Solidity, different types of variables are automatically initialized with default values. For instance, a `uint` variable is initialized to `0`, a `bool` variable is initialized to `false`, and so on. Therefore, explicitly initializing a variable with its default value is redundant and consumes unnecessary gas.


```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // ❌ : unnecessary initialization of variable with default value
    uint num1 = 0;
    
    // ✔️: variable will be automatically initialized with default value
    uint num2;
}
```

|  | Code                                           | Transaction Cost        | Execution Cost         |
| :-: | ---------------------------------------------- | ----------------------- | ---------------------- |
| ❌ | `uint num1 = 0;`<br>(unnecessary initialization)| 94337 gas               | 38093 gas              |
| ✔️ | `uint num2;`<br>(automatically initialized)     | 92079 gas  | 35887 gas |
