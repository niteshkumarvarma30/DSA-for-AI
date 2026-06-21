# Valid Subarrays by Sum Digits

## The Concept

The problem asks us to find the total number of valid subarrays within an array `nums`. A subarray is considered valid based on three core rules:

### 1. The Playground (Subarrays)

A subarray is a contiguous, unbroken slice of the array. You cannot skip numbers.

### 2. The Task (The Sum)

For every subarray, you must calculate the total sum of all the numbers inside it.

### 3. The Test (First and Last Digit)

You look at the sum as a word (string). If the very first digit and the very last digit of that sum both equal the target digit `x`, the subarray is valid!

---

## Example

If:

```python
nums = [3, 20, 10]
x = 3
```

The subarray:

```python
[3, 20, 10]
```

has a sum of:

```python
33
```

The first digit is `3`, and the last digit is `3`.

This perfectly matches `x = 3`, so this subarray is **Valid!**

---

## The Approach (Brute Force)

To solve this, we can systematically test every single possible subarray:

1. Use an outer loop `i` to pick the starting point of the subarray.
2. Use an inner loop `j` to stretch the subarray forward, adding each new number to a running `cur_sum`.
3. Convert the `cur_sum` to a string.
4. Check if the first character (`cur_sum_str[0]`) and the last character (`cur_sum_str[-1]`) are equal to the string version of `x`.

---

## The Code (Python)

```python
class Solution:
    def countValidSubarrays(self, nums: list[int], x: int) -> int:
        valid_count = 0
        n = len(nums)

        # Convert 'x' to a string just once at the top to save time
        x_str = str(x)

        # i represents the start of our subarray
        for i in range(n):
            cur_sum = 0

            # j stretches the subarray forward
            for j in range(i, n):
                cur_sum += nums[j]

                # Convert the current sum to a string to check its digits
                cur_sum_str = str(cur_sum)

                # Test if both the first and last digit match our target 'x'
                if cur_sum_str[0] == x_str and cur_sum_str[-1] == x_str:
                    valid_count += 1

        return valid_count
```

---

# Deep-Dive Complexity Analysis

## Time Complexity: $\mathcal{O}(N^2)$

### Nested Loops

The outer loop runs $N$ times, and the inner loop runs up to $N$ times for each outer loop iteration. This results in testing roughly:

$$
N \times \frac{N}{2}
$$

total subarrays.

### String Conversion

Inside the inner loop, we convert the sum to a string. The time taken for this is proportional to the number of digits in the sum (let's call it $D$).

Since the numbers are generally standard integer sizes, $D$ is very small (at most around 10–15 operations), making it a constant factor.

### Final Time

The dominant operation is the nested loops checking every subarray, making the overall time complexity strictly quadratic:

$$
\mathcal{O}(N^2)
$$

---

## Space Complexity: $\mathcal{O}(D)$ \approx \mathcal{O}(1)$

### Extra Memory

The algorithm only allocates memory for a few variables:

* `valid_count`
* `cur_sum`
* `x_str`

### String Allocation

The only dynamic memory used is `cur_sum_str`, which creates a new string in memory for the sum.

The size of this string is the number of digits in the sum ($D$).

### Final Space

Because a number's string representation takes up a tiny, strictly bounded amount of memory (e.g., a 64-bit integer is at most 20 characters long), we consider the auxiliary space to be constant:

$$
\mathcal{O}(1)
$$
