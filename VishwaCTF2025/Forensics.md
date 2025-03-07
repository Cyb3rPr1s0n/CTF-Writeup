# FORENSICS

---

## Leaky Stream

![ảnh](https://github.com/user-attachments/assets/f0e52312-785e-42b9-a988-c5d357551d33)

### Solution

* They give a chitty-chat.pcapng file

![ảnh](https://github.com/user-attachments/assets/172de55f-9f4e-413d-9454-2b4ecc14e020)

* Quick solution

`strings .\chitty-chat.pcapng | findstr /I "VishwaCTF{"`

`strings .\chitty-chat.pcapng | findstr /I "}"`

![ảnh](https://github.com/user-attachments/assets/8545891d-71a0-4ea5-956e-8ac40e1fe8f6)

### Flag

```
VishwaCTF{this_is_first_part_this_second_part}
```

## Persist

![ảnh](https://github.com/user-attachments/assets/bb7bc521-f7a4-40d7-b9d7-eedd39c3a838)

### Solution

* In this challenge, there are 2 registries: HKCU and HKLM/Software
* First because of scenario is something is on computer screen so I think there must be a malware with a persistence so I check the some common persistence hives but nothing there
* After all, I check the Recent Docs (Tool: registry explorer)

`HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`

![ảnh](https://github.com/user-attachments/assets/3347281b-879e-45b4-a6ea-358e10c6680a)

### Flag

```
VishwaCTF{b3l1ef_in_r3g_p0wer}
```

## Whispers

![ảnh](https://github.com/user-attachments/assets/c8619195-2a37-42ef-af3e-bb68e8452d49)

### Solution

* Firstly, check the all traffic

![ảnh](https://github.com/user-attachments/assets/17a261d0-03b2-4870-8ac9-78842842fb57)

* It's not hard to see that 192.168.31.129 sent 621 TCP packet
* Check all protocol

![ảnh](https://github.com/user-attachments/assets/875d0cce-071c-4fdf-b718-4d74945a0c8a)

* Clearly see that, FTP is keypoint in this challenge
* Filter FTP and FTP-Data, check TCP Stream

![ảnh](https://github.com/user-attachments/assets/a4e4cab8-60d1-44d4-8d63-f5a08dea543a)

![ảnh](https://github.com/user-attachments/assets/b6efb9b6-34ed-47b4-9a49-00023ac2471c)

* Maybe these sentences are hint from author

```
Lines of code in a hidden thread,
0nward they flow, where secrets are led.
Connecting the dots in patterns untold,
Across quiet channels, both subtle and bold.
Layer upon layer, a system unseen,
Paths intertwined, where few have been.
*@*lways it whispers, the code out of sight,
Through silent corridors, beyond the light.
Holding its secrets in digital veins,
Waiting for those who seek through the chains
You must look closely, where silence..remains.
```

* Author use FTP to transfer a zip file `gotyou.zip`

![ảnh](https://github.com/user-attachments/assets/60aeb623-03b1-40ca-aae5-41da1aabfdfb)

* Exctract that zip, but it require a password to extract
* After check everything I can, nothing works
* But when I take all the first Letter of above poetry

`L0CALP@THWY`

* Suprise!!! The zip file was extracted

![ảnh](https://github.com/user-attachments/assets/93a225a0-a864-47e2-9ca8-a1c36e78cfcc)

* There are 4 "jpg" but hehehe.jpg is the different one

![ảnh](https://github.com/user-attachments/assets/dccd3baf-d695-4102-a510-e1d32a67f305)

* It's an ELF file was compile from pyinstaller
* Use pyinstxtractor to decompile this elf file
* there is a hehehe.pyc
* Use `https://pylingual.io/` to read source code

```
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: hehehe.py
# Bytecode version: 3.12.0rc2 (3531)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import socket
import threading
import time
import base64
import random
import string

def generate_random_data():
    """Generate random alphanumeric buffer traffic."""
    length = random.randint(20, 50)
    return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

def start_server():
    """UDP server to receive data (runs in the background)."""
    ip = '127.0.0.1'
    port = 12345
    server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server.bind((ip, port))
    for _ in range(15):
        server.recvfrom(1024)
    server.close()

def send_flag():
    """Send randomized UDP packets along with the real flag."""
    time.sleep(1)
    ip = '127.0.0.1'
    port = 12345
    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    for _ in range(8):
        noise = generate_random_data()
        client.sendto(noise.encode(), (ip, port))
        time.sleep(random.uniform(0.3, 0.7))
    flag = 'VishwaCTF{h1dd3n_l0c4l_traff1c}'
    obfuscated_flag = base64.b64encode(flag.encode()).decode()
    client.sendto(obfuscated_flag.encode(), (ip, port))
    for _ in range(8):
        noise = generate_random_data()
        client.sendto(noise.encode(), (ip, port))
        time.sleep(random.uniform(0.3, 0.7))
    client.close()
if __name__ == '__main__':
    server_thread = threading.Thread(target=start_server, daemon=True)
    server_thread.start()
    send_flag()
```

* Flag is in source code

### FLAG

```
VishwaCTF{h1dd3n_l0c4l_traff1c}
```
