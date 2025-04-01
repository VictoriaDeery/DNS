<h1>DNS Lab</h1>

 ### [YouTube Demonstration-available upon request]
 
<h2>Description</h2>
DNS (Domain naming server/system) converts computer names and website names that humans understand, into binary code that is understood by computers to locate resources. DNS is often integrated with Active Directory (If you have been following my Active Directory repositories, you'll know that we chose to automatically install DNS when we installed Active Directory turning dc-1 into a domain controller. So we already have a DNS server both installed and running on our domain controller dc-1).
<p></p>
In this lab we will use dc-1 and client-1 that was created in the active directory lab, to gain DNS familiarity.

<br />


<h2>Requirements</h2>

- <b>Active Directory must be running in Azure on VM dc-1</b> 
- <b>A client virtual machine must be running in Azure (client-1) and joined to the domain</b>
- <b>Both of these requirements have step by step details in my repositories:
  - [Active Directory Set Up](https://github.com/VictoriaDeery/ActiveDirectorySetUp/blob/main/README.md)
  - [Active Directory Functionality](https://github.com/VictoriaDeery/ActiveDirectoryLab-pt2/blob/main/README.md)

<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Azure</b>
- <b>Active Directory</b>
- <b>remote desktop</b>
- <b>Active Directory Users and Computers</b>
- <b>DNS</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Outline:</h2>

- A. Inspect DNS A-records on the server (hostname to IP address mappings)
- B. Create some of our own A-Records on the server and observe them from the client
- C. Delete records from the server and observe the client DNS cache on the client to gain understanding
- D. Create a "CNMAE" record (Mapping one human readable name to another)
- E. Discuss Root Hints


<h2>Program walk-through:</h2>

<p align="center">

<img src="https://github.com/user-attachments/assets/30f43066-78a7-448f-9a22-eef5090994a6" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
Overview: The domain controller also has a DNS server on it.  Its existing A records on it are dc-1.mydomain.com (10.0.0.4) and client-1.mydomain.com (10.0.0.5). These A records were automatically created. In the command prompt of client-1 if you try to ping anything, for example ping "mainframe," or any word you want, first client-1 will check its local cache (stored in memory) to see if its interacted with this computer before, as using cache is the fastest method. If there's no result in cache, the second place it looks is the host file, which is a file local to the computer that has a domain name to IP address mappings. If there is still no result, then it will reach out over the network to ask the DNS server who has mainframe.com. Again our DNS server in this lab is dc-1. dc-1 will check its record for mainframe and if it doesn't have it then the ping will fail and mainframe host will not be located. So we can create an A record on our DNS server for mainframe host (mainframe.mydomain.com) and then we can map it to any IP address. Then if re-pinged, it will be successful, not from the cache or hostfile but from the DNS server which will return the IP address to client-1; and then client-1 will ping the IP address and then it will give you a success message if the host IP is online and pingable. AT tis point, now client-1 can store this IP address in the local cache. So now if you wait a few minutes and change the mainfrome A record in the DNS server to point to a different IP address (8.8.8.8), client-1 still has its previous IP address in the cache so ping mainframe will attempt to ping the previously mapped IP address even though the main record has changed. So client-1 can run ipconfig /flushdns to wipe the local cache immediately, and so the ping will go out over the network to ask the DNS server as it had before. And then root hints: since our domain controller, doubles as a DNS server, it is responsible for resolving all requests from the network, ie:knowing google's IP address. So if the local serverdoesn't know where to resolve a top level domain, it uses root hints which is automatically installed when a domain controller is installed and opts to install DNS. Root Hints is a network of hundreds of servers around the world.
  
  (an A record maps a hostname to an IP address) 
 <p>
 <br />
- A. Inspect DNS A-records on the server (hostname to IP address mappings)
<p>
<img src="https://github.com/user-attachments/assets/412f0e44-cf11-4ae9-87e0-3cf97abe8377" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/85712f86-a29f-4b3e-9767-bef7c3f8951d" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p></p>

  1. log into both dc-1 and client-1 as an admin like jane_admin. First use client-1. client-1 ->powerShell -> ping mainframe. It will tell you it cannot be found. To see the local DNS cache type ipconfig /displaydns. and if you wantthe cache to be in a text file called test, type ipconfig /displaydns > test.txt hit enter to run. and if you want to open it type notepad test.txt. Here you can do control+F and confirm there is no mainframe here. Now to view the local host file, open notepad as an admin ->file -> open -> change file type to all files -> windows -> systemm32 ->drivers -> etc -> hosts to view the local hostfile which is where you can map IP addresses to hostname, for example if you want to assign zebra to the local loopback address 127.0.0.1 then ping will find it in the hostfile. Even doing nslookup mainframe, it wont be found at this point because there is no DNS record. So let's create a DNS A record on dc-1 for "mainframe" and have it point to dc-1's private IP address.
</p>
- B. Create some of our own A-Records on the server and observe them from the client
<p>
<img src="https://github.com/user-attachments/assets/8f1a01e0-9e82-4464-b172-e60499745723" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
  1.Now use dc-1. Get and copy dc-1's private IP address either from Azureor in dc-1 PowerShell type ipconfig and it will tell you. click the windows symbol -> administrative tools -> DNS to open the DNS server ->dc-1 -> forward lookup zones -> mydomain.com. click mydomain.com to see different records. Right-click in the records space -> new host (A or AAAA) -> set the name ad IP Address as in the picture above (mainframe & dc-1's private IP). It is possible to have two records pointing to the same IP address.
  <p>
<img src="https://github.com/user-attachments/assets/b6c26c2a-ce07-41b3-a422-d50d2a9764c2" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/9fc4b6f4-2d43-48de-9006-c8b5e14d8238" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<p>
    2. Go back to being on client-1 PowerShell and ping mainframe successfully via DNS server.
  </p>
</p>
- C. Delete records from the server and observe the client DNS cache on the client to gain understanding
<p>
  

<img src="https://github.com/user-attachments/assets/c1b51ca2-ff44-493b-aeaf-be72242bb27c" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <p>
<img src="https://github.com/user-attachments/assets/228291c7-6d6b-4749-a0fd-f684713739ab" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/f5bd3aa8-d436-4ae3-bb87-8cb345dfdbfc" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/1d42076c-98a4-4e7c-90a9-ea967abcd9db" height="80%" width="80%" alt="Disk Sanitization Steps"/>

  1. In dc-1 change the A record to point to 8.8.8.8. instead of 10.0.0.4. Doubleclick the mainframe record you made and change the IP address. In client-1 powershell ping mainframe again. You'll notice it outputs the old IP address because that is what has been saved in the cache, which is first looked to. even typing ipconfig /displaydns will display the old IP in this local DNS cache. So we will flush the cache and observe it as empty. oppen powershell, running it as an admin and type ipconfig /flushdns to flush it and then use ipconfig /displaydns to observe that. so now pinging mainframe will get the new IP address 8.8.8.8. flushdns is helpful especially if only one person cannott access a resource when others can.
</p>
- D. Create a "CNMAE" record (Mapping one human readable name to another)
<p>

<img src="https://github.com/user-attachments/assets/e504b551-bbdb-4f92-b208-ae1102264cfb" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <p>
<img src="https://github.com/user-attachments/assets/c0d6a829-ee35-46a5-bbee-1f622f279505" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
  In dc-1, map "search" to "www.google.com" as the fully qualified domain name (FQDN). In the DNS server, rightclick in our domain like before (under mainframe) -> new alias (CNAME). Now in client-1 ping search. since the name doesnt match the certificate of google.com, searching "search" won't work.
</p>
- E. Discuss Root Hints
Review overview.

</p>
