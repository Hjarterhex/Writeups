# Fibonacci

## Challenge description

Here's a wall of text for ya! Not really text though, just random characters. Anyway, the flag is spread out at the positions of the fibonacci numbers.

Note: The fibonacci sequence should start with `[1, 1, ...]` and the indices are 0-based

## Solution

We are given a ~625kb text file filled with random characters. The task is to find the characters of the flag, which are located at the positions of the fibonacci sequence. The fibonacci sequence is the sequence of numbers where each number is the sum of the two previous numbers. In this case we are told that we should start with `[1, 1, ...]`, meaning that the third number is `1 + 1 = 2`, the fourth `1 + 2 = 3`, then `[..., 5, 8, 13]` etc. This can be done using a simple loop:

```python
with open('fibonacci.txt') as f:
    data = f.read()

a, b = 1, 1
flag = data[a]

while b < len(data):
    flag += data[b]
    a, b = b, a + b

print(flag)
```

Flag: `HHCTF{7h475_4_l07_0f_r4bb17s}`