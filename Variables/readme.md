# Gas Optimizations in Variables

## Avoid Explicitly Initializing Variables with Default Values

One way to optimize gas usage is to avoid explicitly initializing variables with their default values. In Solidity, different types of variables are automatically initialized with default values. For instance, a `uint` variable is initialized to `0`, a `bool` variable is initialized to `false`, and so on. Therefore, explicitly initializing a variable with its default value is redundant and consumes unnecessary gas.

**Demo Code:**

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    bool public flag;
}

contract Example2 {
   bool public flag = false;
}
```

|| Contract Name | Transaction Cost | Execution Cost |
| --- | --- | --- | --- |
| ❌ | Example | 94875  | 38487 |
| ✔️ | Example2 | 97498 | 40754 |


## Substituting State Variables with Local Variables

The recommendation is to substitute state variable reads and writes within loops with local variable reads and writes. This is because local variable operations are inexpensive while accessing state variables that are kept in the contract storage can be expensive.

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    uint public counter;
  
    function incrementCounter(uint amount) public {
        for (uint i = 0; i < amount; i++) {
            // Accesses the state variable 'counter' and increments its value.
            counter++; //❌ State variable reads and writes multiple times
        }
    }
}

contract Example2 {
    uint public counter;
  
    function incrementCounter(uint amount) public {
        uint _counter = counter; // Reads the state variable 'counter' once and stores it in a local variable.
        for (uint i = 0; i < amount; i++) {
            // Accesses the local variable and increments its value.
            _counter++; // Local variable reads and writes
        }
        // Assigns the updated value back to the state variable 'counter'.
        counter = _counter; //✔️ Writes to the state variable 'counter' once.
    }
}
```

<details>
<summary><b>Code Explanation</b></summary>

`Example` and `Example2`, with a function named `incrementCounter` that increments the counter state variable. In the `incrementCounter` function of the Example contract, the counter variable is being accessed and incremented within a loop, which results in multiple state variable reads and writes, leading to higher gas consumption during execution.

To make this function more efficient, we can modify it to use a local variable instead of accessing the state variable multiple times. The `incrementCounter` function of the `Example2` contract shows this modification, where the `counter` variable is read once and stored in a local variable `_counter` before the loop. Then, the loop is executed amount times, and the `_counter` variable is incremented within the loop. Finally, the updated value of `_counter` is assigned back to the state variable counter.

</details>


|| Contract Name | Function Name | Execution Gas Cost |
| --- | --- | --- | --- |
| ❌ | Example | incrementCounter() | 7903 |
| ✔️ | Example2 | incrementCounter() | 7040 |

## Single Line Swap

Solidity provides a relatively unique capability to swap variable values within a single statement, making the use of a `temporary variable/xor/arithmetic` methods unnecessary for swapping. Using the `swap` feature can result in lower execution gas costs compared to using a `temporary variable/xor/arithmetic` method.

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // Function to swap two uint variables using a temporary variable
    function swap(uint _a, uint _b) external pure returns(uint a,uint b){
        uint temp = _b;
        _b = _a;
        _a = temp;
        return (_a,_b);
    }
}

contract Example2 {
     // Function to swap two uint variables in a single line using Solidity's swap feature
    function swap(uint _a, uint _b) external pure returns(uint a,uint b){
        (_a, _b) = (_b, _a);
        return (_a, _b);
    }
}
```


|  | Contract | Function | Execution Gas Cost |
| --- | ---| --- | --- |
| ❌ | Example |`swap()` | 896 |
| ✔️ | Example |`swap()` | 893 |

## Optimizing Arithmetic Operations

**1. Use Better Increment**

There are several ways to perform increment and decrement operations in Solidity, including shorthand notation, addition operator, post-increment operator, and pre-increment operator. Here is an example of each technique:

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    uint public num = 1;

    // ❌ This function increments the input value using the shorthand notation.
    function increment() external{
        num += 1;
    }
}

contract Example1 {
    uint public num = 1;
    // ❌ This function increments the input value using the addition operator.
    function increment() external{
        num = num + 1;
    }
}

contract Example2 {
    uint public num = 1;

    // ❌ This function increments the input value using the post-increment operator.
    function increment() external{
        num++;
    }
}

contract Example3 {
    uint public num = 1;

    // ✔️ This function increments the input value using the pre-increment operator.
    function increment() external{
        ++num;
    }
}
```

|  | Contract | Operation | Function | Execution Gas Cost |
| --- |--- |---| --- | --- |
| ❌ | Example | `num += 1;`| `increment()` | 5359 |
| ❌ | Example1 | ` num = num + 1;` | `increment()` | 5346 |
| ❌ | Example2 |  `num++;` |`increment()` | 5301 |
| ✔️ | Example3| `++num` |`increment()` | 5295 |

**2. Do not use the shorthand notation**

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    uint public num = 1;

    // ❌ This function increments the input value using the shorthand notation.
    function add(uint n) external{
        num += n;
    }
}

contract Example2 {
    uint public num = 1;
    // ✔️ This function increments the input value using the addition operator.
    function add(uint n) external{
        num = num + n;
    }
}
```

|  | Contract | Function | Execution Gas Cost |
| --- |---| --- | --- |
| ❌ | Example |`add()` | 5621 |
| ✔️ | Example2 | `add()` | 5608 |

Based on test results, we recommend the following techniques for optimizing gas consumption:

- Use pre-increment and pre-decrement operators (i.e., `++i` and `--i`) instead of post-increment and post-decrement operators (i.e., `i++` and `i--`) where possible. Pre-increment and pre-decrement operators are more efficient and cheaper, resulting in lower gas consumption. However, note that post-increment and post-decrement operators return the old value before incrementing or decrementing while pre-increment and pre-decrement operators return the new value.

- Use `i = i + n` instead of shorthand notation (i.e., `i += n`) for **addition**, **subtraction**, **multiplication**, and **division** operations. This technique results in lower gas consumption and is more efficient, especially for large values of `n`. (⭐ However, it's important to note that the reduction in gas consumption only applies when updating state variables, not local variables. If you are working with local variables, either notation can be used without a significant difference in gas consumption.)

## Variable packing

 Contracts use `32-byte` (`256-bit`) slots for storage and when we arrange multiple variables to fit within a single slot, it is known as variable packing. This technique can be compared to the game of Tetris where we aim to fit various shapes together to minimize wasted space and optimize our gas usage.

 If a variable is too large to fit within the `32-byte` limit of a slot, it will be stored in a new slot. Therefore, we must carefully choose which variables to pack together to reduce the number of required slots and ultimately save gas.

 For example, consider the following variables:


 ```
contract Example1 {
    uint128 a;
    uint256 b;
    uint128 c;
}
 ```

If we were to pack `b` with `a`, it would exceed the `32-byte` limit and thus be stored in a new slot. The same would happen with `c` and `b`.

However, if we reorder the variables like this:

```
contract Example2 {
    uint128 a;
    uint128 c;
    uint256 b;
}
```

We can pack `a` and `c` into the same slot since their combined size does not exceed the limit.

|       | Contract  | Gas Cost |
|-------|----------|--------------------|
| ❌     | Example1 | 67,066 gas         |
| ✔️     | Example2 | 67,054 gas         |

When choosing data types, it is essential to consider whether a smaller version of a data type can help pack the variable into a storage slot. If a `uint128` variable does not pack, it is more efficient to use a `uint256` instead.