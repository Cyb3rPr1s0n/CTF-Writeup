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


