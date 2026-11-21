# Linux Terminal (Bash) Command Cheat Sheet

> Commands work across many Linux distributions, but package managers and some options vary.
> Destructive commands (`rm`, `dd`, partition tools, recursive deletion) can permanently destroy data.

## 1) Help & shell basics
```bash
man ls
ls --help
command -v command_name
clear
exit
echo "Hello"
history
```

## 2) Location & filesystem
```bash
pwd
ls
ls -la
ls -lh
ls -R
cd /etc
cd ..
cd ~
cd -
mkdir newfolder
mkdir -p parent/child
rmdir emptyfolder
rm file.txt
rm -r folder
cp source.txt copy.txt
cp -r folder folder_copy
mv file.txt /tmp/
mv oldname.txt newname.txt
touch newfile.txt
```

## 3) Read & search files
```bash
cat file.txt
less file.txt
head file.txt
head -n 20 file.txt
tail file.txt
tail -n 20 file.txt
tail -f logfile.log
grep "word" file.txt
grep -in "word" file.txt
grep -Rin "word" folder/
find . -name "*.txt"
find /var/log -type f
```

## 4) System information
```bash
uname -a
hostname
whoami
id
date
uptime
cat /etc/os-release
lscpu
free -h
df -h
du -sh folder/
```

## 5) Users & groups (admin/root may be required)
```bash
who
w
id username
getent passwd
getent group
sudo adduser newuser
sudo useradd -m newuser
sudo passwd newuser
sudo userdel newuser
sudo userdel -r newuser
sudo groupadd newgroup
sudo usermod -aG newgroup username
groups username
```

## 6) Processes
```bash
ps
ps aux
top
htop
pgrep processname
pidof processname
kill PID
kill -TERM PID
kill -KILL PID
nice -n 10 command
```
> Prefer graceful termination (`kill`/SIGTERM) before SIGKILL.

## 7) Services (systemd-based distros)
```bash
systemctl status
systemctl list-units --type=service
systemctl status ssh
sudo systemctl start service
sudo systemctl stop service
sudo systemctl restart service
sudo systemctl enable service
sudo systemctl disable service
```

## 8) Network
```bash
ip addr
ip link
ip route
ping -c 4 example.com
traceroute example.com
tracepath example.com
dig example.com
nslookup example.com
ss -tulpn
netstat -tulpn
arp -a
curl -I https://example.com
wget https://example.com/file
```

## 9) Permissions & ownership
```bash
ls -l
chmod 644 file.txt
chmod 755 script.sh
chmod +x script.sh
chown user:group file.txt
chown -R user:group folder/
umask
```
> Use recursive ownership/permission changes carefully.

## 10) Storage & mounts
```bash
lsblk
blkid
mount
df -h
du -sh *
sudo mount /dev/device /mnt
sudo umount /mnt
```
> Do not mount, format, or repartition disks without verifying the device.

## 11) Logs
```bash
journalctl
journalctl -b
journalctl -u ssh
journalctl -n 50
journalctl -f
```

## 12) Packages (choose your distro)
### Debian / Ubuntu / Kali
```bash
sudo apt update
sudo apt upgrade
apt search package
sudo apt install package
sudo apt remove package
sudo apt autoremove
```

### Fedora
```bash
sudo dnf check-update
sudo dnf upgrade
sudo dnf install package
sudo dnf remove package
```

### Arch
```bash
sudo pacman -Syu
pacman -Ss package
sudo pacman -S package
sudo pacman -R package
```

## 13) Archives
```bash
tar -czf archive.tar.gz folder/
tar -xzf archive.tar.gz
zip -r archive.zip folder/
unzip archive.zip
```

## 14) SSH & remote
```bash
ssh user@host
ssh -p 2222 user@host
scp file.txt user@host:/remote/path/
scp user@host:/remote/file.txt .
rsync -av folder/ user@host:/remote/folder/
```

## 15) Disk & hardware
```bash
lsusb
lspci
sudo fdisk -l
```
> Disk-partition tools are for inspection here; modifying partitions is risky.

## 16) Redirection & pipes
```bash
command > file.txt
command >> file.txt
command 2> errors.txt
command &> all_output.txt
command1 | command2
command1 && command2
command1 || command2
```

## 17) Environment & variables
```bash
env
printenv
echo "$PATH"
echo "$HOME"
export MYVAR="value"
unset MYVAR
```

## 18) Quick shortcuts
- `Ctrl+C`: interrupt current foreground command
- `Ctrl+L`: clear screen (similar to `clear`)
- `Tab`: autocomplete
- `Up/Down`: command history
