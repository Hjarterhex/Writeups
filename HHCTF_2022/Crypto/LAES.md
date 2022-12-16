# LAES

## Challenge description

AES is just too complex, so I created LAES - Less Advanced Encryption Standard. It reduces the complexity by just doing one round of encryption, so it doesn't have to worry about key scheduling. I encrypted a message for you, but unfortunately I haven't had the time to write the decryption algorithms, so you will have to to that yourself.

Anyway, here is the message and the key:  
Message (base64): 39l77mfrP3AnWdbKek1UaTNXfe6aJoBqVaZ+QVVaBfU=  
Key (hex): c4 23 6c b7 86 fe 1f 8e 0e 12 e2 13 09 32 ed 55

## Solution

We are given a python file which contains a partial implementation of AES encryption. AES splits the plaintext into 16-byte blocks, and puts them into a 4x4 matrix (in column-major order). Then the block is encrypted in four steps (which in real AES is repeated 10 - 14 times): 
1. SubBytes: using a 256-byte lookup table (a function to create the table, or substitution box, is provided)
2. ShiftRows: shift the rows of the matrix left as many steps as the row index
3. MixColumns: a matrix multiplication in the Galois field `GF(256)` (a function for Galois multiplication is provided)
4. Add key: XOR the block with the encryption key (for real AES a key-scheduling function is used, so it's a different key for each round)

As the description says we need to implement the decryption, which means creating the inverse operations for all the steps, and run those in the reverse order.

The add key step can be reused, since XOR is an invertible operation.

The inverse mix columns is a matrix multiplication with a [different matrix](https://en.wikipedia.org/wiki/Rijndael_MixColumns#InverseMixColumns), which means we can just adapt the provided method, replacing the numbers:

 ```python
     def inv_mix_columns(self):
        for c in range(0, 16, 4):
            temp = self.state[c:c + 4]
            self.state[c] = gmul(temp[0], 14) ^ gmul(temp[1], 11) ^ gmul(temp[2], 13) ^ gmul(temp[3], 9)
            self.state[c + 1] = gmul(temp[0], 9) ^ gmul(temp[1], 14) ^ gmul(temp[2], 11) ^ gmul(temp[3], 13)
            self.state[c + 2] = gmul(temp[0], 13) ^ gmul(temp[1], 9) ^ gmul(temp[2], 14) ^ gmul(temp[3], 11)
            self.state[c + 3] = gmul(temp[0], 11) ^ gmul(temp[1], 13) ^ gmul(temp[2], 9) ^ gmul(temp[3], 14)
```

The inverse shift rows step, can also be implemented with a slight modification of the original, just swapping a `+` for a `-` in the last line

```python
    def inv_shift_rows(self):
        for i in range(1, 4):
            temp = self.state[i:i + 13:4]
            for j in range(4):
                self.state[i + j * 4] = temp[(j - i) % 4]
```

For the inverse SubBytes step, we need to create an inverse S-box, by adding this class method (and calling it from the `__init__` method)

```python
    @classmethod
    def invert_sbox(cls):
        cls.inv_sbox = bytearray(256)
        for i in range(256):
            cls.inv_sbox[cls.sbox[i]] = i
```

The we can just copy the original method, changing it to use the inverted S-box instead

```python
    def inv_sub_bytes(self):
        self.state = self.state.translate(self.inv_sbox)
```

Finally, we put it all together by adding a `decrypt`-method, and creating a modified version of the `encrypt_block`-method

```python
    def decrypt(self, ciphertext, encoding = 'utf-8'):
        plaintext = b''
        for i in range(0, len(ciphertext), 16):
            self.decrypt_block(ciphertext[i:i+16])
            plaintext += bytes(self.state)
        
        return plaintext.decode(encoding)

    def decrypt_block(self, block):
        self.state = bytearray(block)
        self.add_key()
        self.inv_mix_columns()
        self.inv_shift_rows()
        self.inv_sub_bytes()
```

Then we can use our complete LAES class to decrypt the flag

```python
laes = LAES(bytes.fromhex('c4 23 6c b7 86 fe 1f 8e 0e 12 e2 13 09 32 ed 55'))
#print(base64.b64encode(laes.encrypt('REDACTED')))
print(laes.decrypt(base64.b64decode('39l77mfrP3AnWdbKek1UaTNXfe6aJoBqVaZ+QVVaBfU=')))
```

Flag: `HHCTF{R1jnD43l_15_4_g3N1u5!}`
