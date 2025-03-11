# Stegano_ellipto_graphy
![Images](image.png)

When checking the "hakced" folder, I found three files: `key`, `output.png`, and `encrypt.py`. I assumed that the **png** file had been encrypted using `encrypt.py` and the `key` file.

I used the following script:

```py
import argparse
import hashlib
import os
from PIL import Image
import numpy as np

class ECC:
    def __init__(self, p, a, b, g, n):
        self.p = p
        self.a = a
        self.b = b
        self.g = g
        self.n = n

    def add(self, P, Q):
        if P == (0, 0):
            return Q
        if Q == (0, 0):
            return P
        if P[0] == Q[0] and P[1] != Q[1]:
            return (0, 0)

        if P != Q:
            lambd = (Q[1] - P[1]) * pow(Q[0] - P[0], -1, self.p) % self.p
        else:
            if 2 * P[1] % self.p == 0:
                raise ValueError("Point doubling failed: 2 * P[1] is not invertible.")
            lambd = (3 * P[0] ** 2 + self.a) * pow(2 * P[1], -1, self.p) % self.p

        x_r = (lambd ** 2 - P[0] - Q[0]) % self.p
        y_r = (lambd * (P[0] - x_r) - P[1]) % self.p
        return (x_r, y_r)

    def multiply(self, k, P):
        R = (0, 0)
        for i in bin(k)[2:]:
            R = self.add(R, R)
            if i == "1":
                R = self.add(R, P)
        return R

p = 0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f
a = 0
b = 7
g = (55066263022277343669578718895168534326250603453732580969419399252653644613,
     93807190528502704734438820432045625694355265803661227653698390517638372054)
n = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141

curve = ECC(p, a, b, g, n)

def generate_keys():
    private_key = int.from_bytes(os.urandom(32), "big") % n
    public_key = curve.multiply(private_key, g)
    return private_key, public_key

def encrypt_message(message, public_key):
    shared_key_point = curve.multiply(public_key[0], g)
    shared_key = shared_key_point[0]

    shared_key_bytes = b""
    while len(shared_key_bytes) < len(message):
        shared_key_bytes += hashlib.sha256((str(shared_key) + str(len(shared_key_bytes))).encode()).digest()

    encrypted_message = bytes([m ^ k for m, k in zip(message.encode("utf-8"), shared_key_bytes)])
    return encrypted_message

def decrypt_message(encrypted_message, private_key):
    public_key = curve.multiply(private_key, g)
    shared_key_point = curve.multiply(public_key[0], g)
    shared_key = shared_key_point[0]

    shared_key_bytes = b""
    while len(shared_key_bytes) < len(encrypted_message):
        shared_key_bytes += hashlib.sha256((str(shared_key) + str(len(shared_key_bytes))).encode()).digest()

    decrypted_message = bytes([m ^ k for m, k in zip(encrypted_message, shared_key_bytes)])
    return decrypted_message.decode("utf-8")

def embed_message(image_path, message, output_path, public_key):
    img = Image.open(image_path)
    img_array = np.array(img)

    encrypted_message = encrypt_message(message, public_key)

    binary_message = ''.join(format(byte, '08b') for byte in encrypted_message)

    message_length = len(encrypted_message)
    binary_length = format(message_length, '016b')
    full_binary_message = binary_length + binary_message

    if len(full_binary_message) > img_array.size:
        raise ValueError("Image is too small to embed the message.")

    flat_img = img_array.flatten()

    for i in range(len(full_binary_message)):
        current_pixel = int(flat_img[i])
        current_pixel &= ~1
        new_pixel = current_pixel | int(full_binary_message[i])
        flat_img[i] = np.uint8(new_pixel)

    img_array = flat_img.reshape(img_array.shape)
    img_with_message = Image.fromarray(img_array)
    img_with_message.save(output_path)

    print(f"Message embedded successfully in {output_path}")

def extract_message(image_path, private_key):
    img = Image.open(image_path)
    img_array = np.array(img)
    flat_img = img_array.flatten()

    binary_length = "".join(str(flat_img[i] & 1) for i in range(16))
    message_length = int(binary_length, 2)
    binary_message = "".join(str(flat_img[i] & 1) for i in range(16, 16 + message_length * 8))

    encrypted_message = bytes(int(binary_message[i:i + 8], 2) for i in range(0, len(binary_message), 8))
    decrypted_message = decrypt_message(encrypted_message, private_key)
    print(f"Extracted message: {decrypted_message}")

def parse_args():
    parser = argparse.ArgumentParser(description="Embed or extract messages in images using ECC.")
    parser.add_argument("command", choices=["embed", "extract"], help="Command to execute (embed or extract).")
    parser.add_argument("image_path", help="Path to the image file.")
    parser.add_argument("-m", "--message", help="Message to embed.")
    parser.add_argument("-o", "--output_path", help="Path to save the output image.")
    parser.add_argument("-k", "--private_key", type=int, help="Private key for extraction.")
    return parser.parse_args()

def main():
    args = parse_args()

    if args.command == "embed":
        private_key, public_key = generate_keys()
        embed_message(args.image_path, args.message, args.output_path, public_key)
        print(f"Private Key: {private_key}")

    elif args.command == "extract":
        extract_message(args.image_path, args.private_key)

if __name__ == "__main__":
    main()
```

Running it:
```powershell
python .\decrypt.py extract out.png -k 6558684506371667866903020874921700400259832463896378794735041574848246323110
Extracted message: pearl{babe_he's_not_home_U_know_the_drill}
```

Flag:
```
pearl{babe_he's_not_home_U_know_the_drill}
```
