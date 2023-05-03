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

<details>
<summary><b>Reason</b></summary>

The opcode responsible for writing to a storage slot in Ethereum is `SSTORE`. This opcode takes two arguments: the first argument is the storage `slot index`, and the second argument is the `value` to be written to the storage slot.

The gas cost for the `SSTORE` opcode depends on whether the storage slot is being set to `zero` or to a `non-zero` value. 

**Zero to non-zero:** If a storage slot is being set from `zero` to a `non-zero` value for the first time, the gas cost for the `SSTORE` opcode is `20,000 gas`.

**Non-zero to non-zero:**

- If a `non-zero` value is being set to a storage slot that already contains a `non-zero` value, and the new value is different from the current value, the gas cost for the `SSTORE` opcode is `5,000` gas.
- If the new value is the same as the current value, then no actual write operation is performed, and the gas cost is only `200 gas`.

**Non-zero to zero:** If a storage slot is being set from a `non-zero` value to `zero`, the gas cost for the `SSTORE` opcode is `5,000 gas`.

more detailed explanation is here: 👇

https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore

https://eips.ethereum.org/EIPS/eip-2200

</details>

**Recommendations:**

1. When initializing a variable with a default value, the first user will have to pay more gas costs than the subsequent ones. However, the deployment gas cost will be less for the subsequent contracts. Therefore, if you are the first user, it is recommended to use this approach if possible.
   
2. If you are not the first user, `Example3` is preferable. While the deployment gas cost is more expensive than `Example2`, the function execution gas cost is cheaper. Therefore, it is best suited if you want to minimize gas costs for the first user, which can lead to a better user experience. However, it's worth noting that sometimes the default value may need to be something other than zero.

## uint256 vs uint8

There are several integer types available, ranging from `uint8` to `uint256` . While it may seem that using a smaller integer type, such as `uint8`, would result in lower gas consumption, this is not actually the case.

In fact, it is often better to use `uint256` instead of smaller integer types, even if the range of values required can be held in a smaller type. 

**Demo Code:**

```// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    uint8 num;

    function incrementCounter() external returns (uint8) {
        uint8 incrementBy = 50;
        uint8 localValue = num;
        for (uint8 i=0; i < incrementBy; i++) {
            localValue += 1;
        }
        num = localValue;
        return num;
    }
}

contract Example2 {
    uint256 num;

    function increment() external returns (uint) {
        uint incrementBy = 50;
        uint local = num;
        for (uint i=0; i < incrementBy; i++) {
            local += 1;
        }
        num = local;
        return num;
    }
}
```

|  | Contract | Variable  | Deployment |  | Function Execution |  |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  | Transaction Cost | Execution Cost | Transaction Cost | Execution Cost |
| ❌ | Example | `uint8` num | 153469  | 93341  | 63991  | 42927  |
| ✔️ | Example2 | `uint256` num | 145891 | 86335  | 62251 | 41187 |


<details>
<summary><b>Reason</b></summary>
Using `uint256` would be more gas-efficient than `uint8` because the `uint8` value would need to be padded with `24` zeros to fit into a `256-bit` word.

This is how the layout of state variables in storage is designed in `EVM`, with each variable allocated to a `256-bit` word regardless of its actual size. Therefore, if you have a state variable that is smaller than `256 bits`, it will still occupy a full `256-bit` word and incur additional gas costs due to padding.

You can find more detailed information about the layout of state variables in storage in the following link: [https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#layout-of-state-variables-in-storage].

</details>

**Recommendations:**

- When dealing with a single variable, it is more expensive to use `uint8` compared to `uint256`. Therefore, I suggest using `uint256` instead.

- When dealing with multiple variables of different types is through variable packing, although it may not be optimal to use `uint256` for all variables. <details> <summary><b>Example</b></summary>
  ```
    contract Example1 {
        uint public a;
        uint public b;
        uint public c;

        function update(uint _a, uint _b, uint _c) public {
            a = _a;
            b = _b;
            c = _c;
        }
    }

    contract Example2 {
        uint128 public a;
        uint128 public c;
        uint256 public b;

        function update(uint128 _a, uint256 _b, uint128 _c) public {
            a = _a;
            b = _b;
            c = _c;
        }
    }
  ```

    |  | Contract | Deployment |  | Function Execution |  |
    | --- | --- | --- | --- | --- | --- |
    |  | | Transaction Cost | Execution Cost | Transaction Cost | Execution Cost |
    | ❌ | Example1 | 157039 | 96347 | 88597   | 67113 |
    | ✔️ | Example2 | 214664 | 149796 | 66865  | 45381 |

    Based on the gas cost analysis, `Example2` has a higher deployment cost but a lower function execution cost, while `Example1` has a lower deployment cost but a higher function execution cost. Since deployment cost is a one-time payment, while function execution cost is paid by each user, `Example2` is more gas-cost-efficient overall. Therefore, `Example2` is the recommended contract to use. 

  </details>
  
- For variables with an unlimited range of values, `uint256` are recommended. Using a smaller data type than required can lead to errors and security vulnerabilities, such as integer overflow or truncation. Therefore, it's important to choose an appropriate data type based on the expected range of values to avoid potential issues.
  
- Using smaller data types like `uint8` or `uint16` can be more gas-efficient than `uint256` when you need to store multiple variables in a data structure such as an `array` or a `struct`.        <details><summary><b>Example</b></summary>
    ```
    contract Example {
        // Define a struct to store RGB values
        struct Pixel {
            uint8 red;
            uint8 green;
            uint8 blue;
        }

        // Define an array to store pixels
        Pixel[] public pixels;

        // Function to add a pixel to the array
        function addPixel(uint8 _red, uint8 _green, uint8 _blue) public {
            Pixel memory newPixel = Pixel(_red, _green, _blue);
            pixels.push(newPixel);
        }
    }


    contract Example2 {
        // Define a struct to store RGB values
        struct Pixel {
            uint red;
            uint green;
            uint blue;
        }

        // Define an array to store pixels
        Pixel[] public pixels;

        // Function to add a pixel to the array
        function addPixel(uint _red, uint _green, uint _blue) public {
            Pixel memory newPixel = Pixel(_red, _green, _blue);
            pixels.push(newPixel);
        }
    }
    ```

    |  | Contract | Variable  | Deployment |  | Function Execution |  |
    | --- | --- | --- | --- | --- | --- | --- |
    |  |  |  | Transaction Cost | Execution Cost | Transaction Cost | Execution Cost |
    | ✔️ | Example | `uint8` | 222530  | 157202  | 67407  | 45923  |
    | ❌ | Example2 | `uint256` | 186835 | 123971  | 110899 | 89415 |
    
    The following code defines two contracts, `Example` and `Example2`, which both store `RGB` values using a `struct` called `Pixel` and an array to store multiple pixels. However, `Example` uses `uint8` for the color values, while `Example2` uses `uint`. 

    Based on the gas cost analysis, `Example` has a higher deployment cost but a lower function execution cost, while `Example2` has a lower deployment cost but a higher function execution cost. Since deployment cost is a one-time payment, while function execution cost is paid by each user, `Example` is more gas cost-efficient overall. Therefore, `Example` is the recommended contract to use.
    </details> 