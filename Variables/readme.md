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

## Always Initialize `i` Variable in For Loops

It is important to always initialize loop variables. Many developers mistakenly believe that removing the initialization of the loop variable will save gas, but this is not the case.

Consider the following code snippets:

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // ✔️ : Initialized for loop variable i to 0
    function sum(uint[] memory numbers) public pure returns (uint) {
            uint total = 0;
            for (uint i = 0; i < numbers.length; i++) {
                total += numbers[i];
            }
            return total;
    }

    // ❌ : Default-initialized for loop variable i 
    function sum2(uint[] memory numbers) public pure returns (uint) {
        uint total = 0;
        for (uint i; i < numbers.length; i++) {
            total += numbers[i];
        }
        return total;
    }

    // ❌ : Loop variable i declared outside loop and not initialized
    function sum3(uint[] memory numbers) public pure returns (uint) {
        uint total = 0;
        uint i; // variable i is declared but not initialized
        for (; i < numbers.length; i++) {
            total += numbers[i];
        }
        return total;
    }

    // ❌ : Default-initialized for loop variable i 
    function sum4(uint[] memory numbers) public pure returns (uint) {
        uint total = 0;
        uint i = 0;
        for (; i < numbers.length; i++) {
            total += numbers[i];
        }
        return total;
    }
}
```

|  | Function | Gas Cost |
| --- | --- | --- |
| ✔️ | `sum()` | 3047 |
| ❌ | `sum2()` | 3091 |
| ❌ | `sum3()` | 3069 |
| ❌ | `sum4()` | 3113 |

