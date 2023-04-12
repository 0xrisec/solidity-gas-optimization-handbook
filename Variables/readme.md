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

    function incrementCounter2(uint amount) public {
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

In the provided code snippet, we have an example where the state variable `counter` is being incremented within a loop in the `incrementCounter` function. To improve the gas efficiency of this function, we can modify it to use a local variable instead. This can be done by copying the value of `counter` to a local variable before entering the loop then incrementing the local variable within the loop and finally assigning the updated value back to the state variable after the loop completes. This modified code is provided in the `incrementCounter2` function.

|  | Function | Execution Gas Cost |
| --- | --- | --- |
| ❌ | `incrementCounter()` | 75638 |
| ✔️ | `incrementCounter2()` | 36987 |


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

    // Function to swap two uint variables in a single line using Solidity's swap feature
    function singleLineSwap(uint _a, uint _b) external pure returns(uint a,uint b){
        (_a, _b) = (_b, _a);
        return (_a, _b);
    }
}
```


|  | Function | Execution Gas Cost |
| --- | --- | --- |
| ❌ | `swap()` | 918 |
| ✔️ | `singleLineSwap()` | 893 |

## Optimizing Arithmetic Operations

**1. Use Better Increment**

There are several ways to perform increment and decrement operations in Solidity, including shorthand notation, addition operator, post-increment operator, and pre-increment operator. Here is an example of each technique:

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // ❌ This function increments the input value using the shorthand notation.
    function increment(uint num) external pure{
        num += 1;
    }

    // ❌ This function increments the input value using the addition operator.
    function increment2(uint num) external pure{
        num = num + 1;
    }

    // ❌ This function increments the input value using the post-increment operator.
    function increment3(uint num) external pure{
        num++;
    }

    // ✔️ This function increments the input value using the pre-increment operator.
    function increment4(uint num) external pure{
        ++num;
    }
}
```

|  | Function | Execution Gas Cost |
| --- | --- | --- |
| ❌ | `increment()` | 602 |
| ❌ | `increment2()` | 580 |
| ❌ | `increment3()` | 589 |
| ✔️ | `increment4()` | 562 |

**2. Do not use the shorthand notation**
```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.19;

contract Example {
    // This function multiplies the input value by a given value using the shorthand notation.
    function multiplyBy(uint num, uint n) external pure{
        num *= n;
    }

    // This function multiplies the input value by a given value using the multiplication operator.
    function multiply(uint num, uint n) external pure{
        num = num * n;
    }

    // This function divides the input value by a given value using the shorthand notation.
    function divideBy(uint num, uint n) external pure{
        num /= n;
    }

    // This function divides the input value by a given value using the divide operator.
    function divide(uint num, uint n) external pure{
        num = num / n;
    }
}
```

|  | Function | Execution Gas Cost |
| --- | --- | --- |
| ❌ | `multiplyBy()` | 839 |
| ✔️ | `multiply()` | 817 |
| ❌ | `divideBy()` | 768 |
| ✔️ | `divide()` | 746 |


Based on test results, we recommend the following techniques for optimizing gas consumption:

- Use pre-increment and pre-decrement operators (i.e., `++i` and `--i`) instead of post-increment and post-decrement operators (i.e., `i++` and `i--`) where possible. Pre-increment and pre-decrement operators are more efficient and cheaper, resulting in lower gas consumption. However, note that post-increment and post-decrement operators return the old value before incrementing or decrementing while pre-increment and pre-decrement operators return the new value.

- Use `i = i + n` instead of shorthand notation (i.e., `i += n`) for **addition**, **subtraction**, **multiplication**, and **division** operations. This technique results in lower gas consumption and is more efficient, especially for large values of `n`.
