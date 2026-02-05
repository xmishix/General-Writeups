# Pwnlab:init - Writeout

**Perform nmap on target**

To start, when setting up the VM it gives you an IP address.

Perform a _nmap_ with parameters: _\-A_ which allows the detection of OS, the version, script scanning, and traceroute; _\-T4_ indicates the time to send packets from 0-6, the lowest number is the slowest but better because it could evade some firewalls.

<img width="899" height="655" alt="image" src="https://github.com/user-attachments/assets/2ebfa81b-12cd-4e92-88cc-ff1814f06bd7" />

Notice: port 80 http is open, port 111 rcpbind is open, port 3306 mysql.

**Perform nikto**

Perform _nikto -h <http://IP>_ to analyze the vulnerabilities of web service. Notice a service based on PHP, a config.php site that contains possible id's and passwords, a login available for updating pictures.

<img width="906" height="590" alt="image" src="https://github.com/user-attachments/assets/606e1b1c-5ed6-40db-a45a-87c042f78452" />

**Go to the web**

Go to the IP address of the target. The address would be something like _<http://10.219.249.15/?page=login>._

<img width="662" height="342" alt="image" src="https://github.com/user-attachments/assets/74e658d7-bebd-4fc3-8c3c-5044a59968c3" />

**PHP Filter**

By looking at the address and analyzing the _nikto_, one can assume that the website is vulnerable to local file inclusion (LFI). Perform an analysis of the site using php://filter with _source=index, upload, login, config._ Previously I ran the command without decoding and it gave me base64 encoding, on the following I added the base64 -d to decode the encoded text.

<img width="871" height="470" alt="image" src="https://github.com/user-attachments/assets/da1f5873-bd05-4f1c-875a-0ff9b98c5e0d" />

<img width="873" height="611" alt="image" src="https://github.com/user-attachments/assets/1c29f8b2-612b-4587-899d-2aa660877511" />

<img width="858" height="528" alt="image" src="https://github.com/user-attachments/assets/4c5defca-0879-4306-bc5c-cdc784b5b883" />

<img width="923" height="130" alt="image" src="https://github.com/user-attachments/assets/2e266707-fae1-472c-bbe3-2568b6a223e7" />

_Resource=config_ gives credentials to mysql database with users.

**Connect to mysql**

Proceed to connect to the database using the credentials obtained from config. I had trouble with SSL so I decided to ignore it. After accessing mysql, send a query to select all users and decode the given passwords.

<img width="839" height="300" alt="image" src="https://github.com/user-attachments/assets/beb27bae-9e7d-44e7-a771-04ee2df5130f" />

<img width="509" height="285" alt="image" src="https://github.com/user-attachments/assets/21146aee-3c31-4473-b999-ea8495a2483d" />

<img width="469" height="251" alt="image" src="https://github.com/user-attachments/assets/06572457-7514-41bd-920d-db0c63a3190d" />


**Login with credentials**

Login to the web with one of the users found in the database, I selected kent. After login in, we try uploading a file but it only allows pictures, gif, etc.

<img width="638" height="340" alt="image" src="https://github.com/user-attachments/assets/7ffe31b1-a2d8-4171-89b0-e68736a8a480" />

<img width="644" height="367" alt="image" src="https://github.com/user-attachments/assets/9bd61462-9adf-4640-bba3-f0483419c172" />

I created a shell gif that contains a valid GIF header and a line of code (payload) that executes any command via POST parameter "cmd".

<img width="637" height="223" alt="image" src="https://github.com/user-attachments/assets/e0076806-9dde-4517-87f4-3b93fc45311d" />


Select the path of the gif on the website and trigger the payload in upload shell.gid by using the "lang" cookie flag in the index.php.

<img width="904" height="75" alt="image" src="https://github.com/user-attachments/assets/7d176aef-82e5-44ff-9438-25f74afd8c56" />

**Create Meterpreter and start server**

Create a _meterpreter_ and start server, as well as starting a server for the attacker's machine (your machine). We created the _meterpreter_ with _msfvenom,_ setting payload, host to the local machines, and port number. This will write out to a file called evil_dhn.

<img width="915" height="136" alt="image" src="https://github.com/user-attachments/assets/93ff0dcc-fca6-4732-9fe0-06ec815f768f" />

<img width="906" height="121" alt="image" src="https://github.com/user-attachments/assets/82651458-948e-4e7e-984c-90cb3e794557" />

**Use webshell to download and execute meterpreter**

Send evil_dhn file

<img width="898" height="281" alt="image" src="https://github.com/user-attachments/assets/7de413e2-3527-42c3-8c9a-261ae35a3a7b" />

**Gather information**

Start _metasploid_ and use the local host and port as set up previously in the msfvenom and you will obtain the reverse shell.

<img width="880" height="230" alt="image" src="https://github.com/user-attachments/assets/551928b9-78bb-4276-8a62-3196ca0365a3" />

Gather information inside the shell. First look at the system information and you'll realize you are inside the pwnlab (10.219.249.15). Then log in with one of the previous users and passwords, I used Kane.

Some of the following steps are pretty straightforward like looking at where you're at, listing elements, navigating through paths, etc.

<img width="545" height="344" alt="image" src="https://github.com/user-attachments/assets/7c46d46a-92a6-4f5a-bd28-bc70d1f64ea8" />

<img width="620" height="353" alt="image" src="https://github.com/user-attachments/assets/874aabc6-98c1-486f-a15a-df12d0ed298a" />

Notice that there is a _msgmike_ file with SUID, but the moment we execute it we get an error message with an unclear path, insinuating that the path might be incomplete. When trying to execute this file, it shows that it is trying to _cat_ a wrong path.

Then we overwrite a _cat_ fale with '_/bin/sh_' inside to use it for an attack and we change the permission to execute the _cat_ file. Then we export PATH to our current directory, to let us run the _cat_ malicious file.

Then we execute the _./msgmike_ and we become mike thanks to the SUID.

<img width="639" height="626" alt="image" src="https://github.com/user-attachments/assets/42f3b474-413c-47ed-b5a5-25ebdddc21e5" />

<img width="379" height="365" alt="image" src="https://github.com/user-attachments/assets/92b10753-7ee4-4e40-b7e3-1471568d7583" />


After navigating to _mike_, we do a file on the file we find in the directory, notice how that is another SUID (setuid) and if we run a string we'll find "_/bin/echo %s >> /root/messages.txt_". After writing anything we type '_;_' to interrupt the echo execution and perform a privilege escalation with '/bin/sh'.

<img width="876" height="253" alt="image" src="https://github.com/user-attachments/assets/ca4ec04e-5537-45ed-bcdf-7133198960af" />

Got root access!!! Now navigate through the directories until you are in the root directory.

<img width="811" height="422" alt="image" src="https://github.com/user-attachments/assets/d75a6b25-f092-44cf-86ac-6b58e1b3e409" />

CATCH THE FLAG!!!!

<img width="695" height="499" alt="image" src="https://github.com/user-attachments/assets/ce3306fa-a3df-46f2-9e6f-760f84c16afb" />

