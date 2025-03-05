# **Write-Up: VishwaCTF - Cryptography**

## **Aria of the Lost Code**  
### **Solution:**  
Used the **Hymnos Alphabet** table to decode the message.

### Flag: 
```
VishwaCTF{H4v3_y0u_7ri3d_Ar_70n3l1c0}
```
**Explanation:**  
The challenge involved decrypting a message encoded using the Hymnos Alphabet, a fictional language system.

---

## **Chaos**  
### **Solution:**  
A Python script was used to decrypt the given encrypted message.  
The encryption method involved **Base85 encoding** and an **XOR operation with an index-based key**.

### **Python Script:**
```python
import base64

def xor_decrypt(encrypted_str):
    encrypted_str = encrypted_str.strip()
    try:
        transformed = base64.b85decode(encrypted_str)  # Decode from Base85
    except Exception as e:
        print(f"Error decoding Base85: {e}")
        return None
    original = bytearray()
    for i in range(len(transformed)):
        original_byte = transformed[i] ^ (i % 256)  # XOR with index
        original.append(original_byte)
    return original

encrypted_messages = []
with open(r"output.txt", 'r') as f:
    content = f.read().strip()
    print("File content:", content)  
    encrypted_messages = content.split('\n\n')

for msg in encrypted_messages:
    decrypted = xor_decrypt(msg)
    if decrypted:
        print("Decrypted message:", decrypted.decode())

```

### Flag:  
```
VishwaCTF{CrYpt0_cRyPT0_1g_It_1s_sOm3_7hiNg_t0_D0}
```
**Explanation:**  
- The message was first encoded using Base85.
- Then, each byte was XOR-ed with its index value.
- The decryption script reversed the process to retrieve the original message.

---

## **Forgotten Cipher**  
### **Solution:**  
A Python script was used to decrypt a **VIC Cipher**, which involved **bit rotations (left and right)** and **XOR operations with a generated key**.

### **Python Script:**
```python
def rotate_right(val, r_bits, max_bits=8):
    return ((val >> r_bits) | (val << (max_bits - r_bits))) & ((1 << max_bits) - 1)

def rotate_left(val, r_bits, max_bits=8):
    return ((val << r_bits) | (val >> (max_bits - r_bits))) & ((1 << max_bits) - 1)

def decrypt_vic_cipher(ciphertext_hex, initial_key):
    ciphertext = bytes.fromhex(ciphertext_hex)
    plaintext = bytearray(len(ciphertext))
    key = initial_key
    for i, c in enumerate(ciphertext):
        key = (key * 3 + i) % 256  # Generate a key dynamically
        if i % 2 == 0:
            temp = rotate_right(c, 2)  # Rotate right for even indices
        else:
            temp = rotate_left(c, 2)  # Rotate left for odd indices
        plaintext[i] = temp ^ key  # XOR with generated key
    return plaintext.decode('utf-8', errors='replace')

ciphertext_hex = "0d4ac648a2f0bee7bccf0231c35e13ba7bc93a2d8f7d9498885e3f4998"
result = decrypt_vic_cipher(ciphertext_hex, 7)
print("Decrypted Flag:", result)
```

### Flag: 
```
VishwaCTF{VIC_Decoded_113510}
```
**Explanation:**  
- The ciphertext was given in **hex format**.
- The script performed **bitwise rotations** (right for even indices, left for odd indices).
- A dynamically generated key was used for **XOR decryption**.
- Finally, the result was converted back to a readable string.
