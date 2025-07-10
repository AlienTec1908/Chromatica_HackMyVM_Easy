# HackMyVM: Chromatica - Easy

![Chromatica Icon](Chromatica.png)

*   **Difficulty:** Easy 🟢
*   **Author:** DarkSpirit
*   **Date:** 13. Juni 2025
*   **VM Link:** [https://hackmyvm.eu/machines/machine.php?vm=Chromatica](https://hackmyvm.eu/machines/machine.php?vm=Chromatica)
*   **Full Report (HTML):** [Link zum vollständigen Pentest-Bericht](https://alientec1908.github.io/Chromatica_HackMyVM_Easy/)

## Overview

This report documents the penetration testing process of the "Chromatica" virtual machine from HackMyVM, rated as an Easy difficulty challenge. The objective was to identify and exploit vulnerabilities to gain root access to the system. The machine involved web application exploitation (SQL Injection), a clever SSH login bypass, and privilege escalation via a misconfigured Sudo permission.

## Methodology

The approach followed a standard penetration testing methodology, starting with reconnaissance and enumeration to identify potential attack vectors, gaining an initial foothold, and finally escalating privileges to root.

### Reconnaissance & Web Enumeration

1.  **Host Discovery:** Identified the target IP (192.168.2.42) using `arp-scan` and configured the hostname `chromatica.hmv` in `/etc/hosts`.
2.  **Port Scanning (Nmap):** Discovered open ports 22 (SSH), 80 (HTTP), and 5353 (DNS/dnsmasq REFUSED). Identified OpenSSH 8.9p1 and Apache httpd 2.4.52.
3.  **Web Application Analysis (Nikto, Gobuster, Curl):** Confirmed Apache 2.4.52, found standard web directories (`assets`, `css`, `js`, `javascript`), `robots.txt`, and noted missing security headers and directory listing enabled on `/css/`.
4.  **Robots.txt Discovery:** Found a crucial entry in `robots.txt`: `user-agent: dev` and `Allow: /dev-portal/`.
5.  **Dev Portal Discovery:** Accessed `/dev-portal/` using `curl` and Burpsuite with the `User-Agent: dev` header. Found `search.php`.
6.  **SQL Injection Identification:** Analyzed the response from `search.php` and manually tested the `city` parameter for SQL Injection vulnerabilities.

### Initial Access

The primary initial access vector was a SQL Injection vulnerability found in the `/dev-portal/search.php` script.

1.  **SQL Injection Exploitation (sqlmap):** Used `sqlmap` with the correct URL, `User-Agent: dev`, and `--batch` to confirm SQL Injection (Time-based Blind and UNION Query) on the `city` parameter.
2.  **Database Enumeration:** Dumped the `Chromatica` database and identified the `users` table.
3.  **Credential Dumping:** Dumped the `users` table, obtaining 5 usernames (`admin`, `dev`, `user`, `dev-selim`, `intern`) and their MD5 password hashes.
4.  **Password Cracking:** Cracked the MD5 hashes using John the Ripper and CrackStation. Obtained cleartext passwords for `admin` (`adm!n`), `dev` (`flaghere`), `user` (`keeptrying`), and `intern` (`intern00`). The hash for `dev-selim` was not cracked.
5.  **SSH Login Attempt:** Attempted to log in via SSH as user `dev` with the password `flaghere`. Direct login was denied ("THIS ACCOUNT IS NOT A LOGIN ACCOUNT"), but the login banner displayed the User Flag.
6.  **SSH Banner Escape:** Exploited the SSH login banner by reducing the terminal size (`stty`) to force the output into a pager (`more`/`less`), then used the pager's escape function (`!/bin/bash`) to gain an interactive shell as user `dev`.

### Privilege Escalation

From the `dev` shell, enumeration led to a Sudo misconfiguration allowing privilege escalation to root.

1.  **Shell Stabilization & Enumeration:** Stabilized the `dev` shell using `stty` and began internal system enumeration.
2.  **Sudo Rights Check:** Executed `sudo -l` as `dev` (or implicitly `analyst`) and discovered the permission `(ALL : ALL) NOPASSWD: /usr/bin/nmap`. This meant `nmap` could be run as root without a password.
3.  **Nmap Sudo Exploit (Attempt 1):** Attempted the common `sudo nmap --interactive` exploit found on GTFOBins, but it failed as the Nmap version (7.80) was too old and did not support the `--interactive` option.
4.  **Nmap Sudo Exploit (Attempt 2):** Used an alternative `nmap` Sudo exploit involving the Nmap Scripting Engine (NSE). Created a small NSE script (`/tmp/shell.nse`) containing `os.execute("/bin/bash")` and executed it using `sudo /usr/bin/nmap --script=/tmp/shell.nse`.
5.  **Root Access:** Successfully obtained a root shell via the NSE script execution with Sudo privileges.

### Flags

Both the user.txt and root.txt flags were successfully retrieved after gaining the respective privileges.

*   User Flag: `brightctf{ALM0ST_TH3R3_34897ffdf69}` (Found in SSH login banner for user `dev`)
*   Root Flag: `1b56eefaab2c896e57c874a635b24b49` (Found in `/root/root.txt`)

---

[Link zum vollständigen Pentest-Bericht](https://alientec1908.github.io/Chromatica_HackMyVM_Easy/)
