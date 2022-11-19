# Lian_YuWriteUp

First, we use NMAP to discover any open port on the host by running the command: nmap 10.10.58.201

![image](https://user-images.githubusercontent.com/118617364/202856505-54a84a2a-84cc-48d6-ad0d-b92fa598cdf4.png)

Well, now we use the -A flag in NMAP to view more information about the open ports that we did find: nmap -A -p 21,22,80,111 

![image](https://user-images.githubusercontent.com/118617364/202856533-40326b12-3d55-426e-ab92-91fbe9c52575.png)

We know that SSH port is open we can maybe use it later?
also, ftp doesn’t allow anonymous login so there should be a username and password for it to login. NMAP would show if ftp allowed anonymous login that’s why I guessed it.
Anyway, let’s explore the webpage on port 80 maybe we will find something useful. 
