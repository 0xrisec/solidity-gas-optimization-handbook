# Gas Optimizations in Data Type

## Storing nonzero values on zero values is more costly


It is more expensive to store nonzero values on `zero` values. Initializing `uint/int` to `1` can help to reduce the gas cost of `SSTORE` operations.

**Demo Code:**

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // ❌ : unnecessary initialization of variable with default value
    uint num = 0;

    function increment() public {
        ++num;
    }
}

contract Example2 {
    // ✔️: variable will be automatically initialized with default value
    uint num;

    function increment() public {
        ++num;
    }
}

contract Example3 {
    // ✔️: variable will be initialized with 1
    uint num = 1;

    function increment() public {
        ++num;
    }
}
```

In the provided example, `Example2` is a more efficient implementation than `Example` because the variable `num` is automatically initialized with the default value of `0`, while in `Example`, `num` is unnecessarily initialized with the value `0`. 

However, `Example3` is even more efficient than `Example2` because num is initialized with the value `1`, which reduces the gas cost of `SSTORE` operations.

The gas costs for deploying and executing each contract are provided in the table.

|  | Contract | Variable | Initialization | Deployment |  | Function Execution |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  ||   |  | Transaction Cost | Execution Cost | Transaction Cost | Execution Cost |
| ❌ | Example | num | 0 | 109757 | 52705 | 43437 | 22373 |
| ✔️ | Example2 | num | Default value (0) | 107499 | 50499 | 43437 | 22373 |
| ✔️ | Example3 | num | 1 | 129673 | 72605 | 26337 | 5273 |

**Recommendations:**

1. When initializing a variable with a default value, the first user will have to pay more gas costs than the subsequent ones. However, the deployment gas cost will be less for the subsequent contracts. Therefore, if you are the first user, it is recommended to use this approach if possible.
   
2. If you are not the first user, `Example3` is preferable. While the deployment gas cost is more expensive than `Example2`, the function execution gas cost is cheaper. Therefore, it is best suited if you want to minimize gas costs for the first user, which can lead to a better user experience.

<details>
<summary><b>Reason</b></summary>

The opcode responsible for writing to a storage slot in Ethereum is `SSTORE`. This opcode takes two arguments: the first argument is the storage `slot index`, and the second argument is the `value` to be written to the storage slot.

The gas cost for the `SSTORE` opcode depends on whether the storage slot is being set to `zero` or to a `non-zero` value. 

**Zero to non-zero:** If a storage slot is being set from `zero` to a `non-zero` value for the first time, the gas cost for the `SSTORE` opcode is `20,000 gas`.

**Non-zero to non-zero:**

- If a `non-zero` value is being set to a storage slot that already contains a `non-zero` value, and the new value is different from the current value, the gas cost for the `SSTORE` opcode is `5,000` gas.
- If the new value is the same as the current value, then no actual write operation is performed, and the gas cost is only `200 gas`.

**Non-zero to zero:** If a storage slot is being set from a `non-zero` value to `zero`, the gas cost for the `SSTORE` opcode is `5,000 gas`.

More detailed explanation is here: 👇

https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore

https://eips.ethereum.org/EIPS/eip-2200

</details>

