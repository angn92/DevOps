# Linux - Base commands

1. File & Directory Management

```bash
ls        # list files
cd        # change directory
pwd       # print working directory
cp        # copy files
mv        # move/rename
rm        # delete files
mkdir     # create directory
find      # search files
```

2. Viewing & Editing Files
```bash
cat       # print file
less      # scroll file
head      # first lines
tail      # last lines
tail -f   # follow logs live
nano      # simple editor
vim       # advanced editor
```

Example:
```bash
less config.yml 
```
Using: 
- arror up and down -> move
- space -> next page
- q -> quite
- /any_word -> search  

```bash
head -n 10 config.yml # show first 10 lines
```

```bash
tail -n 10 config.yml # show last 10 lines

tail -f app.log # show live logs in real time
```

```bash
vim config.yml
```
Press i -> enter INSERT mode
Press ESC -> exit INSERT mode
 :wq -> sqve and quit


3. Text Processing
Used in automation, pipelines, parsing logs:

```bash
grep      # search text
awk       # pattern processing
sed       # stream editing
cut       # extract columns
sort      # sort lines
uniq      # remove duplicates
```

Example:

```bash
grep "ERROR" app.log # Find line containing a pattern
grep -i "error" app.log     # case insensitive
grep -r "db_host" .         # search recursively looks inside folder and subfolders  . (dot) current directory
grep -v "INFO" app.log      # exclude matches
grep -n "ERROR" app.log     # show line numbers
```

```bash
sed 's/old/new/' file.txt # Replace or modify text automatically
```

4. Process & System Monitoring
```bash
ps        # running processes
top       # real-time processes
htop      # better top
kill      # stop process
uptime    # system load
free -h   # memory usage
df -h     # disk usage
du -sh    # folder size du -sh logs/
```

Example:
```bash
ps aux # it shows all running processes, CPU / memory usage, user, PID (process ID)
ps aux | grep docker
```

```bash
top
```
Shows:
What it shows:
- live CPU usage
- memory usage
- running processes
Key controls:
- q → quit
- k → kill process
- M → sort by memory
- P → sort by CPU

```bash
kill 1234 # stop process 1234
kill -15 1234   # graceful stop (default)
kill -9 1234    # force kill (strong)
```


5. Networking
```bash
ping        # check connectivity
curl        # make HTTP requests
wget        # download files
netstat     # network stats
ss          # modern netstat
ip a        # IP info
traceroute  # route tracking
```

6. Permissions & Users
```bash
chmod     # change permissions
chown     # change owner
whoami    # current user
sudo      # run as admin
```

Example:
```bash
chmod +x script.sh        # make executable
chmod go-w file.txt       # remove write from group & others
chmod u=rwx,g=rx,o=r file # set exact permissions
```
Each permission has a number:

r = 4
w = 2
x = 1

Add them:

7 = rwx
6 = rw-
5 = r-x
4 = r--

```bash
chown [options] owner[:group] file

chown chown user file.txt # change owner 
chown user:developers file.txt # change owner and group
chown :developers file.txt # change only group
```



7. Package Management (depends on distro)
Debian/Ubuntu:
```bash
apt update
apt install nginx
apt upgrade
```

RHEL/CentOS:
```bash
yum install nginx
dnf install nginx
```

8. DevOps Essentials (containers & infra)
Docker:
```bash
docker ps
docker images
docker run
docker build
docker logs
docker exec -it
```

9. Archives & Compression
```bash
tar -czf archive.tar.gz folder/
tar -xzf archive.tar.gz
zip
unzip
```

Example:
```bash
tar -czf archive.tar.gz folder/
```
What it means:
-c → create a new archive
-z → compress with gzip
-f → filename (the archive name comes next)

```bash
tar -xzf archive.tar.gz
```
What it means:
-x → extract
-z → decompress gzip
-f → file to extract

10. Shell & Automation
```bash
echo
export
env
history
alias
xargs
```

11. SSH
```bash
ssh user@server
scp file user@server:/path
rsync -avz folder user@server:/path
```