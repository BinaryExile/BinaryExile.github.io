 ---
title: Port-Service Scanning
category: Scanning
order: 5
---

>**Windows and AD**

SMB Signing | <code> nmap --script smb-security-mode.nse -p445 192.168.1.1 </code>
SMB2 Signing | <code> nmap --script smb2-secuirty-mode -p445 192.168.1.1 </code>


>**Scanning a Large Network**

Network Sweeping | <code> sudo masscan 17.0.0.0/8 -p0-1023 --rate 20000</code>
Network Sweeping | <code> sudo masscan 17.0.0.0/8 -oG apple-masscan.gnmap -p 7,9,13,21-23,25-26,37,53,79-81,88,106,110-111,113,119,135,139,143-144,179,199,389,427,443-445,465,513-515,543-544,548,554,587,631,646,873,990,993,995,1025-1029,1110,1433,1720,1723,1755,1900,2000-2001,2049,2121,2717,3000,3128,3306,3389,3986,4899,5000,5009,5051,5060,5101,5190,5357,5432,5631,5666,5800,5900,6000-6001,6646,7070,8000,8008-8009,8080-8081,8443,8888,9100,9999-10000,32768,49152-49157 --rate 100000 </code>
Filter List | <code> egrep '^Host: ' apple-masscan.gnmap | cut -d" " -f2 | sort | uniq > apple-alive </code>
Port/Version Scan | <code> nmap -PN -n -A -iL apple-alive -oA apple-nmap-advanced-scan </code>

> **NMAP**

Update Scripts | <code> nmap --script-updatedb </code>
example command |  <code>nohup ./nmap -n --randomize-hosts --reason -A -sT -vv -oA [OUTPUTFILE] -p 1-1024 10.10.10.1-255 > output.txt & </code>
example command 2 | <code>nohup ./nmap -n --randomize-hosts --reason -A -sT -vv -oA [OUTPUTFILE] > -p 1-1024 -iL [HOSTLIST]  > output.txt & </code>
example command 3 | <code> nmap -A  -p- --script nmap-vulners,vulscan,vuln,default --script-args [vulscandb=scipvuldb.csv,exploitdb.csv,securitytracker.csv] --open -sV [target IP] -oA TCP_10.11.1.x </code>
Example UDP | <code> nmap -A -sV -sU --script=default,vuln --open -oA udp_10.11.1.x 10.11.1.x </code>
Ping |  <code>-Pn (no ping), -sP (ping sweep) </code>
Scans | <code> -sS (syn)<br> -sT (TCP)<br> -sF (FIN)<br> -sX (FIN,PUSH, URG)<br> -sM (FIN, ACK)<br> -sU (UDP) </code>
Check for firewall |  <code>--badsum <br> //If you recieve a rest or ICMP unreachable, it is likely a firewall) </code>
Numbers instead of machine |  <code>-n </code>
Choose ports | <code>-p [START]-[END]</code>
timing | <code>-T (0-5)</code>
Store output | <code> -oN (human readable)<br> -oG (grepable)<br> -oX (XML)<br> -oA (all formats)</code>
Fingerprint |  <code>-O</code>
Version scaning | <code>-sV </code>
Fingerprint, version, script, and traceeroute | <code>-A </code>
Reason for result | <code>--reason </code>
vulnerability Scan | <code> --script vuln </code>

> **NSE**

Example Command | <code>./NMAP -n -sV --script= [all, category, dir, script...] [target] -p [ports] </code>
All scripts  | <code>./NMAP -sC [target] -p [ports] </code>
Script details | <code>--script-trace </code>
Script help |  <code>--script-help</code>
Script arguments | <code>--script-args[arguments]</code>
Script database | <code>/opt/NMAP/share/NMAP/scripts/script.db </code>
Script files | <code>/opt/NMAP/share/NMAP/scripts/[name].nse </code>

> **Service Specific Scans**

* [FTP, HTTP,MSSQL, MYSQL, NTP, ORACLE, RDP,SMB, SMTP, SNMP, VNC](https://github.com/BinaryExile/pentest/tree/master/enumeration)

> **MASSCAN**

Example Command | <code> nohup masscan -p [PORTS] --rate 500 -vv --includefile [HOSTLIST] --output-format list --output-filename [OUTPUTFILE] > output.txt & </code>

>**Netcat Port Scan**

{% highlight bash %}
#TCP
nc -vv -z -n -w 1 127.0.0.1 1-65535 2>&1 | grep " open"
#UDP
nc -u -vv -z -n -w 1 127.0.0.1 1-65535 2>&1 | grep " open"

{% endhighlight %}

> **Powershell Port Scan**

{% highlight bash %}
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("192.168.1.1", $_)) "Port $_ is open" } 2>$null 
{% endhighlight %}

> **SMB Scanning**
{% highlight bash %}
#NULL Sessions are mostly Windows 2000 and Windows XP SP1-2 and eariler
#NBTSCAN
nbtscan [IP Range]

#RPCclient
rpcclient -U "" [IP]
[empty password]
#usefule commands
> srvinfo
> enumdomusers
> getdompwinfo

#ENUM4LINUX
enum4linux -v [IP]

#NMAP
nmap -p 139,445 --script smb-enum-users [IP]
nmap -p 139,445 --script smb-os-discovery.nse [IP]
nmap -p 139,445 --script vulns --script-args=unsafe=1 [IP]

#SMBClient
smbclient --list=[IP]
{% endhighlight %}

> **SNMP:**

***note: 1, 2, 2c have no encryption***

Find SNMP Machines | <code>nmap -sU --open -p 161 192.168.31.200-254 --open</code>
Bruteforce Community Strings | <code> onesixyone -c community =i ips [MiB Value] </code>
Common Community Strings | [Common Strings](https://github.com/danielmiessler/SecLists/blob/master/Discovery/SNMP/common-snmp-community-strings.txt)
MiB Values | [MiB Values](http://www.oidview.com/mibs/311/md-311-1.html)
Metasploit | <code>use auxiliary/scanner/snmp/snmp_enum </code>



> **Usefule Resources**
* scanrand
* zmap

