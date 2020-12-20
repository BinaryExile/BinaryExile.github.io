---
title: Linux
category: Misc
order: 90
---

>**Screen**

Function | Command
------------- | -------------
Create Screen: | screen 
Create Window | Ctrl-a c
Kill a screen: | screen -X -S 12937 quit
Attach to screen: | screen -r  31844.pts-0.name
Reattach to Screen: | screen -r -d 30608
Switch Window: | Ctrl-a n
Split Screen: | <code> Ctrl-a S [horiz] or | [vertical] </code>
Switch Split Window:| Ctrl-a tab
Detatch from screen: | Ctrl-a d
Logging Screen Output | Ctrl-a H
Kill current screen: | Ctrl-a K
Scroll | Ctrl-a Esc [Enter to end]

>**Accounts and Privileges**

Function | Command
------------- | -------------
User Information | <code>id, whoami </code>
Add User (root) | <code>useradd -o -u 0 [login_name] </code>
Change Password | <code>passwd [login name] </code>
Generat SSH-Key | <code>ssh-keygen </code>


>**Drives, Files, Inventory, Search, and Counting**

Function | Command
------------- | -------------
Most Common word in file | <code>cat /home/ruk/dinner_receipts/Gandalf.txt | sort | uniq -c | sort -nr </code>
Locate a file | <code>find [path] -name [application] </code>
Locate a file | <code> updatedb && locate [application] </code>
Locate a file based on path | <code> which [application] </code>
Locate a file and execute command | <code> find [path] -name [application] -exec [command] {} \;</code>
search files | <code>grep [word] * </code>
Search files Regex | <code> grep -o '[expression]' [file] <code>
Line count | <code>wc -l </code>
Display File | <code>type [file] </code>
Delete a file | <code>Shred --remove </code>
Determine file type | <code>file [filename]</code>
Extract Sections from Line | <code>cut -d["delimiter"] -f[field you are interested in - eg 3] </code>
Unique Entries | <code> sort -u </code>
(Old) Find files only reable by root (flags for ctf) | <code> find / -user root -perm +400 ! -perm +044 </code>
Find files only reable by root (flags for ctf) | <code> find / -user root -perm /400 ! -perm /044 </code>
Find a drive | <code> sudo fdisk -l </code>
Clone a drive | <code> dd if=/dev/sda of=/dev/sdd bs=1M </code>
Create and Image | <code> sudo dd if=/dev/sdb of=~/USB_image </code>
Mount an Image | <code> mount ~/USB_image /mnt/USB_image -o loop </code>
Binary to hex text file | <code> xxd file.dll | cut -d" " -f 2-9 | sed 's/ //g' | tr -d '\n' > file.dll.txt </code>

>**Network and Firewalls**

Function | Command
------------- | -------------
Ports | <code>netstat -nap | less </code>
TCP/UDP Ports | <code>netstat -an[t/u]p | less </code>
IP address and alias from host name | <code>host [domain] </code>
List open files with network usage and port numbers | <code>lsof -Pi [ | grep PORTNUM]</code>
ssh config file | <code>/ect/ssh/sshd_config </code>
SimpleHTTPServer | <code>python -m SimpleHTTPServer 8000 </code>
Firewall rule drop | <code>iptables -A INPUT -s 10.10.76.1 -p tcp --dport 22 -j DROP </code>
Firewall rule drop negate | <code> iptables -A INPUT -p tcp --destination-port 13327 \! -d 127.0.0.1 -j DROP </code>
Firewall rule accept | <code>iptables -A INPUT 1 -s 10.10.76.1 -p tcp --dport 22 -j ACCEPT </code>
Firewall rule remove | <code>iptables -D INPUT 1 -s 10.10.76.1 -p tcp --dport 22 -j ACCEPT </code>
Firewall list rules | <code>iptables -n list </code>
Firewall Account Traffic | <code> iptables -I INPUT 1 -s 10.0.0.19 -j ACCEPT <br> iptables -I OUTPUT 1 -d 10.0.0.19 -j ACCEPT <br> iptables -Z  <br> After Done:  iptables -vn -L </code> 
  
**SSH and proxy**

Open terminal through SSH connection:
{% highlight bash %}
$ ssh -i [path to private key] -p [ssh port] [username]@server.com -D 9000
{% endhighlight %}
Note: Private key permissions should be set to 400.<br>
Note: The "-D 9000" option created a local SOCKS5 proxy on port 9000, which Firefox can be configured to use to access internal assets. 


>**Processes and Services**

Function | Command
------------- | -------------
Running Processes | <code>ps aux | less </code>
Background a process | <code>CTRL-Z and resume with bg </code>
Background Running | <code><command> &, jobs, fg job# </code>
View Background Processes | <code>jobs </code>
Move Process to foreground | <code>fg </code>
Restart process | <code>Killall -HUP sshd </code>
Start/Stop Service | <code>service sshd start/stop, /etc/init.d/sshd start </code>
Run command in background | <code>nohup [command] & </code>
Kill process | <code>killall [processname] </code>
Start/Stop Apache | <code> service apache2 start </code>
Apache Document Root | <code> /var/www/ </code>
Persistent Service Start | <code> update-rc.d ssh enable </code>
View persistent for All Services | <code> rcconf </code>

>**Remote commands and file upload**

Function | Command
--------- | -------
Mount Windows CIFS/SMB Share | <code> mount -t cifs -o username=user,password=pass,domain=blah //192.168.1.X/share-name /mnt/cifs </code>
Mount NFS share | <code> mount 192.168.1.1:/vol/share /mnt/nfs </code>
GUI for SMB shares | apt-get install smb4k -y
Copy file over network | <code>$ scp -r foo your_username@remotehost.edu:/some/remote/directory/bar <br> $ sftp tecmint@27.48.137.6 </code>
LF/CR (0a0d) | <code>Scripts developed on Windows will not execute unless the CR/LF are converted to LF (0a). FTP does this automatically. </code>


>**Scripting and Shell**

Function | Command
------------- | -------------
Start Bash Script | <code> #!/bin/bash </code>
Search History | <code>CTRL-R </code>
Clear Screen | <code>CTRL-L </code>
Start of Command | <code>Home </code>
End of Command | <code>End </code>
locate command | <code>locate [application], updatedb </code>
locate command in path | <code>which ls </code>
Untarring (.tar) | <code>tar xvf [file] </code>
Untarring (.tar.gz) | <code>tar xvfz [file] </code>
search for command | <code>apropos <topic> or man -k network </code>
Path | <code>echo $PATH, PATH=$PATH:[another_dir] </code>
Learn More | <code>man, info, whitis, apropos [topic] </code>
Shutdown/reboot | <code>shutdown -h now, shutdown -r now, reboot </code>
For loop | <code>  for [variable] in $([command]); do [command] [variable] | [additional commands] done </code>

>**Terminal from Shell**

Function | Command
------------- | -------------
Linux Terminal from Shell - Python | <code> python -c 'import pty; pty.spawn("/bin/sh")' </code>
Linux Terminal from Shell - echo| <code> echo os.system('/bin/bash')</code>
Linux Terminal from Shell - /bin/sh| <code> /bin/sh -i </code>
Linux Terminal from Shell - Perl 1| <code> perl —e 'exec "/bin/sh";' </code>
Linux Terminal from Shell - Perl 2| <code> perl: exec "/bin/sh"; </code>
Linux Terminal from Shell - Ruby| <code> ruby: exec "/bin/sh" </code>
Linux Terminal from Shell - lua| <code> lua: os.execute('/bin/sh') </code>
Linux Terminal from Shell - From within IRB | <code> exec "/bin/sh" </code>
Linux Terminal from Shell - From within VI | <code> :!bash </code>
Linux Terminal from Shell - From within VI | <code> :set shell=/bin/bash:shell </code>
Linux Terminal from Shell - From within nmap | <code> !sh </code>


>**Password Reset**

1. Reboot, holding down the left Shift key as Linux boots. 
2. When you see the "GNU GRUB" prompt, press "e" on the first entry to edit it. 
3. Arrow down to the line beginning with "linux", append the word "init=/bin/bash" to the end of the line, change "ro" to "rw" in the same line, and press Ctrl-x. The system will a bash shell as root. 
4. Run passwd and enter your desired password twice. 
5. Run sync to make sure your changes to /etc/shadow (via passwd) are flushed to disk. 
6. Reboot the machine. This procedure is valid on most Linux systems.

References: 
* https://help.ubuntu.com/community/LostPassword
* https://askubuntu.com/questions/91188/authentication-token-manipulation-error
* https://wiki.archlinux.org/index.php/Reset_root_password Look


