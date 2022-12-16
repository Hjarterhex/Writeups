# Python!

## Challenge description

My friend sent me this large piece of weird-looking python code. To my surprise it actually works, but I wonder if there is something more to it.

## Solution

We are given a python file with an `exec()`-function that gets sent a huge `bytes`-list constructed from the values of another `bytearray`, and decoded as an ascii string. What the `exec()` function does is to interpret and execute the given string as python code. To get the actual actual string, we can just replace `exec` with `print`. This gives us a similar (but shorter) code, so it seems we need to do the same thing again. We make it into a variable instead, and loop through it, replacing `exec` with `eval`, which evaluates the python code and returns the result.

```python
data = bytes([a[85],a[3],a[85],a[54],a[80],a[49], ... ,a[35],a[55],a[55]]).decode("ascii")

data = data.strip().removeprefix('exec(').removesuffix(')')

while data.startswith('bytes'):
    data = eval(data).strip().removeprefix('exec(').removesuffix(')')

print(data)
```
This gives us the original code, where the flag is set to a variable, but never used.

```
"""
flag = "HHCTF{C0d3_08fU5c4t10n}"
print("I can't believe this code actually works!")
"""
```

Flag: `HHCTF{C0d3_08fU5c4t10n}`