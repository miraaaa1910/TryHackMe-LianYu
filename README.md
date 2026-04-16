# TryHackMe-LianYu
Lab 6 Vulnerability Attack

Target IP: 10.49.164.148
### Reconaissance
Start by scanning the target IP to see which open ports are open. `nmap -sV -vv 1-.49.164.148`

<img width="948" height="535" alt="image" src="https://github.com/user-attachments/assets/350f87e7-6c9f-4b50-8f96-d145e29adef0" />

### Open Ports:
1. 21/tcp : vsftpd 3.0.2
2. 22/tcp : OpenSSH 6.7p1
3. 80/tcp : Apache httpd
4. 111/tcp : rpcbind(2-4)

### Web Directory
1. Visiting http://10.49.164.148 shows a basic page. We need to find hidden directory by using gobuster. Navigate to `/usr/share/wordlists/dirbuster` to locate the standard directory wordlists; we will use `directory-list-2.3-medium.txt` for our initial brute-force attack.

   <img width="938" height="134" alt="Screenshot 2026-04-16 185250" src="https://github.com/user-attachments/assets/fd703d7b-7ea1-412d-a17e-ca1af6c4d9d4" />

3. Run Gobuster `gobuster dir -u http://10.49.164.148 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

   <img width="933" height="289" alt="image" src="https://github.com/user-attachments/assets/537d71cb-0da8-41f5-a657-b29ff7428eff" />

4. You will find `/island`. visiting to http://10.49.164.148/island/ reveals a hidden message. However, the "Code Word" isn't immediately visible on the page, suggesting we need to inspect the page source and found "vigilante"

   <img width="905" height="791" alt="Screenshot 2026-04-16 193932" src="https://github.com/user-attachments/assets/bdd8049e-4bde-4d0d-aacf-765860e86f16" />

4. Run Gobuster again `gobuster dir -u http://10.49.164.148/island -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`. The scan has identified a new path `/2100`. This is the web directory.

   <img width="971" height="274" alt="image" src="https://github.com/user-attachments/assets/07b8f539-8266-495d-a9f9-6ba3a49c1644" />
   
   <img width="805" height="119" alt="image" src="https://github.com/user-attachments/assets/bfd89de7-be13-411c-bad5-d2b897428168" />

### File Name
1. Visiting http://10.49.164.148/island/2100/ . While the video on the page is unavailable, the "inspector" tool reveals a crucial piece of information hidden in a comment. The comment specifically mentions `ticket` which strongly suggests there is a file named something like `[filename].ticket` hidden in this directory.
   
   <img width="1919" height="815" alt="image" src="https://github.com/user-attachments/assets/cf4023d6-5324-4bcf-afa8-658043d6ed81" />

2. Run Gobuster `gobuster dir -u http://10.49.164.148/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x ticket
`. Gobuster successfully combined the word `green_arrow` from the wordlist with the `.ticket` extension provided earlier.

   <img width="1114" height="300" alt="image" src="https://github.com/user-attachments/assets/9e9a99a0-a1a2-48e9-9326-28b67661d562" />
   
   <img width="815" height="116" alt="image" src="https://github.com/user-attachments/assets/676a5a2a-d202-4ec6-afda-5aeac58cc152" />

### FTP Password

1. Visiting http://10.49.164.148/island/2100/green_arrow.ticket . This `RTy8yhBQdscX` string doesn't look like a standard plaintext password.
   
   <img width="1025" height="187" alt="image" src="https://github.com/user-attachments/assets/25c5bbcb-9906-429f-bd5b-f7c54e2dca7b" />

2. Based on the hint TryHackMe gave, we can decode the string in https://gchq.github.io/CyberChef/ by using base58. So, `!#th3h00d` is likely the password for FTP
   
   <img width="1211" height="638" alt="Screenshot 2026-04-16 170830" src="https://github.com/user-attachments/assets/697e1e1b-7706-4cce-97c5-58eb3b803cd9" />
   <img width="317" height="141" alt="image" src="https://github.com/user-attachments/assets/306e27f5-aec5-4ff0-80d5-fe6f1aa2a9d4" />
   <img width="812" height="120" alt="image" src="https://github.com/user-attachments/assets/c98ac31b-5cee-4000-92b4-ed0353c81a6c" />

### File Name with SSH Password

1. Once inside, I listed the contents of the current directory by using `ls`, revelaling three interesting image files which are `Leave_me_alone.png`, `Queen's_Gambit.png` and `aa.jpg`. I used `get` command for each file to downloads them from the remote target server to local Kali machine `~/Downloads`
   
   <img width="748" height="611" alt="image" src="https://github.com/user-attachments/assets/cefa9b4f-7044-4e7b-b2f3-7571e0bc5e43" />

2. When I try open the file one by one, I noticed that `Leave_me_along.png` cannot be open.

   <img width="735" height="754" alt="Screenshot 2026-04-16 172007" src="https://github.com/user-attachments/assets/190dc49a-e9fd-4fbe-90b7-ee1fdfd0dd0a" />
   
   <img width="1328" height="741" alt="image" src="https://github.com/user-attachments/assets/c7f77f3c-4d18-4158-98dc-47db86eba711" />
   
   <img width="678" height="726" alt="image" src="https://github.com/user-attachments/assets/7e2dd46f-b828-4fb6-b50a-11445cd8e1c3" />

3. I open the file to look for anomalies in its structure by running `hexeditor Leave_me_alone.png` and noticed that the file is corrupted. To fix it, i cross-referencing the official PNG File Header specification against the actual data of the file.
   
   <img width="331" height="60" alt="image" src="https://github.com/user-attachments/assets/931bf63c-421e-4e6b-b51e-c567886cb45a" />
   
   <img width="718" height="868" alt="Screenshot 2026-04-16 172156" src="https://github.com/user-attachments/assets/2da72479-880f-4547-936a-7c6288ab2f33" />
   
   <img width="1041" height="189" alt="Screenshot 2026-04-16 172339" src="https://github.com/user-attachments/assets/3d20ddee-a670-47e4-be94-57fd61301a60" />

   <img width="911" height="747" alt="Screenshot 2026-04-16 172403" src="https://github.com/user-attachments/assets/57867777-dc88-4446-a3c2-95c54c4ba025" />

4. Perform steganographic extraction by using `steghide extract -sF aa.jpg` and found a container for a hidden archive. The passphrase for this is `password` that we found earlier in hexeditor step. Navigate the ZIP fie in local directory and found 2 files which are `passwd.txt` and `shado`. So, the file name with SSH password is `shado`.
   
   <img width="1444" height="154" alt="Screenshot 2026-04-16 230756" src="https://github.com/user-attachments/assets/f3945278-2c9b-4156-b910-e430600362f2" />

   <img width="622" height="256" alt="image" src="https://github.com/user-attachments/assets/f6f84454-d49e-424c-ac98-1bad88966390" />

   <img width="646" height="200" alt="Screenshot 2026-04-16 172805" src="https://github.com/user-attachments/assets/bd149f30-b58c-426e-96ca-91c8cb6a2d4a" />

   <img width="810" height="126" alt="image" src="https://github.com/user-attachments/assets/44eb13cd-4243-40f1-ac63-685e7d58ebb2" />

### user.txt

1. Based on the Arrow theme and our previous discoveries that the password from the ZIP file we just cracked, I pivoted from the FTP user vigilante to the system user `slade` by using `ssh slade@10.49.164.148`. Inside slade's home directory, I found a file named `user.txt`. To capture the flag use `cat user.txt`.

   <img width="662" height="490" alt="Screenshot 2026-04-16 232448" src="https://github.com/user-attachments/assets/73274315-758c-4885-8b24-2b4f02cca665" />

   <img width="811" height="108" alt="image" src="https://github.com/user-attachments/assets/0eeb1705-93c6-44e0-b4e8-745fc7461be6" />

### root.txt

1. Final goal is to escalate to the root account. We can check what commands `slade` is allowed to run with elevated privileges by running`sudo -l` . `slade` is allowed to run `pkexec` as the root user. This is a massive security hole if `pkexec` can be used to spawn a shell. Then I looked up `plexec` on GTFOBins, and found that it can spawn an interactive system shell if executed via `sudo` because it does not drop the acquired privileges. Run `sudo /bin/sh` the prompt changed to `#`. In Linux, the hash symbol indicates we are now the root user. Run `ls` to see directory of the root and found `root.txt`. To capture the flag use `cat root.txt`
   
   <img width="890" height="119" alt="Screenshot 2026-04-16 233748" src="https://github.com/user-attachments/assets/9946a49c-26f0-4b38-adbc-7a78829c26da" />
   
   <img width="928" height="809" alt="Screenshot 2026-04-16 173545" src="https://github.com/user-attachments/assets/7f299342-e0bf-4391-a455-f23d0131e9d2" />
   
   <img width="768" height="291" alt="Screenshot 2026-04-16 233854" src="https://github.com/user-attachments/assets/457ab516-66a2-439f-bb38-53a6c0899683" />

   <img width="815" height="114" alt="image" src="https://github.com/user-attachments/assets/0d44cddb-293e-4fb8-9967-9f86d0210366" />

   <img width="832" height="618" alt="Screenshot 2026-04-16 173717" src="https://github.com/user-attachments/assets/5daea9b6-5cd0-4fb5-b073-43116c83cc2f" />





   

