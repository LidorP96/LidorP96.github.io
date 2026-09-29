# Appointment

---

## **Background**

- Linux machine
- A MySQL/MariaDB database instance running on port 3306 that allows anonymous access with the root user.
- Enumerating the databases will reveal a custom database and a table containing the flag.

---

### **Q1: During our scan, which port do we find serving MySQL?**

1. Run nmap scan:
    
    ```
    nmap -T5 -p- --open -Pn -n <IP> -oN open-ports
    ```
    
    `Answer: 3306`
    

### **Q2: What community-developed MySQL version is the target running?**

1. Used banner grabbing technique:
    
    ```
    nc -vn target.com 3306
    ```
    
    `Answer: MariaDB`
    

### **Q3: When using the MySQL command line client, what switch do we need to use in order to specify a login username?**

1. Used man page:
    
    ```
    man mysql | grep -i -E "(user|username)"
    ```
    
    `Answer: -u`
    

### **Q4: Which username allows us to log into this MariaDB instance without providing a password?**

`Answer: root`

### **Q5: In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?**

`Answer: *`

### **Q6: In SQL, what symbol do we need to end each query with?**

`Answer: ;`

### **Q7: There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?**

1. Connect remotely and execute query:
    
    ```
    mysql -u root -h <IP> -P 3306 -p -e "SHOW databases;"
    ```
    
    `Answer: htb`
    

### **Q8: What is the command in MySQL to select a database to interact with?**

1. Used MySQL syntax:
    
    ```
    mysql -u root -h <IP> -P 3306 -p -e "USE htb;"
    ```
    
    `Answer: use`
    

### **Q9: What is the command in MySQL to show the different columns for a given table?**

`Answer: describe`

### **Q10: Which table has a column named "flag"?**

1. Run a remote query:
    
    ```
    mysql -u root -h <IP> htb -P 3306 -p -e "SELECT * FROM config;"
    ```
    
    `Answer: config`
    

### **Q11: Submit the flag located in the database**

`Answer: 7b4bec00d1a39e3dd4e021ec3d915da8`