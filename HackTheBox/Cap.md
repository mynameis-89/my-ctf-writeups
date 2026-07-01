---INTRODUCTION--- 
platform : Hack The Box
challenge/machine name : Cap
machine description / about the machine: Cap is an easy difficulty Linux machine running an HTTP server that performs administrative functions including performing network captures. Improper controls result in Insecure Direct Object Reference (IDOR) giving access to another user's capture. The capture contains plaintext credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root.


---Task 1:How many TCP ports are open?
as soon as you get the victim's ip go ahead and scan it using the tool "nmap" and type the following in the terminal : 
nmap (victim ip)
you will find out that there are 3 TCP ports open :
1- port 21/tcp with the service FTP ( it's the File Transfer Protocol which allows you to upload, download, and manage files remotely)
2- port 22/tcp with the service SSH (which simply lets you control another device in the terminal)
3- port 80/tcp with the service HTTP (which is an old and unsecured version of https which is used widely nowadays)
so the answer  = 3
---Task 2: After running a "Security Snapshot", the browser is redirected to a path of the format /[something]/[id], where [id] represents the id number of the scan. What is the [something]?
since the task mentions "browser" you have to simply go the protocol we found opened "http" go ahead and type in your browser :
http://(victim ip)
it will take you to a specific website where you can see 4 things on the left :
1- dashboard
2-security snapshot 
3- ip config
4- network status
right now we need "security snapshot". once you click on it you will be redirected to the following path : (victim ip)/data/2
so answer = data
---Task 3:Are you able to get to other users' scans (yes/no) ?
you can exploit this easily by using the vulnerability IDOR ( when you see in the path or request there is a parameter "id=2", you can modify it and make it "id=1" as a result you can now see the data of another user without any effort. this is called IDOR)
so the answer  = yes

---Task 4: What is the ID of the PCAP file that contains sensative data?
( by the way this task took me 3h+ just because i thought that the id he meant is a built in thing is the file probabilities)
after entering the "security snapshot" tab you will see a download button. we need to know where is that sensitive data file pcap file first. why don't we use the IDOR vulnerability again ? after you change the number "2" in the path to "0" you will get a completely other results. download that file and you find a lot of sensitive data
so the answer  = 0


---Task 5: Which application layer protocol in the pcap file can the sensitive data be found in?

after downloading the file and opening it in wireshark you will find a lot of data. go ahead and filter the search to "FTP" press right click + follow + tcp, and you now see the username and password used to login in the FTP which are username : nathan and password : Buck3tH4TF0RM3!
so the answer  = ftp


---Task 6: We've managed to collect nathan's FTP password. On what other service does this password work?
earlier when we scanned the victim ip using the tool "nmap' we found out that the service "ssh" was opened. why don't we try to login in it using the same "ftp" password we got ? first off you open the terminal then type "ssh (victim ip)" then you will get this "ssh (your machine username)@(victim ip)" then you will be asked to enter the password, however any password you will enter won't work because earlier when you wrote "ssh (victim ip)" you told the computer "hey, i wanna login in this ip using the username im using right now (which is your computer username)" and guess what....your username doesn't exist in the victim ip computer. so you need to do the following :
1- ssh nathan@(victim ip)
2- try the password we got earlier which is Buck3tH4TF0RM3!
3- and boom !! you are in
so the answer = ssh


---Submit User Flag: Submit the flag located in the nathan user's home directory.

we were able to get in using 2 ways :
1- ftp
2- ssh
we can get this flag with both methods we did. lets start by how to get it using ftp.
okay now we are in using ftp....lets see what files are there by performing the command "ls"
and we found a file called "user.txt". great lets open and see the content in it....you try to write "cat user.txt" and it gives you and error or "invalid command" thats because service ftp doesn't support regular commands due to ftp main job, remember it ? yes, transferring files remotely. nice job.
there is a command that ftp supports which is "get" it transfers the specified file to your computer so you can inspect it easily. now write "get user.txt" and close the service by writing "bye". now open the user.txt on your computer see the flag and submit it.


---task 8: What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

now this is a tricky one, but you can solve it by using the following command "getcap -r / 2>/dev/null" 
Breaking down the command:
-r: Tells the command to look recursively through folders.

/: Starts the search at the absolute root directory.

2>/dev/null: Hides any error messages so your screen stays clean.
What to look for:
When you run this, it will output a short list of paths and their assigned capabilities. Look for a common system binary (like an interpreter or tool) that has the cap_setuid capability enabled.
after doing so the results will be :
nathan@cap://$ getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
nathan@cap://$ 
Take a look at the very first line:
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip 
yes its our boy right here.
the answer = /usr/bin/python3.8

---Submit Root Flag: Submit the flag located in root's home directory.

when you go into the absolute first directory and perform the command "ls" a lot of folders will appear, one of them is "root". now when you go and type "cd root" you will get a "permission denied" because you are smart you will think that "sudo" is the key and type "sudo cd root" and enter the password and then "permission denied" again, thats becasue you are not a "root" you are just a regular "user". since we knew the full path to the binary on this machine that has special capabilities that can be abused to obtain root privileges we can abuse it right now to get the root privileges. go to the python3.8 from the path " /usr/bin/python3.8" and write these codes : 
import os
os.setuid(0)
os.system("/bin/bash")
Because Python had that cap_setuid power on the keychain, it became a golden opportunity.

When you entered the Python interactive shell (>>>), you executed three specific lines of code. Here is exactly what they did line-by-line:

1. import os
The os module is Python's built-in toolkit for talking directly to the underlying Operating System (Linux). By importing it, you gave Python the ability to send low-level system commands.

2. os.setuid(0)
This is where the magic happened.

In Linux, every user has a number called a UID (User ID).

Regular users have high numbers (like 1000).

The root user always has a UID of 0.

By running os.setuid(0), you told Python: "Hey, use that cap_setuid capability you have to change my current identity to User ID 0." Because Python had the capability permission from Task 1, the operating system allowed it, and your Python process instantly became an administrator process.

3. os.system("/bin/bash")
Now that Python was running with root privileges, you used os.system to tell it to launch a brand new system terminal terminal (/bin/bash).

Because a child program always inherits the permissions of the parent program that launched it, the new terminal opened up with full root authority. This is why your prompt changed to #, allowing you to walk right into /root/root.txt and read the flag

and congratulations you have finished the challenge successfully.


for more details check official hack the box write-up for this machine : https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/machines/writeup/9e4d90d2-73c7-4da0-a15f-662bbc048868.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260701%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260701T145130Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=a7acb9b319abd00e5a5aa12b0b8fd6130b26eafb159e22bc70ca5349deefdaf7
