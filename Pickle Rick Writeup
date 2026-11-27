# 🥒 TryHackMe — Pickle Rick Writeup

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20%7C%20Linux-blue?style=for-the-badge)

> A beginner-friendly walkthrough covering web enumeration, directory discovery, Linux commands, file enumeration and sudo permissions.

---

## 📌 1. Lab Information

| Details | Information |
|:---|:---|
| Platform | TryHackMe |
| Room Name | Pickle Rick |
| Difficulty | Easy |
| Target IP | `10.49.153.150` |
| Operating System | Linux |
| Tools Used | Kali Linux, Gobuster, curl |
| Objective | Find three ingredients |

**Room Link:** [TryHackMe - Pickle Rick](https://tryhackme.com/room/picklerick)

---

## 🎯 2. Objective

The objective of this room is to help Rick find three ingredients required to make his potion.

During this lab, I practised:

- Web reconnaissance
- HTML source inspection
- Directory enumeration
- Identifying useful information in `robots.txt`
- Linux filesystem enumeration
- Reading files using Linux commands
- Understanding sudo permissions

---

## 🔍 3. Initial Web Enumeration

First, I checked whether the target web server was accessible.

### Command

```bash
curl http://10.49.153.150
```

### What does `curl` do?

`curl` is a command-line tool used to transfer data from or to a server using protocols such as HTTP and HTTPS.

It can retrieve webpage HTML directly in the terminal.

### Observation

While inspecting the HTML source, I found a comment containing a username.

```text
Username: R1ckRul3s
```

**Learning:** HTML comments can sometimes contain useful information that is not visible on the webpage.

---

## 📂 4. Directory Enumeration with Gobuster

Next, I used Gobuster to discover directories and files on the web server.

### Command

```bash
gobuster dir -u http://10.49.153.150 \
-w /usr/share/wordlists/dirb/common.txt
```

### Command Breakdown

| Option | Description |
|:---|:---|
| `gobuster` | Directory enumeration tool |
| `dir` | Directory scanning mode |
| `-u` | Target URL |
| `-w` | Wordlist location |

### Interesting Results

```text
/assets
/index.html
/robots.txt
```

### What is Gobuster?

Gobuster is a tool used to discover hidden directories, files, DNS subdomains and other resources.

**Learning:** Directory enumeration helps identify web resources that may not be linked directly from the homepage.

---

## 🤖 5. Checking robots.txt

I checked the `robots.txt` file.

### Command

```bash
curl http://10.49.153.150/robots.txt
```

### Output

```text
Wubbalubbadubdub
```

This string became a useful clue during the lab's login stage.

### What is robots.txt?

`robots.txt` is a file that provides instructions to web crawlers about which parts of a website they should or should not crawl.

**Important:** It is not a security mechanism. Its contents are publicly accessible.

---

## 🔐 6. Accessing the Login Portal

The login page was located at:

```text
http://10.49.153.150/login.php
```

I used the username discovered in the HTML source and the clue found in `robots.txt` during the lab's login stage.

After successful authentication, I accessed the command panel.

---

## 🖥️ 7. Exploring the Web Command Panel

After logging in, I checked the current directory.

### Command

```bash
ls -la
```

### Command Explanation

| Option | Description |
|:---|:---|
| `ls` | Lists files and directories |
| `-l` | Displays detailed information |
| `-a` | Includes hidden files |

### Interesting Files

```text
Sup3rS3cretPickl3Ingred.txt
clue.txt
portal.php
login.php
robots.txt
```

These files provided useful starting points for further enumeration.

---

## 📖 8. Understanding Linux File-Reading Commands

When I attempted to read the first ingredient file using `cat`, the command panel displayed:

```text
Command disabled to make it hard for future PICKLEEEE RICCCKKKK
```

This was a restriction implemented by the lab's web command panel.

It does not mean that Linux itself has disabled the `cat` command.

### A. cat

```bash
cat filename.txt
```

**Purpose:** Displays the entire contents of a file in the terminal.

### B. less

```bash
less filename.txt
```

**Purpose:** Opens a file in a scrollable viewer.

Useful keyboard shortcuts:

| Key | Function |
|:---|:---|
| `Space` | Next page |
| `b` | Previous page |
| `↑` / `↓` | Move line by line |
| `/word` | Search for a word |
| `q` | Exit |

### C. head

```bash
head -n 5 filename.txt
```

**Purpose:** Displays the first five lines of a file.

### D. nl

```bash
nl -ba filename.txt
```

**Purpose:** Displays file contents with line numbers.

**Learning:** Linux provides multiple ways to inspect files. Understanding these commands is useful when working with different environments.

---

## 🔎 9. Searching the Filesystem

I needed to locate the other ingredients, so I started searching the filesystem.

### Initial Search

```bash
find / -type f 2>/dev/null
```

### Command Breakdown

| Part | Meaning |
|:---|:---|
| `find` | Searches for files and directories |
| `/` | Starts searching from the root directory |
| `-type f` | Searches only regular files |
| `2>/dev/null` | Hides error messages |

The command returned a large number of files.

To narrow down the results, I used a more specific search.

### Searching for Text Files

```bash
find /home /var/www /root -type f -iname "*.txt" 2>/dev/null
```

### Explanation

- `/home /var/www /root` — Directories to search.
- `-type f` — Only regular files.
- `-iname "*.txt"` — Finds text files, case-insensitively.
- `2>/dev/null` — Hides permission errors.

### Further Search

```bash
find /home /root /tmp /opt -type f 2>/dev/null
```

### Important Discovery

The search revealed:

```text
/home/rick/second ingredients
```

The filename contained a space, so I used quotation marks.

### Reading the File

```bash
less "/home/rick/second ingredients"
```

**Learning:** Always use quotes around filenames containing spaces. Otherwise, the shell may interpret the filename as multiple separate arguments.

---

## 🔑 10. Checking Sudo Permissions

Next, I checked the sudo permissions of the current user.

### Command

```bash
sudo -l
```

### What does sudo -l do?

It displays the commands that the current user is permitted to execute through `sudo`.

### Important Output

```text
User www-data may run the following commands:

(ALL) NOPASSWD: ALL
```

### Understanding the Output

| Term | Meaning |
|:---|:---|
| `sudo` | Executes commands with elevated privileges |
| `-l` | Lists permitted sudo commands |
| `NOPASSWD` | Does not require a password for the permitted rule |
| `ALL` | All commands are allowed under that rule |

**Learning:** Sudo permissions are important when understanding Linux privilege boundaries.

---

## 👑 11. Inspecting the Root Directory

The sudo permission output indicated that the current user had extensive elevated command permissions.

I inspected the root user's home directory.

### Command

```bash
sudo ls -la /root
```

### Command Breakdown

| Part | Meaning |
|:---|:---|
| `sudo` | Executes the command with elevated privileges |
| `ls` | Lists directory contents |
| `-la` | Displays all files with detailed information |
| `/root` | Root user's home directory |

### Important Discovery

The directory listing revealed:

```text
3rd.txt
```

This was the location of the final ingredient.

### Reading the Final File

The normal file-reading command was:

```bash
sudo cat /root/3rd.txt
```

However, the lab command panel also restricted `cat`.

An alternative was:

```bash
sudo less /root/3rd.txt
```

Another alternative:

```bash
sudo head -n 5 /root/3rd.txt
```

**Learning:** The `sudo` command allows permitted commands to run with elevated privileges. File ownership and permissions determine who can access protected files.

---

## 🧰 12. Linux Commands Cheat Sheet

| Command | Purpose |
|:---|:---|
| `curl URL` | Retrieves webpage content |
| `gobuster dir` | Discovers web directories and files |
| `ls` | Lists files and directories |
| `ls -la` | Lists all files with detailed information |
| `cat file.txt` | Displays file contents |
| `less file.txt` | Reads files page by page |
| `head -n 5 file.txt` | Displays the first five lines |
| `nl -ba file.txt` | Displays file contents with line numbers |
| `find / -type f` | Searches for regular files |
| `-iname "*.txt"` | Searches for text files |
| `2>/dev/null` | Hides error messages |
| `sudo -l` | Lists permitted sudo commands |
| `sudo command` | Executes a permitted command with elevated privileges |

---

## 🧠 13. Key Learnings

After completing this room, I learned:

1. How to inspect HTML source code.
2. How to use `curl` for basic web enumeration.
3. How to discover directories using Gobuster.
4. Why checking `robots.txt` can be useful.
5. How to list files using `ls -la`.
6. The difference between `cat`, `less`, `head` and `nl`.
7. How to search a Linux filesystem using `find`.
8. How to handle filenames containing spaces.
9. How to inspect sudo permissions using `sudo -l`.
10. Why Linux file permissions and ownership matter.

---

## 🏁 14. Conclusion

The Pickle Rick room was a useful beginner-level hands-on lab.

It helped me understand the basics of:

- Web enumeration
- Directory discovery
- Linux command-line operations
- Filesystem enumeration
- File permissions
- Sudo privileges

**My biggest takeaway:**

> Enumeration is one of the most important steps in cybersecurity. Understanding command output is more valuable than blindly running commands.

---

## 📚 References

- [TryHackMe — Pickle Rick](https://tryhackme.com/room/picklerick)
- [GNU Coreutils Documentation](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [Gobuster GitHub Repository](https://github.com/OJ/gobuster)

---

**Disclaimer:** This writeup documents learning in an authorized TryHackMe practice environment. All testing was performed against the assigned lab target.
