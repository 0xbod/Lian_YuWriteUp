# TryHackMe Writeup 
Lian_Yu Room Link -----> https://tryhackme.com/room/lianyu

Hello Guys , I'm Abdelrahman Mohamed. My username on THM is “Sylvesterr”.

# #Step1: Scanning the host
First, we use NMAP to discover any open port on the host by running the command: nmap 10.10.58.201

![image](https://user-images.githubusercontent.com/118617364/202856505-54a84a2a-84cc-48d6-ad0d-b92fa598cdf4.png)

Well, now we use the -A flag in NMAP to view more information about the open ports that we did find: nmap -A -p 21,22,80,111 

![image](https://user-images.githubusercontent.com/118617364/202856533-40326b12-3d55-426e-ab92-91fbe9c52575.png)

We know that SSH port is open we can maybe use it later?
also, ftp doesn’t allow anonymous login so there should be a username and password for it to login. NMAP would show if ftp allowed anonymous login that’s why I guessed it.
Anyway, let’s explore the webpage on port 80 maybe we will find something useful. 

![image](https://user-images.githubusercontent.com/118617364/202856738-dbbd0904-113d-47ef-955a-6bf95f1a15b9.png)

Nothing interesting in the webpage here or even the page source as we should always view the page source.
# #Step2: Finding hidden directories
Running “gobsuter” to get any hidden directories from the webpage is our target now so let’s use the command: gobuster dir -u http://10.10.58.201 -w /usr/share/wordlists/dirb/big.txt

![image](https://user-images.githubusercontent.com/118617364/202856824-9b50c902-3e5d-43e4-8bba-a8041009da2d.png)

Hooraay, we found a directory called /island, let’s navigate to this directory and see where it gets us

![image](https://user-images.githubusercontent.com/118617364/202856871-17222b0c-827b-49c8-aeab-5369f1202cef.png)

Okay if we check the page source as I mentioned above that’s what we should do cause maybe the developer left any hard coded passwords or evidence that could help us for exploitations.

![image](https://user-images.githubusercontent.com/118617364/202856893-14f951cb-d92c-48f9-ac23-e941ba299042.png)

This code word “vigilante” is interesting it looks like a password or a username for something, who knows. Let’s explore more by running gobuster one more time but we define the directory we found this time: gobuster dir -u http://10.10.58.201/island -w /usr/share/dirbusterwordlists/directory-list-2.3medium.txt

![image](https://user-images.githubusercontent.com/118617364/202856922-8b6769e5-88cf-4e6d-a8b0-e5c04eea3de4.png)

Well well well, we found another directory called “2100” inside the island. Let’s explore it and see what it is.

![image](https://user-images.githubusercontent.com/118617364/202856936-2bdc3d34-4d81-4bf4-8eb4-96b2f0bf6cea.png)

Okay, the video is deleted which confused me, I thought it had something to do with solving the machine, but it turned out to be garbage. Now let’s check the page source.

![image](https://user-images.githubusercontent.com/118617364/202856960-8e26870e-4dc9-4d18-9a03-b6f24aaa5c8a.png)

This “.ticket” looks like some kind of extension. We could try another gobuster but adding the -x flag for extension and search for .ticket. Use the command: gobuster dir -u http://10.10.58.201/island/2100 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x .ticket

![image](https://user-images.githubusercontent.com/118617364/202856970-df287aeb-45a6-4406-b3ed-e29abe87ea5e.png)

Let’s navigate to /green_arrow.ticket and see its contents 

![image](https://user-images.githubusercontent.com/118617364/202856980-4495a160-6615-4987-89d1-1777839a8768.png)

This doesn’t look like a hash, it is some kind of encoding. Let’s try cyberchef to decode. After trying  base64 and  base32 it didn’t get us anywhere, so I decided to try all base encodings.

![image](https://user-images.githubusercontent.com/118617364/202856996-f141a8ec-1282-4d67-b7a4-c43a1adc5f02.png)

It turned out to be base58, for next time you should be aware that it is not only base62-32. Look at the left  side and you will see that we have 6 types of base encodings. Well, now we have what looks like a password “!#th3h00d”. Remember the code name we found earlier in the island page “vigilante”. We now have a username and a password let’s try to login in ftp.

![image](https://user-images.githubusercontent.com/118617364/202857130-826b47bc-73b1-4259-ab8a-d27a722bb45d.png)
# #Step3: Using what we enumerated
We successfully logged on ftp, let’s enumerate it and look for any files to download.

![image](https://user-images.githubusercontent.com/118617364/202857153-4302145a-6426-4bd8-99fc-859d9a2a0462.png)

We see 3 images with extension .png inside the vigilante directory.
It’s time for steganography!!! Before we get distracted with the images. We should look for more information inside ftp, if we try to change directory back we will find that we have 2 users, one of them we know which is vigilante and other one is “slade” this could come in play later so don’t forget to look for any information that you can gain.

Use mget <filename> to get the files to your local machine
  
![image](https://user-images.githubusercontent.com/118617364/202857172-93795942-8bc6-4412-b581-dccac52a78c0.png)
# #Step4: steganography

Okay, let’s play with the images that we downloaded to our local machine with stego tools. https://0xrick.github.io/lists/stego/ you should checkout this list of useful tools written by Ahmed Hesham.
  
First, we tried to run exiftool bcs Sometimes important stuff is hidden in the metadata of the image or the file. However, nothing showed up
  
Then, we tried steghide that checks for any hidden or embedded data inside of image and audio files it only supports these file formats : JPEG, BMP, WAV and AU. If we look at the images we have here, only one image is jpg which steghide supports.
  
![image](https://user-images.githubusercontent.com/118617364/202857198-f313818c-a58a-4638-b76c-fb0960fbf1da.png)

There a file embedded inside the aa.jpg, but it is encrypted with a password. Let’s see the other images maybe the clue is hidden somewhere in those .png ones

We can open the other 2 images easily but the image Leave_me_alone.png  does not open. Let’s try to run file command to check the type.
  
 ![image](https://user-images.githubusercontent.com/118617364/202857304-f30f06bf-08bf-434c-8562-11bd47ffb132.png)
  
Looks like that .png extension is not the real file type or the magic bytes are manipulated. We should check the magic bytes of Queen’s_gambit.png and compare it to Leave_me_alone.png 

We can do that by using xxd command: xxd Leave_me_alone.png > magic.txt
  
 ![image](https://user-images.githubusercontent.com/118617364/202857338-c0deca74-9a84-4ddd-96ba-601080e4ad55.png)

Save both files so we can read them easily out of the terminal 
  
  ![image](https://user-images.githubusercontent.com/118617364/202857493-aaa2194b-83e6-433b-9524-cad660549312.png)

 As we expected the magic bytes of Leave_me_alone.png starts with XEo and that’s not how the magic bytes of PNG extensions should be.
  
Let’s open hexeditor [a tool already installed in kali] and change the bytes of Leave_me_alone.png with the bytes from Queen’s_gambit.png 

  ![image](https://user-images.githubusercontent.com/118617364/202857504-baafd500-0bdf-486a-90fa-d70ea3582d0a.png)

  The file now turned into real .PNG and we FINALLY get the image and this is how it looks

![image](https://user-images.githubusercontent.com/118617364/202857509-4605b900-f526-43cf-8924-7ad9b17bfac5.png)

  Remember that we had 3 files, the aa.jpg which we found by steghide, it has an embedded password protected file. Let’s now try to extract it using the “password” password in this image.
  
  ![image](https://user-images.githubusercontent.com/118617364/202857517-08fd527a-2c59-4991-b6ae-32aff23d9bf0.png)

  The embedded zip file was ss.zip after extracting it we found passwd.txt which was a note from the machine owner [garbage] the real password was inside shado file.
Let’s go back to the ssh, remember the use “slade” that we found his directory on ftp. 
  
  ![image](https://user-images.githubusercontent.com/118617364/202857525-3fcb515a-291a-4209-9b6c-31fdc9214c3e.png)
# #Step5: privilege Escalation
IT WORKED!!!!!!! We got a shell now and the user flag is here.
Time for privesc, the very first command I run is sudo -l to see if there is any command that we could run as root. 

![image](https://user-images.githubusercontent.com/118617364/202857534-e765ffb0-212d-4b37-b31b-273349b404e7.png)
  
  Running the sudo -l, we find pkexec that we could run as root 
Instantly go to https://gtfobins.github.io/ it has a great list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems. Search for pkexec and you will find a command that elevates our privilege: sudo pkexec /bin/sh  
Now we have gained root access. Let’s get the final root flag.
  
  ![image](https://user-images.githubusercontent.com/118617364/202857540-a3d225da-eb13-4b71-8582-cbb7d290f473.png)

### Thanks for reading this.
