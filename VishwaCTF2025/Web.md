# **Flames** 

### **Analysis and Exploitation**

- After attempting **XSS** without success, I explored other methods and discovered the `db.php` file, which suggested a potential **SQL Injection (SQLi)** vulnerability.
- I used SQLi payload:

```sql
' UNION SELECT 1,2,3,4--  
```

### Flag: 

```
VishwaCTF{SQL_1nj3ct10n_C4n_Qu3ry_Your_He4rt}
```

---

# **Scan-it-to-stay-safe** 

Retrieved the flag using a **webhook**:

### Flag: 

```
VishwaCTF{Y0u_7R4c30lI7_3000_4rK}
```

---

# **Forgot-h1-login** 

### **Analysis and Exploitation**

- Sent an **email array** to `ark.dev@hackerone.com` and included yourself email address.
- The flag was found in the `X-CTF-SECRETS` header of the response.

### Flag: 

```
VishwaCTF{y0u_4r3_7h3_h4ck3r_numb3r_0n3_2688658}
```
